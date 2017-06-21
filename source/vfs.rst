Virtual File Systems
====================


GVFS
----

:wikipedia:`Gvfs` is a userspace virtual filesystem where mount runs
as a separate processes which you talk to via D-Bus. It also contains
a gio module that seamlessly adds gvfs support to all applications
using the gio API. It also supports exposing the gvfs mounts to
non-gio applications using fuse.

Gvfs is used through collection of daemons which communicate with each
other and the GIO module over D-Bus. `Supported backends
<https://wiki.gnome.org/Projects/gvfs/backends>`_
include file operations sftp, ftp, webdav, smb, smb-browse, http,
google drive, obexftp; medias (burn, cdda, gphoto2, mtp),
:ref:`archive mounting <gvfs_libarchive>` support,
:ref:`Gnome Online Account <goa>`, admin access for local filesystem.

..  _goa:

gvfs-goa is for Gnome Online Account see the
`GNOME Online Accounts (GOA) project
<https://wiki.gnome.org/Projects/GnomeOnlineAccounts>`__
and `Debarshi Ray posts tagged "Online Account"
<https://debarshiray.wordpress.com/category/gnome/online-accounts/>`_.

..  _gvfs_libarchive:

Archive backend allow to read and write `all formats
<https://github.com/libarchive/libarchive/wiki/LibarchiveFormats>`_
supported by `libarchive <https://github.com/libarchive/>`_:

-   read and write: tar, cpio, pax , gzip , zip, bzip2,  xz, lzip, lzma, ar,
    mtree, iso9660, compress,
-   read only:  7-Zip, mtree, :wikipedia:`xar <Xar_archiver>`,
    lha/lzh, rar, microsoft cab,

Gvfs is directly enabled in all gpio enabled applications, which include gnome
applications. For other applications you have to either use
*Fuse* as shown below, a special bridge as
`Gigolo <http://www.uvena.de/gigolo/>`_
or `Tramp Gvfs backend
<http://www.gnu.org/software/emacs/manual/html_node/tramp/GVFS-based-methods.html>`__
for Emacs.

Gvfs references
~~~~~~~~~~~~~~~

-   Wikipedia: :wikipedia:`Gvfs`.
-   `Gnome gvfs doc <https://wiki.gnome.org/Projects/gvfs/doc>`_ is a
    presentation of the gvfs system architecture.
-   `ArchWiki: File manager functionality - Mounting
    <https://wiki.archlinux.org/index.php/File_manager_functionality#Mounting>`_

gvfs memo
~~~~~~~~~

The *gvfs* daemon by itself is not big 2.6M resident (2M shared), Each
of the backend daemon take also the same size, and I have usually at
least 7 of them (*gvfsd-archive*, *gvfsd-trash*, *gvfsd-sftp*,
*gvfs-fuse-daemon*, *gvfs-afc-volume-monitor*,
*gvfs-gphoto2-volume-monitor*, *gvfs-goa-volume-monitor*).
*gvfs-gdu-volume-monitor* and *gdm-simple-slave* are bigger at 4M/2M
shared. Some of this daemons are launched on demand, but some of them
are launched at start by systemd, the services are in
``/usr/lib/systemd/user/gvfs-<backend-name>.service``. If you never use
some of them like *gphoto2* and have no *iphone* to make use of the
*afc* backend; they can be disabled at session start and only launch
them on demand.


There is a set of command line programs starting with "gvfs-" that
lets you run commands (like cat, ls, stat, etc) on files in the gvfs
mounts.

As a command line example you mount a gvfs share by

::

    $ gvfs-mount sftp://192.168.1.1
    $ gvfs-mount smb://192.168.1.1/share
    $ gvfs-mount dav://192.168.1.1/owncloud/files/webdav.php
    $ gvfs-mount davs://dav.box.com/dav

You will unmount it with

::

    $ gvfs-mount -u sftp://192.168.1.1

If gvfs-fuse is enabled your mount are available in
``/run/user/<id>/gvfs``.

You get info on the file system by

::

    $ gvfs-info sftp://192.168.1.1
    $ gvfs-info davs://dav.box.com/dav
    $ gvfs-info /run/user/12345/gvfs/dav:host=dav.box.com,ssl=true

You can then use it from any gio enabled application or in command line
with

::

    $ gvfs-ls sftp://192.168.1.1/my/path
    $ gvfs-ls smb://192.168.1.1/
    $ gvfs-cat http://192.168.1.1/path
    $ gvfs-cat dav://192.168.1.1/owncloud/files/webdav.php/pim/address.txt
    $ gvfs-tree smb://192.168.1.1/share

And unmount it by

::

    $ gvfs-mount -u sftp://192.168.1.1

You can use it under emacs in
`Tramp
<http://www.gnu.org/software/emacs/manual/html_node/tramp/GVFS-based-methods.html>`__
with a slightly different syntax. To load a file:

::

    /dav:user@192.168.1.1:/owncloud/files/webdav.php/pim/address.txt

To browse a directory:

::

     /dav:user@192.168.1.1:/owncloud/files/webdav.php:

Archive backend allow to read and write `all formats
<https://github.com/libarchive/libarchive/wiki/LibarchiveFormats>`__
supported by `libarchive <https://github.com/libarchive/>`__:

    - read and write: tar, cpio, pax , gzip , zip xz, lzip, lzma, ar
    - read only: iso9660, 7-Zip, mtree,
      :wikipedia:`xar <xar (archiver)>`, lha/lzh, microsoft CAB.

To command-line use of the archive backend is a bit harder, because the
path of the archive is url-encoded. If you want to read the archive
whose path is ``//path/to/my/archive.tgz`` you do:

::

    $ gvfs-mount archive://file%3A%2F%2F%2Fpath%2Fto%2Fmy%2Farchive.tgz

Of course it's somewhat painful to do it by hand, you can use a script to
url encode and better do:

::

    $ gvfs-mount archive://file$(urlencode ':///path/to/my/archive.tgz')

The easier way to access your ``gvfs`` share with a non ``gpio`` enabled
software is to use the ``gvfs-fuse`` gateway. It needs that the
``gvfs-fuse-daemon`` is running. Otherwise launch it with

::

    $ /usr/lib/gvfs-fuse-daemon ~/.gvfs

Then you should see the virtual file system mounted in your directory
``/run/<user_id>/gvfs``

::

    $ ls /run/1234/gvfs
    archive.tgz sftp on 192.168.1.1

You can also mount your obex enabled device, (it may be a phone) by

::

    $ gvfs-mount obex://[00:0F:DE:72:22:D5]

Other medias (gphoto2, cdda) and network file system are also available.

Gvfs is directly enabled in all gpio enabled applications,it includes gnome
applications.

In many modern file managers like *nautilus* or *pcmanfm* you can
directly open gvfs file system like: `sftp://192.168.1.1` or
`davs://dav.box.com/dav`.


For other applications you have to either use
`udiskctl
<http://storaged.org/doc/udisks2-api/latest/udisksctl.1.html>`_,
*Fuse* as shown above, a special bridge as
`Gigolo <http://www.uvena.de/gigolo/>`_ *this is a Debian package* or
the `Tramp Gvfs backend
<http://www.gnu.org/software/emacs/manual/html_node/tramp/GVFS-based-methods.html>`__
for Emacs.

The gvfs deamon can automount gvfs backends, this is an option set in
``/usr/share/gvfs/mounts/<gvfs service>``. The following one is set by
default:

::

    $ cat /usr/share/gvfs/mounts/network.mount
    [Mount]
    Type=network
    Exec=/usr/lib/gvfs/gvfsd-network
    AutoMount=true

To use one's own rules, create ``~/.gvfs/mounts``.

MTP
---
mtp devices
~~~~~~~~~~~

Detecting mtp devices
+++++++++++++++++++++

Mtp devices will not show with :ref:`disk devices commands
<disk-devices>`

As they are usb devices they will appear with ``lsusb``
::

    $ lsusb
    Bus 001 Device 006: ID 0fce:01b5 Sony Ericsson Mobile Communications AB
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

Then you can have a mode detailed output for the device with
::

    $ lsusb -v -s 001:006

To list them with *libmtp* use
::

    $ mtp-detect

It will give a long detailled list of connected devices, that include
the bus and device address of the device, and all its capabilities.


With *jmtpfs* you can list them with
::

    $ jmtrpfs -l

give a short list of bus and device address of the connected devices.


With *mtp-tools* you don't need to mount your device, you connect
with ``mtp-connect`` and use any individual tool.


To mount the device with jmtpfs use
::

    $ jmtpfs /path/to/mountpoint
    $ jmtpfs --device=<busnum>,<devnum> /path/to/mountpoint.

The second command is used when you have many devices connected.

The mountpoint appear as any filesystem and you can use the common
utilities.


You unmount with ``fusermount``.
::

    $ fusermount -u /path/to/mountpoint

To automate the process you can put in your fstab
::

    jmtpfs /media/mtp fuse  noauto,rw,nosuid,nodev,user 0 0

Then you can mount and unmount with the ordinary
::

    $ mount /media/mtp
    $ umount /media/mtp

``/media/mtp`` should be in the ``fuse`` group, with group permissions
``rwx``.

To mount using *gvfs* with the backend *gvfs-mtp*, you first need to
know the location of your device, you can use ``lsusb``, then use
::

    $ gvfs-mount mtp://[001,006]

or::

    $ gvfs-mount --device '/dev/bus/usb/001/006'

The command output the location of the mount point in
``/run/user/<id>/gvfs``. You can get more info on the root node of the
device by one of
::

    $ gvfs-info mtp://[001,006]
    $ gvfs-info /run/user/<id>/gvfs/<mountpoint>

To list the file under any node in the fs tree:
::

    $ gvfs-ls mtp://[001,006]
    $ gvfs-ls /run/user/<id>/gvfs/<mountpoint>

Again ``gvfs-info`` will give a detailled listing of any node,
including all the `GIO GFile attributes
<https://developer.gnome.org/gio/stable/gio-GFileAttribute.html>`_
::

    $ gvfs-info mtp://[001,006]/path/to/file
    $ gvfs-info /run/user/<id>/gvfs/<mountpoint>/path/to file


or to any folder or file under the mountpoint with
::

    $ gvfs-info /run/user/<id>/gvfs/<mountpoint>/path/to/folder_or_file


You can then use all the gvfs file management on this folder:
``gvfs-ls``, ``gvfs-copy``, ``gvfs-rm``, ``gvfs-trash``,
``gvfs-move``, ``gvfs-less``, ``gvfs-cat``, ``gvfs-save``.

Ordinary file management command ``cat``, ``cp``, ... may not work on this
file system as they cannot manage the gio file attributes.


You unmount it with
::

    $ gvfs-mount -u mtp://[usb:001,008]
    $ gvfs-mount -u /run/user/<id>/gvfs/<mountpoint>

Or eject with
::

    $ gvfs-mount --eject mtp://[usb:001,008]



Automounting
~~~~~~~~~~~~
..
    0fce:01b5
    bus 4, dev 3

automounting rules

.. code-block:: cfg

    # Sony D2005 mount & unmount rules
    SUBSYSTEM=="usb", ATTR{idVendor}=="0fce", ATTR{idProduct}=="01b5", MODE="0666", OWNER="your-login"
    ENV{ID_MODEL}=="D2005", ENV{ID_MODEL_ID}=="01b5", ACTION=="add", RUN+="/usr/bin/sudo -b -u your-login /usr/bin/go-mtpfs -dev=0fce:01b5 -allow-other=true /media/D2005"
    ENV{ID_MODEL}=="D2005", ENV{ID_MODEL_ID}=="4ee1", ACTION=="remove", RUN+="/bin/umount /media/D2005"

For the rule to apply, you should  disconnect and reconnect the device
either physically or by resetting the port with:

.. code-block:: console

    # echo -n "0000:00:1d.7" | tee /sys/bus/pci/drivers/ehci_hcd/unbind
    # echo -n "0000:00:1d.7" | tee /sys/bus/pci/drivers/ehci_hcd/bind
