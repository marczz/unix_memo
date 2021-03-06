Shell Memo
==========

.. contents::
   :local:
   :depth: 1

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
for zsh which is very similar for scripting. bash and zsh differences
are mainly in the interactive. use It is explained
`in this linux Magazine article <http://www.linux-mag.com/id/1053/>`_
and `in zsh wiki <http://zshwiki.org/home/convert/bash>`_.

I summarise the scripting differences in the next
paragraph.

..  index::
    single: bash; vs zsh
    single: zsh; vs bash

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

..  index::
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

..  index::
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

..  index::
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

..  index::
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

..  index::
    reflink

Reflinks
^^^^^^^^
Reflinks are :wikipedia:`Copy-on-write` *COW* of a file; they are available
on :wikipedia:`OCFS2` and :wikipedia:`Btrfs` file systems. Reflinks
creates a new inode that shares the same disk blocks as the original
file. Reflinks works only inside the boundaries of a file system; and
in contrast to hardlinks, changes to a file are not reflected to the copy.

You create manually reflinks with:

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

..  index::
    input; caturing

capturing input
---------------
The :man:`tee` command allow to duplicate stdout.

The :man:`script` command can be used
to capture the input and output to/from an application.

To replay a session you want to
capture only the input. I achieve it by using:

..  code-block:: shell-session

     $ cat /dev/stdin|tee /tmp/session_input| application

..  index::
    return; status
    command; return status


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
-   an alternative is to use a number comparison:

    ..  code-block:: shell-session

        if [ $? -eq 0 ]; then echo "ok"; fi

..  index::
    command; expansion

Command Expansion.
------------------

The `Bash Reference`_ gives a detailled description of each part of
`shell expansion`_, wich takes place during `simple command expansion`_.

The order of expansion is very important; it is: brace expansion;
tilde expansion, `parameter <parameter expansion>`_ and variable expansion, arithmetic
expansion, `command substitution`_ from left-to-right,
`word splitting`_, `filename expansion`_, `process substitution`_, quote
removal.

To quote the manual: *Only brace expansion, word splitting, and
filename expansion can change the number of words of the expansion;
other expansions expand a single word to a single word.*

..  index::
    parameter; substitution
    command; substitution

..  _parameter_and_command_substitution:

Parameter and Command Substitution
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Parameter expansion commes before command substitution, it explains why we have:

..  code-block:: shell-session

    $ a=foo; a=bar echo $a
    foo
    $ a=foo; a=bar /bin/echo $a
    foo
    $ a=foo; a=bar; echo $a
    bar
    $ a=foo; a=bar; /bin/echo $a
    bar


In the second assignement the parameter substitution is done *before
the assignment*.

This is true for a builtin like ``echo`` or a command like ``/bin/echo``

You can find more on this subject in
`Bruce Barnett Grymoire - sh, a subtle point
<http://www.grymoire.com/Unix/Sh.html#uh-14>`_.

..  index::
    filename; expansion

Filename expansion
^^^^^^^^^^^^^^^^^^
After `word splitting`_ comes `filename expansion` where all the `patterns
<Pattern Matching>`_ found in the filename are expanded.

The following are done with default shopt settings which disable ``dotglob`` and ``nullglob``.

..  code-block:: shell-session

    $ mkdir dir
    $ cd dir
    $ touch b .b a aa aaa .a .aa ..a ba ab bab
    $ echo a*
    a  aa  aaa  ab
    $ echo *a
    a aa aaa ba
    $ echo a?
    aa ab
    $ echo ?a
    aa ba
    $ echo ??
    aa ab ba
    $ echo *
    a aa aaa ab b ba bab
    $ echo .*
    . .. .a ..a .aa .b
    $ echo * .*
    a aa aaa ab b ba bab . .. .a ..a .aa .b
    $ echo .?*
    .. .a ..a .aa .b
    $ echo .??*
    ..a .aa

As we see there is no easy way to catch all files from a directory, the default does not
match dotfiles, and if we add the initial dot we catch too less or too much.
If we use bash you we can set an option ``dotglob`` so pattern can cath the dot, let try
again:

..  code-block:: shell-session

    $ shopt -s dotglob
    $ echo *a
    a  .a  ..a  aa  .aa  aaa  ba
    $ echo a?
    aa  ab
    $ echo ?a
    .a  aa  ba
    $ echo ??
    .a  aa  ab  .b  ba
    $ echo *
    a .a ..a aa .aa aaa ab b .b ba bab
    $ echo .*
    . .. .a ..a .aa .b
    $ echo .?*
    .. .a ..a .aa .b
    $ echo .??*
    ..a .aa

It is now easy to catch all files from the directory, but
as we see for the last three commands none of these patterns expand to the *dotfiles*,
the first catch also the directory itself and the parent directory, the next one catch
also the  parent directory, and the last one miss some filenames. Making an operation
only on dotfiles is quite common so we must find a way to match all of them, and only
them.

In bash we have an environment variable ``GLOBIGNORE`` which is not set by default, f
``GLOBIGNORE`` is set, each matching file name that also matches one of the patterns in
``GLOBIGNORE`` is removed from the list of matches.
After setting globignore by ``GLOBIGNORE=.:..`` we get

..  code-block:: shell-session

   $ echo .*
   .a ..a .aa .b

which is the proper list of dotfiles.

A special feature is that setting ``GLOBIGNORE`` to a non null value always disable matching
``.`` and ``..`` so we have had the same result by setting ``GLOBIGNORE=qwertyuop`` !

Note that if we want to change ``GLOBIGNORE`` only for some expansions we ca do it in a
subshell.

..  code-block:: shell-session

    $ (GLOBIGNORE=.:..; echo .*)
    .a ..a .aa .b

but as  :ref:`explained above <parameter_and_command_substitution>`,
the following does not work :

..  code-block:: shell-session

    $ GLOBIGNORE=.:.. echo .*
    . .. .a ..a .aa .b

The previous use of ``shopt`` is only possible in bash, these options don't exist in
plain bourne shell. But we can always match all dotfiles with

..  code-block:: shell-session

    $ echo .[!.]* ..?*
    .a .aa .b ..a
    $ echo .[^.] .??*
    .a .b ..a .aa

When a range match begins with ``!`` or a ``^`` any character not
enclosed is matched, so with these two patterns we get rid of the unwanted ``.`` and
``..``.

..  index::
    parameter; expansion
    word; splitting
    quote; removal

Parameter Expansion, Splitting and Quote Removal.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As `parameter expansion`_ is done before `word splitting`_, and `quote removal`_
after `word splitting`_, we have:

..  code-block:: shell-session

    $ IFS=: set -- a:b:c; echo $1
    a b c
    $ string="a:b:c"
    $ IFS=: set -- $string; echo $1
    a
    $ IFS=: set -- "$string"; echo $1
    a b c


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

..  index::
    command; expansion
    assignment; expansion
    word; splitting
    string; quoting

..  _expansion:


Quoting and splitting.
^^^^^^^^^^^^^^^^^^^^^^

The bash reference describe `Quoting`_ but sometime the combination of
quoting with `Command Expansion`_ can be difficult to sort out.

If you set a variable to a string, when use it as parameter in the
process of `Simple Command Expansion`_ it is subject to
`Shell Expansion`_ a complex process which involves
`Shell Parameter Expansion`_, `Command Substitution`_, *Arithmetic
Expansion*, `Process Substitution`_, and then `Word Splitting`_
followed by `Filename Expansion`_ and `Quote removal`_.

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

..  _assignment_expansion:

But if you use your parameter in assignment the
rules are different, **assignment are not commands** but a preliminary to
`Simple Command Expansion`_ and quoting this section

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

Now if you use the special characters used for glob patterns like
``?`` and ``*`` in a word assigned to a variable, and evaluate the
variable afterward the `Filename Expansion`_ can occur so you get:

..  code-block:: shell-session

    $ x=sound*
    $ echo $x
    sound.wav
    $ echo "$x"
    sound*
    $ y=$x
    $ echo "$y"
    sound*
    $ y=$(echo $x)
    $ echo "$y"
    sound.wav
    $ y="$(echo $x)"
    $ echo "$y"
    sound.wav

Note that in the first assignement we have not used quotes, because as
:ref:`set previously<assignment_expansion>` in the text after an
assignement the is no `Filename expansion`_ but of course there is
quote removal, so the three following assignements are identical:

..  code-block:: shell-session

    $ x='sound*'
    $ x="sound*"
    $ x='sound*'


In the same way the line breaks in a string assigned to a variable are
used for word splitting so:

..  code-block:: shell-session

    $ x="
    > one
    > two
    > three"
    5231$ echo $x
    one two three
    5232$ echo "$x"

    one
    two
    three
    $ y=$x
    $ echo "$y"

    one
    two
    three


As usual a trailing backslach remove the following newline:

..  code-block:: shell-session

    $ x="one \
    two \
    three"
    $ echo "$x"
    one two three

The other backslash escape are uninterpreted, but you can force the
interpretation with the command :man:`echo` with the ``-e`` argument
or the command :man:`printf`. Both commands exists as
`Bash builtins`_.
As *printf* is not a dash builtin, and the dash ``echo`` builtin does
not support the  ``-e`` argument; in portable shell you better rely on
the command :man:`printf` or the command :man:`echo`.

..  code-block:: shell-session

    $ x="one\ntwo\nthree"
    $ echo $x
    one\ntwo\nthree
    $ echo -e $x
    one
    two
    three
    $ printf "%b\n" $x
    one
    two
    three


..  index::
    see: file descriptor; fd
    fd
    redirection




file descriptors
----------------

Reference
^^^^^^^^^
-   `Wikipedia: File descriptor
    <https://en.wikipedia.org/wiki/File_descriptor>`_
-   Redirections are described in the
    `Redirections`_ section of the `Bash Reference`_  Manual.
-   Advanced bash scripting guide has also a `section on redirection
    <http://tldp.org/LDP/abs/html/io-redirection.html>`_
    that has more elaborated examples, than the following recipes.
-   There are also many topics on redirection in the
    `Bash FAQ <http://mywiki.wooledge.org/BashFAQ>`_ :
    `Redirect stderr to a pipe
    <http://mywiki.wooledge.org/BashFAQ/047>`_,
    `Redirect the output of multiple commands at once
    <http://mywiki.wooledge.org/BashFAQ/014>`_,
    `Send all  output to a log file
    <http://mywiki.wooledge.org/BashFAQ/106>`_.

.. index::
   fd; open
   fd; close
   fd; list

Opening - Closing - Listing
^^^^^^^^^^^^^^^^^^^^^^^^^^^
To assign fd 3 to myfile:

..  code-block:: shell-session

    $ exec 3>myfile

To close fd 3:

..  code-block:: shell-session

    $ exec >&3-

For input descriptors:

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

..  index::
   fd; copy
   fd; move

Copying - Moving
^^^^^^^^^^^^^^^^
To copy a file descriptor you can use:

..  code-block:: shell-session
    :linenos:

    $ exec 3>&1
    $ exec 1>|/tmp/output1
    $ ls
    $ exec 1>&3
    $ exec 3>&-

In line *(1)* the file descriptor 1 is copied to fd 3, then *(2)* 1 is redirected to the
file ``/tmp/output1``, *(3)* the first ls goes in this file, then *(4)* fd 3 is
copied back to fd 1 which comes back to it's previous value; *(5)* the
descriptor 3 is then closed.

In Bash *but not dash* the last two lines can be abbreviated in:

..  code-block:: shell-session

    $ exec 1>&-3

..  index::
    fd; swap

Swapping stdout and stderr.
^^^^^^^^^^^^^^^^^^^^^^^^^^^

..  code-block:: shell-session
    :linenos:

    $ f(){ echo out; echo error >&2; }
    $ f
    out
    error
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

    $ x=$(f 3>&2 2>&1 1>&3)
    out
    $ echo $x
    error
    $ exec 3>&2; x=$(f 2>&1 1>&3); 3>&-
    out
    $ echo $x
    error
    $ exec 3>&1; x=$(f 2>&1 1>&3); 3>&-
    out
    $ echo $x
    error


\(2)
    Normal call to ``f``, both error and output go to standard output.

\(5)
    The output is assigned to ``x`` and the error go to stdout.

\(9)
    The output fd replace the error, so both fd go to stdout and are
    assigned to ``x``.

\(12)
    The error fd replace the output, so both echos are going to error
    and nothing on stdout, ``x`` is empty.

\(17)
    We execute f in a context where error is saved as fd 3, then
    output replacing error, previous saved error replace output.

\(21)
    is similar to *(17)* except that we save stderr before evaluating
    the expression.

\(25)
    Is harder to understand, and can seem paradoxal first, what use of
    saving fd (**f**\ ile **d**\ escriptor) 1 for copying it later to
    the same fd 1?

    But the standard output outside and inside of the `Command
    Substitution`_ are not the same.  Before the `Command
    Substitution`_ the current output is untouched, if we execute this
    script from a pseudo terminal it is /dev/pts/0 and this value is
    saved to fd 3.

    Inside ``$(...)`` which is a `Command Substitution`_ construct,
    the standard output is redirected to a pipe to capture the output;
    we copy this pipe to fd 2 with ``2>&1``, then the fd 3 containing
    the original output is copied back to the current output that is
    fd 1.

    We execute the function in this context and affect the content of
    the pipe, that is what goes to stderr inside the function, to the
    variable; then fd 3 is closed.

..  index::
    read; from text bloc
    read; from string


Reading from a bloc of text.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The shell builtin **read** first aim is reading from standard input or
any file descriptor. But we can use it with `Here Documents`_, or with
`Here Strings`_ *in bash or zsh but not dash, it is not a portable
construct*, in a pipe, or with `process substitution`_.

In standard shell like dash when we read from an `Here Documents`_
the builtin *read* read one line each time. In
bash or zsh we can change the delimiter and read the whole *Here
Document* or *Here String* in a single shell variable.

..  code-block:: shell-session
    :linenos:

    $ f=foo
    $ read -d'' x <<EOF
    $f and bar
            go in a boat
    EOF
    $ echo "$x"
    foo and bar
            go in a boat
    $ read -d'' x << "EOF"
    $f and bar
            go in a boat
    EOF
    $ echo "$x"
    $f and bar
        go in a boat
    $ read -d'' x <<- EOF
    $f and bar
            go in a boat
    EOF
    $ echo "$x"
    foo and bar
    go in a boat

As seen in the previous example *(9)* quoting the end string disable
variable expansion, and *(16)* using ``<<-`` delete initial
tabulations.

..  index::
    substring; extract

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

Or using `Here Documents`_ that is found in any shell:

..  code-block:: bash

    string="a:123:456"
    IFS=: read v1 v2 v3 <<EOF
    $string
    EOF

We can also use in dash, busybox ash, yash, bash, zsh and others POSIX
compatibles shells
`Shell Parameter expansion`_

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

..  index::
    seealso: processes; shell subprocess
    shell subprocess; asynchronous

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
temporary file, a fifo or `process substitution`_.

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

If our shell and system admit `process substitution`_, which is the case
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

This can even be used in yash with `process redirection`_ whose syntax is different
from `process substitution`_ which is found in bash or zsh.

..  code-block:: shell-session

    $ read x y <(echo a b)
    $ echo $x $y
    a b

As these custom syntaxes are not in Posix; and for portability it is
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

We illustrate the use of `process substitution`_ to parse the ouput of
the ``route`` command to find the different fields of the default
route; that we `did previously <parsing-route-1>` sequentially.

..  code-block:: bash

    read dest gateway mask flags metric ref use iface < \
    <(route -n | grep '^0\.0\.0\.0' )


If we don't need  the asynchronous processing of the previous scripts
we can store the output of the first command in a string and read from
that string this is illustrated by the paragraph on splitting output
of a command.

..  index::
    command; time

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


..  index::
    seealso : arrays;  shell array
    shell array
    shell array; index
    shell array; globbing


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

The same pathname expansion is done after an ``unset`` command, so doing
``unset t[1]`` may result in unsetting the array element ``t[1]`` or
the unsetting the variable ``t1`` or causing an error or doing
nothing, depending on the presence of a file named ``t1`` and of the
setting of the options ``nullglob``, ``failglob``, ``extglob``, and the
environment variable ``GLOBIGNORE``

So you are advised always quoting the argument of an ``unset`` and
write: ``unset 't[1]'``.

*Note that within an expression like* ``${t[1]}`` *braces disable pathname
expansion*


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


Using ``getopts``.
------------------
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

using ``getopt``.
-----------------

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

``getopt`` and whitespaces.
^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

..  _Bash Reference: http://www.gnu.org/software/bash/manual/bashref.html
..  _Command Expansion:
..  _Simple Command Expansion: https://www.gnu.org/software/bash/manual/bashref.html#Simple-Command-Expansion
..  _Filename expansion: https://www.gnu.org/software/bash/manual/bashref.html#Filename-Expansion
..  _Pattern Matching: https://www.gnu.org/software/bash/manual/bashref.html#Pattern-Matching
..  _Command Substitution: https://www.gnu.org/software/bash/manual/bashref.html#Command-Substitution
..  _Quoting: https://www.gnu.org/software/bash/manual/bashref.html#Quoting
..  _Quote Removal: https://www.gnu.org/software/bash/manual/bashref.html#Quote-Removal
..  _Shell Expansion: https://www.gnu.org/software/bash/manual/bashref.html#Shell-Expansions
..  _parameter expansion:
..  _Shell parameter expansion: https://www.gnu.org/software/bash/manual/bashref.html#Shell-Parameter-Expansion
..  _Word Splitting: https://www.gnu.org/software/bash/manual/bashref.html#Word-Splitting
..  _Here Documents: https://www.gnu.org/software/bash/manual/bashref.html#Here-Documents
..  _Here Strings: https://www.gnu.org/software/bash/manual/bashref.html#Here-Strings
..  _Bash Builtins: https://www.gnu.org/software/bash/manual/bashref.html#Bash-Builtins
..  _Redirections: https://www.gnu.org/software/bash/manual/bashref.html#Redirections
..  _Process Substitution: https://www.gnu.org/software/bash/manual/bashref.html#Process-Substitution
..  _Process Redirection: https://yash.osdn.jp/doc/redir.html#process
