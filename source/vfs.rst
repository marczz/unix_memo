Virtual File Systems
====================



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

The command ouptput the location of the mount point in
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
