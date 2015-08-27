Shell Memo
==========

.. index:
   pair: shell; exit
   pair: shell; return

``exit`` and ``return``
-----------------------

Refs:
    `Bash Reference Manual: Bourne Shell Builtins
    <http://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html>`_

return
^^^^^^

``return [n]`` return n or if not supplied the exit status of the last
command. In a function it  return with the  value. When in a script
being sourced with the ``.`` or ``source`` builtin terminate the
script with the return value.

Any command associated with the RETURN trap is executed before
execution resumes after the function or script.

A return outside of a function or sourced script, will not terminate
the script nor return the value, it produces an error.
If the shell is running with ``-e`` option, it interrupt the script
with an error code.

exit
^^^^

``exit [n]``: Exit the shell, returning a status of n to the shell’s
parent. If n is omitted, the exit status is that of the last command
executed. Any trap on EXIT is executed before the shell terminates.

.. index:
   query string
   single: html; query

Reading a query string
----------------------
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

.. index:
   symlink

Symlinks
^^^^^^^^

Symlinks store in an inode any path in the system hierarchy. The
symlink act as a pointer to another file name. The path
can be absolute or relative; existing or dangling.
Permissions are ignored in symlink inodes.

Unlike hard links, symbolic links can be made to directories or across
file systems with no restrictions.

You create symlinks with:

.. code-block:: shell-session

   $ ln -s existing-path alias-or-directory
   $ cp --symbolic-link name1 name2

The value of a symlink is returned by:

.. code-block:: shell-session

   $ readlink name

and the absolute path stripped from any symbolic link component, any
``.`` or ``..`` or repeated ``/`` is given by:

.. code-block:: shell-session

   $ readlink --canonicalize name
   $ readlink -f name

.. index:
   hardlink

Hardlinks
^^^^^^^^^

In POSIX systems, one file can have many names at the same time.
Since hardlinks reference inodes directly, they're restricted to the
same file system.

Since they reference the same inode the owner and permissions of to
hardlinks are always identical.

You create hardlinks with:

.. code-block:: shell-session

   $ ln  existing-path alias-or-directory
   $ cp --link name1 name2

.. index:
   reflink

Reflinks
^^^^^^^^
Reflinks are :wikipedia:`Copy-on-write` *COW* of a file; they are available
on :wikipedia:`OCFS2` and :wikipedia:`Btrfs` file systems. Reflinks
creates a new inode that shares the same disk blocks as the original
file. Reflinks works only inside the boundaries of a file system; but
in contrast to hardlinks changes to a file are not reflected to the copy.

You create hardlinks with:

.. code-block:: shell-session

   $ cp --reflink name1 name2

References
^^^^^^^^^^

See also:

-   :coreutils:`ln`, :coreutils:`readlink`, :coreutils:`cp`.

-   `libc manual: Symbolic Links
    <http://www.gnu.org/savannah-checkouts/gnu/libc/manual/html_node/Symbolic-Links.html>`_,
    `libc manual: Hardlinks
    <http://www.gnu.org/savannah-checkouts/gnu/libc/manual/html_node/Hard-Links.html>`_


capturing input
---------------
The :man:`tee` command allow to duplicate stdout.

The :man:`script` command can be used
to capture the input and output to/from an application.

To replay a session you want to
capture only the input. I achieve it by using:

..  code-block:: shell-session

     $ cat /dev/stdin|tee /tmp/session_input| application

command return status
---------------------
-   You can test the status of a command by executing it in a conditional
    block:

    .. code-block:: shell-session

        $ if echo "foo"; then echo "ok"; fi
        foo
        ok

     If you want to keep this status you get it in the ``$?`` variable.
     You can later on test it against the ``0`` value that mean *true* for
     the shell.

     .. code-block:: shell-session

         $ echo "foo"; if [ $? -eq 0 ]; then echo "ok"; fi
         foo
         ok

-   but this convention is the opposite of C where 0 means false, so if
    you want in a shell supporting numerical expressions in ``((...))``
    to use the numerical testing, which is C compatible you have to
    negate it.

     .. code-block:: shell-session

        $  echo "foo"; if ! (($?)); then echo "ok"; fi
        foo
        ok

-   **avoid this**: ``$?`` is a number not a test so you cannot put it
    directly in an if expression.
-   **avoid this**: ``[ 1 ]`` is **true** so a test like ``[ $? ]`` fail
    when $? is not defined, but succeed with both 0 and 1 values.

quoting and splitting
---------------------
The `bash reference
<http://www.gnu.org/software/bash/manual/bashref.html>`__
gives a long description of `command expansion
<http://www.gnu.org/software/bash/manual/bashref.html#Simple-Command-Expansion>`__
and of
`quoting
<http://www.gnu.org/software/bash/manual/bashref.html#Quoting>`__,
but sometime the combination of the two can be difficult to sort out.

If you set a variable to a string, when use it as parameter in the
process of `simple command expansion
<http://www.gnu.org/software/bash/manual/bashref.html#Simple-Command-Expansion>`__
it is subject to `shell expansion
<http://www.gnu.org/software/bash/manual/bashref.html#Shell-Expansions>`__
a complex process which involves `shell parameter expansion
<http://www.gnu.org/software/bash/manual/bashref.html#Shell-Parameter-Expansion>`__
and then then `word splitting
<http://www.gnu.org/software/bash/manual/bashref.html#Word-Splitting>`__.

Let's apply with a simple command, made from a simple script named
``countargs``:

.. code-block:: bash

    #! /bin/sh
    echo nbargs: $#
    i=0
    for a in "$@"; do
        i=$((i+1)) # ++i in bash!
        echo $i ':' "$a"
    done

    $ x="one two three"
    $ ./countargs $x
    nbargs: 3
    ...
    $ ./countargs "$x"
    nbargs: 1
    1 : one two three

Very simple indeed, but if you use your parameter in assignment the
rules are different, assignment are not commands but a preliminary to
`Simple Command
Expansion <http://www.gnu.org/software/bash/manual/bashref.html#Simple-Command-Expansion>`__
and quoting this section
..  highlights:

    The text after the ‘=’ in each variable
    assignment undergoes tilde expansion, parameter expansion,  command
    substitution, arithmetic expansion, and quote removal before being
    assigned to the variable.

There is **no word splitting** there, so the last two assignments are
valids and equivalents:

.. code-block:: bash

    $ x="one two three"
    $ y=$x
    $ z="$x"
    $ echo $y
    one two three
    $ echo $z
    one two three

file descriptors
----------------

Reference
^^^^^^^^^
-   `Wikipedia: File descriptor
    <https://en.wikipedia.org/wiki/File_descriptor>`_
-   Redirections are described in the
    `Redirection section of the bash reference manual
    <http://www.gnu.org/software/bash/manual/bashref.html#Redirections>`_,
-   Advanced bash scripting guide has also a `section on redirection
    <http://tldp.org/LDP/abs/html/io-redirection.html>`_
    that has more elaborated examples, than the following recipes.

Opening - Closing -Listing
^^^^^^^^^^^^^^^^^^^^^^^^^^
To assign fd 3 to myfile:

..  code-block:: shell-session

    # exec 3>myfile

To close fd 3:

..  code-block:: shell-session

    # exec >&3-

For input descripors:

..  code-block:: shell-session

    # exec 3<myfile
    # exec <&3-

To open a fd for read-write:

..  code-block:: shell-session

    # exec 3<>myfile

To list open file descriptors:

..  code-block:: shell-session

    # ls -l /dev/fd/*

or:

..  code-block:: shell-session

    # lsof -a -p $$ -d 0-10

Copying - Moving
^^^^^^^^^^^^^^^^
To copy a file descriptor you can use:

..  code-block:: shell-session

    # exec 3>&1
    # exec 1>|/tmp/output1
    # ls
    # exec 1>&3
    # exec 3>&-

The file descriptor 1 is copied to fd 3, then 1 is redirected to the
file ``/tmp/output1``, the first ls goes in this file, then fd 3 is
copied back to fd 1 which comes back to it's previous value; the
descriptor 3 is then closed.

The last two lines can be abbreviated in:

..  code-block:: shell-session

    # exec 1>&-3

Swapping stdout and stderr
^^^^^^^^^^^^^^^^^^^^^^^^^^

..  code-block:: shell-session

    # f(){ echo out; echo error >&2; }
    # x=$(f)
    error
    # echo $x
    out
    # x=$(f 2>&1)
    # echo $x
    out error
    # x=$(f 1>&2)
    out
    error
    # echo $x

    # exec 3>&1; x=$(f 2>&1 1>&3); 3>&-
    out
    # echo $x
    error

avoiding a subshell
^^^^^^^^^^^^^^^^^^^
Variable in a subshell are inaccessible from the parent shell, as a
pipe open a subshell, we often meet this problem while trying to
read from the output of a pipe:

..  code-block:: shell-session

    # echo one two | { read x y; echo $x $y; }
    one two
    # echo one two | read x y
    # echo $x $y

We can use redirection to avoid a subshell, we can either use a
temporary file or a process substitution.

..  code-block:: shell-session

    # echo "a b">|/tmp/tmpfile
    # exec 4< /tmp/tmpfile
    # read x y <&4
    # echo $x $y
    #  exec 4<&-

Or if our system admit process substitution:

..  code-block:: shell-session

    # exec 4< <(echo a b)
    # read x y <&4
    # echo $x $y
    a b
    # exec 4<&-

Or simply:

..  code-block:: shell-session

    # read x y < <(echo a b)
    # echo $x $y
    a b

Note that implicit fifo created by a pipe, force to use a subshell,
but we can also avoid it ankeep the benefit of forking a producer by
using an explicit fifo.

..  code-block:: shell-session

    # mkfifo /tmp/fifo
    # echo a b >/tmp/fifo &
    [1] 6934
    # read x y </tmp/fifo
    # echo $x $y
    a b
    [1]+  Done    echo a b > /tmp/fifo


Elapsed time of a command
-------------------------
To get the time of a command we can use the
:man:`time` command

..  code-block:: shell-session

    $ /usr/bin/time -f "%e elapsed, %U user, %S sys" locate xzuv
    Command exited with non-zero status 1
    1.72 elapsed, 1.61 user, 0.04 sys

We can also under bash use the internal time bash command, this one can
be used not only with a command but before any pipe, command group, or
subshell

..  code-block:: shell-session

    $ time locate xzuv
    $ time (ls >/dev/null; cat /etc/passwd >/dev/null)
    $ time { ls >/dev/null; cat /etc/passwd >/dev/null; }
    real    0m0.021s
    user    0m0.000s
    sys     0m0.012s

To know the elaped time of some part of a script we can also use the
:man:`date` command:

.. code-block:: shell

    before="$(date +%s)"
    ..... #some shell commands
    after="$(date +%s)"
    echo "elapsed: $(date -u -d @$(($after - $before)) +%H:%M:%S)"



array indexes and globbing
--------------------------
Bash can store arrays in a shell variable, but a
badly documented aspect of bash, is interaction between array indexes
and pathname expansion.

This is summarized by the next small script

..  code-block:: bash

    # echo t[1]
    t[1]
    # shopt -s nullglob
    # echo t[1]

    # touch t1
    # echo t[1]
    t1

The same pathname expansion is done after an unset command, so doing
``unset t[1]`` may result in unsetting the array element ``t[1]`` or
the unsetting the variable ``t1`` or causing an error or doing
nothing, depending on the presence of a file named ``t1`` and of the
setting of the options ``nullglob``, ``failglob``, ``extglob``, and the
environment variable ``GLOBIGNORE``

So you are advised always quoting the argument of an ``unset`` and
write: ``unset 't[1]'``.

*Note that within an expression like* ``${t[1]}`` *braces disable pathname
epansion*


bash regex expressions
----------------------
In bash, since version 3.0, you can match gnu regex, it allows to
dispense with the call to ``expr`` or ``sed`` (the price is a lesser compatibility
with older release of batch or other bourne shells, it is definitely not
posix).

You use it like this:

..  code-block:: bash

    # [[ "abbbaaaaabbb" =~ 'a*(b*)(a*)(ab*)' ]]
    # echo "${ldelim}BASH_REMATCH[@]{rdelim}"
    abbbaaaaabbb bbb aaaa abbb

The array variable ``BASH_REMATCH`` contains substrings matched by parenthesized
subexpressions.

Using ``getopts``
-----------------

The details is reviewed in `abs: example 11-8
<http://tldp.org/LDP/abs/html/internal.html#EX33>`_,
and you get a summary by typing ``help getops`` under the shell.

When using it in a function it looks like that:

..  code-block:: bash

    local opt OPTARG
    local -i OPTIND=1
    while getopts :d:D:p:F opt; do
        case $opt in
            d|D) myoptarg1=$OPTARG ;;
            p) myoptarg2=$OPTARG ;;
            F) myopt3=true ;;
            *) help $FUNCNAME
            exit 2
        esac
    done
    shift $(( OPTIND - 1 ))

If not in function replace ``local`` by ``declare``

using ``getopt``
----------------

An example is given with :man:`getopt(1) <getopt>` in
``/usr/share/doc/util-linux/examples/``, it is recalled here:

..  code-block:: bash

    # We need TEMP as the `eval set --' would nuke the return value of getopt.
    TEMP=$(getopt -o ab:c:: --long a-long,b-long:,c-long:: \
         -n 'example.bash' -- "$@")
    if [ $? != 0 ] ; then echo "Terminating..." >&2 ; exit 1 ; fi
    # Note the quotes around `$TEMP': they are essential!
    eval set -- "$TEMP"
    while true ; do
       case "$1" in
           -a|--a-long) echo "Option a" ; shift ;;
           -b|--b-long) echo "Option b, argument \`$2'" ; shift 2 ;;
           -c|--c-long)
                # c has an optional argument. As we are in quoted mode,
                # an empty parameter will be generated if its optional
                # argument is not found.
                case "$2" in
                    "") echo "Option c, no argument"; shift 2 ;;
                     *)  echo "Option c, argument \`$2'" ; shift 2 ;;
                esac ;;
           --) shift ; break ;;
           *) echo "Internal error!" ; exit 1 ;;
       esac
    done
    echo "Remaining arguments:"
    for arg do echo '--> '"\`$arg'" ; done

``getopt`` and whitespaces
^^^^^^^^^^^^^^^^^^^^^^^^^^

he old version of ``getopt`` does preserve whitespaces in arguments so you

get:

..  code-block:: shell-session

    $ getopt a: -- -a "one two" "three four"
     -- -a one two three four

This stand either with the old ``getopt`` or the enhanced one, as the
latter generate output that is compatible with that of other versions,
as long as his first parameter is not an option.

This defect is the cause of ``getopt`` rejection in some manuals as in
the `SHELLdorado good coding practices
<http://www.shelldorado.com/goodcoding/cmdargs.html>`_

But if you use the enhanced version with it's new syntax you get:

..  code-block:: shell-session

    $ getopt --options a: -- -a "one two" "three four"
     -a 'one two' -- 'three four'

So the whitespaces are preserved, with the new ``getopt``;. You can
check that yourgetopt; is the enhanced one by doing ``getopt -V`` or in
a script:

..  code-block:: shell-session

    $ getopt -T
    $ if [ $? -eq 4 ]; then
    # code for new getopt

The return value of 4 is the sign of the enhanced version.
