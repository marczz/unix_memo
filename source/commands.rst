.. _linux_command_memo:

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

files
-----
.. csv-table::
   :delim: %
   :widths: 50, 60

   :man:`rename` 's/.jpeg$/.jpg/' \*.jpeg%Mass rename
   `find`_ ./ -type f -print | `xargs`_ :coreutils:`chmod` 640%Change permissions to 640 for all files in subtree.
   `find`_ ./ -type d -print | `xargs`_ :coreutils:`chmod` 751%Change permissions to 751 for all sub-directories.

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
   `find`_ -maxdepth 1 -type f -print0 | `xargs`_ -0 ls -lS |min2|\ block-size=1k%List files by decreasing size
   `find`_ -size +1M -ls%List files bigger than 1 Megabyte.
   `find`_ -name '\*.[ch]' | `xargs`_ grep -E 'expr'%Search 'expr' in this dir and below.
   `find`_ -type f -print0 | `xargs`_ -r0 grep -F 'example'%Search all regular files for 'example' in this dir and below
   `find`_ -maxdepth 1 -type f | `xargs`_ grep -F 'example'%Search all regular files for 'example' in this dir
   `find`_ -maxdepth 1 -type d | while read dir; do echo $dir; echo cmd2; done%Process each item with multiple commands (in while loop)
   `find`_. -xtype l%Find broken links
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
   :coreutils:`du` -sh * | :coreutils:`sort` -k1,1rh | :coreutils:`head`%Show larger directories in current dir.
   sudo :coreutils:`du` -hs /home/* | :coreutils:`sort` -k1,1h%Sort paths by increasing use
   :coreutils:`du` -ah |min2|\ max-depth=0 * | :coreutils:`sort` -k1,1rh | :coreutils:`head` -n 15%Show 15 larger directories or files in current dir.
   :coreutils:`df` -h%Show free space on mounted filesystems
   :coreutils:`df` -i%Show free inodes on mounted filesystems
   sudo :man:`sfdisk` -l /dev/sda%Show disks partitions sizes and types (MBR part)
   sudo :man:`sgdisk` -p /dev/sda%Show disks partitions sizes and types (GUID part)
   :coreutils:`dd` bs=1 seek=2TB if=/dev/null of=ext3.test%Create a large sparse test file (taking no space).
   >| file%truncate data of file or create an empty file

text handling
-------------
.. csv-table::
   :delim: §
   :widths: 50, 60

   :coreutils:`tr` -dc '[:print:]' < /dev/urandom§Filter non printable characters
   :coreutils:`tr` -s '[:blank:]' '\t' </proc/diskstats | :coreutils:`cut` -f4§cut fields separated by blanks
   :coreutils:`tr` -s '[:blank:]' </proc/diskstats | :coreutils:`cut` -d' ' -f4§cut fields separated by blanks
   :man:`dmesg`| :coreutils:`wc` -l§count lines (``w`` words, ``-b`` bytes)
   :coreutils:`cut` -d: -f1 /etc/passwd | :coreutils:`sort`§Lists all usernames in alphabetical order.
   :coreutils:`paste` -d ',:' file1 file2 file3§Merges given files line by line
   :man:`mount` | :bsdman:`column` -t§table of mounted filesystems
   :coreutils:`join` -t'\0' -a1 -a2 file1 file2§Union of sorted files
   :coreutils:`join` -t'\0' file1 file2§Intersection of sorted files
   :coreutils:`join` -t'\0' -v2 file1 file2§Difference of sorted files
   :coreutils:`join` -t'\0' -v1 -v2 file1 file2§Symmetric Difference of sorted files
   :bsdman:`column` -s, -t <tmp.csv§pretty print csv
   :coreutils:`printf` "%03o\\n" "\'%"§octal code of ascii character ``%``
   :coreutils:`printf` "Ox%02x\\n" "\'%"§hexacimal code of ascii character ``%``
   :coreutils:`printf` "%d\\n" "\'%"§decimal code of ascii character ``%``
   :man:`iconv` -f ISO8859-1 -t UTF-8 -o file.utf8 file.txt§convert encoding
   :man:`iconv` -l§List known coded character sets

archives and compression
------------------------

.. csv-table::
   :delim: %
   :widths: 50, 60

   `gpg`_ -c file%Encrypt file. More commands in the :ref:`gnupg_memo`.
   `gpg`_ file.gpg%Decrypt file.
   :man:`tar` -cjf dir.tar.bz2 dir/%Make bzip2 compressed archive of dir/
   :man:`tar` -jxf dir.tar.bz2%Extract archive (replace **j**, by **z** for gzip, or `--lzip`)
   :man:`tar` -cxf dir.tgz --exclude '\*.o' --exclude '\*~' dir/
   :man:`tar` -xf dir.tgz --to-stdout  dir/file.txt%Print file to stdout
   :man:`tar` -c dir/ | gzip | `gpg`_ -c | :man:`ssh` user\@remote 'dd of=dir.tar.gz.gpg'%Make encrypted archive of dir/ on remote machine.
   `find`_ dir/ -name '\*.txt' | :man:`tar` -c |min2|\ files-from=- | bzip2 > dir\_txt.tar.bz2%Make archive of subset of dir/ and below.
   `find`_ dir/ -name '\*.txt' | `xargs`_ :coreutils:`cp` -a |min2|\ target-directory=dir\_txt/ |min2|\ parents%Make copy of subset of dir/ and below.
   ( :man:`tar` -c /dir/to/copy ) | ( cd /where/to/ && :man:`tar` -x -p )%Copy (with permissions) copy/ dir to /where/to/ dir
   ( cd /dir/to/copy && :man:`tar` -c **.** ) | ( cd /where/to/ && :man:`tar` -x -p )%Copy (with permissions) contents of copy/ dir to /where/to/
   ( :man:`tar` -c /dir/to/copy ) | :man:`ssh` -C user\@remote 'cd /where/to/ && :man:`tar` -x -p'%Copy (with permissions) copy/ dir to remote:/where/to/ dir
   :man:`zip` -r /path/to/archive.zip dir%zip a directory
   :man:`unzip` archive.zip%extract archive
   :man:`unzip` -l archive.zip%list archive content
   :man:`unzip` archive.zip file.txt%Extract one file from archive
   :coreutils:`dd` if=/dev/vg0/vol0 of=/dev/vg1/vol1 bs=4096%Copy a partition to another one (bs must be a divider of volume blocksize)
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
   :man:`ps` ax -o pcpu,cpu,nice,state,cputime,args |min2|\ sort -pcpu | :man:`sed` '/^ 0.0 /d'%List processes by  decreasing cpu rate (see also :man:`top`).
   :man:`ps` ax -opid=,rss=,args= |min2|\ sort=+rss | :man:`sed` '/^\s*0\>/d' | :coreutils:`pr` -TW$COLUMNS%List processes by mem (KB) usage (see also :man:`top`).
   :man:`ps` -o user |min2|\ sort user| :coreutils:`uniq` -c| :coreutils:`sort` -n -k1%number of processes per user.
   :man:`ps` -C lighttpd -o pid=%pid of *lighttpd*.
   :man:`pgrep` light%pid of processes having *light* in their name.
   :man:`pgrep` -a daemon%pid/command-line of all processes having *daemon* in their name
   `pidof <http://linux.die.net/man/8/pidof>`_  lighttpd%pid of *lighttpd*.
   :man:`ps` uw -C lighttpd%user oriented list of process *lighttpd*.
   :man:`ps` -C firefox-bin -L -o pid,tid,pcpu,state%List all threads for a particular process.
   :man:`ps` -p 666 -o etime=%List elapsed wall time for process id 666
   :man:`ps` ew 666%show command and environment of process 666
   :coreutils:`kill` -9 1234%Send SIGKILL to process 1234
   :man:`killall` -s USR1 dd%Send signal USR1 to the dd program
   :bsdman:`pkill` -s USR1 dd%Send signal USR1 to the dd program
   :man:`pmap` 1234%Memory map of process 1234

monitoring, process admin
-------------------------
.. csv-table::
   :delim: %
   :widths: 50, 60

   :coreutils:`tail` -f /var/log/messages%Monitor messages in a log file.
   :man:`lsof` -p 666%List paths that process id 666 has open.
   :man:`lsof` /path/to/file%List processes that have specified path open.
   :man:`lsof` -u foo%Processes and files of user foo
   :man:`lsof` -u foo%Processes no of user foo
   :man:`lsof` -t -c  pcmanfm%files open by pcmanfm
   :man:`fuser` -va 22/tcp%List processes using port 22
   :man:`fuser` -va /home%List processes accessing the /home
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

*Use the --dry-run option for testing and environment*
`RSYNC_PARTIAL_DIR=.rsync-tmp` *to keep partial files separates.*

.. csv-table::
   :delim: %
   :widths: 50, 60

   :man:`diff` -r  /path/to/dir1/ /path/to/dir2/%diff recursively two directories.
   :man:`diff` -rq /path/to/dir1/ /path/to/dir2/| :man:`sort`%list files that differs between two directories
   :man:`diff` -rq /path/to/dir1/ /path/to/dir2/| :man:`diffstat`%summarize differences  between two directories
   :man:`rsync` -avn source-dir/ target-dir/%what files differs (size mod time) between two directories.
   :man:`rsync` -avnc source-dir/ target-dir/%what files differs (checksum) between two directories.
   `rsync`_ -P rsync://rsync.server.com/path/to/file file%Use partial transfer, repeat for troublesome downloads.
   `rsync`_ |min2|\ bwlimit=1000 fromfile tofile%Locally copy with rate limit. It's like nice for I/O
   `rsync`_ -az  |min2|\ delete ~/public\_html/ remote.com:'~/public\_html'%Mirror web site (using compression and encryption)
   `rsync`_ -auz  remote:/dir/ **.** && `rsync`_ -auz  **.** remote:/dir/%Synchronize current directory with remote one.

.. _ssh_commands:

:bsdman:`ssh`
-------------
More info in the :ref:`ssh section <ssh_section>`.

.. csv-table::
   :delim: %
   :widths: 50, 60

   :bsdman:`ssh` $USER\@$HOST command%Run command on $HOST as $USER (default command=shell)
   :bsdman:`ssh` -f -Y $USER\@$HOSTNAME xterm%Run GUI command on $HOSTNAME as $USER
   :bsdman:`ssh` -c arcfour128 -f -Y $USER\@$LANHOST xterm%Run GUI command on $LANHOST as $USER with :ref:`faster crypto <ssh_ciphers>`.
   :man:`tar` -cf- src | :bsdman:`ssh` -q -c arcfour128 $LANHOST tar -xf- -Cdest% :ref:`quick directory transfer <ssh_file_transfer>`.
   :bsdman:`scp` -p -r -C $USER\@$HOST: file dir/%Copy with permissions to $USER's home directory on $HOST, compress  for slow links.
   :bsdman:`scp` -c arcfour128 $USER\@$LANHOST: bigfile%Use :ref:`faster crypto <ssh_ciphers>` for local LAN, but :ref:`tar over ssh is to be preferred <ssh_file_transfer>`.
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
   `wget`_ |min2|\ limit-rate=20k url%Do a low priority download (limit to 20KB/s)
   `wget`_ -nv |min2|\ spider |min2|\ force-html -i bookmarks.html%Check links in a file
   `wget`_ |min2|\ mirror http://www.example.com/%Efficiently update a local copy of a site (handy from cron)

networking
----------

.. csv-table::
   :delim: %
   :widths: 50, 60

   `ethtool <http://linux.die.net/man/8/ethtool>`_ eth0%Show status of ethernet interface eth0
   `ethtool <http://linux.die.net/man/8/ethtool>`_ |min2|\ change eth0 autoneg off speed 100 duplex full%Manually set ethernet interface speed
   :man:`iwconfig` eth1%Show status of wireless interface eth1
   :man:`iwconfig` eth1 rate 1Mb/s fixed%Manually set wireless interface speed
   :man:`iwlist` scan%List wireless networks in range
   :man:`ip` link show%List network interfaces
   :man:`ip` link set dev eth0 name wan%Rename interface eth0 to wan
   :man:`ip` link set dev eth0 up%Bring interface eth0 up (or down)
   :man:`ip` addr show%List addresses for interfaces
   :man:`ip` addr add 1.2.3.4/24 dev eth0%Add (or del) ip and mask (255.255.255.0)
   :man:`ip` route show%List routing table
   :man:`ip` route add default via 1.2.3.254%Set default gateway to 1.2.3.254
   :man:`ip` route add 192.168.16.0/28 via 192.168.31.254%route subnet
   :man:`ifconfig` -a%List network interfaces
   :man:`ifconfig` eth0 1.2.3.4 up%Bring interface eth0 up (or down)
   :man:`ifconfig` eth0 1.2.3.4 netmask 255.255.255.0%Add first ip and mask
   :man:`ifconfig` eth0:0 1.2.3.5 netmask 255.255.255.0%Add additional ip and mask
   :man:`route` -n%List routing table
   :man:`route` add default gw 1.2.3.254%Set default gateway to 1.2.3.254
   :man:`route` add -net 192.168.16.0 netmask 255.255.240.0 gw 192.168.31.254%route subnet
   :bsdman:`host` github.com%Lookup DNS ip address for name or vice versa
   :man:`hostname` -i%Lookup local ip address (equivalent to host \`hostname\`)
   :man:`whois` mzlinux.org%Lookup whois info for hostname or ip address
   sudo :man:`netstat` -tupl%List internet services on a system
   sudo :man:`netstat` -tup%List active connections to/from system
   sudo :man:`ss` -tup%List tcp and udp active connections to/from system
   :man:`iptraf`%interactive ncurses colorful IP LAN monitor.
   :man:`vnstat`%Console hourly, daily and monthly network traffic.
   :man:`lsof` -i tcp:443%What tcp connection is using `port 443 <http://www.whatportis.com/443>`_.
   :man:`lsof` -i :5800%What is using `port 5800 <http://www.whatportis.com/5800>`_.
   :man:`lsof` -i @192.168.1.5:22%connections to host 192.168.1.5 port 22
   `curl`_ -I htps://github.org%Display the server headers for a web site.
   `curl`_ -s https://ftp-master.debian.org/keys/archive-key-7.0.asc | :man:`gpg` |min2|\ import%Import a gpg key from the web
   `curl`_ ifconfig.me%get your external address through `ifconfig.me <http://ifconfig.me>`_
   sudo :man:`apache2ctl` -S%Display a list of apache virtual hosts


network manager
---------------
.. csv-table::
   :delim: %
   :widths: 55, 55

   :man:`nm-tool`%state of network-manager including Wireless Access Points
   :man:`nmcli` dev status%status of all devices
   :man:`nmcli` -p dev wifi list%list all availables wifi access points
   :man:`nmcli` -p dev list iface *wlan0*%detailled list of device and APs
   :man:`nmcli` con list%list of registered connections
   :man:`nmcli` dev wifi connect *FreeWifi*%setup and activate a new connection
   :man:`nmcli` dev wifi connect *apssid* name *conname* password *private*%new connection with name and password
   :man:`nmcli` con status id *MyWifi*%details of connection
   :man:`nmcli` con up id *MyWifi* password *mypasswd*%connect with password
   `curl`_ -F login=\ *myid*  -F password=\ *mypasswd* *https://wifi.provider.org/Auth*%Connect to open spot

sed
---

See `sed manual <sed>`_ and
`sed1line <http://sed.sourceforge.net/sed1line.txt>`_.

.. csv-table::
   :delim: %
   :widths: 55, 55

   ``sed -n '8,12p'``%Print lines 8 to 12
   ``sed -n '/regexp/p'``%Print lines which match regular expression
   ``sed '/regexp/d'``%Print lines which don't match regular expression
   ``sed -n '/begregexp/,/endregexp/p'``%Print section of file between two regexp
   ``sed '/begregexp/,/endregexp/d'``%Print file except section between two regexp
   ``sed '/^#/d; /^ *$/d'``%Remove comments and blank lines
   ``sed -i 's/[ \t]\*$//' file.txt``%Delete trailing space at end of lines
   ``sed -e :a -e '/^\n*$/N;/\n$/ba'``%Delete blank lines at end of file.
   ``sed -i 42d ~/.ssh/known_hosts``%Delete a particular line
   ``sed ':a; /\\$/N; s/\\\n//; ta'``%Concatenate lines with trailing ``\``
   ``sed = filename | sed 'N;s/\n/\t/'``%Put a left count number on each line of a file
   ``sed = filename | sed 'N; s/^/     /; s/ *\(.\{6,\}\)\n/\1  /'``%Put a right aligned count on each line
   ``sed 's/\x0D$//'``%Dos to unix eol
   ``sed 's/$/\\r/'``%Unix to dos eol

:wikipedia:`ACL` and :wikipedia:`Extended Attributes`
-----------------------------------------------------
*Note: for ext 2/3/4 fs you may need to (re)mount with "acl" or
"user_xattr" options. Or set the filesystem default with tune2fs. On
btrfs acl and xattr are enabled by default.*

.. csv-table::
   :delim: %
   :widths: 50, 60

   :man:`getfacl` .%Show ACLs for file.
   :man:`setfacl` -m u:nobody:r .%Allow a specific user to read file.
   :man:`setfacl` -x u:nobody .%Delete a specific user's rights to file.
   :man:`setfacl` |min2|\ default -m group:users:rw- dir/%Set umask for a for a specific dir.
   :bsdman:`getcap` file%Show capabilities for a program.
   :bsdman:`setcap` cap_net_raw+ep your_gtk_prog%Allow gtk program raw access to network
   `getfattr <http://linux.die.net/man/1/getfattr>`_ -m- -d%Show all extended attributes (includes selinux,acls,...)
   `setfattr <http://linux.die.net/man/1/getfattr>`_ -n "user.foo" -v "bar" .%Set arbitrary user attributes

Desktop management
------------------
.. csv-table::
   :delim: %
   :widths: 50, 60

   :man:`xset` q%display X user preferences.
   :man:`xset -b`%Turn off system beep
   :man:`xset -b`%Turn on system beep
   :man:`xwininfo`%Info of the window selected by mouse click.
   :man:`xwininfo -name emacs`%Emacs window info.
   :man:`xprop`%Xserver properties of the window selected by mouse click.
   :man:`xdpyinfo`%Xserver dimension and resolution.
   :man:`wmctrl` -lG%List managed windows with their geometry.
   :man:`wmctrl` -l -x%List managed windows with their ``WM_CLASS``.
   :man:`wmctrl` -d%List desktops, current desktop has a ``*``
   :man:`wmctrl` -s 3%switch to desktop 3
   :man:`wmctrl` -a emacs%switch to  desktop containing emacs and raise it.
   :man:`wmctrl` -r emacs -t2%send emacs to third desktop
   :man:`wmctrl` -r emacs -e 0,-1,-1,756,495%resize emacs to 756x495 pixels
   :man:`xdotool` search |min2|\ onlyvisible |min2|\ class *emacs* windowsize |min2|\ usehints |percent|\ 1 *80 24*%resize emacs to 80 columns x 24 lines.
   :man:`xwit` -columns 80 -rows 24 -names foo%resize  *foo* window.
   :man:`xwit` -columns 80 -rows 24 -select%select and resize a window.
   :man:`xwit` -rows 34 -columns 80 -property WM_CLASS -names emacs%resize all emacs windows.

Images manipulation
-------------------
You can use `ImageMagick`_ or
`GraphicsMagick <http://www.graphicsmagick.org>`_ with similar syntax except that you prepend the command ``gm`` for GraphicsMagick, and the option for input come before the input file in GraphicsMagick and **after** for `ImageMagick`_.

.. csv-table::
   :delim: %
   :widths: 50, 60


   `identify <http://www.imagemagick.org/script/identify.php>`_ *photo.jpg*%information about an image file
   `convert`_ *photo.png* -resize 2048x1536 -quality 80 *photo.jpg*%resize an image
   `convert`_ *apple.jpg* -crop 128×128+50+50 *apple_crop.jpg*%crop an image
   `convert`_ *lying.jpg* -rotate 90 *standing.jpg*%rotate an image
   `convert`_ \*.jpg ouput.pdf%Create a single PDF from multiple images with `ImageMagick`_
   `import <http://www.imagemagick.org/script/import.php>`_ *snapshot.jpg*%Take a snapshot of a mouse selected desktop area.


Refs
----

-  This page is a fork of *pixelbeat*
   `command line reference <http://www.pixelbeat.org/cmdline.html>`_
   see also the `unix commands page
   <http://www.pixelbeat.org/docs/unix_commands/>`_,
   `More Linux commands <http://www.pixelbeat.org/docs/linux_commands.html>`_,
   the `programming notes <http://www.pixelbeat.org/programming/>`_,
   the `scripts <http://www.pixelbeat.org/scripts/>`_
-  Other system command memos:
   `Unix Toolbox <http://cb.vu/unixtoolbox.xhtml>`_,
   `commandlinefu <http://www.commandlinefu.com/>`_,
   `shell-fu <http://www.shell-fu.org/>`_.

.. |percent| unicode:: 0x25 .. % sign
.. |min2| unicode:: 0x2d 0x2d .. - -
.. _convert: http://www.imagemagick.org/script/convert.php
.. _curl: http://curl.haxx.se/docs/manpage.html
.. _dirstack: http://www.gnu.org/software/bash/manual/html_node/Directory-Stack-Builtins.html
.. _find: http://www.gnu.org/software/findutils/manual/html_node/find_html/index.html
.. _gpg: http://www.gnupg.org/documentation/manuals/gnupg/
.. _grep: http://www.gnu.org/software/grep/manual/html_node/index.html
.. _ImageMagick:  http://www.imagemagick.org
.. _rsync: http://www.samba.org/ftp/rsync/rsync.html
.. _sed: http://www.gnu.org/software/sed/manual/sed.html
.. _tcpdump: http://www.tcpdump.org/tcpdump_man.html
.. _sed: http://www.gnu.org/software/sed/manual/sed.html
.. _wget: http://www.gnu.org/software/wget/manual/wget.html
.. _xargs: http://www.gnu.org/software/findutils/manual/html_node/find_html/xargs-options.html

..
   TODO: Complete with other commands from http://cb.vu/unixtoolbox.xhtml
