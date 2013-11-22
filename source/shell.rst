Shell Memo
==========

Reading query string
--------------------
To define variable ``a`` and ``b`` with respective values from the
``QUERY`` ``a=1234&b=9876``

.. code-block:: bash

   oldifs=$IFS
   IFS="&"
   set -- $QUERY
   IFS=$oldifs
   for l in $@; do
       eval $l
   done

Or in bash 4+ to define an associative array var with each value:

.. code-block:: bash

   declare -A var
   oldifs=$IFS
   IFS="&"
   set -- $QUERY
   IFS=$oldifs
   for i in $@; do
       IFS="="
       set -- $i
       IFS=$oldifs
       var[$1]=$2
   done


A more complete function that does uudecode is given in
`Parsing Web Form Input in CGI Shell Scripts
<http://cfajohnson.com/shell/articles/parse-query/>`_

.. code-block:: bash

   parse_query() #@ USAGE: parse_query var ...
   {
       local var val
       local IFS='&'
       vars="&$*&"
       [ "$REQUEST_METHOD" = "POST" ] && read QUERY_STRING
       set -f
       for item in $QUERY_STRING
       do
         var=${item%%=*}
         val=${item#*=}
         val=${val//+/ }
         case $vars in
             *"&$var&"* )
                 case $val in
                     *%[0-9a-fA-F][0-9a-fA-F]*)
                          printf -v val "%b" "${val//\%/\\x}"
                          ;;
                 esac
                 ;;
         esac
       done
       set +f
   }

Links
-----

Symlinks
^^^^^^^^

Symlinks store in an inode any path in the system hierarchy. The
symlink act as a pointer to another file name. The path
can be absolute or relative; existing or dangling.
Permissions are ignored in symlink inodes.

Unlike hard links, symbolic links can be made to directories or across
file systems with no restrictions.

You create symlinks with::

   ln -s existing-path alias-or-directory
   cp --symbolic-link name1 name2

The value of a symlink is returned by::

   readlink name

and the absolute path stripped from any symbolic link component, any
``.`` or ``..`` or repeated ``/`` is given by::

   readlink --canonicalize name
   readlink -f name

Hardlinks
^^^^^^^^^

In POSIX systems, one file can have many names at the same time.
Since hardlinks reference inodes directly, they're restricted to the
same file system.

Since they reference the same inode the owner and permissions of to
hardlinks are always identical.

You create hardlinks with::

   ln  existing-path alias-or-directory
   cp --link name1 name2



Reflinks
^^^^^^^^
Reflinks are :wikipedia:`Copy-on-write` *COW* of a file; they are available
on :wikipedia:`OCFS2` and :wikipedia:`Btrfs` file systems. Reflinks
creates a new inode that shares the same disk blocks as the original
file. Reflinks works only inside the boundaries of a file system; but
in contrast to hardlinks changes to a file are not reflected to the copy.

You create hardlinks with::

   cp --reflink name1 name2

References
^^^^^^^^^^

See also:

-   :coreutils:`ln`, :coreutils:`readlink`, :coreutils:`cp`.

-   `libc manual: Symbolic Links
    <http://www.gnu.org/savannah-checkouts/gnu/libc/manual/html_node/Symbolic-Links.html>`_,
    `libc manual: Hardlinks
    <http://www.gnu.org/savannah-checkouts/gnu/libc/manual/html_node/Hard-Links.html>`_
