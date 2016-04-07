===================
Access Control List
===================

ACL References
==============

-  Acl is described in :man:`acl` (5), its use is in :man:`setfacl` (1)
   and :man:`getfacl` (1).
-  `ArchWiki: Access Control Lists
   <https://wiki.archlinux.org/index.php/Access_Control_Lists>`_
-  `POSIX Access Control Lists on Linux
   <http://www.suse.de/~agruen/acl/linux-acls/online/>`_ by
   Andreas Grünbacher discuss ACL and extended attributes on Linux file
   systems.
-  `Debian Wiki: Access Control Lists in Linux
   <https://wiki.debian.org/Permissions#Access_Control_Lists_in_Linux>`_
-  `Using ACLs with Fedora Core 2
   <http://www.vanemery.com/Linux/ACL/linux-acl.html>`_ by Van Emery
   is an old How-To *But neither outdated nor Fedora specific*.
-  `eiciel <http://rofi.roger-ferrer.org/eiciel/>`_ is the GNOME file
   ACL editor.
-  `ACLbit <http://aclbit.sourceforge.net/>`__ is an ACL Backup and
   Inspect Tool
-  `python-pylibacl <http://pylibacl.k1024.org/>`__ provide an acl
   interface to python
-  `NFSV4 has an ACL support
   <http://www.citi.umich.edu/projects/nfsv4/linux/using-acls.html>`_
   The nfsv4 acls are thinner than the posix acls, even if they get
   translated to posix. you manage nfsv4 acl with *nfs4-acl-tools*
   *nfs4_getfacl* and *nfs4_setacl*.

Access acl
==========

Access acl use *owner*, *group*, *other* inherited from file
permission bits and a list of named user and group access. The group
owner, named used and group access form the *group class*.

In a system call when the new object is created in a directory the
access is further modified by the *mode* parameter (9 bits fields), in
case of default acl the 'umask' is not used.

To access an object we select an acl entry by following the order:
*owner*, named users, (all owning or named) groups, *others*. If the
permission bit is not set in any of these acl, the access is denied.

If the permission bit is set, access is granted for an *owner* or
*other* permission. For a named user, owning group, or named group entry
the permission is granted if the corresponding bit of the group is 1.

When a new directory is created his mask permission is the union of all
permissions in the group class. The group class contains the owning
group, other, and all named user and named group permission. So the
initial mask of a directory without acl is the union of group and other.

In a directory with a mask chmod change the mask (which limit the *group
class* permission), nor the *owning group* permission.

Default acl.
============

A new directory inherit from the default acl of his parent directory both
as *access acl* and *default acl*. When a directory has extra acls it
is listed by ``ls -l`` with an extra ``+`` after the access bits.

You can reset the acls to the default value by one of:
::

    $ setfacl -b /path/to/directory
    $ setfacl --remove-all /path/to/directory


Using acl to control access.
============================

You may have some directory that contains sensible data and you don't
want to give others a read permission in any (or most) part of this
directory. It is of course easy to change the permission on all objects
of a directory, but it may be more complicated to ensure, that all files
that you will create in the future will have the same access rights.

The mask for new files is controlled by the 'umask'. Suppose your umask
is 022, you may want to have a directory with 077 mask but once *umask*
set in your environment by your .profile , it is difficult to ensure
that every process creating a file in this directory will get not the
default *umask* but a stricter one.

But acl can help to solve this problem, if you set the default acl for
your protected directory to some sensible value every process accessing
this directory, will not use the mask but the *mode* field, and files or
directory are created in accordance to your default acl.

My acl to protect crypt and other sensitive data.
=================================================


::

    setfacl -R -d user::rwx,group::---,other::---,user:root:rwx directory

    setfacl -R -d user::rwx,group::---,other::---,user:root:rwx directory

.. todo

    This section is slurped from www.mzlinux, only references are updated,
    should be made clearer.
