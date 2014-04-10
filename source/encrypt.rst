========================
 Encrypted file systems
========================
See :mzlinux:`mzlinux: Encrypted File System <157>`

DM-Crypt
========

DM-crypt encrypted loopback file device.
----------------------------------------

An example of creation and use of an aes encrypted loopback file device.


-   Fill a file with random bits::

        dd if=/dev/urandom of=/tmp/crypt.img bs=1M count=10

-   Setup a loopback *unencrypted* device for the file system::

        losetup   /dev/loop0 /tmp/crypt.img

    As the encryption is at the device mapper level the loop device is
    created unencrypted.
-   Setup the aes encrypted device mapper::

        cryptsetup -v -c aes luksFormat /dev/loop0
        cryptsetup luksOpen /dev/loop0 crypt

-   Create a filesystem and use it::

        mkfs.ext3 /dev/mapper/crypt
        mkdir /tmp/mnt
        mount /dev/mapper/crypt /tmp/mnt
        echo "foo" >/tmp/mnt/file.txt
        cat /tmp/mnt/file.txt

-   Unmount the filesystem, close the device mapper
    and release the loopback device::

        umount /tmp/mnt
        cryptsetup luksClose crypt
        losetup -d /dev/loop0
