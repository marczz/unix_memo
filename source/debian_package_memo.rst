==========================
Debian Package Config Memo
==========================

dpkg Memo
=========

See also :man:`dpkg(1)`, :man:`dpkg-deb(1)`, :man:`dpkg.cfg(5)`,
:man:`dlocate(1)`, :man:`apt-file(1)`


-   Find out all the options::

      dpkg --help

-   Print out the control file (and other information) for a specified
    package::

      dpkg --info foo_VVV-RRR.deb

-  Install a package (including unpacking and configuring) onto the file
   system of the hard disk::

     dpkg --install foo_VVV-RRR.deb

-  status of all the packages installed on a system::

     dpkg --list

-  status of packages matching foo\* installed on a system::

     dpkg --list 'foo*'

-  Detailed status of foo, including dependencies and configuration::

     dpkg --status foo

   Note that configuration files are followed by the md5sum  of the
   original configuration file provided by the package. It allows the
   package manager to know when they are changed. You can also use it
   in the same way.
   ::

       $ dpkg --status mysql-common
       .....
       Conffiles:
       /etc/mysql/conf.d/.keepme d41d8cd98f00b204e9800998ecf8427e
       /etc/mysql/my.cnf 77f15d6c87f9c136c4efcda072017f71
       $ md5sum /etc/mysql/my.cnf # unchanged conf
       77f15d6c87f9c136c4efcda072017f71  /etc/mysql/my.cnf

-  files provided by the installed package foo::

     dpkg --listfiles foo

-  :man:`dlocate` cache the packages content and allow a quicker
   processing:

   -   list status with::

         dlocate -l 'foo*'

   -   list files with::

         dlocate -L foo

   -   files long list with::

         dlocate -ls foo

-   what installed package produced a particular file::

      dpkg --search filename

    or::

      dlocate -S  filename

-   what package (installed or not) produced a particular file::

      apt-file search foo

-   Determine what files are contained in a Debian archive file::

      dpkg-deb --contents foo_VVV-RRR.deb

-   Extract the files contained in a named Debian archive into a
    directory without installing the archive::

      dpkg-deb --extract foo_VVV-RRR.deb tmp

-   Unpack (but do not configure) a Debian archive, This command removes
    any already-installed version of the program and runs the preinst but
    does not necessarily leave the package in a usable state; it has to
    be configured::

      dpkg --unpack foo_VVV-RRR.deb

-   Configure a package that already has been unpacked, this action runs
    the postinst script and updates the files listed in the conffiles for
    this package::

      dpkg --configure foo

-  reconfigure a package::

     dpkg-reconfigure foo

-  Extract all files matching glob pattern "blurf*" from a Debian
   archive::

     dpkg --fsys-tarfile foo_VVV-RRR.deb | tar -xf - blurf*

-   Remove a package (but not its configuration files)::

      dpkg --remove foo

    or::

      aptitude remove foo

-   Remove a package (including its configuration files)::

      dpkg --purge foo

    or::

      aptitude purge foo

-   List the installation status of packages containing the string (or
    regular expression) ``'foo*'``::

      dpkg --list 'foo*'

-   Configuration files policy, without prompt:
    They are listed in the ``--force-things`` section of the
    :man:`dpkg(1) manpage <dpkg>`.

    -   List the *force* options::

          dpkg --force-help

    -   Do not modify the current configuration file touched or not::

          dpkg --install --force-confold foo


    -   Do not modify the current configuration file when touched, but
        apply the default policy when untouched (usually update it!)::

          dpkg --install --force-confold --force-confdef foo

    -   Install the new version of a modified configuration file,
        the current version is kept in a file with the .dpkg-old::

          dpkg --install --force-confnew foo

    -   If a conffile is missing and the version in the  package
        did  change,  always  install  the missing conffile without
        prompting::

          dpkg --install --force-confmiss foo

    -   If a conffile has been modified always offer to replace
        it, even if the version in the package did not change::

          dpkg --install --force-confask foo

apt/aptitude memo
=================

References
----------

-   :man:`apt(8)`,
    :man:`apt-get(8)`,
    :man:`apt.conf(5)`,
    :man:`sources.list(5)`.
-   :man:`apt-cache(8)`,
    :man:`apt-file(1)`
-   :man:`apt-offline(8)`
-   `Debian Reference: Debian package management
    <https://www.debian.org/doc/manuals/debian-reference/ch02.en.html>`_
-   `aptitude User Manual <http://aptitude.alioth.debian.org/doc/en/>`_,
    `command line use <http://aptitude.alioth.debian.org/doc/en/rn01.html>`_ and
    `aptitude Command-Line Reference
    <http://aptitude.alioth.debian.org/doc/en/rn01re01.html>`_.
-   `Aptitude reference guide: search  patterns
    <http://aptitude.alioth.debian.org/doc/en/ch02s04.html>`_.
-   The commands that install, upgrade, and remove packages all accept
    the parameter ``-s``, which stands for “simulate”. When ``-s`` is passed on
    the command line, the program performs all the actions it would
    normally perform, but does not actually download or install/remove
    any files.

Install/Remove
--------------

-   update the list of available packages at the repositories::

      aptitude update

    or::

      apt-get update

-   upgrade each package on the system, after installing versions of
    packages upon which it depends::

      aptitude update
      aptitude safe-upgrade
      aptitude full-upgrade

-   in the *safe* version installed packages are not removed unless they
    are unused.

-   with apt: ``apt-get upgrade`` or ``apt-get dist-upgrade`` use
    the *safe* command.
-   installs package from the unstable distribution while installing its
    dependencies from the current distribution::

      aptitude install package/unstable

-   installs package from the unstable distribution while installing its
    dependencies also from the unstable distribution by setting the
    Pin-Priority of unstable to 990::

      aptitude install -t unstable package

-   checks the status of packages foo bar ::

      aptitude show foo bar ... | less

    or::

      apt-cache show foo bar ... | less

-   installs the particular version 2.2.4-1 of the foo package::

      aptitude install foo=2.2.4-1

-   installs the foo package and removes the bar package::

      aptitude install foo bar-

-   removes the bar package but not its configuration files::

      aptitude remove bar

-   removes the bar package together with all its configuration files::

      aptitude purge bar

-   Use ``--force-things`` when calling dpkg from apt and aptitude::

      apt-get install --reinstall -o Dpkg::Options::="--force-confmiss" foo
      aptitude reinstall -o Dpkg::Options::="--force-confmiss" foo

informations about packages
---------------------------

-   update cache and check for broken packages
    ::

        apt-get   check

-   search package from text description:
    ::

        apt-cache search  pattern

    or ::

        aptitude search foo

-   Search all manually installed packages (~i: installed, !~M not
    automatic)
    ::

        aptitude search '~i!~M'

-   Search all packages with tag hardware::input:keyboard
    ::

        aptitude search ~Ghardware::input:keyboard

-   Search all packages whose description contains the word "switcher"
    ::

        aptitude search ~dswitcher

-   Search all installed packages that contains "firewall" in
    description.
    ::

        aptitude search '~dfirewall~i'

-   Search all package installed from an other archive than debian
    ::

        aptitude search '!~Odebian'~i

-   Search or show all packages of priority *standard* (priority must be
    extra, important, optional, required, or standard. )
    ::

        aptitude search '?priority(standard)'
        aptitude search '~p standard'
        aptitude show '?priority(standard)'
        aptitude show '~p standard'

-   search patterns description is in
    `Debian Reference: The aptitude regex formula
    <http://www.debian.org/doc/manuals/debian-reference/ch02.en.html#_the_aptitude_regex_formula>`_
    and
    `Aptitude reference guide: search  patterns
    <http://aptitude.alioth.debian.org/doc/en/ch02s04.html>`_.


+--------------+----------------+----------------------+-----------------+--------------------+------------+
| key          | val            | key                  | val             | key                | val        |
+==============+================+======================+=================+====================+============+
| ~A<archive\> | archive        | ~G<tag\>             | tag             | ~s                 | section    |
+--------------+----------------+----------------------+-----------------+--------------------+------------+
| ~a<action\>  | action         | ~i                   | installed       | ~T                 | true       |
+--------------+----------------+----------------------+-----------------+--------------------+------------+
| ~B<type\>    | Broken-<type\> | ~M                   | automatic       | ~t<task\>          | task       |
+--------------+----------------+----------------------+-----------------+--------------------+------------+
| ~b           | broken         | ~m<name>             | maintainer      | ~U                 | upgradable |
+--------------+----------------+----------------------+-----------------+--------------------+------------+
| ~C<pattern\> | conflict       | ~N                   | new             | ~V<version\>       | version    |
+--------------+----------------+----------------------+-----------------+--------------------+------------+
| ~c           | config-files   | ~n<name\>            | name            | ~v                 | virtual    |
+--------------+----------------+----------------------+-----------------+--------------------+------------+
| ~D           | dependency     | ~O<origin\>          | origin          | ~w<pattern\>       | widen      |
+--------------+----------------+----------------------+-----------------+--------------------+------------+
| ~d           | description    | ~P<pattern\>         | provides        | !<pattern\>        | not        |
+--------------+----------------+----------------------+-----------------+--------------------+------------+
| ~e           | essential      | ~p<priority\>        | priority        | <patt1\>  <patt2\> | and        |
+--------------+----------------+----------------------+-----------------+--------------------+------------+
| ~F           | false          | ~R<type\>:<patt\>    | reverse-<type\> | <patt1\>\|<patt2\> | or         |
+--------------+----------------+----------------------+-----------------+--------------------+------------+
| ~g           | garbage        | ~S <filter\> <patt\> | narrow          |                    |            |
+--------------+----------------+----------------------+-----------------+--------------------+------------+


<type\> is one of ``depends``, ``predepends``, ``recommends``,
``suggests``, ``breaks``, ``conflicts``, or ``replaces``.

-    package priority/dists information:
     ::

         apt-cache policy  package
         aptitude versions package

-    show description of package:
     ::

         aptitude show package

-    show description of package in archive:
     ::

         aptitude show package/archive

     or

     ::

         aptitude show -t archive package

-    show the installed version of a package:
     ::

         apt-show-versions -p package
         apt-show-versions -r regex

-    show all versions in archives:

     ::

         apt-show-versions -a package

-    show description of all versions of a package:

     ::

         aptitude -v show package

-    show description of package in all dists:

     ::

         apt-cache show -a package

-    show description of matching source package:

     ::

         apt-cache showsrc package

-    package information including what repositories provide available
     versions and forward and reverse dependencies
     ::

         apt-cache showpkg package

-    Print the full package record of a package including all aptitude
     show output and md5, sha1, sha256 sums, and tags:
     ::

         apt-cache show package
         dpkg --print-avail package

-    Transitive dependencies and reverse dependencies of a package:
     ::

         apt-cache depends package
         apt-cache rdepends package

-    You can also use aptitude
     ::

         apt-cache rdepends xdg-utils

     can be replaced by:
     ::

         aptitude search '?dependency(xdg-utils)'

     but the to search all dependencies of the package like
     ``apt-cache depends``:
     ::

         aptitude search
         '?reverse-depends(xdg-utils)\|?reverse-recommends(xdg-utils)\|reverse-suggest(xdg-utils)'

-    Detailed information about the priority selection of the named
     package. It helps to debug your preferences pinning.
     ::

         apt-cache policy <package>

-    Look for a file matching a pattern among the sources.list packages,
     first update the ``apt-file`` cache with:
     ::

         apt-file update

     Then search with:
     ::

         apt-file search <pattern>

     We can switch from the default glob pattern to a regex or a fixed
     string with:
     ::

         apt-file --regexp search <pattern>
         apt-file --fixed-string search <pattern>

-    Look for a file matching a pattern among installed packages *only'*:
     ::

         dpkg --search <pattern>
         dlocate -S <string>

-    Content of all packages (among the sources.list packages) whose name
     match a pattern:
     ::

         apt-file list <pattern>

     for installed packages *only* use:
     ::

         dpkg {-listfiles|-L} <pattern>
         dlocate -l <pattern>

     for deb package files:
     ::

         dpkg -c </path/to/pkg.deb>

-    Dependencies and reverse dependencies of a package:
     ::

         apt-cache depends pkg(s)
         apt-cache rdepends pkg(s)

-    how many packages you have from testing:
     ::

         apt-show-versions | fgrep /testing | wc

-    list of upgradeable packages *including upgrades not in
     preferences*:
     ::

         apt-show-versions -u

-    upgrade all unstable packages to their newest versions
     *(dangerous)*:
     ::

         aptitude install `apt-show-versions -u -b | fgrep /unstable`

importing a key
---------------
Reference: :man:`apt-key(8)`

-    With :man:`apt-key` the command *adv* allow to use :man:`gpg` to
     receive a key, you will use either the default keyserver or give
     one explicitly::

       sudo apt-key adv --recv-keys --keyserver hkp://pgp.mit.edu <missing key>

-   You can also directly provide the key in stdin::

      wget -q http://fr.packages.medibuntu.org/medibuntu-key.gpg -O- | \
      sudo apt-key add -

-   or put it in your keyring::

      gpg --keyserver hkp://subkeys.pgp.net --recv-keys KEY_ID
      gpg -a –export KEY_ID | sudo -H apt-key add -
