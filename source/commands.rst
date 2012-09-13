Linux Commands Memo
===================

This is a linux command line reference for common operations.


basic
-----
.. csv-table::
   :delim: %
   :widths: 50, 60

   :man:`apropos` whatis%Show commands pertinent to string.
   :man:`man` -t ascii | ps2pdf - > ascii.pdf%make a pdf of a manual page
   :man:`which` command%Show full path name of command
   :man:`time` command%See how long a command takes
   :man:`time` cat%Start stopwatch. Ctrl-d to stop.


dir navigation
--------------

.. csv-table::
   :delim: %
   :widths: 50, 60

   :man:`cd` -%Go to previous directory
   :man:`cd`%Go to $HOME directory
   (:man:`cd` dir && command)%Go to dir, execute command and return to current dir
   `pushd <dirstack>`_ *dir*%Put *dir* on stack so you can **popd** back to it
   `pushd <dirstack>`_ +3%Rotate the dir stack, putting third entry at top

File searching
--------------
.. csv-table::
   :delim: %
   :widths: 50, 60

   :coreutils:`ls` -lt%List files by date, newest first
   :coreutils:`ls` /usr/bin | :coreutils:`pr` -T9 -W$COLUMNS%Print in 9 columns to width of terminal
   `find`_ -name '\*.[ch]' | xargs grep -E 'expr'%Search 'expr' in this dir and below.
   `find`_ -type f -print0 | xargs -r0 grep -F 'example'%Search all regular files for 'example' in this dir and below
   `find`_ -maxdepth 1 -type f | xargs grep -F 'example'%Search all regular files for 'example' in this dir
   `find`_ -maxdepth 1 -type d | while `read <http://www.pixelbeat.org/programming/readline/>`_ dir; do echo $dir; echo cmd2; done%Process each item with multiple commands (in while loop)
   `find`_ -type f ! -perm -444%Find files not readable by all (useful for web site)
   `find`_ -type d ! -perm -111%Find dirs not accessible by all (useful for web site)
   :bsdman:`locate` -r '*file*.txt'%Search cached path index for names.
   :bsdman:`locate` -r 'file[^/]*\\.txt'%Search cached path index for names. 'file' must be in last component.


disk space
----------

.. csv-table::
   :delim: %
   :widths: 50, 60

   :coreutils:`ls` -lkS%Show files by size in kb, biggest first.
   :coreutils:`ls` -lt%sort by modification time, newest first
   :coreutils:`du` -s * | :coreutils:`sort` -k1,1rn | :coreutils:`head`%Show top disk users in current dir.
   sudo :coreutils:`du` -hs /home/* | :coreutils:`sort` -k1,1h%Sort paths by increasing use
   :coreutils:`df` -h%Show free space on mounted filesystems
   :coreutils:`df` -i%Show free inodes on mounted filesystems
   sudo :man:`sfdisk` -l /dev/sda%Show disks partitions sizes and types (MBR part)
   sudo :man:`sgdisk` -p /dev/sda%Show disks partitions sizes and types (GUID part)
   :coreutils:`dd` bs=1 seek=2TB if=/dev/null of=ext3.test%Create a large sparse test file (taking no space).
   >| file%truncate data of file or create an empty file

archives and compression
------------------------

.. csv-table::
   :delim: %
   :widths: 50, 60

   `gpg`_ -c file%Encrypt file.
   `gpg`_ file.gpg%Decrypt file.
   :man:`tar` -cj dir/  > dir.tar.bz2%Make bzip2 compressed archive of dir/
   :man:`tar` -jxf dir.tar.bz2%Extract archive (replace **j**, by **z** for gzip, or `--lzip`)
   :man:`tar` -c dir/ | gzip | `gpg`_ -c | :man:`ssh` user\@remote 'dd of=dir.tar.gz.gpg'%Make encrypted archive of dir/ on remote machine.
   `find`_ dir/ -name '\*.txt' | :man:`tar` -c --files-from=- | bzip2 > dir\_txt.tar.bz2%Make archive of subset of dir/ and below.
   `find`_ dir/ -name '\*.txt' | xargs :coreutils:`cp` -a --target-directory=dir\_txt/ --parents%Make copy of subset of dir/ and below.
   ( :man:`tar` -c /dir/to/copy ) | ( cd /where/to/ && :man:`tar` -x -p )%Copy (with permissions) copy/ dir to /where/to/ dir
   ( cd /dir/to/copy && :man:`tar` -c **.** ) | ( cd /where/to/ && :man:`tar` -x -p )%Copy (with permissions) contents of copy/ dir to /where/to/
   ( :man:`tar` -c /dir/to/copy ) | :man:`ssh` -C user\@remote 'cd /where/to/ && :man:`tar` -x -p'%Copy (with permissions) copy/ dir to remote:/where/to/ dir
   :coreutils:`dd` bs=1M if=/dev/sda | gzip | :man:`ssh` user\@remote 'dd of=sda.gz'%Backup harddisk to remote machine.
   :man:`killall` -s USR1 dd%Ask dd to print the state of the current transfer.

process management
------------------
.. csv-table::
   :delim: %
   :widths: 50, 60

   :man:`ps` axww%list all processes
   :man:`ps` axuww%list all processes and resource used
   :man:`ps` axmu%list all processes and threads
   :man:`ps` axf -o pid,args%List processes in a hierarchy.
   :man:`ps` ax -o pcpu,cpu,nice,state,cputime,args --sort pcpu | `sed`_ '/^ 0.0 /d'%List processes by  cpu rate (see also top).
   :man:`ps` ax -orss=,args= | :coreutils:`:coreutils:`sort`` -b -k1,1n | :coreutils:`pr` -TW$COLUMNS%List processes by mem (KB) usage (see also top).
   :man:`ps` -C lighttpd -o pid=%pid of lighttpd
   :man:`pgrep` lighttpd%pid of lighttpd
   `pidof <http://linux.die.net/man/8/pidof>`_  lighttpd%pid of lighttpd
   :man:`ps` -C firefox-bin -L -o pid,tid,pcpu,state%List all threads for a particular process.
   :man:`ps` -p 666 -o etime=%List elapsed wall time for process id 666
   :coreutils:`kill` -9 1234%Send SIGKILL to process 1234
   :man:`killall` -s USR1 dd%Send signal USR1 to the dd program
   :bsdman:`pkill` -s USR1 dd%Send signal USR1 to the dd program


monitoring, process admin
-------------------------
.. csv-table::
   :delim: %
   :widths: 50, 60

   :coreutils:`tail` -f /var/log/messages%Monitor messages in a log file.
   :man:`lsof` -p 666%List paths that process id 666 has open.
   :man:`lsof` /path/to/file%List processes that have specified path open.
   sudo `tcpdump`_ not port 22%Show network traffic except ssh.
   | sudo `tcpdump`_ -ni eth0 'dst 192.168.1.5 and tcp and port http'%all HTTP session to 192.168.1.5.
   :man:`last` reboot%Show system reboot history.
   :man:`free` -m%Show amount of (remaining) RAM (-m displays in MB)
   :man:`watch` -n.1 'cat /proc/interrupts'%Watch changeable data continuously.
   :man:`watch` -t -n1 :man:`uptime`%Clock with system load.
   :man:`nice` *command*%Low priority *command*.
   sudo :man:`renice` 19 -p 666%Set process 666 to low scheduling priority (0<pr<20)
   sudo :man:`renice` +2 -p 666%Lower the scheduling priority.
   :man:`chrt` -i 0 *command*%Low priority command (more effective than nice)
   sudo :man:`ionice` -p 666%io class and priority of process 666. Higher priority 0
   sudo :man:`ionice`  -c3 -p 666%Sets process 666 as an idle io process.
   :man:`htop` -d 5%Better top (scrollable, tree view, lsof/strace integration, ...)
   :man:`iotop`%What's doing I/O.
   sudo :man:`iftop`%What's using the network.
   :bsdman:`vmstat` 3%monitor processes, memory, paging, block IO, traps, and cpu activity.(columns are explained in the :bsdman:`manual <vmstat>`.)
   :bsdman:`vmstat` -m%usage of kernel dynamic memory.

Users
-----
.. csv-table::
   :delim: %
   :widths: 50, 60

   :man:`id` -a%Show the active user id with login and groups.
   :man:`last`%Show last logins on the system.
   :bsdman:`w`%users logged on, and their processes.
   :man:`groupadd` admin%Add group "admin"
   :man:`useradd` -c "Linus Torvald" -g admin -m linus%Add new user
   :man:`usermod` -a -G sudo linus%add group "sudo" to linus groups.
   :man:`userdel` linus%Delete user linus

system information
------------------
.. csv-table::
   :delim: %
   :widths: 50, 60

   :coreutils:`uname` -a%Show kernel version and system architecture.
   :coreutils:`cat` /etc/debian_version%Get Debian version
   :man:`lsb_release` -a%Full release info of any LSB distribution
   :coreutils:`cat` /etc/issue%Show name and version of distribution.
   :coreutils:`cat` /proc/partition%Show all partitions registered on the system.
   `grep`_ MemTotal /proc/meminfo%Show RAM total (see also *free*, *vmstat*)
   :coreutils:`cat` /proc/cpuinfo%Show CPU(s) info
   :man:`lsdev`%hardware info from the /proc directory
   :man:`lspci` -tv%Show PCI info
   :man:`lsusb` -tv%Show USB info
   :man:`mount` | :coreutils:`column` -t%List mounted fs on the system (and align output)
   `grep`_ -F capacity: /proc/acpi/battery/BAT0/info%Show state of cells in laptop battery
   :man:`dmidecode` -q | less%Display SMBIOS/DMI information
   :man:`dumpe2fs` -h /dev/part1 | `grep`_ -e '\\([mM]ount\\)\\|\\([Cc]heck\\)'%info about fs check
   sudo :man:`e2fsck` -f -v -t -C 0 /dev/part1%Check health of partition
   sudo :man:`sdparm` -C stop /dev/sdb%Stop scsi (also usb) disk
   sudo :man:`hdparm` -i /dev/sda%Show info about disk sda
   :man:`dmesg`%Detected hardware and boot messages

`rsync`_
--------

*Use the --dry-run option for testing and environment* `RSYNC_PARTIAL_DIR=.rsync-tmp` *to keep partial files separates.*

.. csv-table::
   :delim: %
   :widths: 50, 60

   `rsync`_ -P rsync://rsync.server.com/path/to/file file%Use partial transfer, repeat for troublesome downloads.
   `rsync`_ --bwlimit=1000 fromfile tofile%Locally copy with rate limit. It's like nice for I/O
   `rsync`_ -az  --delete ~/public\_html/ remote.com:'~/public\_html'%Mirror web site (using compression and encryption)
   `rsync`_ -auz  remote:/dir/ **.** && `rsync`_ -auz  **.** remote:/dir/%Synchronize current directory with remote one.

:bsdman:`ssh`
-------------
.. csv-table::
   :delim: %
   :widths: 50, 60

   :bsdman:`ssh` $USER\@$HOST command%Run command on $HOST as $USER (default command=shell)
   :bsdman:`ssh` -f -Y $USER\@$HOSTNAME xterm%Run GUI command on $HOSTNAME as $USER
   :bsdman:`scp` -p -r -C $USER\@$HOST: file dir/%Copy with permissions to $USER's home directory on $HOST, compress  for slow links.
   :bsdman:`scp` -c arcfour256 $USER\@$LANHOST: bigfile%Use `faster crypto <http://blog.famzah.net/2010/06/11/openssh-ciphers-performance-benchmark/>`_ for local LAN. Use `blowfish <http://en.wikipedia.org/wiki/Blowfish_(cipher)>`_ for a quick cypher stronger than `RC4  <http://en.wikipedia.org/wiki/RC4>`_.
   :bsdman:`ssh` -g -L 8080:localhost:80 root\@$HOST%Forward connections to $HOSTNAME:8080 out to $HOST:80
   :bsdman:`ssh` -R 1434\:imap\:143 root\@$HOST%Forward connections from $HOST:1434 in to imap\:143
   :bsdman:`ssh-copy-id` $USER\@$HOST%Install public key for $USER\@$HOST for password-less log in

`wget`_
-------
.. csv-table::
   :delim: %
   :widths: 50, 60

   (cd dir/ && `wget`_ -nd -pHEKk http://rest-sphinx-memo.readthedocs.org/)%Store local browsable version of a page to the current dir
   `wget`_ -c http://www.example.com/large.file%Continue downloading a partially downloaded file
   `wget`_ -r -nd -np -l1 -A '\*.jpg' http://www.example.com/dir/%Download a set of files to the current directory
   `wget`_ ftp://remote/file[1-9].iso/%FTP supports globbing directly
   `wget`_ -q -O-  http://www.example.com/page%cat to /dev/stdout
   echo '`wget`_ url' | at 01:00%Download url at 1AM to current dir
   `wget`_ --limit-rate=20k url%Do a low priority download (limit to 20KB/s)
   `wget`_ -nv --spider --force-html -i bookmarks.html%Check links in a file
   `wget`_ --mirror http://www.example.com/%Efficiently update a local copy of a site (handy from cron)

networking
----------

.. csv-table::
   :delim: %
   :widths: 50, 60

   `ethtool <http://linux.die.net/man/8/ethtool>`_ eth0%Show status of ethernet interface eth0
   `ethtool <http://linux.die.net/man/8/ethtool>`_ --change eth0 autoneg off speed 100 duplex full%Manually set ethernet interface speed
   :man:`iwconfig` eth1%Show status of wireless interface eth1
   :man:`iwconfig` eth1 rate 1Mb/s fixed%Manually set wireless interface speed
   :man:`iwlist` scan%List wireless networks in range
   :man:`ip` link show%List network interfaces
   :man:`ip` link set dev eth0 name wan%Rename interface eth0 to wan
   :man:`ip` link set dev eth0 up%Bring interface eth0 up (or down)
   :man:`ip` addr show%List addresses for interfaces
   :man:`ip` addr add 1.2.3.4/24 brd + dev eth0%Add (or del) ip and mask (255.255.255.0)
   :man:`ip` route show%List routing table
   :man:`ip` route add default via 1.2.3.254%Set default gateway to 1.2.3.254
   :bsdman:`host` github.com%Lookup DNS ip address for name or vice versa
   :man:`hostname` -i%Lookup local ip address (equivalent to host \`hostname\`)
   :man:`whois` mzlinux.org%Lookup whois info for hostname or ip address
   sudo :man:`netstat` -tupl%List internet services on a system
   sudo :man:`netstat` -tup%List active connections to/from system
   sudo `ss <http://linux.die.net/man/8/ss>`_ -tup%List active connections to/from system
   :man:`iptraf`%interactive ncurses colorful IP LAN monitor.
   :man:`vnstat`%Console hourly, daily and monthly network traffic.
   `curl <http://curl.haxx.se/docs/manpage.html>`_ -I htps://github.org%Display the server headers for a web site.
   :man:`lsof` -i tcp:80%What's using port 80.
   sudo :man:`apache2ctl` -S%Display a list of apache virtual hosts
   `curl <http://curl.haxx.se/docs/manpage.html>`_ -s http://www.pixelbeat.org/pixelbeat.asc|`gpg`_ --import%Import a gpg key from the web

sed
---

Seed `sed manual <sed>`_ and `sed1line <http://sed.sourceforge.net/sed1line.txt>`_.

.. csv-table::
   :delim: %
   :widths: 50, 60

   sed -n '8,12p'%print lines 8 to 12
   sed -n '/regexp/p'% print lines which match regular expression
   sed '/regexp/d'% print lines which don't match regular expression
   sed -n '/begregexp/,/endregexp/p'%print section of file between two regexp
   sed '/begregexp/,/endregexp/d'%print file except section between two regexp
   sed '/^#/d; /^ *$/d'%Remove comments and blank lines
   sed -i 's/[ \\t]*$//' file.txt%delete trailing space at end of lines
   sed -e :a -e '/^\\n*$/{$d;N;ba' -e '}'%Delete blank lines at end of file.
   sed -i 42d ~/.ssh/known_hosts%Delete a particular line
   sed ':a; /\\\\$/N; s/\\\\\\n//; ta'%Concatenate lines with trailing \\
   sed = filename | sed 'N;s/\\n/\\t/'%put a left count number on each line of a file
   sed = filename | sed 'N; s/^/     /; s/ *\\(.\\{6,\\}\\)\\n/\\1  /'%put a right aligned count on each line
   sed 's/\\x0D$//'%Dos to unix eol
   sed 's/$/\\r/'%Unix to dos eol

ACL and Extended Attributes
---------------------------
*Note you may need to (re)mount with "acl" or "user_xattr" options. Or set the filesystem default with tune2fs*

.. csv-table::
   :delim: %
   :widths: 50, 60

   :man:`getfacl` .%Show ACLs for file.
   :man:`setfacl` -m u:nobody:r .%Allow a specific user to read file.
   :man:`setfacl` -x u:nobody .%Delete a specific user's rights to file.
   :man:`setfacl` --default -m group:users:rw- dir/%Set umask for a for a specific dir.
   :bsdman:`getcap` file%Show capabilities for a program.
   :bsdman:`setcap` cap_net_raw+ep your_gtk_prog%Allow gtk program raw access to network
   `getfattr <http://linux.die.net/man/1/getfattr>`_ -m- -d%Show all extended attributes (includes selinux,acls,...)
   `setfattr <http://linux.die.net/man/1/getfattr>`_ -n "user.foo" -v "bar" .%Set arbitrary user attributes

Refs
----

-  This page is a fork of *pixelbeat*
   `<http://www.pixelbeat.org/cmdline.html>`_
   see also the `unix commands page
   <http://www.pixelbeat.org/docs/unix_commands/>`_,
   `More Linux commands <http://www.pixelbeat.org/docs/linux_commands.html>`_,
   the `programming notes <http://www.pixelbeat.org/programming/>`_,
   the `scripts <http://www.pixelbeat.org/scripts/>`_

.. _wget: http://www.gnu.org/software/wget/manual/wget.html
.. _gpg: http://www.gnupg.org/documentation/manuals/gnupg/
.. _rsync: http://www.samba.org/ftp/rsync/rsync.html
.. _tcpdump: http://www.tcpdump.org/tcpdump_man.html
.. _sed: http://www.gnu.org/software/sed/manual/sed.html
.. _find: http://www.gnu.org/software/findutils/manual/html_node/find_html/index.html
.. _dirstack: http://www.gnu.org/software/bash/manual/html_node/Directory-Stack-Builtins.html
.. _grep: http://www.gnu.org/software/grep/manual/html_node/index.html
.. _sed:http://www.gnu.org/software/sed/manual/sed.html

..
   TODO: Complete with other commands from http://cb.vu/unixtoolbox.xhtml
