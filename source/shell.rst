Shell Memo
==========
I only give some points which are sometime tricky in shell. There are
many good shell scripting guides, to which you can refer.

The following recipes are usually in portable shell, when it is
specifically for bash it is indicated.

For the numerous differences betwween bash and bourne shell see
`Bashism - How to make bash scripts work in dash
<http://mywiki.wooledge.org/Bashism>`_ to transform your scripts you
can use
`Autoconf manual - Portable Shell Programming
<http://www.gnu.org/savannah-checkouts/gnu/autoconf/manual/autoconf-2.69/html_node/Portable-Shell.html>`_
.

Most of the constructs given here for bash work also in the same way
for zsh which is very similar for scripting, the main [differences are
in the interactive use as set
`in linux Magazine <http://www.linux-mag.com/id/1053/>`_ and
`in zsh wiki <http://zshwiki.org/home/convert/bash>`_.
I summarise the scripting differences in the next
paragraph.

..  index:
    single: bash, vs zsh
    single: zsh, vs bash

Differences between zsh and bash scripts.
-----------------------------------------
-   zsh does not allow ``${var/#old/new}`` and ``${var/%old/new}`` for
    anchoring the match of old to the start or end of the parameter
    text, respectively. It is the only difference mentioned in the
    `zsh FAQ <http://zsh.sourceforge.net/FAQ/zshfaq02.html>`_.
-   While bash array index begin by ``0`` those of zsh begin by ``1``.
    More details in `Compatibility between Zsh and Bash
    <http://slopjong.de/2012/07/02/compatibility-between-zsh-and-bash/>`_.
-   There are some tricky differences in pattern matching. In both
    shell with the double ``[[  ... ]]`` operator the comparison for
    ``==``, ``!=``, ``=~`` is done whith shell patterns.

    The commands:

    .. code-block:: shell-session

       $ if [[ "abc" = a*c ]]; then echo true; else echo false;fi
       $ if [[ "abc" == a*c ]]; then echo true; else echo false;fi
       $ [[ "abc" = a*c ]] && echo true || echo false
       $ [[ "abc" = a*d ]] && echo true || echo false

    work in the expected way both zsh and bash.

    But in zsh we have:

    .. code-block:: shell-session

       $ if [ "abc" == a*c ]; then echo true; else echo false;fi
       zsh: = not found
       $ if [ "abc" = a*c ]; then echo true; else echo false;fi
       zsh: no matches found: a*c
       $ if [ a*c == a*c ]; then echo true; else echo false;fi
       zsh: = not found
       $ if [ a*c = a*c ]; then echo true; else echo false;fi
       zsh: no matches found: a*c



       While in bash:

    .. code-block:: shell-session

       $ if [ "abc" = a*c ]; then echo true; else echo false;fi
       false
       $ if [ "abc" == a*c ]; then echo true; else echo false;fi
       false
       $ if [ a*c = a*c ]; then echo true; else echo false;fi
       true
       $ if [ a*c == a*c ]; then echo true; else echo false;fi
       true

   As the single ``[`` does not make pattern matching in bash and the
   comparison is on the raw strings.

   But note that all these last lines are *not corrects*, as we are
   using single ``[ ... ]``; in bash the comparison is on *strings* not
   patterns; so the right way was to quote the stings and use a single
   ``=``.

   And the right expression

   ..  code-block:: shell-session

       $ if [ "abc" = "a*c" ]; then echo true; else echo false;fi
       false
       $ if [ "a*c" = "a*c" ]; then echo true; else echo false;fi
       true

   Is correct in any shell: bash, zsh, or dash.

..  index:
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

..  index:
    query string
    single: html; query

Reading a query string
----------------------
To define variable ``a`` and ``b`` with respective values from the
``QUERY`` ``a=1234&b=9876``

A very simple code that **should be avoided** because the ``eval`` here
are uncontrolled is

..  code-block:: bash

    IFS="&" set -- $QUERY
    for l in $@; do
        eval $l
    done

Read `BASH FAQ: Eval command and security issues
<http://mywiki.wooledge.org/BashFAQ/048>`_ for an explanation of the
danger, and how to avoid it.

A better solution is to use indirect variables or associative arrays,
their use is explained in the
`BASH FAQ: How can I use variable variables or associative arrays?
<http://mywiki.wooledge.org/BashFAQ/006>`_

In bash 4+ we can define an associative array var with each value:

..  code-block:: bash

    declare -A var
    IFS="&" set -- $QUERY
    for i in $@; do
        IFS="=" set -- $i
        var[$1]=$2
    done

Chris F.A Johnson gave a more complete function that does uudecode in
`Parsing Web Form Input in CGI Shell Scripts
<http://web.archive.org/web/20140807203929/http://cfajohnson.com/shell/articles/parse-query>`_
(also available `in Dr. Dobbs
<http://www.drdobbs.com/parsing-web-form-input-in-cgi-shell-scri/199103035>`_
)

..  code-block:: bash

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

You find a more detailled solution using associative bash arrays; and
another with `indirect variables
<http://mywiki.wooledge.org/BashFAQ/006>`_
in
`Bash FAQ: How do I write a CGI script that accepts parameters?
<http://mywiki.wooledge.org/BashFAQ/092>`_

Links
-----

..  index:
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

..  code-block:: shell-session

    $ ln -s existing-path alias-or-directory
    $ cp --symbolic-link name1 name2

The value of a symlink is returned by:

..  code-block:: shell-session

    $ readlink name

and the absolute path stripped from any symbolic link component, any
``.`` or ``..`` or repeated ``/`` is given by:

..  code-block:: shell-session

    $ readlink --canonicalize name
    $ readlink -f name

..  index:
    hardlink

Hardlinks
^^^^^^^^^

In POSIX systems, one file can have many names at the same time.
Since hardlinks reference inodes directly, they're restricted to the
same file system.

Since they reference the same inode the owner and permissions of to
hardlinks are always identical.

You create hardlinks with:

..  code-block:: shell-session

    $ ln  existing-path alias-or-directory
    $ cp --link name1 name2

..  index:
    reflink

Reflinks
^^^^^^^^
Reflinks are :wikipedia:`Copy-on-write` *COW* of a file; they are available
on :wikipedia:`OCFS2` and :wikipedia:`Btrfs` file systems. Reflinks
creates a new inode that shares the same disk blocks as the original
file. Reflinks works only inside the boundaries of a file system; but
in contrast to hardlinks changes to a file are not reflected to the copy.

You create hardlinks with:

..  code-block:: shell-session

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

    ..  code-block:: shell-session

        $ if echo "foo"; then echo "ok"; fi
        foo
        ok

     If you want to keep this status you get it in the ``$?`` variable.
     You can later on test it against the ``0`` value that mean *true* for
     the shell.

     ..  code-block:: shell-session

         $ echo "foo"; if [ $? -eq 0 ]; then echo "ok"; fi
         foo
         ok

-   but this convention is the opposite of C where 0 means false, so if
    you want in a shell supporting numerical expressions in ``((...))``
    to use the numerical testing, which is C compatible you have to
    negate it.

    ..  code-block:: shell-session

        $  echo "foo"; if ! (($?)); then echo "ok"; fi
        foo
        ok

-   **avoid this**: ``$?`` is a number not a test so you cannot put it
    directly in an if expression.
-   **avoid this**: ``[ 1 ]`` is **true** so a test like ``[ $? ]`` fail
    when ``$?`` is not defined, but succeed with both 0 and 1 values.

Command Epansion.
-----------------


The `bash reference
<http://www.gnu.org/software/bash/manual/bashref.html>`_
gives a long description of `command expansion
<http://www.gnu.org/software/bash/manual/bashref.html#Simple-Command-Expansion>`_

The order of expansion is very important; it is: brace expansion;
tilde expansion, parameter and variable expansion, arithmetic
expansion, command substitution from left-to-right,
word splitting; filename expansion, process substitution, quote
removal.

To quote the manual: *Only brace expansion, word splitting, and
filename expansion can change the number of words of the expansion;
other expansions expand a single word to a single word.*

parameter and command substitution.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The order of parameter and command substitution explain why we have:

..  code-block:: shell-session

    $ a=foo; a=bar echo $a
    foo

In the second assignement the parameter substitution is done *before
the assignement*. You can find more on this subject in
`Bruce Barnett Grymoire - sh, a subtle point
<http://www.grymoire.com/Unix/Sh.html#uh-14>`_.

parameter expansion, splitting and quote removal.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The order of the previous operation explains we have:

..  code-block:: shell-session

    $ IFS=: set -- a:b:c; echo $1
    a b c
    $ string="a:b:c"
    $ IFS=: set -- $string; echo $1
    a
    $ IFS=: set -- "$string"; echo $1
    a b c

as parameter expansion is done before word splitting, and quote
removal after word splitting.

But when we have:

..  code-block:: shell-session

    $ IFS=: read x y z <<< "a:b:c"; echo $x
    a
    $ string="a:b:c"
    $ IFS=: read x y z <<< $string; echo $x
    a
    $ IFS=: read x y z <<< "$string"; echo $x
    a
    $ echo $string | { IFS=: read x y z; echo $x; }
    a

In these four the input file descriptor is built before the read, and
the three input are the same.



Quoting and splitting.
^^^^^^^^^^^^^^^^^^^^^^

The bash reference decribe
`quoting
<http://www.gnu.org/software/bash/manual/bashref.html#Quoting>`_,
but sometime the combination of quoting with  *command expansion*
can be difficult to sort out.

If you set a variable to a string, when use it as parameter in the
process of `simple command expansion
<http://www.gnu.org/software/bash/manual/bashref.html#Simple-Command-Expansion>`__
it is subject to `shell expansion
<http://www.gnu.org/software/bash/manual/bashref.html#Shell-Expansions>`__
a complex process which involves `shell parameter expansion
<http://www.gnu.org/software/bash/manual/bashref.html#Shell-Parameter-Expansion>`__
and then  `word splitting
<http://www.gnu.org/software/bash/manual/bashref.html#Word-Splitting>`__.

Let's apply with a simple command, made from a simple script named
``countargs``:

..  code-block:: bash

     #! /bin/sh
     echo nbargs: $#
     i=0
     for a in "$@"; do
         i=$((i+1)) # ++i in bash!
         echo $i ':' "$a"
     done

..  code-block:: shell-session

     $ x="one two three"
     $ ./countargs $x
     nbargs: 3
     ...
     $ ./countargs "$x"
     nbargs: 1
     1 : one two three

Very simple indeed, this is the expected behavior of quotting and word
splitting.

But if you use your parameter in assignment the
rules are different, **assignment are not commands** but a preliminary to
`Simple Command Expansion
<http://www.gnu.org/software/bash/manual/bashref.html#Simple-Command-Expansion>`__
and quoting this section
..   highlights:

     The text after the ‘=’ in each variable
     assignment undergoes tilde expansion, parameter expansion,  command
     substitution, arithmetic expansion, and quote removal before being
     assigned to the variable.

There is **no word splitting** there, so the last two assignments are
valids and equivalents:

..  code-block:: shell-session

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

    $ exec 3>myfile

To close fd 3:

..  code-block:: shell-session

    $ exec >&3-

For input descripors:

..  code-block:: shell-session

    $ exec 3<myfile
    $ exec <&3-

To open a fd for read-write:

..  code-block:: shell-session

    $ exec 3<>myfile

To list open file descriptors:

..  code-block:: shell-session

    $ ls -l /dev/fd/*

or:

..  code-block:: shell-session

    $ lsof -a -p $$ -d 0-10

Copying - Moving
^^^^^^^^^^^^^^^^
To copy a file descriptor you can use:

..  code-block:: shell-session

    $ exec 3>&1
    $ exec 1>|/tmp/output1
    $ ls
    $ exec 1>&3
    $ exec 3>&-

The file descriptor 1 is copied to fd 3, then 1 is redirected to the
file ``/tmp/output1``, the first ls goes in this file, then fd 3 is
copied back to fd 1 which comes back to it's previous value; the
descriptor 3 is then closed.

The last two lines can be abbreviated in:

..  code-block:: shell-session

    $ exec 1>&-3

Swapping stdout and stderr
^^^^^^^^^^^^^^^^^^^^^^^^^^

..  code-block:: shell-session

    $ f(){ echo out; echo error >&2; }
    $ x=$(f)
    error
    $ echo $x
    out
    $ x=$(f 2>&1)
    $ echo $x
    out error
    $ x=$(f 1>&2)
    out
    error
    $ echo $x

    $ exec 3>&1; x=$(f 2>&1 1>&3); 3>&-
    out
    $ echo $x
    error

Extracting the parts of a string.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Very often we want to extract fiels from a strings separated with a
single character. It may be a space or an other character, we often
choose ``,``, ``:``, ``;``, ``!``, ``|``, ``%``, ``/`` or ``\`` but
any char can be choosen as far it is not member of the strings; if we
want to be free from this limitation we have to add an escaping
mechanism which is not dealed here.

A related problem is when the fields are separated by any sace
sequence constituted by space characters and tab characters, but it is
easily converted to the previous case with one of:

..  code-block:: bash

    string=$(echo $string0 | sed 's/[[:space:]]\{1,\}/ /g')
    string=$(echo $string0 | sed -r 's/[[:space:]]{1,}/ /g')

using either old *basic* regex or standard posix 2 *extended regex*.
But when you use the shell primitives you don't need this step as the
shell with the default *IFS* use sequence of spaces to delimit words.

For the example we suppose the fields are delimited by ``:``.

We can of course use basic coreutils

..  code-block:: bash

    string="a:123:456"
    v1=$(echo $string | cut -d: -f1)
    v2=$(echo $string | cut -d: -f2)
    v3=$(echo $string | cut -d: -f3)

But we can use only the shell even basic bourne shell or dash.

..  code-block:: bash

    string="a:123:456"
    IFS=":" set -- $string
    v1=$1
    v2=$2
    v3=$3

Or using *Here Documents* that is found in any shell:

..  code-block:: bash

    string="a:123:456"
    IFS=: read v1 v2 v3 <<EOF
    $string
    EOF

We can also use in dash, busybox ash, yash, bash, zsh and others POSIX
compatibles shells
`parameter expansion
<http://www.gnu.org/software/bash/manual/bashref.html#Shell-Parameter-Expansion>`_:

..  code-block:: bash

    string="a:123:456"
    var="${string}::"
    i=0
    while [ "$var" != ':' ]; do
      i=$((i+1))
      # drop part of string from first ':' to the end
      iter=${var%%:*}
      echo "v_$i=$iter"
      # drop begin of string upto first ':'
      var="${var#*:}"
    done

Here we just echo the variables name we don't set them. To set the
variables ``v_1``,  ``v_2``,  ``v_3``, in bash or zsh we can replace
the echo by:

..  code-block:: bash

    declare v_$i="$iter"

or use the POSIX directive *typeset* that is synonim to *declare* in
*bash* and *zsh*; but is also available in *yash*:

..  code-block:: bash

    typeset v_$i="$iter"


in any bourne shell, *dash* or *ash* we fallback to an eval.

..  code-block:: bash

    eval v_$i="$iter"

But quite harmless as we know the range of values of ``$i``.

In bash, yash or zsh we can also use *Here String*

..  code-block:: bash

    string="a:123:456"
    IFS=: read v1 v2 v3 <<< "$string"

Also bash yash and zsh can use arrays to read the values; it is peculiarly
useful when you don't know how many fields can be present.

In bash you write:

..  code-block:: shell-session

    $ string="a:123:456"
    $ IFS=: read -a v <<< "$string"
    $ echo "${v[@]}"
    a 123 456
    $ declare -p v
    declare -a v=([0]="a" [1]="123" [2]="456")

You can also use the previous code in yash or zsh; with a slightly
different syntax:

..  code-block:: shell-session

    $ string="a:123:456"
    $ IFS=: read -A v <<< "$string"
    $ echo "${v[@]}"
    a 123 456
    $ typeset -p v
    v=('a' '123' '456')



An other way in bash or zsh is to assign directly the array.

..  code-block:: shell-session

    $ string="a:123:456"
    $ IFS=: v=(${string})
    $ declare -p v
    declare -a v=([0]="a" [1]="123" [2]="456")


..  parsing-route-1_

As an example of use we parse the ouput of the ``route`` command to
find the different fields of the default route.

..  code-block:: bash

    read dest gateway mask flags metric ref use iface <<EOF
    $(route -n | grep '^0\.0\.0\.0' )
    EOF

But when we use the previous methods to parse the output of a command;
the command should be executed *before* the parsing; in some case e
might want to prefer, or be obliged, to process asynchronously the
results. Unix use pipes for this, and everything works as long as you
want to use the result of the previous process in the subsequent
one. But it can be more difficult to get the result of a subprocess
inthe parent process. This is the subject of next paragraph.

Using an asynchronous subprocess.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
A pipe open a subshell. As variables in a subshell are inaccessible
from the parent shell,  all the variables set inside a pipe are also
unavailable out of the pipe.

We often meet this problem while trying to
read from the output of a pipe:

..  code-block:: shell-session

    $ x="unset"
    $ y="unset"
    $ echo one two | { read x y; echo $x $y; }
    one two
    $ echo $x $y
    unset unset

We can use redirection to avoid a subshell, we can either use a
temporary file, a fifo or process substitution.

The use of a temporary file is allowed in bare bourne shell, dash,
ash, yash and usefull for portable script.

..  code-block:: shell-session

    $ echo "a b">|/tmp/tmpfile
    $ exec 4< /tmp/tmpfile
    $ read x y <&4
    $ echo $x $y
    a b
    $ exec 4<&-
    $ rm /tmp/tmpfile

If we need to split our string on another character than a sequence of
spaces, tabs and newlines, which constitute the default IFS, we only
change the IFS before the script and reset it or change it into a
subprocess.

But here changing IFS for only the read works as well.

..  code-block:: shell-session

    $ echo "a:b">|/tmp/tmpfile
    $ exec 4< /tmp/tmpfile
    $ IFS=":" read x y <&4
    $ echo $x $y
    a b
    $ exec 4<&-
    $ rm /tmp/tmpfile

If our shell and system admit process substitution, which is the case
of bash, and zsh on systems that support named pipes (FIFOs) or the
``/dev/fd`` files:

..  code-block:: shell-session

    $ exec 4< <(echo a b)
    $ read x y <&4
    $ echo $x $y
    a b
    $ exec 4<&-

Or simply, a more condensed form:

..  code-block:: shell-session

    $ read x y < <(echo a b)
    $ echo $x $y
    a b

This can even be used in yash with `Process redirection
<https://yash.osdn.jp/doc/redir.html#process>`_ which differs from
*Process substitution* thet is found in bash or zsh.
The syntax is slightly different:

..  code-block:: shell-session

    $ read x y <(echo a b)
    $ echo $x $y
    a b

But these custom syntaxes are not in Posix; and for portability it is
better to avoid them.

A pipe create an implicit fifo, which imply to use a subshell,
but we can also avoid it and keep the benefit of forking a producer by
using an explicit fifo. This is also available in any shell.

..  code-block:: shell-session

    $ mkfifo /tmp/fifo
    $ echo a b >/tmp/fifo &
    [1] 6934
    $ read x y </tmp/fifo
    $ echo $x $y
    a b
    [1]+  Done    echo a b > /tmp/fifo
    $ rm /tmp/fifo

We illustrate the use of process substitution to parse the ouput of
the ``route`` command to find the different fields of the default
route; that we `did previously <parsing-route-1>` sequentially.

..  code-block:: bash

    read dest gateway mask flags metric ref use iface < \
    <(route -n | grep '^0\.0\.0\.0' )


If we don't need  the asynchronous processing of the previous scripts
we can store the output of the first command in a string and read from
that string this is illustrated by the paragraph on splitting output
of a command.


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



array indexes and globbing.
---------------------------
Bash can store arrays in a shell variable, but interaction between
array indexes and pathname expansion is a tricky and badly documented
aspect of bash.

This is summarized by the next small script

..  code-block:: bash

    $ echo t[1]
    t[1]
    $ shopt -s nullglob
    $ echo t[1]

    $ touch t1
    $ echo t[1]
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


bash and zsh regex expressions.
-------------------------------

In bash, since version 3.0, you can match gnu regex, it allows to
dispense with the call to ``expr`` or ``sed`` (the price is a lesser
compatibility with older release of batch or other bourne shells, it
is definitely not posix).

You use it like this:

..  code-block:: shell-session

    $ [[ "abbbaaaaabbb" =~ a*(b*)(a*)(ab*) ]]
    $ echo "${BASH_REMATCH[@]}"
    abbbaaaaabbb bbb aaaa abbb

The array variable ``BASH_REMATCH`` contains substrings matched by parenthesized
subexpressions.

In zsh if the option ``BASH_REMATCH`` is set the result is identical
to the bash one. Otherwise we get the global match with ``$MATCH``;
and the groups with ``${match[@]}``

..  code-block:: shell-session

    $ [[ "abbbaaaaabbb" =~ a*(b*)(a*)(ab*) ]]
    $ echo "$MATCH"
    abbbaaaaabbb
    $necho "${match[@]}"
    bbb aaaa abbb

The match in bash and the default on zsh is using POSIX regex
functions, the same we use in *grep*.

Zsh can also test the regexp as a PCRE regular expression by setting the option
``RE_MATCH_PCRE``.


Using ``getopts``
-----------------
``getopts`` is a POSIX function; defined in bash and zsh.

In bash the details are reviewed in `abs: example 11-8
<http://tldp.org/LDP/abs/html/internal.html#EX33>`_,
and you get a summary by typing ``help getops`` under the shell.

In zsh ``$ getopts`` followed by  ``Esc-h`` give the interactive help.

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
