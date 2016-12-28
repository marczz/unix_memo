..  _udev:

Udev device Management and usb devices
======================================
..  highlight:: shell-session

-  :mzlinux:`MZlinux udev/hotplug Page </node/151>`
-  :man:`udev(7)`
-  :man:`lsusb(8)`.
-  :man:`udevadm(8)`.
-  to detect hardware including bus information :man:`lspci(8)`,
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

My local udev *pre* rule are: /etc/udev/pre-local.rules, and the *post*
rules /etc/udev/post-local.rules

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

Ref: `Writing udev rules by Daniel Drake
<http://www.reactivated.net/writing_udev_rules.html>`_,
`ArchWiki: udev
<https://wiki.archlinux.org/index.php/udev>`_.

We obtain the keys by looking for dev ``/sys`` info, with
``udevadm info``:

1.  find the key by either

    -   Report with ``udevadm info`` providing the device name
        ::

            $ udevadm info --query=property  --name /dev/usb/lp0

   -   use the path that you also get from device by
       ::

           $ udevadm info --query=path  --name /dev/usb/lp0
           /class/usb/lp0

       and get dev ``/sys`` info by
       ::

           $ udevadm info --query=property  --path /class/usb/lp0

2.  We can then find appropriate keys to identify uniquely our device::


        SYSFS{manufacturer}=="Hewlett-Packard "
        SYSFS{product}=="DeskJet 840C"

3.  We then add our rule in /etc/udev/rules.d/10-local.rules

.. code-block:: cfg

       BUS=="usb", SYSFS{product}=="DeskJet 840C", NAME="%k", KERNEL=="lp[0-9]*", NAME="usb/%k", GROUP="lp", SYMLINK="deskjet"

4.  Reload udev conf by::

        $ udevcontrol reload_rules

5.  Test the config with::

        $ udevtest   $(udevadm info -q path -n /dev/usb/lp0)

    or::

        $ udevtest /class/usb/lp0 usb
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

In the same way we can mount a specific mass-storage by looking at the
keys by::

    $ udevadm info -a -p $(udevadm info -q path -n /dev/uba1)

then add in /etc/udev/rules.d/10-local.rules

.. code-block:: cfg

    BUS="usb", SYSFS{serial}="0402170100000020EB5D00000000000", KERNEL="ub?1", NAME="%k", SYMLINK="usbfoo"

Note that you can find all disk devices by::

    $ ls -l /dev/disk/by-uuid/

that gives something like::

    lrwxrwxrwx 1 root root 10 Jul 26 22:31 0ae675ac-482e-4789-a7cc-e1505adf539a -> ../../hda1
    lrwxrwxrwx 1 root root 10 Jul 26 22:31 15d94fad-67ea-4de5-b304-ec224eeb4554 -> ../../hda5
    lrwxrwxrwx 1 root root 10 Jul 31 16:20 37712fde-ab06-4957-b9cb-13d2978532a8 -> ../../uba1

You can also use their **id** with::

    $ ls -l/dev/disk/by-id

you will get more devices by id than
uuid, because some devices does not contain (at least at first level) a
file system so have no fs uuid, like a lvm partition or an full disk.

There is some information in `Gentoo HOWTO USB Mass Storage
Device <http://gentoo-wiki.com/HOWTO_USB_Mass_Storage_Device>`__

Automounting USB devices
------------------------

/etc/udev/rules.d/sda.rules:

.. code-block:: cfg

    KERNEL=="sd[a-z]", NAME="%k", SYMLINK+="usbhd-%k", GROUP="users", OPTIONS="last_rule"
    ACTION=="add", KERNEL=="sd[a-z][0-9]", SYMLINK+="usbhd-%k", GROUP="users", NAME="%k"
    ACTION=="add", KERNEL=="sd[a-z][0-9]", RUN+="/bin/mkdir -p /media/usbhd-%k"
    ACTION=="add", KERNEL=="sd[a-z][0-9]", PROGRAM=="/sbin/vol_id -t %N", RESULT=="vfat", RUN+="/bin/mount -t vfat -o rw,noauto,sync,dirsync,noexec,nodev,noatime,dmask=000,fmask=111 /dev/%k /media/usbhd-%k", OPTIONS="last_rule"
    ACTION=="add", KERNEL=="sd[a-z][0-9]", RUN+="/bin/mount -t auto -o rw,noauto,sync,dirsync,noexec,nodev,noatime /dev/%k /media/usbhd-%k", OPTIONS="last_rule"
    ACTION=="remove", KERNEL=="sd[a-z][0-9]", RUN+="/bin/umount -l /media/usbhd-%k"
    ACTION=="remove", KERNEL=="sd[a-z][0-9]", RUN+="/bin/rmdir /media/usbhd-%k", OPTIONS="last_rule"


If you are using any fixed devices
(for example SATA hard disks - check your /etc/fstab) which are
recongized as /dev/sdX change all occurrences of sd[a-z] to the first
unused letter for a sd\* device.

debugging udev
~~~~~~~~~~~~~~

To debug udev we can:

1.  use ``udevtest``
2.  log ``udevd`` by issuing::

    ..  code-block: cfg

        log="yes"

3.  in /etc/udev.conf and change the level of debugging with::

        $ udevcontrol log_priority=level

the priority is a  numerical or symbolic level from systlog
**err**, **info** and **debug**

-  ``udevmonitor`` reports to the console the udevd activity
