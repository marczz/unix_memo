Operating on devices
====================

Listing devices
---------------

.. _lsblk:

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



A detailled information with UUID, but not graphical is given by
`blkid <http://linux.die.net/man/8/blkid>`_, it should be run as root
to give a full information.
::

    $ sudo blkid -c /dev/null

The ``-c /dev/null`` is to prevent blkid from using cache, and
reporting devices which are no more available.

A more readable output is with:
::

    $ sudo blkid -o list -c /dev/null
    $ sudo blkid -o list -c /dev/null /dev/sdd1



`udisksctl
<http://udisks.freedesktop.org/docs/latest/udisksctl.1.html>`_ is a
higher level control part of the `udisk Disk Manager
<http://udisks.freedesktop.org/docs/latest/>`_, if installed you can
use it as:

::

    $ udisksctl info -p 'block_devices/sdd1'
    $ udisksctl info -p 'block_devices/sdd'
    $ udisksctl info -b '/dev/sdb'

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
:ref:`lsblk and blkid <lsblk>`.

If these command are not available you can work at low level with the
*proc and *sys* virtual filesystem.
::

    $ cat /proc/partitions
    $ sudo file -s /dev/dm-*
    $ sudo file -s /dev/sd*
    $ cat /sys/block/sr0/device/model

..  Comment


    2075  ls /etc/dbus-1/system.d/ | grep freedesktop

Mounting Devices
----------------

To mount the device as root you can of course use the :man:`mount`
command, for removable devices, usually you prefer to mount them as
user.
You can still use :man:`mount` if the fstab has a ``user`` option for
the device, but not for arbitrary plugged devices.

The old way is to use :man:`pmount`, but if you have
`udiskd daemon
<http://udisks.freedesktop.org/docs/latest/udiskd>`_ running on your
system, you shoul use `udisksctl
<http://udisks.freedesktop.org/docs/latest/udisksctl.1.html>`:

::

    $ udisksctl mount -p /dev/sdd1
    $ udisksctl umount -p /dev/sdd1
    $ udisksctl power-off -p /dev/sdd1
