BTRFS
=====

Filesystem
----------

For further refs see `Btrfs Wiki - Basic Filesystem Commands`_.


Resizing
~~~~~~~~
To resize a filesystem, the underlying device must have room to accommodate the changes,
you first may have to resize the partition or logical volume.

The following commands are self explanatory.
::

    # btrfs filesystem resize +2g /path/to/filesystem
    # btrfs filesystem resize -2g /path/to/filesystem
    # btrfs filesystem resize 20g /path/to/filesystem
    # btrfs filesystem resize max /path/to/filesystem

You can use as unit 'K', 'M', 'G', 'T', 'P' or   'k', 'm', 'g', 't', 'p' the units are
counted with a 1024 base, i.e KiB, MiB, GiB, TiB, PiB.

Btrfs label
~~~~~~~~~~~

You can put a label on the filesystem at creation with::

  # mkfs.btrfs -L mylabel /dev/sdx

It can be changed with ::

  # btrfs filesystem label /mountpoint newlabel

On an unmounted volume ::

  # btrfs filesystem label /dev/by-uuid/1234567890 newlabel


Changing uuid
~~~~~~~~~~~~~

If you copy at low level a btrfs filesystem you end up with two filesystems with the
same UUID, and even if you don't use the UUID for mounting your filesystem the output of
:man:`mount` or :man:`findmnt` is incoherent mixing the two filesystems, and you cannot
mount both as the kernel see the filesystem as yet mounted. Even the command
::

    # btrfs filesystem show

see only one of the two duplicate filesystem.

So after copying the filesystem you need to change the UUID of the new fs with
:man:`btrfstune(8)`.
::

    # btrfs check /dev/<device>
    # btrfstune -u /dev/<device>

The new UUID is randomly choosen, you can also provide a new valid and unique UUID.

::

     # btrfstune -u b532bc62-9468-4fe9-83af-9e9de6aed2d4 /dev/<device>

Filesystem integrity
--------------------

The :man:`btrfs-scrub(8)` command executes as a background process for a mounted
volume. It verifies the checksums for all data and metadata. If the checksum fails it
marks it as bad, and if a good copy is available on another device it replaces it. This
operation runs at a default IO priority of idle to minimize the impact on other active
processes.

::

    # btrfs scrub start /mountpoint

To monitor scub progress::

  # btrfs scrub status /mountpoint

Scrub should be run via a periodic system service at least each month.

There is also a command to make a read-only check of  metadata and filesystem structures
on an unmounted btrfs filesystem using :man:`btrfs-check(8)`::

  # btrfs check -p /dev/by-partuuid/UUID

*(you can choose any device naming!)*.

This command is not usually necessary and scrub should be preferred.

To know what errors have been detected on a btrfs volume you can issue::

  # btrfs dev stats /mountpoint


Subvolume
---------

Ref: :man:`btrfs-subvolume(8)`

Se also :ref:`snapshots`.

Creating, listing, deleting
~~~~~~~~~~~~~~~~~~~~~~~~~~~


To create a subvolume::

  # btrfs subvolume create /path/to/subvolume

List subvolumes under some path::

  # btrfs subvolume list -p /path

Delete it ::

  # btrfs subvolume delete /path/to/subvolume



Moving data across subvolumes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To move data between subvolumes you can use reflinks to avoid a physical copy of the
data::

  $ cp -a --reflink=always volume1/dir volume2/dir
  $ rm -rf volume1/dir

It can seem surprising that a simple move don't do the work, more quickly, but mv uses
the rename syscall and when a cross subvolumes move is detected, it fall back to a plain
copy.

Mounting
~~~~~~~~

When you mount a btrfs filesystem, the default subvolume mounted. The default subvolume
is initially set to be the top-level subvolume which has an id of 5, but it can be
changed as shown below.

If you want to mount an other subvolume you have to give as mount option either
``subvolid=<id>`` or  ``subvol=path/from/toplevel`` where id is the subvolume id that you
get by ::

  # btrfs subvolume list -p /path/to/filesystem
  ID 267 gen 5264 top level 5 path mysubvolume

or by ::

  # btrfs subvolume show /path/to/filesystem/subvolume


Changing default subvolume
~~~~~~~~~~~~~~~~~~~~~~~~~~

You can change the subvolume which is mounted by default by ::

   # btrfs subvolume set-default <id> /path/to/filesystem

The top level will become accessible by mounting it as subvolume with path ``/`` or
id 5.

Changing the label
~~~~~~~~~~~~~~~~~~

The current label is displayed with ::

  # btrfs filesystem label /path/to/filesystem

to change it::

  # btrfs filesystem label /path/to/filesystem a_new_label

You can also display or change or display the label of an unmounted filesystem
::

    # btrfs filesystem label /dev/mydevice a_new_label


.. _snapshots:

Snapshots
---------

Snapshots are :man:`btrfs-subvolume` operations `described in the Btrfs wiki
<Btrfs Wiki - Snapshots>`_.
They are ordinary subvolumes and files data, i.e. *extents*, are shared using the COW
feature of btrfs. In this aspect they differ from lvm snaphots that are done
block-level.

To create a new writable snapshot::

  # btrfs subvolume snapshot <source> <dest>/<name>

If you omit *<name>* the the name of the source is used.

If you want your snapshot to be readonly add the ``-r`` option
::

    # btrfs subvolume snapshot -r <source> <dest>/<name>

You often want to give the snapshot a meaningfull name like
::

    # btrfs subvolume snapshot /rootfs /.snapshots/root_$(date -Iminutes)
    Create a snapshot of '/rootfs' in '/.snapshots/root_2019-01-19T09:29+01:00'
    # btrfs subvolume snapshot -r /rootfs /.snapshots/root_$(date -Iminutes)
    Create a readonly snapshot of '/rootfs' in '/.snapshots/root_2019-01-19T15:43+01:00'

The snapshots are in their own subvolume, and the snapshot does not cross subvolume
boundaries. So the previous snapshot do not include other snapshots that may reside in
``/.snapshots/``.

Btrfs send
----------

:man:`btrfs-send` and :man:`btrfs-receive` allow to send a snapshot to an other btrfs
filesystem.  As example make a snapshot and send it *for the clarity we put the date in
clear* ::

    # btrfs subvolume snapshot -r /rootfs /.snapshots/root_2019-01-18

And send it elsewhere
::

    # btrfs send .snapshots/root_2019-01-18 | btrfs receive /backup/snapshots

Now make a new snapshot and send the difference between the two snapshots.
::

     # btrfs subvolume snapshot -r /rootfs /.snapshots/root_2019-01-19
     # btrfs send -p /.snapshots/root_2019-01-18 /.snapshots/root_2019-01-19 |\
       btrfs receive /backup/snapshots
     # btrfs subvolume delete  /.snapshots/root_2019-01-18

After this operation in the ``/backup`` btrfs filesystem we have two readonly snapshots
in the directory ``/backup/snapshots`` named ``root_2019-01-18`` and
``root_2019-01-19``.

As :man:`btrfs-send` and  :man:`btrfs-receive` communicate with a data stream, the
transfer is not necessarily synchronous, and the send and receive can be on different
computers by using a data transport, like ssh, to convey the stream.

The `incremental backup page of btrfs wiki <Btrfs Wiki - Incremental Backup>`_
give more details. The same page list also some tools to automate backups, more recent
tools in the `Uses Cases page of the same Wiki <Btrfs Wiki - Btrfs For Backups>`_.


Btrfs raid
----------

We can create raid over many partitions, or over multiple logical volumes of the same
group allocated to physical volumes on distinct disks.

We use here a raid between lvm volumes of the same volume group but allocated to
Physical Volumes on distinct disks:

We can check that the logical volumes are on proper devices by ::

  # lvs  -o lv_name,attr,lv_size,devices myvg
    LV      Attr       LSize   Devices
    mylv0   -wi-ao---- 351.56g /dev/sdc2(0)
    mylv1   -wi-ao---- 351.56g /dev/sda4(0)

Here the attributes means (w) writable, (i) inherited allocation, (a) active, (o) open.
More details on :man:`lvs` fields in `Red Hat LVM Administration - Custom Report`_.

Create a raid-1 btrfs filesystem
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
::

    # mkfs.btrfs -m raid1 -d raid1 /dev/myvg/mylv0 /dev/myvg/mylv1


Conversion to raid
~~~~~~~~~~~~~~~~~~

..  _btrfs_add:

::

    # mount /dev/myvg/mylv0 /mnt
    # btrfs device add /dev/myvg/mylv1 /mnt
    # btrfs balance start -dconvert=raid1 -mconvert=raid1 /mnt

The balance operation duplicate the metadata with option ``-mconvert=raid1`` and the
data with ``-dconvert=raid1``, note that these two operations are independent and could
have been done in two steps as shown in :man:`btrfs-device(8)`.

You will see the raid devices and used space on each *which should be identical after
the balance operation* with

::

    # btrfs filesystem show /mnt

or any of

::

   # btrfs filesystem show  /dev/myvg/mylv0
   # btrfs filesystem show 0c0c9d36-ae9f-41d0-a4fc-fb7368ddf7b1
   # btrfs filesystem show myfs_label

The used space for data, system, metadata is shown by

::

    # btrfs filesystem df /mnt

Or a more detailled report, with detail for each of the underlying devices by

::

    # btrfs filesystem usage /mnt

References: :man:`btrfs-device(8)`,  :man:`btrfs-balance(8)`,:man:`btrfs-filesystem(8)`.

Replacing a failed device
~~~~~~~~~~~~~~~~~~~~~~~~~

Suppose you have a Raid-1 filesystem with two devices ::

  # btrfs filesystem show /mnt
  Label: 'myfs'  uuid:  0c0c9d36-ae9f-41d0-a4fc-fb7368ddf7b1
        Total devices 2 FS bytes used 501.40GiB
        devid    1 size 551.56GiB used 505.06GiB path /dev/mapper/myvg-mylv0
        devid    2 size 551.56GiB used 505.06GiB path /dev/mapper/myvg-mylv1

If the device 2 fail the output will be

::

  # btrfs filesystem show /mnt
  Label: 'myfs'  uuid:  0c0c9d36-ae9f-41d0-a4fc-fb7368ddf7b1
        Total devices 2 FS bytes used 501.40GiB
        devid    1 size 551.56GiB used 505.06GiB path /dev/mapper/myvg-mylv0
        *** Some devices missing

If a device fail you can only mount the filesystem in degraded mode

::

    # mount -o degraded /dev/myvg/lv0 /mnt

You have to provide a replacement device, it implies on our scheme of using pair of
logical volume, to remove the failed physical volume *in our example /dev/sda4* and the
lv it includes *in our example mylv1* , and providing a new PV added to *myvg* and
creating a new lv in the new PV *we name it mylva for clarity, but we could use the same
previous name mylv1*.

Then there are two possibilities for replacing the failed device.

::

    # btrfs replace start 2 /dev/myvg/mylva /mnt

You can monitor the progress with:
::

    # btrfs replace status /mnt

Or you can add a new device as we have shown before in the :ref:`btrfs add example
<btrfs_add>`, and delete the missing device with:

::

    # btrfs device delete missing /mnt

Note that we must first add a device and only after delete the misssing one, as raid-1
needs at least two devices.

You need after any addition of device to balance the filesystem, to distribute metadata
and data.
::

    # btrfs balance  /mnt

References:

-   :man:`btrfs-replace(8)`.
-    `Red Hat Storage Administration Guide -
     Integrated Volume Management of Multiple Devices`_.


Btrfs References
----------------

-  The main site is the `Brtfs Wiki`_ it holds on the Home page a
   `list of Guides and articles <Btrfs Wiki - List of Guides>`_
   and the `Btrfs FAQ`_.
-  `Red Hat Storage Administration Guide - Chapter 6. Btrfs`_
-  Suse storage administration - Btrfs filesystem`_
-  `Oracle Linux - The Btrfs File System`_
-  `ArchWiki - Btrfs`_,
   `Btrfs Tips and Tricks  <ArchWiki - Btrfs Tips and Tricks>`_



..  _Brtfs Wiki: https://btrfs.wiki.kernel.org
..  _Btrfs Wiki - Basic Filesystem Commands:
    https://btrfs.wiki.kernel.org/index.php/Getting_started#Basic_Filesystem_Commands
..  _Btrfs FAQ: https://btrfs.wiki.kernel.org/index.php/FAQ
..  _Btrfs Wiki - Snapshots:
    https://btrfs.wiki.kernel.org/index.php/SysadminGuide#Snapshots
..  _Btrfs Wiki - List of Guides:
    https://btrfs.wiki.kernel.org/index.php/Main_Page#Guides_and_usage_information
..  _Btrfs Wiki - Incremental Backup:
    https://btrfs.wiki.kernel.org/index.php/Incremental_Backup
..  _Btrfs Wiki - Btrfs for Backups:
    https://btrfs.wiki.kernel.org/index.php/UseCases#How_can_I_use_btrfs_for_backups.2Ftime-machine.3F
..  _Red Hat LVM Administration - Custom Report:
    https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/logical_volume_manager_administration/custom_report
..  _Red Hat Storage Administration Guide - Chapter 6. Btrfs:
    https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-btrfs
..  _Red Hat Storage Administration Guide - Integrated Volume Management of Multiple Devices:
    https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/btrfs-integrated_volume_management
..  _Suse storage administration - Btrfs filesystem:
    https://www.suse.com/documentation/sles-12/stor_admin/data/sec_filesystems_major_btrfs.html>
..  _Oracle Linux - The Btrfs File System:
    https://docs.oracle.com/cd/E37670_01/E37355/html/ol_btrfs.html
..  _ArchWiki - Btrfs: https://wiki.archlinux.org/index.php/Btrfs
..  _ArchWiki - Btrfs Tips and Tricks:
    https://wiki.archlinux.org/index.php/Btrfs_-_Tips_and_tricks
