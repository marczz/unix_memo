.. _network_commands_memo:

Network Commands Memo
=====================

:man:`rsync`
------------

*Use the* |min2|\ dry-run *option for testing and environment*
``RSYNC_PARTIAL_DIR=.rsync-tmp`` *to keep partial files separates.*

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
   :bsdman:`ssh` -c 'chacha20-poly1305@openssh.com' -f -Y $USER\@$LANHOST xterm%Run GUI command on $LANHOST as $USER with :ref:`faster crypto <ssh_ciphers>`.
   :man:`tar` -cf- src | :bsdman:`ssh` -q -c arcfour128 $LANHOST tar -xf- -Cdest% :ref:`quick directory transfer <ssh_speed_tests>`.
   :bsdman:`scp` -p -r -C $USER\@$HOST: file dir/%Copy with permissions to $USER's home directory on $HOST, compress  for slow links.
   :bsdman:`scp` -c arcfour128 $USER\@$LANHOST: bigfile%Use :ref:`faster crypto <ssh_ciphers>` for local LAN, but :ref:`tar over ssh is to be preferred <ssh_speed_tests>`.
   :bsdman:`ssh` -g -L 8080:localhost:80 root\@$HOST%Forward connections to $HOSTNAME:8080 out to $HOST:80
   :bsdman:`ssh` -R 1434\:imap\:143 root\@$HOST%Forward connections from $HOST:1434 in to imap\:143
   :bsdman:`ssh` -D 9999 $USER\@$HOST%create a SOCKS proxy on localhost and port 9999
   :man:`ssh-copy-id` $USER\@$HOST%Install public key for $USER\@$HOST for password-less log in

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
   :man:`ip` link set dev eth0 address 00:80:c8:f8:be:ef%Change mac address.
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
   :man:`nmcli` con show%list of registered connections
   :man:`nmcli` dev wifi connect *FreeWifi*%setup and activate a new connection
   :man:`nmcli` dev wifi connect *apssid* name *conname* password *private*%new connection with name and password
   :man:`nmcli` con status id *MyWifi*%details of connection
   :man:`nmcli` con up id *MyWifi* password *mypasswd*%connect with password
   `curl`_ -F login=\ *myid*  -F password=\ *mypasswd* *https://wifi.provider.org/Auth*%Connect to open spot


.. |min2| unicode:: 0x2d 0x2d .. - -
.. _curl: http://curl.haxx.se/docs/manpage.html
