..  _udev:

Udev device Management and usb devices
======================================
..  highlight:: shell-session

References
----------

- :man:`udev(7)` explains the key and variables you can use in rules.
- :man:`lsusb(8)`.
- :man:`udevadm(8)`.
- ArchWiki: :archwiki:`udev`.
- `Writing udev rules by Daniel Drake
  <http://www.reactivated.net/writing_udev_rules.html>`_
  *2008 - the udev command names are obsolete, but the guide is still useful*,
- `Gentoo Wiki - Udev <https://wiki.gentoo.org/wiki/Udev>`_
- To detect hardware including bus information use :man:`lspci(8)`,
  :man:`lshw(1)` and the *Suse* tool
  `hwinfo <https://github.com/openSUSE/hwinfo>`_ (also in debian).


localrules
----------

Some local udev rules must be placed before defaults because the first
matched rule is applied, Some other rules can be placed *after* the main
rules and will only modify a yet created device as when you change
permissions, add a symlink or a ``RUN`` action.

We also need to check that each rule matches on all keys, and the rule
match exactly one device.

You can put your rules in ``/etc/udev/rules.d/``, and as the rules are used in
lexicographical order you give a proper priority such they are used before or after a
corresponding rule in ``/lib/udev/rules.d/``.

If you want to *replace* a provided rule in ``/lib/udev/rules.d/``, you can just write a
rule with the same name in ``/etc/udev/rules.d/`` as they take precedence over the
``/lib`` rules.

finding usb devices
-------------------

-  To have a list of usb devices just do::

       $ lsusb

-   A slightly more informative list is got by::

        $ lsusb -v \| grep -E '<(Bus\|iProduct\|bDeviceClass\|bDeviceProtocol)' 2>/dev/null

-   To get detailled information you can

    -   Use the *bus:devnum* reported by lsusb as in:

        ::

            lsusb -v -s 003:001

    - Use the *vendor:product* that you find with ``udevadm info``
      like:

        ::

            $ lsusb -v -d 04b8:082b

Many of the keys in ``lsusb`` are attributes of ``udevadm info`` and can
be used in udev configuration.

Using /sys keys
-------------------

We obtain the keys by looking for dev ``/sys`` info, with
``udevadm info``:

1.  find the key by either

    -   Report with ``udevadm info`` providing the device name
        ::

            $ udevadm info --query=property --name /dev/usb/lp0

   -   use the path that you also get from device by
       ::

           $ udevadm info --query=path --name /dev/usb/lp0
           /class/usb/lp0

       and get dev ``/sys`` info by
       ::

           $ udevadm info --query=property --path /class/usb/lp0

2.  We can then find appropriate keys to identify uniquely our device:

.. code-block:: cfg

        SYSFS{manufacturer}=="Hewlett-Packard"
        SYSFS{product}=="DeskJet 840C"

3.  We then add our rule in /etc/udev/rules.d/10-local.rules

.. code-block:: cfg

       BUS=="usb", SYSFS{product}=="DeskJet 840C", NAME="%k", KERNEL=="lp[0-9]*", NAME="usb/%k", GROUP="lp", SYMLINK="deskjet"

4.  Reload udev conf by::

        $ udevadm control --reload

5.  Test the config with::

        $ udevadm test $(udevadm info -q path -n /dev/usb/lp0)

    or::

        $ udevadm test /class/usb/lp0 usb
        main: looking at device '/class/usb/lp0' from subsystem 'usb'
        main: opened class_dev->name='lp0'
        udev_rules_get_name: reset symlink list
        udev_rules_get_name: add symlink 'deskjet'
        udev_rules_get_name: rule applied, 'lp0' becomes 'usb/lp0'
        create_node: creating device node '/dev/usb/lp0', major = '180', minor     = '0', mode = '0660', uid = '0', gid = '7'
        create_node: creating symlink '/dev/deskjet' to 'usb/lp0'


    -   note that it is only a udev simulation, not the true udev creating
        devices, I have experimented cases where udevtest was working, but
        udev did not. In some case it seems it was caused by multiple
        devices matching the same key.
    -   If the device was yet present reloading the rules or restarting
        udev, is not sufficient to have the new device, you have to unplug
        the device, it can be a hot plugging when available, otherwise you
        need to restart the computer.

6.  We must now have::

        $ ls -l /dev/usb/lp0
        crw-rw----  1 root lp 180, 0 Mar 14 21:42 /dev/usb/lp0
        $ ls -l /dev/deskjet
        lrwxrwxrwx  1 root root 7 Apr  7 18:23 /dev/deskjet -> usb/lp0

In the same way we can link a specific mass-storage to a special ``/dev`` entry by
looking at the keys by::

    $ udevadm info -a -p $(udevadm info -q path -n /dev/uba1)

then add in /etc/udev/rules.d/10-local.rules

.. code-block:: cfg

    BUS="usb", SYSFS{serial}="0402170100000020EB5D00000000000", KERNEL="ub?1", NAME="%k", SYMLINK="usbfoo"


But usually it it is better to use the provided symlinks which yet allow
:ref:`persistent naming`.


If we want to automount some removable storage we can create a rule in
``/etc/udev/rules.d/`` and use :man:`systemd-mount` to mount it. As the recent man page
explain in the section :man:`The udev database <systemd-mount#THE_UDEV_DATABASE>`
a udev rule like the following automatically mount all USB storage plugged in:

.. code-block:: cfg

  ACTION=="add", SUBSYSTEMS=="usb", SUBSYSTEM=="block", ENV{ID_FS_USAGE}=="filesystem", \
  RUN{program}+="/usr/bin/systemd-mount --no-block --automount=yes --collect $devnode"

In this case ``systemd-mount`` honors the of additional udev properties
``SYSTEMD_MOUNT_OPTIONS=`` to give additional mount options, ``SYSTEMD_MOUNT_WHERE=``
The file system path to place the mount point at, instead of ``/run/media/system/``.


Here to mount a specific usb key we create a rule by using some keys to identify the
file system using keys from the top sysfs entry and possibly also from a parent, like
this:

.. code-block:: cfg

  ACTION=="add"
  SUBSYSTEMS=="usb"
  SUBSYSTEM=="block"
  ATTRS{serial}=="0709289778d6a5"
  ATTR{partition}=="1"
  RUN{program}+="/usr/bin/systemd-mount --no-block --automount=yes --discover $devnode"

Here the ``serial`` identify the usb key so I have to add a partition number to properly
identify the partition.  The mounted device in ``/run/media/system`` directory

For disk devices you can use any :ref:`persistent naming` for identifying them.
The same information used by symlinks in ``/dev/disk/`` tree and ref:`reported by ldblk
<uuid_with_lsblk>`, is also reported by::

  $ udevadm info --query=property -n /dev/sdc

as ``ID_SERIAL``, ``ID_SERIAL_SHORT``, ``ID_WWN``   for a disk and also for a partition
filesystem ``ID_FS_UUID``, ``ID_FS_LABEL``, ``ID_PART_ENTRY_UUID`` (instead of ``PARTUUID``
for lbslk). These variables are available in udev rules as ``ENV{``*variable*``}``.

If I add a label on the partition of my usb key, I can now use:

.. code-block:: cfg

  ACTION=="add"
  SUBSYSTEMS=="usb"
  SUBSYSTEM=="block"
  ENV{ID_FS_LABEL}=="VFAT_SHARE"
  ENV{SYSTEMD_MOUNT_OPTIONS}="gid=john,uid=john"
  ENV{SYSTEMD_MOUNT_WHERE}="/run/media/john/$env{ID_FS_LABEL}"
  RUN{program}+="/usr/bin/systemd-mount --no-block --automount=yes $devnode"

and I find the mounted device at ``/run/media/john/VFAT_SHARE`` with the *john* user and
group.

debugging udev
~~~~~~~~~~~~~~

To debug udev we can:

1.  use ``udevadm test``
2.  log ``udevd`` by issuing:

    ..  code-block:: cfg

        log="yes"

    in ``/etc/udev.conf`` and change the level of debugging with::

        $ udevadm control log_priority=level

the priority is a  numerical or symbolic level from systlog
**err**, **info** and **debug**

3. ``udevadm monitor`` reports to the console the udevd activity
