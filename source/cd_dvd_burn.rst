DVD and CD
==========

Knowing about your drive.
-------------------------

To know what are your devices

Or with *xorisso* or *cdrskins*:

::

    $ xorriso -devices
    $ xorriso -device_links
    $ cdrskin --devices
    $ cdrskin --device_links

with wodim:
::

    $ wodim --devices

You may have to use it with sudo.

while *xorriso* and *cdrskin* find any device, *wodim*
works only when you have the
*obsolete* symlinks ``/dev/scd*``. The following :ref:`-prcap
<wodim_prcap>` and :ref:`-checkdrive<wodim_checkdrive>` are free from
these limitations and usually find your device even if ``--devices``
fail.


You may also find the drive by looking at
``/dev/sr*`` or the symlinks ``/dev/cdrom``,
``/dev/cdrw``, ``/dev/dvd``, ``/dev/dvdrw``,   and check
with the following :ref:`-checkdrive dev=/dev/dvd <wodim_checkdrive>`
command.

The most basic and secure way, is to use directly the proc filesystem
with:
::

    $ cat /proc/sys/dev/cdrom/info
    CD-ROM information, Id: cdrom.c 3.20 2003/12/17

    drive name:		sr0
    drive speed:	62
    ....
    Can write CD-R:	1
    Can write CD-RW:	1
    Can read DVD:	1
    Can write DVD-R:	1
    Can write DVD-RAM:	1
    .....


..  _wodim_checkdrive:

To have a summary of the model and capacities of your drive:

::

    $ wodim dev=/dev/sr0 -checkdrive
    $ cdrskin dev=/dev/sr0 -checkdrive
    $ xorrecord dev=/dev/sr0 -checkdrive

*xorrecord* is a shortcut for *xorriso -as cdrecord*.

Without *dev* parameter *wodim* or *cdrskin* will check all cd
devices, *xorriso* needs a device.


..  _wodim_prcap:

To get all the capacities of your cd/dvd driver and of the media
inserted:

::

    $ wodim dev=/dev/sr0 -prcap

This ``-prcap`` command is not implemented by *xorriso* or *cdrskin*,
it works only with the original *cdrecord* or *wodim*, and as
:ref:`-checkdrive<wodim_checkdrive>` explore all drives when *dev* is
not given.

Working with iso images.
------------------------

To get the label of a dvd:
::

    $ dd if=/dev/sr0 bs=1 skip=32808 count=32

This work also with an image file:
::

    $ dd if=/boot/grml/grml64-full_2017.05.iso  bs=1 skip=32808 count=32 2>/dev/null
    grml64-full 2017.05

To mount *read only* an iso image as root:
::

    # mount -t iso9660 -o ro,loop /path/to/image.iso /mountpoint

The system usually can guess the options, so you can use:
::

    # mount  /path/to/image.iso /mountpoint

Unmount as usual:
::

    # umount /mountpoint


..  _fstab_cdrom_entry:

This work also for a disk inserted in a drive, but usually you have
yet put in your fstab a line like:
::

    /dev/sr0   /media/cdrom0   udf,iso9660 user,noauto   0     0

so a simple user can mount the device with:
::

    $ mount /media/cdrom0


To mount an iso image as user:
::

    $ udisksctl loop-setup -r -f /path/to/image.iso

The option  ``-r`` means read-only
and ``-f`` gives the path of the file.

and mount it under ``/media/$USER/`` with:
::

    $ udisksctl mount -b /dev/loop0p1

Where you replace *loop0* with the loop device given by ``loop-setup``,
don't forget to add the partition *p1*.

As usual unmount it with:
::

    $ udisksctl unmount -b /dev/loop0p1

and detach the loop device with:
::

    $ udisksctl loop-delete -b /dev/loop0

You can also mount true cd/dvd disk devices with udisksctl.
::

    $ udisksctl mount -b /dev/sr0
    $ udisksctl unmount -b /dev/sr0

*udisksctl* allow to dispense with the
:ref:`user mount in fstab<fstab_cdrom_entry>`


Make iso image from a directory.
--------------------------------

Make an iso image of a directory with Joliet ``-J``, and Rock Ridge
``-R`` extensions. Use a label (max 32 chars) ``-V``, be verbose
``-v``. *Joliet is only useful to use it on windows*.  ::

    $ genisoimage -v -o cd.iso -V DISK_LABEL -R -J /path/to/cd_dir

If you want to use this iso on an other system, you don't want to keep
owner and acces bits, so you will replace ``-R`` with ``-r`` to get
ownership cleared to uid and gid 0; read access to each file and
execute for everybody if the file was executable.


*xorriso -as mkisofs* aliased as *xorrisofs* use exactly the same
options:
::

    $ xorrisofs -v -o cd.iso -V DISK_LABEL -r -J /path/to/cd_dir


Extract an iso image from a CD/DVD.
-----------------------------------

..  _iso_size_on_cd:

Some media types will possibly return more bytes than those found in
the ISO image, because cd writers are allowed to add "run out" sectors
at the end of an iso9660 image.  This trailing garbage MAY HAPPEN with
CD written in TAO mode, incrementally recorded DVD-R[W], formatted
DVD-RW, DVD+RW, BD-RE, and also with USB keys.

Nevertheless if you copy the full disk content, may be constitued by
an iso file and garbage trailing sectors, it will still be
mountable. It should still fit onto a medium of the same type as the
medium from which the image was copied.

So if you want to copy the full content:
::

    $ dd bs=2048 if=/dev/sr0 of=isoimage.iso status=progress

If you want to extract only the iso9660 image, first determine the
size of the image with:
::

    $ isosize -x /dev/sr0
    sector count: 2309214, sector size: 2048

Then extract with:
::

     $ dd if=/dev/sr0 of=isoimage.iso bs=2048 count=2309214 status=progress


Verifying the burnt image.
--------------------------

First you have to know the hash of the iso image, either you have a
sha that you have used to check a download was correct or you compute
it.

Any hash sum will do the job, distributions usually use sha256 or
sha512, and even if md5 is now to be avoided, it is still much used.

::

    $ sha256sum isoimage.iso

Then you can either extract the size in blocks of the isoimage on disk
:ref:`like shown previously<iso_size_on_cd>` or use the size of of the isoimage file.
Both numbers should be the same, *or your write surely failed*, but
reading from hard disk is quicker.

And you compare this number of sectors ignoring
:ref:`garbage trailing sectors<iso_size_on_cd>`.

::

    $ isosize -x isoimage.iso
    sector count: 2309214, sector size: 2048
    $ dd if=/dev/sr0 bs=2048 count=2309214 | sha256sum


Media Type and Capacity
-----------------------

For a dvd :man:`dvd+rw-mediainfo` gyve the type, available speeds,
status, number of sessions, capacity and free blocks of the media.
::

    dvd+rw-mediainfo /dev/sr0

:ref:`wodim -prcap<wodim_prcap>` gives the type of inserted media,
and read and write speed, but not the capacity.

The type of media and supported modes are also given by:
::

    $ wodim dev=/dev/sr0 -atip


To know the capacity of CD or DVD use *cdrskin* or xorriso :
::

    $ cdrskin dev=/dev/sr0 --tell_media_space
    2298496
    $ xorriso -dev /dev/sr0 -tell_media_space
    Drive current: -dev '/dev/sr0'
    Media current: DVD-RW restricted overwrite
    Media status : is blank
    Media summary: 0 sessions, 0 data blocks, 0 data, 4488m free
    Media space  : 2297856s


Here  ``2298496`` is the number of 2kiB sectors, so the capacity is
``2298496/512 = 4489.25MiB`` or ``2298496/(512*1024) = 4.3840GiB``.

Burn an iso image
-----------------

To burn a CD with wodim:
::

    $ wodim -v dev=/dev/sr0 -dao /path/to/file.iso

``-dao`` *disk at once* is used for a single session, a multi session
would require ``-tao`` *track at once*.


Used CD-RW media need to be erased before you can rewrite them, a
*fast* blank is sufficient, you may also want to and eject the drive
at the end of write: ::

    $ wodim -v dev=/dev/sr0 blank=fast -dao -eject /path/to/file.iso

It is not recommended to use *wodim* with DVD or Blu-ray.

To burn a CD, DVD or Blu-ray  with *xorriso* or *cdrkit*:
::

    $ cdrskin -v dev=/dev/sr0 -dao /path/to/file.iso
    $ xorriso -as cdrecord -v dev=/dev/sr0 -dao /path/to/file.iso

``-dao`` has only meaning for CD and DVD-R, DVD-RW, let *xorriso* or
*cdrkit* choose the write mode for other medium or multi-session.

As we have seen used CD-RW need to be blanked before rewrite, this is
also true for DVD-RW but DVD-RAM, DVD+RW, BD-RE are overwritable
without blanking.
::

    $ cdrskin -v dev=/dev/sr0 -dao blank=fast -eject  /path/to/file.iso
    $ xorriso -as cdrecord -v dev=/dev/sr0 -dao blank=fast -eject /path/to/file.iso


In addition to *fast*, *xorriso* and *cdrskin* have a value
*as_needed* which apply the proper blanking to the media, and resolve
to *fast* for used CD-RW or DVD-RW.

To write a DVD  or Blu-ray with *growisofs*:
::

    $ growisofs -dvd-compat -Z /dev/sr0=/home/user/file.iso
