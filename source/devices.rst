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

You can get more info using the device entry, in many ways::

    $ udisksctl info -b /dev/sdd
    $ udisksctl info -p 'block_devices/sdd'

but it will not give you a partition list like :ref:`lsblk <lsblk>`
or :ref:`blkid <blkid>`
which I find more informative for discovering new devices.

When you know your partitions, by using any mean like the
:ref:`proc filesystem <devices_proc_sys_fs>`, the
:ref:`dev filesystem <dev_fs>`, :man:`fdisk` or :man:`gdisk`

::

    $ udisksctl info -p 'block_devices/sdd1'
    $ udisksctl info -b '/dev/sdd1'

``-p`` *is an abbrev for* ``--object-path`` *and* ``b`` *an abbrev
for* ``--block-device``.

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
:man:`file -s <file>`:

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
`udiskd daemon
<http://udisks.freedesktop.org/docs/latest/udiskd.8.html>`_ running on your
system, you should use `udisksctl
<http://udisks.freedesktop.org/docs/latest/udisksctl.1.html>`_:

::

    $ udisksctl mount -b /dev/sdd1
    $ udisksctl unmount -b /dev/sdd1
    $ udisksctl power-off -b /dev/sdd1
