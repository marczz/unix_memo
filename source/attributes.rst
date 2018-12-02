===============
File Attributes
===============



References
==========
-  `Extended attributes: the good, the not so good, the bad.
   <http://www.lesbonscomptes.com/pages/extattrs.html>`_
-  :wikipedia:`chattr`
-  man pages: :man:`lsattr(1)`, :man:`chattr(1)`
----------

attributes memo
===============

Only the superuser or a process possessing the ``CAP_LINUX_IMMUTABLE``
capability can set or clear attribute.

-  file ‘a’ attribute: can only be open in append mode for writing.
-  directory ‘D’ attribute write synchronously on the disk; this is
   equivalent to the ‘dirsync’ mount option, but only applied to the
   directory.
-  file ‘d’ attribute means no backup with *dump*.
-  directory 'I' unchangeable) used by the htree code to indicate that a
   directory is being indexed using hashed trees.
-  file ‘i’ ''immutable'' attribute cannot be modified: it cannot be
   deleted or renamed, no link can be created to this file and no data
   can be written to the file.
-  file ‘j’ attribute all data is written to the ext3 journal before
   being written to the file itself, if the filesystem is mounted with
   the "data=ordered" or "data=writeback" options. No effect if mounted
   with the "data=journal"
-  file ‘S’ attribute equivalent to the ‘sync’ mount option applied to a
   subset of the files.
-  directory ’T’ attribute used by the Orlov block allocator
   to indicate the top of directory hierarchies
-  file 'u' attribute ask for undeletion support.
-  ‘c’ 's' 'u' are auto compression flags to attribute compress the file
   on disk and uncompress at read, not yet implemented on ext2/ext3
-  'E' ’X’ 'Z' attribute are used by the experimental compression
   patches

Attribute are listed py :man:`lsattr(1)` and changed by
:man:`chattr(1)`
-  ArchWiki: :archwiki:`File attributes
   <File_permissions_and_attributes#File_attributes>`.


Extended file attributes
========================

The ext2, ext3, ext4, JFS, Squashfs, ReiserFS, XFS, Btrfs and OCFS2
1.6 filesystems support extended attributes (abbreviated xattr) when
enabled in the kernel configuration. Any regular file or directory may
have extended attributes consisting of a name and associated data.

:man:`getfattr(1)` and :man:`setfattr(1)` utilities retrieve and set xattrs.
-  ArchWiki: :archwiki:`Extended attributes
   <File_permissions_and_attributes#Extended_attributes>`.

