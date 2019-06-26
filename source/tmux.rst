Tmux
====

In this page I suppose that the tmux prefix is the default one that is ``Ctrl-B``, you
can change the prefix, but it is easy to translate ``B`` to any other symbol.

*In this memo the list of keys, commands, options of a command are not exhaustive. For a
full list look at* :man:`tmux(1)`.

Tmux commands
-------------

All tmux commands can be either issued by a command line
::

    $ tmux <command> <flags>

or in a tmux session by opening the command prompt with ``Ctrl_B :``.

The default command is ``new-session`` and can be omitted, so the command
::

    $ tmux

start a new *default* session.

You can have a list of commands and keys by:

::

    $ tmux list-keys
    $ tmux list-commands

but usually you will get the keys by typing ``C-B ?`` inside a tmux session.


Sessions
--------

commands
~~~~~~~~
.. csv-table::
   :delim: %
   :widths: 20, 80


   ``new -s sessname``%start new *sessname* session
   ``attach``%attach to any open session
   ``a``%attach to any open session
   ``attach -t sessname``%attach to *sessname* session
   ``new -As sessname``%attach to a session *sessname*, create it if necessary.
   ``list-sessions``%list sessions
   ``ls``%list sessions
   ``switch-client -t sessname``%switch to session *sessname*
   ``kill-session -a``%kill all sessions
   ``kill-session -t sessname``%kill  *sessname* session
   ``kill-session -at sessname``%kill all sessions but *sessname*

Keys
~~~~

.. csv-table::
   :delim: %
   :widths: 20, 80

    ``s``%list and switch to sessions
    ``$``%name session
    ``d``%detach current client
    ``(``%switch to the previous session
    ``)``%switch to the next session

Windows
-------

Keys
~~~~

.. csv-table::
   :delim: %
   :widths: 20, 80

    ``c``%create window
    ``w``%choose windows from list
    ``n``%next window
    ``p``%previous window
    ``<digit>``%go to window n° <digit>
    ``f``%find window
    ``,``%name window
    ``&``%kill window
    ``[``%enter :ref:`copy mode`
    ``Page Up``%enter :ref:`copy mode` and scroll one page up
    ``]``%paste last coppied text
    ``#``%list all paste buffers
    ``=``%choose paste buffer and paste
    ``?``%show keys in :ref:`copy mode`
    ``:``%prompt for entering commands
    ``t``%show a big clock

Execute a shell command in a new window
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can create a new window with the command
::

    :new-window [-d] [-n window-name] <shell-command>

- With ``-d`` the current window in unchanged.
- ``neww`` is an alias to ``new window``.

Examples:

  ::

    $ tmux neww 'vi ~/.tmux.conf'
    $ tmux neww -d 'rsync -avz ~/documents ssh:example.org'



Panes
-----

Keys
~~~~

.. csv-table::
   :delim: %
   :widths: 20, 80

    "``%``"%vertical split
    ``"``%horizontal split
    ``o``%next pane
    ``q``%show panes numbers [*]_,
    ``x``%kill pane
    ``+``%break pane into window
    ``-``%restore pane from window
    ``⍽``%space - toggle between layouts
    ``{``%swap current pane with previous one
    ``}``%swap current pane with next one
    ``z``%toggle pane zoom
    ``Up``%go to upper pane
    ``Down``%go to lower pane
    ``Left``%go to left pane
    ``Right``%go to right pane
    ``C-Up``%resize pane up by 1 lines
    ``M-Up``%resize pane up by 5 lines
    ``C-Down``% resize pane down by 1 line
    ``M-Down``% resize pane down by 5 lines
    ``C-Left``% resize pane left by 1 lines
    ``M-Left``% resize pane left by 5 lines
    ``C-Right``% resize pane right by 1 line
    ``M-Right``% resize pane right by 5 lines

..   [*] if you type the number you go to this pane.

Synchronize panes
~~~~~~~~~~~~~~~~~

You can synchronize panes, i.e. send the keyboard to multiple panes with the command:

::

    :setw synchronize-panes

Toggle it off again by repeating the command.

Execute a shell command in a pane
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can create a new pane with the command:

::

    :split-window [-dhv] [p percentage] <shell-command>

The flag ``-h`` means horizontal split, and ``-v`` stand for vertical split; if you add
``d`` the new pane will not get the focus. ``splitw`` is an alias for ``split-window``.

It is usefull to launch a long running command in forground.  Example:

::

    $ tmux splitw -dh htop
    $ tmux splitw -v -p 90 man tmux

These two command can also be entered at tmux command prompt with:

::

    C-b:splitw -dh htop
    C-b:splitw -v -p 90 man tmux

If you use often such commands, an alias makes it easier:

::

    alias tvspl "tmux splitw -dh'
    alias thspl "tmux splitw -v"
    alias tmman "tmux splitw -v -p 90 man"

Move a window to a pane
~~~~~~~~~~~~~~~~~~~~~~~

When you want to bring an other window as pane in the current window you can use the
command:

::

    :joinp -s :2

Or you can prefer to send your window inside another one as new pane:

::

    :joinp -t :1

The post `join window to pane <https://unix.stackexchange.com/a/14301/266187>`_
propose to add to tmux.conf
::

    # pane movement
    bind-key j command-prompt -p "join pane from:"  "join-pane -s '%%'"
    bind-key s command-prompt -p "send pane to:"  "join-pane -t '%%'"

or

::

     bind-key j choose-window 'join-pane -h -s "%%"'
     bind-key s choose-window 'join-pane -t "%%"'

*The* ``s`` *binding will hide the default binding for* ``choose-tree``.

..  _copy mode:

Copy mode:
----------

``C-b [`` switch to *Copy mode*, then ``q`` comes back to default mode.

In *copy mode*  we can move with the arrow keys, and *Page
Up/Down*

There are two modes for key bindings: *emacs* is the default, you can switch to vi mode
by the command ``setw mode-keys vi``, to make it permanent put in your configuration:

::

    setw -g mode-keys vi

In *vi mode* we use h, j, k, and l to move around our buffer.

Keys
~~~~

The following keys are bound in copy mode *(for an exhaustive list see* :man:`tmux(1)` *)* :

+-----------------+-------------------------+--------------+---------+
|                 |Function                 |vi            |emacs    |
+=================+=========================+==============+=========+
|                 |up                       |k             |Up       |
|                 +-------------------------+--------------+---------+
|Move by          |down                     |j             |Down     |
|characters       +-------------------------+--------------+---------+
|                 |left                     |h             |Left     |
|                 +-------------------------+--------------+---------+
|                 |right                    |l             |Right    |
+-----------------+-------------------------+--------------+---------+
|                 |Start of line            |0             |C-a      |
|                 +-------------------------+--------------+---------+
|Move in the line |End of line              |$             |C-e      |
|                 +-------------------------+--------------+---------+
|                 |Back to indentation      |^             |M-m      |
|                 +-------------------------+--------------+---------+
|                 |Next word                |w             |M-f      |
|                 +-------------------------+--------------+---------+
|                 |Previous word            |b             |M-b      |
+-----------------+-------------------------+--------------+---------+
|                 |Jump forward <char>      | f<char>      |f<char>  |
|                 +-------------------------+--------------+---------+
| Jump in line    |Jump backward <char>     | F<char>      | F<char> |
|                 +-------------------------+--------------+---------+
|                 |Jump next occurrence     | ;            | ;       |
|                 +-------------------------+--------------+---------+
|                 |Jump previous occurence  | ,            | ,       |
+-----------------+-------------------------+--------------+---------+
|                 |Search forward           |/             |C-s      |
|                 +-------------------------+--------------+---------+
| Search          |Search backward          |?             |C-r      |
|                 +-------------------------+--------------+---------+
|                 |Search again             |n             |n        |
|                 +-------------------------+--------------+---------+
|                 |Search again in reverse  |N             |N        |
+-----------------+-------------------------+--------------+---------+
|                 |Goto line                |:             |g        |
|                 +-------------------------+--------------+---------+
| Move to a line  |Bottom line              |L             |         |
|                 +-------------------------+--------------+---------+
|                 |Middle line              |M             |M-r      |
|                 +-------------------------+--------------+---------+
|                 |Top line                 |H             |M-R      |
+-----------------+-------------------------+--------------+---------+
|                 |Half page up             |C-u           |M-Up     |
|                 +-------------------------+--------------+---------+
| Move by pages   |Half page down           |C-d           |M-Down   |
|                 +-------------------------+--------------+---------+
|                 |Next page                |C-f           |Page down|
|                 +-------------------------+--------------+---------+
|                 |Previous page            |C-b           |Page up  |
+-----------------+-------------------------+--------------+---------+
|                 |Scroll up                |C-Up or C-y   |C-Up     |
| Scroll          +-------------------------+--------------+---------+
|                 |Scroll down              |C-Down or C-e |C-Down   |
+-----------------+-------------------------+--------------+---------+
|                 |Start selection          |Space         |C-Space  |
|                 +-------------------------+--------------+---------+
| Selection       |Clear selection          |Escape        |C-g      |
|                 +-------------------------+--------------+---------+
|                 |Copy selection           |Enter         |M-w      |
|                 +-------------------------+--------------+---------+
|                 |Paste buffer             |p             |C-y      |
+-----------------+-------------------------+--------------+---------+
|                 |Delete entire line       |d             |C-u      |
|  Delete         +-------------------------+--------------+---------+
|                 |Delete to end of line    |D             |C-k      |
+-----------------+-------------------------+--------------+---------+
|                 |Quit mode                |q             |Escape   |
| Misc            +-------------------------+--------------+---------+
|                 |Transpose chars          |              |C-t      |
+-----------------+-------------------------+--------------+---------+




Mouse support
-------------

If you set the mouse option, mouse events can be bound to keys. The default is to use
the mouse to select and resize panes, to copy text and to change window using the status
line.

You turn on the mouse with the command *for tmux 2.1 and above*:

::

    setw -g mouse on

Configurations Options:
-----------------------

::

    # Mouse support - set to on if you want to use the mouse
    setw -g mouse on

    # split panes using | and -
    bind | split-window -h
    bind - split-window -v
    unbind '"'
    unbind %

    # reload config file
    # will hide the default refresh-client binding
    bind r source-file /path/to/tmux.conf

    # Set the default terminal mode to 256color mode
    set -g default-terminalq "screen-256color"

    # enable activity alerts
    setw -g monitor-activity on
    set -g visual-activity on

    # Center the window list
    set -g status-justify centre

References
----------
-  tmux manual: :man:`tmux(1)`
-  `GitHub - tmux <https://github.com/tmux/tmux>`_
-  `tmux FAQ <https://github.com/tmux/tmux/wiki/FAQ>`_
-  `ArchWiki: Tmux <https://wiki.archlinux.org/index.php/Tmux>`_ is a
   good introduction with references to complementary articles.
-  `The Tao of Tmux <https://leanpub.com/the-tao-of-tmux/read>`_
   an online book
-  `Awesome tmux <https://github.com/rothgar/awesome-tmux>`_ a list of
   helpful tmux links for various tutorials, plugins, and configuration
   settings.
