FreeDesktop Structure
=====================

These are notes of the structure of desktops following the Freedesktop
specifications.

..  index::
    see: freedesktop; xdg


References
----------

-   `Freedesktop.org <http://www.freedesktop.org/>`__ a base platform
    (both software and standard) for desktop software.
-   `Freedesktop Software <http://freedesktop.org/wiki/Software/>`__
-   `freedesktop specifications
    <http://www.freedesktop.org/wiki/Specifications>`__:

    -   `Menu specifications
        <http://standards.freedesktop.org/menu-spec/latest/>`__
    -   `Mime Actions Specification`_
    -   `Shared Mime Info Specification
        <http://www.freedesktop.org/wiki/Specifications/shared-mime-info-spec>`__
    -   `Base-Directory Specification
        <http://standards.freedesktop.org/basedir-spec/latest/>`__
    -   `Desktop Entry Specification
        <http://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html>`__:
        `the exec key
        <http://standards.freedesktop.org/desktop-entry-spec/latest/ar01s06.html>`__
    -   `Recent File Specification
        <http://www.freedesktop.org/wiki/Specifications/recent-file-spec>`__
    -   `Icon Themes Specification
        <http://standards.freedesktop.org/icon-theme-spec/icon-theme-spec-latest.html>`__
    -   `Desktop Application Autostart Specification
        <http://standards.freedesktop.org/autostart-spec/latest/>`__

-   `python-xdg <http://freedesktop.org/wiki/Software/pyxdg>`__ is a
    python library to access freedesktop.org standards, the
    `python-xdg documentation
    <http://pyxdg.readthedocs.org/en/latest/index.html>`_ is on
    ReadTheDocs.

..  index::
    pair: xdg; menu
    pair: desktop; menu


XDG menus
---------

-   `XDG Menu in LXDE <http://wiki.lxde.org/en/Main_Menu>`__
-   ArchLinux: :archwiki:`Xdg-menu`
-   `xdg-utils <http://portland.freedesktop.org/>`__
    is a set of command line utilities for Free Desktop: creating
    menus, opening files, setting mime types it includes the command
    :man:`xdg-desktop-menu` which is used for (un)installing
    freedesktop menu items i.e.  ``.desktop`` files for freedesktop
    compatible environments.
-   To build the full desktop menu many distribution use a command to
    browse the freedesktop menu hierarchy :man:`xdg-menu`.
    It is decribed in Archlinux :archwiki`Xdg-Menu`.
-   In Debian the freedesktop menus are generated from
    `Debian Menus System
    <http://www.debian.org/doc/packaging-manuals/menu.html/>`__ by
    the command
    `install-menu
    <http://www.debian.org/doc/packaging-manuals/menu.html/ch7.html>`__
    with the configuration ``/etc/menu-methods/menu-xdg`` from the
    package ``menu-xdg``. To use them with default config
    ``$XDG_MENU_PREFIX`` must be set to ``debian-``.
-   Debian is using the alternate debian menu, with the increasing
    spreading of Free desktop menus through ``.desktop`` files, it
    becomes more difficult to maintain a debian menu file for each
    package. There was a
    `Proposal: using Desktop entries with the Debian menu system
    <https://wiki.debian.org/Proposals/DebianMenuUsingDesktopEntries>`__
    which compare Debian menus with ``.xdesktop`` files. An alternative
    is to switch debian packaging to freedesktop menus, it is developped
    in `Debian and application-menu policies
    <http://lwn.net/Articles/597697/>`__.

..  index::
    xdg; default application
    default application
    application; default
    mime; action
    mime; type
    xdg-mime
    xdg-open

XDG Default application.
------------------------

The Freedesktop way of opening applications is through `xdg-open`_
and the Mime type of the file.

Each application provide a ``.desktop`` file that conform to
`Desktop Entry Specification
<http://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html>`__,
amon many key, value pairs this file contain a key ``MimeType`` that
indicates the MIME Types that an application knows how to handle.
An application is expected to be able to reasonably open files of
these types using the command listed in the ``Exec`` key.
It is specified in `Mime Actions Specification`_.


Example::

  $ cat /usr/share/applications/feh.desktop
  [Desktop Entry]
  Name=Feh
  .......
  Exec=feh %F
  Type=Application
  .......
  MimeType=image/jpeg;image/png;image/gif;image/tiff;image/bmp;
           image/x-icon;image/x-xpixmap;image/x-xbitmap;

`ArchLinux wiki <https://wiki.archlinux.org/>` has also many related
documentation : :archwiki:`Default applications`
*describe mime database, xdg-open, xdg-mime, ect.*
:archwiki:`Xdg user directories, :archwiki:`Desktop Entries`.


It list also some :archwiki:`alternatives <Default_Applications#Utilities>`,
that try to provide more flexibility than the official Freedesktop
mechanism.

Managing default applications with xdg-utils.
---------------------------------------------

To know the mime file type of a file we use `xdg-mime`_::

    $ xdg-mime query filetype example.png
    image/png

An URI is associated with a special mime type ``x-scheme-handler/scheme`` where
``scheme`` is the URI scheme like ``http``, ``https``, ``ftp``, ``mms``, ``rtsp`` ....
(see `URI scheme handlers`_ in freedesktop specification)

What application open this file type::

    $ xdg-mime query default image/png
    feh.desktop

Change the default application::

    $ xdg-mime default geeqie.desktop image/png

Open a file with the default application with `xdg-open`_::

    $ xdg-open example.png

The command `xdg-settings`_ allow to change at once all defaults for web scheme handlers
or other url scheme handlers::

  $ xdg-mime query default x-scheme-handler/http
  org.kde.falkon.desktop
  $ xdg-mime query default x-scheme-handler/https
  org.kde.falkon.desktop
  $ xdg-settings get default-web-browser
  org.kde.falkon.desktop
  $ xdg-settings set default-web-browser firefox.desktop
  $ xdg-settings get default-web-browser
  firefox.desktop
  $ xdg-mime query default x-scheme-handler/http
  firefox.desktop
  $ xdg-settings get default-url-scheme-handler http
  firefox.desktop
  $ xdg-settings get default-url-scheme-handler mms
  smplayer.desktop
  $ xdg-mime query default x-scheme-handler/mms
  smplayer.desktop

As seen above `xdg-settings`_ is a convenience tool that replace one or many operations
that can also be done with `xdg-mime`_, but usually we want to use the same browser for
all web url

If you have the :man:`gio` command from the ``libglib2.x-bin`` package
(glib is used in GTK+ and Gnome applications) you can use it to see all packages that
declare the mime type by issuing a query like::

  $ gio mime x-scheme-handler/http

*you can use any mime type*. With gio you don't have to dig manually in the mime
databases, in the file system, that we now describe.



The *mimeapps* file.
--------------------
Each *mimeapps* file is composed of many sections:

   1. *Default Applications*: are made of lines like::

        mimetype=application1.desktop;application2.desktop...

      They give the default to open this mime type, many defaults can be specified in
      the same or different *mimeapps* file they are used in their order in the file or
      in the *mimeapps* browse order. The first application installed is used *some may
      be missing*.

   2. *Added Associations*: The association between an application and the mime types
      elle can open is usually defined in the desktop file, but this section define some
      added associations.

   3. *Removed Associations*: removes associations of applications with mimetypes, this
      mean that this asociation is not used, even if present in a desktop.

   Added associations should be in preference order, if a valid default application is
   not used the higher preferrence association will be used.

   The adding and removal of associations only applies to desktop files in the current
   directory, or a later one, this mean that if a desktop file is defined say in
   ``/etc/xdg`` directory you cannot add or remove an association related to it in
   ``/usr/local/share/applications/mimeapps.list`` that has a lower priority.


You can see all applications are defined for each mime type in
``/usr/share/applications/mimeinfo.cache``, they are not prioritized in this file which
only summarize the ``MimeType=`` field of each desktop file.

The priority for each type is shown in the first file with this type in this list:
``~/.config/$desktop-mimeapps.list``, ``~/.config/mimeapps.list``,
``/etc/xdg/$desktop-mimeapps.list``, ``/etc/xdg/mimeapps.list``,
``/usr/local/share/applications/$desktop-mimeapps.list``,
``/usr/local/share/applications/mimeapps.list``,
``/usr/share/applications/$desktop-mimeapps.list``,
``/usr/share/applications/mimeapps.list``.

Each of these files can be absent, the ``$desktop-mimeapps.list`` are seldom used, in
these entry *$desktop* is stand for ``$XDG_CURRENT_DESKTOP`` environment variable, which
contain the desktop name in lowercase.

Here we have used the defaults for the examined directories, of course you should use
instead the `Freedesktop Directory environment variables`_ instead if they are not let
at their default values.

For a detailed description of the *mimeapps* traversal algorithm look at
`Adding/removing associations`_ in `Mime Actions Specification`_.

The previous utilities changes the entries in ``~/.config/mimeapps.list``.


Follow the `Gnome System Administration Guide`_ instructions, if you want to
`add a custom MIME type for all users`_ or
`add a custom MIME type for individual users`_.


..  index::
    xdg; directory

..  _freedesktop_directories:

Freedesktop Directories
-----------------------

The Base Directories are used when looking for for user configuration.

-   XDG Base Directories are specified in
    `Freedesktop Base-Directory Specification
    <http://standards.freedesktop.org/basedir-spec/latest/>`__.
-   ArchWiki: :archwiki:`XDG Base Directory support`.
    catalog software using the XDG Base Directory Specification.
-   `GNOME Goal: XDG Base Directory Specification Usage
    <https://wiki.gnome.org/Initiatives/GnomeGoals/XDGConfigFolders>`__
    explains why and how Gnome software should implement XDG base
    directories, and list the present support in Gnome programs.
-   ArchWiki: :archwiki:`XDG user directories`.

The `freedesktop base directories
<http://standards.freedesktop.org/basedir-spec/latest/>`__
that follow is used by all freedesktop compatible application. They have
a default that can be overrided by exporting in your environment the
variables.

..  _freedesktop directory environment variables:

-   ``$XDG_DATA_HOME`` default ``$HOME/.local/share`` contains
    user-specific data files.
-   ``$XDG_CONFIG_HOME`` default ``$HOME/.config`` contains user specific
    configuration files.
-   ``$XDG_DATA_DIRS`` default ``/usr/local/share/:/usr/share/`` are
    directories seperated with a colon ':' to search for data in
    addition of ``$XDG_DATA_HOME``
-   ``$XDG_CONFIG_DIRS`` default ``/etc/xdg`` are directories seperated
    with a colon ':' to search for configuration files in addition of
    ``$XDG_CONFIG_HOME``. Configurations are searched in directory order,
    using the first match.
-   ``$XDG_CACHE_HOME`` default ``$HOME/.cache`` for temporary data.
-   ``$XDG_RUNTIME_DIR`` temporary runtime, his life must be the session,
    and it must be owned by the user with access mode 0700 *see full
    requirement in the `specification
    <http://standards.freedesktop.org/basedir-spec/latest/>`__*
    Usually it is set by ``pam_systemd`` at login and there is no need to
    change it. You can get its value from the environment variable
    ``$XDG_RUNTIME_DIR``.

The user directories are the directories under ``$HOME`` used by your
desktop to store your data their default set in
``$XDG_CONFIG_DIRS/user-dirs.defaults`` usually
``/etc/xdg/user-dirs.defaults`` it default to:

::

    DESKTOP=Desktop
    DOWNLOAD=Downloads
    TEMPLATES=Templates
    PUBLICSHARE=Public
    DOCUMENTS=Documents
    MUSIC=Music
    PICTURES=Pictures
    VIDEOS=Videos

These system defaults can be changed in ``user-dirs.defaults``.

The program :man:`xdg-user-dirs-update` is run very early in the login
phase. This program reads a configuration file, and a set of default
directories. It then creates localized versions of these directories
in the users home directory and sets up a config file in
``$(XDG_CONFIG_HOME)/user-dirs.dirs`` *defaults to* ``~/.config`` that
applications read to find these directories.

You can customize the values in your ``~/.config/user-dirs.dirs``; as
an example if you have a non english locale and wish to force these
directories to keep their default english names run:

::

    $ LC_ALL=C xdg-user-dirs-update

That will create the ``~/.config/user-dirs.dirs``. It also creates an
``~/.config/user-dirs.locale`` used to remember the locale used and
allowing to translate names if it changes.

An other popular alternative to avoid to create too many directories
under ``$HOME`` is:

::

    MUSIC=Documents/Music
    PICTURES=Documents/Pictures
    VIDEOS=Documents/Videos

The `Debian Wiki <https://wiki.debian.org/DotFilesList>`__ list the
dotfiles we can find in a Debian system, their role and the programs
that use them. Most of them are not yet following the XDG standard,
many programs may be launched with a specific environment on command
line option to make them comply with xdg satndard as explained in
ArchWiki: :archwiki:`XDG Base Directory support`.
You can also symlink many of these files or directories inside the
corresponding XDG Base directory.

..  index::
    xdg; menu
    !menu

Menu specification.
-------------------

The reference is `Freedesktop Menu Specification
<http://www.freedesktop.org/wiki/Specifications/menu-spec>`__
see also the Gnome: `Desktop Menu Specification
<http://developer.gnome.org/menu-spec/>`__.

-   ``$XDG_CONFIG_DIRS/menus/${XDG_MENU_PREFIX}applications.menu`` is a
    file containning the XML definition of the main application menu
    layout, with the first match strategy you can overide the system
    wide menu with
    ``$XDG_CONFIG_HOME/${XDG_MENU_PREFIX}applications.menu``.
-   ``$XDG_CONFIG_DIRS/menus/applications-merged/`` is the default merge
    directory included in the ``<DefaultMergeDirs>`` element of the
    previous file.
-   ``$XDG_DATA_DIRS/applications/`` contains a ``.desktop`` file for each
    menu item. Desktop entries are collected from all of them, but in
    case of name conflict the first one is used.
-   ``$XDG_DATA_DIRS/desktop-directories/`` contains ``.directory`` files
    giving directory entries in the menu layout.

..  index::
    pair: application; autostart
    xdg; autostart


Autostart applications
----------------------

-   ArchWiki :archwiki:`Autostarting`, :archwiki:`Desktop entries`

Applications referenced by a ``.desktop`` file in
``$XDG_CONFIG_DIRS/autostart`` and ``$XDG_CONFIG_HOME`` may be
autostarted by xdg compliant window managers.

In additions to generic keys, autostart ``.desktop`` files may contain
additional keys:

-   ``Hidden`` when true, the application is ignored
-   ``OnlyShowIn`` and ``NotShowIn`` can list desktop environments in
    which the application is only (not?) started. These two keys are
    exclusives each other.
-   ``TryExec``: Tha application is started only when the named exec
    exist. It can be an absolute path or a name to be looked for in
    ``$PATH``.

..  _xdg-open: https://portland.freedesktop.org/doc/xdg-open.html
..  _xdg-mime: https://portland.freedesktop.org/doc/xdg-mime.html
..  _xdg-desktop-menu:
    https://portland.freedesktop.org/doc/xdg-desktop-menu.html
..  _xdg-settings: https://portland.freedesktop.org/doc/xdg-settings.html
..  _Mime Actions Specification:
    https://specifications.freedesktop.org/mime-apps-spec/latest/
..  _URI scheme handlers:
    https://specifications.freedesktop.org/shared-mime-info-spec/shared-mime-info-spec-latest.html#idm140625828587776
..  _Adding/removing associations:
    https://specifications.freedesktop.org/mime-apps-spec/latest/ar01s03.html
..  _Gnome System Administration Guide:
    https://help.gnome.org/admin/system-admin-guide/stable/
..  _add a custom MIME type for all users:
    https://help.gnome.org/admin/system-admin-guide/stable/mime-types-custom.html.en
..  _add a custom MIME type for individual users:
    https://help.gnome.org/admin/system-admin-guide/stable/mime-types-custom-user.html.en
