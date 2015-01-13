Operating on devices
====================

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

Using *udisk disk manager*.
~~~~~~~~~~~~~~~~~~~~~~~~~~~

`udisksctl
<http://udisks.freedesktop.org/docs/latest/udisksctl.1.html>`_ is a
higher level control part of the `udisk Disk Manager
<http://udisks.freedesktop.org/docs/latest/>`_.
If it is running on your system, you get the managed devices with
their model, revision and serial number by::

    $ udisksctl status

but it will not give you a partition list like :ref:`lsblk <lsblk>`
or :ref:`blkid <blkid>`
which I find more informative for discovering new devices.

When you know your partitions, by using any mean like the
:ref:`proc filesystem <devices_proc_sys_fs>`, the
:ref:`dev filesystem <dev_fs>`, :man:`fdisk` or :man:`gdisk`

You can get more info using the device entry:

::

    $ udisksctl info -b /dev/sdd
    $ udisksctl info -b '/dev/sdd1'
    $ udisksctl info -p 'block_devices/sdd'
    $ udisksctl info -p 'block_devices/sdd1'

``-p`` *is an abbrev for* ``--object-path`` *and* ``b`` *an abbrev
for* ``--block-device``. In any case the *block-device* the
*object-path* are told in the answer.

You can also use ``udisksctl monitor`` to monitor devices
before connecting the device and see the
device entry attributed by udev.

It is also shown in your kernel messages, and can be read with
:man:`dmesg` but it is quite laborious to find the proper line.


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
*proc and *sys* virtual filesystem.
::

    $ cat /proc/partitions
    $ cat /sys/block/sr0/device/model

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

Mounting devices.
-----------------

To mount the device as root you can of course use the :man:`mount`
command, for removable devices, usually you prefer to mount them as
user.
You can still use :man:`mount` if the fstab has a ``user`` option for
the device, but not for arbitrary plugged devices.

The old way is to use :man:`pmount`, but if you have
`udisdk daemon
<http://udisks.freedesktop.org/docs/latest/udiskd.8.html>`_ running on your
system, you should use `udisksctl
<http://udisks.freedesktop.org/docs/latest/udisksctl.1.html>`_:

::

    $ udisksctl mount -b /dev/sdd1
    $ udisksctl unmount -b /dev/sdd1
    $ udisksctl power-off -b /dev/sdd1

Front ends
----------
You may also want to have some frontend that allows to alleviate the
burden of remembering the commands or to read the manual, *but which add
the the load of remembering the frontend api and make you depend on
the presence of an added software piece*.

-   `bashmount <https://github.com/jamielinux/bashmount/>`__ is a bash
    script to help mounting with *udisks2*.
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
    is packaged in Debian.  *Last commit 2012*
-   `udisk-glue <https://github.com/fernandotcl/udisks-glue>`_
    (BSD Licence) is a daemon that can perform user-configurable
    actions when a certain udisks event is detected. It can be
    configured to automatically mount devices. It is packaged in
    Debian.
-   `udiskie <https://github.com/coldfix/udiskie>`_
    *(MIT License)* is an automounter for usb devices written in
    python. It uses the dbus interface through *udisks*.
    It comes with optional mount notifications and gtk
    tray icon and a command-line client ``udiskie-mount``.
-   `UDisksEvt <https://github.com/dpx-infinity/udisksevt>`__ (GPL) by
    Vladimir Matveev is a daemon written ih haskell which listens for
    D-Bus signals emitted by UDisks daemon and execute configured
    actions. *Last commit 2011*
-   `udevil <http://ignorantguru.github.io/udevil/>`_
    is a command line program which mounts
    and unmounts removable devices. Udevil is written in C with libudev
    and glib without dependency on udisks or gvfs. It is part of the
    Spacefm project whose development stopped in April 2014.
-   `usbmount <http://usbmount.alioth.debian.org/>`_
    automatically mounts USB mass storage devices when they are
    plugged in, and unmounts them when they are removed. The
    mountpoints (``/media/usb[0-7]`` by default), filesystem types to
    consider, and mount options are configurable. If the device
    provides a model name, a symlink ``/var/run/usbmount/MODELNAME``
    pointing to the mountpoint is automatically created.
    `usbmount git source
    <https://alioth.debian.org/scm/browser.php?group_id=30641>`_
    The old home page set that *usbmount* is unmaintained since 2007
    at release 0.0.14.1, but development continued to 2012 release
    0.0.22 which is packaged in Debian.
-   `udisksvm <https://github.com/berbae/udisksvm>`__ is a small (280
    loc) python GUI oriented script to automount removable medias using udisks.
-   `udisks_functions <https://gist.github.com/ledti/838039>`_
    are bash functions to help mounting and unmounting with udisks2.

All modern file managers can automount devices for lxde desktops see
`PCManFM <http://wiki.lxde.org/en/PCManFM>`_

Udisks references
-----------------

-   `ArchWiki: Udisks <https://wiki.archlinux.org/index.php/Udisks>`_.
-   `Gentoo: Udisks <http://wiki.gentoo.org/wiki/Udisks>`_.
-   `Introduction to Udisks
    <http://blog.fpmurphy.com/2011/08/introduction-to-udisks.html>`_.
