.. _disk-devices:

Operating on disk devices
=========================

Listing devices.
----------------

.. _lsblk:

Using ``lsblk``.
~~~~~~~~~~~~~~~~

You can get the block devices on your system by using :man:`lsblk`::

    $ lsblk -o NAME,TYPE,FSTYPE,LABEL,SIZE,MODEL,MOUNTPOINT

Which gives a schema like this::

    sda                    disk              465.8G ST9500325AS
    ├─sda1                 part                 10M
    └─sda2                 part              419.2G
      ├─vg0-debian (dm-0)  lvm                13.7G                  /
      ├─vg0-home (dm-1)    lvm                23.4G                  /share/home
      ├─vg0-vguests (dm-2) lvm                  56G
      └─vg0-seafile (dm-3) lvm                   2G
    sdd                    disk                3.7G STORAGE DEVICE
    └─sdd1                 part vfat           3.7G
    sr0                    rom                1024M DVD+-RW GSA-T11N

You can also use the simpler ``lsblk -f`` with less columns.

.. _blkid:

Using ``blkid``.
~~~~~~~~~~~~~~~~

A detailled information with UUID, but not graphical is given by
:man:`blkid`, it should be run as root
to give a full information.
::

    $ sudo blkid -c /dev/null

The ``-c /dev/null`` is to prevent ``blkid`` from using cache, and
reporting devices which are no more available.

A more readable output is with:
::

    $ sudo blkid -o list -c /dev/null /dev/sd*
    device     fs_type label    mount point    UUID
    ------------------------------------------------------
    /dev/sda2  LVM2_member      (in use)       Fdh5Am-cNBo...
    /dev/sdb1  vfat             (not mounted)  B49E-A10C

    Or any device with:

::

    $ sudo blkid -o list -c /dev/null
    $ sudo blkid -o list -c /dev/null /dev/sdd1
    $ sudo blkid -o list -c /dev/null /dev/mapper/*

..  _udisksctl_use:

Using *udisk disk manager*.
~~~~~~~~~~~~~~~~~~~~~~~~~~~

`udisksctl`_ is a
higher level control part of the `udisk disk manager`_
If it is running on your system, you get the managed devices with
their model, revision and serial number by::

    $ udisksctl status

but it will not give you a partition list like :ref:`lsblk <lsblk>`
or :ref:`blkid <blkid>`
which I find more informative for discovering new devices.

There are many way to know what partitions compose your storage
devices like
:ref:`proc filesystem <devices_proc_sys_fs>`, the
:ref:`dev filesystem <dev_fs>`, or the commands: :man:`fdisk`, :man:`sfdisk`,
:man:`gdisk`, :man:`sgdisk` or :man:`parted`.

You may have to use a second level when the partition is not a
physical partition but a logical partition.
lvm volumes are grouped in *volume groups* divided in *logical
volumes* that you can list using
:man:`lvdisplay`.

If the partition host a btrfs file system, you can list the
subvolumes that compose it by using :man:`btrfs-subvolume`.

When you add the device entry to *udiskctl* you obtain a more detailed
info:

::

    $ udisksctl info -b /dev/sdd
    $ udisksctl info -b '/dev/sdd1'
    $ udisksctl info -p 'block_devices/sdd'
    $ udisksctl info -p 'block_devices/sdd1'

``-p`` *is an abbrev for* ``--object-path`` *and* ``b`` *an abbrev
for* ``--block-device``. In any case the *block-device* the
*object-path* is told in the answer.

You can also use ``udisksctl monitor`` to monitor devices
before connecting the device and see the
device entry attributed by udev.

It is also shown in your kernel messages, and can be read with
:man:`dmesg` but it is quite laborious to find the proper line.


References
++++++++++

-   `udisksctl`_
-   ArchWiki: :archwiki:`Udisks`

Using the *udev* level for usb devices.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For an usb device you can also use `lsusb
<http://linux.die.net/man/8/lsusb>`_:
::

    $ lsusb -v | grep -E \
    '\<(Bus|iProduct|bDeviceClass|bDeviceProtocol)' 2>/dev/null

You can also get the udev keys with `udevadm
<http://linux.die.net/man/8/udevadm>`_, you have more details
in the :ref:`udev section <udev>`.
::

    $ udevadm info -a  -n /dev/usb/sdb1


`lsusb <http://linux.die.net/man/8/lsusb>`_ is aimed at usb devices,
it can often be replaced  by the more general commands
:ref:`lsblk <lsblk>` and :ref:`blkid <blkid>`.

.. _devices_proc_sys_fs:

Interacting with *proc* and *sys*.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If these command are not available you can work at low level with the
*proc* and *sys* virtual filesystem.
::

    $ cat /proc/partitions
    major minor  #blocks  name
       8        0  976762584 sda
       8        1     409600 sda1
       8        2     307200 sda2
    ....
    $ ls -l /sys/block/*/device
    lrwxrwxrwx 1 root root 0 Jan 25 21:10 /sys/block/sda/device -> ../../../0:0:0:0
    lrwxrwxrwx 1 root root 0 Jan 25 21:12 /sys/block/sr0/device -> ../../../1:0:0:0
    $ cat /sys/block/sr0/device/model
    DVDRAM GUA0N

..  _dev_fs:

Using the *dev* filesystem.
~~~~~~~~~~~~~~~~~~~~~~~~~~~
:ref:`udev <udev>` populate the *dev* filesystem, you can explore it with
:man:`file` ``--special-files`` abridged in ``file -s``, as the
`/dev` entry also used symlinks to find device by *label*, *id*,
or *uuid*, you may need to use also the option  ``--dereference`` (``-L``).

:man:`file` give you how the type of boot sector of device, the type
of partition, it can detect also the physical volumes of *LVM* and
their *UUID*.
::

    $ sudo file -s /dev/dm-*
    $ sudo file -s /dev/sd*
    $ sudo file -L -s /dev/disk/by-uuid/*
    $ sudo file -L -s /dev/disk/by-label/*

..  Comment


    2075  ls /etc/dbus-1/system.d/ | grep freedesktop

Determine the file system of an unmounted partition.
----------------------------------------------------
When the partition is mounted the output of :man:`mount` show the file
system type.

You can also use :man:`df` with the command::

  $ df --print-type --human-readable
  Filesystem                   Type      Size  Used Avail Use% Mounted on
  /dev/mapper/vg0-root         ext3       19G   12G  6.2G  65% /
  /dev/sda2                    vfat      296M   50M  247M  17%
  /boot/efi
  ....

Using short options the command is ``df -Th``.

When the partition is not mounted whe have seen :ref:`above <blkid>`
that :man:`blkid` also give the partition type, and it can even be run
as a user, in this case he cannot tell if the partition is in use or
not, but it still gives the fs type.

When you want to know the size and alignement of a partition you can
use :man:`fdisk` or :man:`sfdisk` with *mbr* partition table
:man:`gdisk` or :man:`sgdisk` with *gpt* partition table, and
:man:`parted` with both of them to issue one of::

  $ sudo fdisk -l /dev/sdd
  $ sudo sfdisk -l /dev/sdd
  $ sudo gdisk -l /dev/sdd
  $ sudo sgdisk -i -p /dev/sdd
  $ sudo parted /dev/sdd print

All these command are to be run as root.

..  _udisksctl_mount:

Mounting devices.
-----------------

To mount the device as root you can of course use the :man:`mount`
command, for removable devices, usually you prefer to mount them as
user.
You can still use :man:`mount` if the fstab has a ``user`` option for
the device, but not for arbitrary plugged devices.

The old way is to use :man:`pmount`, but if you have
`udisdk daemon`_ running on your system, you should use `udisksctl`_:

::

    $ udisksctl mount -b /dev/sdd1
    $ udisksctl mount -b /dev/disk/by-label/key64G001
    $ udisksctl unmount -b /dev/sdd1
    $ udisksctl power-off -b /dev/sdd1

The *udisks* daemon mount your block device in a directory
``/media/<user>`` that it creates if necessary. If ther is a label it is
used; so the device above *key64G001* is mounted as
``/media/<user>/key64G001``.

The directory ``/media/<user>`` belongs to root, but has an ACL giving
you the ``r-x`` access. The directory ``/media/<user>/key64G001`` and
its content belongs to to you with ``rwx`` access.

You can also use `udisksctl`_ to mount a loop device:

::

    $ udisksctl loop-setup -f someimage.iso
    Mapped file someimage.iso as /dev/loop0.
    $ udisksctl mount -b /dev/loop0
    Mounted /dev/loop0 at /media/john/someimage.


Mounting a partition in a FileManager
-------------------------------------

The modern file managers like *Nautilus*, *Thunar*, *Pcmanfm* use
*gvfs* and *udisk2* to mount removabele media. They accept that you
give them a *gvfs* mountpoint.

They also list a list of partition, either yet mounted, or unmouted,
and allow to mount removable partitions.

Partitions listed in ``/etc/fstab`` would (by default) only show up if
they are mounted under ``/media``, ``$HOME`` or ``/run/media/$USER``
or if there is an entry in fstab for them pointing to these
directories.

If you want the partition to be mounted under a different directory
(e.g. ``/mnt``) and still be shown in the sidebar, you can override the
default behaviour by adding ``x-gvfs-show`` to your mount options in
fstab:


Partitions not listed in ``/etc/fstab`` are handled by udisks2 and will be
mounted under ``/run/media/$USER/VolumeName`` or ``/media/VolumeName``
depending on the value of ``UDISKS_FILESYSTEM_SHARED`` (see
:man:`udisks` you can :archwiki:`change it in an udev rule
<udisks#Mount_to_.2Fmedia_.28udisks2.29>`),
hence they will be shown under Devices in the sidebar.


Front ends for mounting removable devices.
------------------------------------------
You may also want to have some frontend that allows to alleviate the
burden of remembering the commands or to read the manual, *but which add
the the load of remembering the frontend api, and make you depend on
the presence of an added piece of software*.

-   `bashmount <https://github.com/jamielinux/bashmount/>`__ is a bash script to help
    mounting with *udisks2*. It is not updted since 2014 but there are more recent
    forks.
-   `lightweight device mounter
    (ldm) <https://github.com/LemonBoy/ldm>`_ (MIT License)
    is a lightweight daemon that mounts removable devices
    automatically. Ut requires only libudev, libmount and libusb. The
    daemon uses 3.3M resident with 2.5M shared. There are few
    configuration options as it relies on fstab for mounting
    partitions. There is no easy way to configure what you want to be
    mounted by the daemon and my regular partitions yet mounted on a
    system path get mounted again under ``/mnt``.
-   `triggerhappy <https://github.com/wertarbyte/triggerhappy>`_ (GPL)
    is a hotkey daemon developed for small and embedded systems. It
    attaches to the input device files and executes scripts on events. It
    is packaged in Debian.
-   `udisk-glue <https://github.com/fernandotcl/udisks-glue>`_
    (BSD Licence) is a daemon that can perform user-configurable
    actions when a certain udisks event is detected. It can be
    configured to automatically mount devices. *Last commit 2013*.
-   `udiskie <https://github.com/coldfix/udiskie>`_
    *(MIT License)* is an automounter for usb devices written in
    python. It uses the dbus interface through *udisks*.
    It comes with optional mount notifications and gtk
    tray icon and a command-line client ``udiskie-mount``. It is in *pypi*.
-   `UDisksEvt <https://github.com/dpx-infinity/udisksevt>`__ (GPL) by
    Vladimir Matveev is a daemon written ih haskell which listens for
    D-Bus signals emitted by UDisks daemon and execute configured
    actions. *Last commit 2011*
-   `udevil <http://ignorantguru.github.io/udevil/>`_
    is a command line program which mounts
    and unmounts removable devices. Udevil is written in C with libudev
    and glib without dependency on udisks or gvfs. It is part of the
    Spacefm project whose development stopped in April 2014.
-   `usbmount <https://github.com/rbrito/usbmount>`_
    automatically mounts USB mass storage devices when they are
    plugged in, and unmounts them when they are removed. The
    mountpoints (``/media/usb[0-7]`` by default), filesystem types to
    consider, and mount options are configurable. If the device
    provides a model name, a symlink ``/var/run/usbmount/MODELNAME``
    pointing to the mountpoint is automatically created.
    *usbmount* is unmaintained since 2007 as a debian package and the last release is in
    *Jessie*, a `git repository <https://github.com/rbrito/usbmount>`_ contains some new
    developpements.
-   `udisksvm <https://github.com/berbae/udisksvm>`__ is a small (280
    loc) python GUI oriented script to automount removable medias using udisks.
-   `udisks_functions <https://gist.github.com/ledti/838039>`_
    are bash functions to help mounting and unmounting with udisks2.

All modern file managers can automount devices for lxde desktops see
`PCManFM <http://wiki.lxde.org/en/PCManFM>`_

Udisks references
-----------------

-   `ArchWiki: :archwiki:`Udisks`.
-   `Gentoo: Udisks <http://wiki.gentoo.org/wiki/Udisks>`_.
-   `Introduction to Udisks
    <http://blog.fpmurphy.com/2011/08/introduction-to-udisks.html>`_.


..  _udisksctl:
    http://udisks.freedesktop.org/docs/latest/udisksctl.1.html
..  _udisk disk manager:
    http://udisks.freedesktop.org/docs/latest/
..  _udisdk daemon:
    http://udisks.freedesktop.org/docs/latest/udiskd.8.html
