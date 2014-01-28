SSH
===
..  highlight:: shell-session

References
----------

-  :mzlinux:`MzLinux: SSH <63>`,
   :mzlinux:`MzLinux: Security and Encryption section <155>`  and
   :mzlinux:`MzLinux: Strong Passwords <286>`

-  Introduction: :wikipedia:`Secure Shell`,
   :wikipedia:`OpenSSH`, :wikipedia:`SSh tunnel`,
   `Openssh susefaq how-to
   <http://susefaq.sourceforge.net/howto/openssh.html>`_,
   `OpenSSH FAQ <http://www.openssh.com/faq.html>`_
-  The man pages are

+---------------------------------+---------------------------------------------------------+
|:bsdman:`ssh`                    |Basic rlogin/rsh-like client program.                    |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`sshd`                   |Daemon that permits you to login.                        |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`ssh_config`             |Client configuration file.                               |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`sshd_config`            |Daemon configuration file.                               |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`ssh-agent`              |Authentication agent that can store private keys.        |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`ssh-add`                |Tool which adds keys to in the above agent.              |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`sftp`                   |FTP-like program over SSH protocol.                      |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`scp`                    |File copy program.                                       |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`ssh-keygen`             |Key generation tool                                      |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`ssh-keygen#CERTIFICATES`|use of certificates.                                     |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`sftp-server`            |SFTP server subsystem (started automatically by sshd).   |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`ssh-keyscan`            |Utility for gathering public host keys from a number of  |
|                                 |hosts.                                                   |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`ssh-keysign`            |Helper program for host based authentication.            |
+---------------------------------+---------------------------------------------------------+

-   `ArchWiki: ssh <https://wiki.archlinux.org/index.php/Secure_Shell>`_,
    `sshfs <https://wiki.archlinux.org/index.php/Sshfs>`_,
    `SSH\_Keys <https://wiki.archlinux.org/index.php/SSH_Keys>`_,
    `Sshguard <https://wiki.archlinux.org/index.php/Sshguard>`_ *daemon
    that protects SSH and other services against brute-force attacts* .
-   `The 101 Uses of OpenSSH: Part
    II <http://www.linuxjournal.com/article/4413>`_ by Mick Bauer explain
    the public key crypto aspect of ssh.
-   Ibm Developer Work: `OpenSSH key
    management <http://www.ibm.com/developerworks/linux/library/l-keyc.html>`_
    by Daniel Robbins introduces RSA/DSA key authentication, the `second
    article <http://www-106.ibm.com/developerworks/linux/library/l-keyc2/>`_
    shows you how to use ssh-agent, ssh-add and keychain. The `third
    article <http://www-106.ibm.com/developerworks/linux/library/l-keyc3/>`_
    explains ssh-agent authentication forwarding mechanism.
-   Nick Burch has written a two parts tutorial `SSH Tips and Tricks Part
    1 <http://www.torchbox.com/blog/ssh-tips-and-tricks-part-1>`_ and
    `Part 2 <http://www.torchbox.com/blog/ssh-tips-and-tricks-part-2>`_;
    it reminds us about the ``ControlMaster`` and ``ControlPath`` that
    allow the connection sharing on the same socket.
-   `Suno Ano ssh pages <http://sunoano.name/ws/public_xhtml/ssh.html>`_
    Includes detailled recipes for ssh daily use and daemon
    configuration.
-   Van Emery: `Useful OpenSSL
    Tricks <http://www.vanemery.com/Linux/Apache/openSSL.html>`_, `X over
    SSH <http://www.vanemery.com/Linux/XoverSSH/X-over-SSH2.html>`_
-   The eecs departement of berkeley has some `quick text help
    files <http://inst.eecs.berkeley.edu/usr/pub/>`_ among with
    `ssh.help <http://inst.eecs.berkeley.edu/usr/pub/ssh.help>`_ and
    `ssh-agent.help <http://inst.eecs.berkeley.edu/usr/pub/ssh-agent.help>`_.
-   `OpenSSH certificates
    tutorial <http://blog.habets.pp.se/2011/07/OpenSSH-certificates>`_
-   While
    `ssh-agent <http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-agent>`_
    is a daemon that cache your decrypted private keys during your
    session `Keychain <http://www.funtoo.org/wiki/Keychain>`_ is a
    front-end to ssh-agent, allowing you to have one long-running
    ssh-agent process per system, rather than one per login session.
    Keychain was `introduced by Daniel Robins in
    2001 <http://www.ibm.com/developerworks/linux/library/l-keyc2/>`_ for
    Gentoo *Keychain has evolved since this article*, It is now available
    in most distributions.

    -   `Gentoo Guide:
        Keychain <http://www.gentoo.org/doc/en/keychain-guide.xml>`_.
    -   `ArchWiki:
        Keychain <https://wiki.archlinux.org/index.php/SSH_keys#Keychain>`_
    -   `man: keychain(1) <http://man.cx/keychain(1)>`_

-   Gnome Keyring is a daemon that keeps user's security credentials,
    such as user names and passwords encrypted in a keyring file in the
    user's home folder. The default keyring uses the login password for
    encryption.

    -   `ArchLinux: Gnome
        Keyring <https://wiki.archlinux.org/index.php/GNOME_Keyring>`_
        describe also how to use it without gnome.

-   `autossh <http://www.harding.motd.ca/autossh/>`_ (modified BSD) is a
    program to start a copy of ssh and monitor it, restarting it as
    necessary should it die or stop passing traffic. A small included
    script ``rscreen`` or ``rtmux`` allow a *perpetual* ssh session. It
    is in Debian. To use autossh a monitoring port should be choosen
    using the ``-M`` option, but the debian version of autossh uses a
    wrapper to automatically select a free monitoring port. In any case
    you could also disable the monitoring port with ``-M 0`` and have ssh
    do itself the monitoring by setting ``ServerAliveInterval`` and
    ``ServerAliveCountMax`` options to have the SSH client exit if it
    finds itself no longer connected to the server. If not set in the
    [man:ssh\_config] file your command line looks like:

    ::

        $ autossh -M 0 -o "ServerAliveInterval 45" -o "ServerAliveCountMax 2" username@myserver

    To use sshfs with autossh you can use:

    ::

         $ sshfs -o reconnect,compression=yes,transform_symlinks,\
             ServerAliveInterval=45,ServerAliveCountMax=2,\
             ssh_command='autossh -M 0' username@server:/\
             /mnt/remote

-   `mosh <http://mosh.mit.edu/>`_ (GPL with OpenSSL exceptions) is a
    replacement for SSH that allows roaming, supports intermittent
    connectivity, and provides intelligent local echo and line editing of
    user keystrokes. Mosh improve ssh usability for mobile users. It is
    in Debian. Mosh does not use the ssh tcp protocol, but runs a
    terminal emulator at the server and transmits this screen to the
    client through udp. This udp protocol may conflict with firewall
    rules. Mosh cannot forward ssh-agent nor X11, and does not support
    IPv6.

    -  :wikipedia:`mosh`
    -  `ArchWiki:
       autossh <https://wiki.archlinux.org/index.php/Secure_Shell#Autossh_-_automatically_restarts_SSH_sessions_and_tunnels>`_

ssh memo
--------

-   You can fix the control path of your connections by putting in
    ``~/.ssh/config``

    ..  code:: cfg

        Host *
        ControlPath ~/.ssh/sshsocket-%r@%h:%p

    then you can set first a master connection by adding the option
    ``-M`` to your ssh command. The following connections will use the
    same control socket. and will not ask for any authentication If you
    don't want to use ``-M`` you can put in your ssh config

    .. code:: cfg

        Host *
        ControlMaster auto

    you can also use ``ask`` to be asked if you want to reuse an existing
    connection and ``autoask`` to combine both options
-   If you use ``ControlMaster`` you need to specify
    ``-o ControlMaster=no`` when using ssh to do ssh tunneling.
-   in the file ``authorized-keys`` protocol 2 public key consist of:
    options, keytype, base64-encoded key, comment. Where options are
    separated by a comma
-   You can secure ssh when using a key without passphrase by putting
    **options** in your authorized\_keys file. Options allow you to
    restrict to some clients, limit port forwarding, or force the use of
    a predefined command. The options are listed in the `SSHRC section of
    sshd man
    page <http://www.openbsd.org/cgi-bin/man.cgi?query=sshd#SSHRC>`_ that
    also gives some examples like

    ..  code:: cfg

        # Comments allowed at start of line
        ssh-rsa AAAAB3Nza...LiPk== user@example.net
        from="*.sales.example.net,!pc.sales.example.net" ssh-rsa AAAAB2...19Q== john@example.net
        command="dump /home",no-pty,no-port-forwarding ssh-dss   AAAAC3...51R== example.net
        permitopen="192.0.2.1:80",permitopen="192.0.2.2:25" ssh-dss  AAAAB5...21S==
        tunnel="0",command="sh /etc/netstart tun0" ssh-rsa AAAA...==  jane@example.net


-   To get the public key from the private one::

      $ openssl rsa -in rsa_key.priv -pubout

Ssh port forwarding
-------------------

-   ssh port forwarding and tunneling is explained in the `Tcp forwarding
    section <http://www.openbsd.org/cgi-bin/man.cgi?query=ssh#TCP+FORWARDING>`_
    and `X11 forwarding
    section <http://www.openbsd.org/cgi-bin/man.cgi?query=ssh#X11+FORWARDING>`_
    of the man page, `SSH Port
    Forwarding <http://www.symantec.com/connect/articles/ssh-port-forwarding>`_
    by Brian Hatch see also `Compressed-TCP
    HOWTO <http://en.tldp.org/HOWTO/Compressed-TCP.html>`_ by Sebastian
    Schreiber.
-   The general syntax for port forwarding is: -L port:host:hostport --
    redirect a local port to a remote host:hostport -R port:host:hostport
    -- redirect a remote port to a local host:hostport

-   An example from *tychoish* is a tunnel to a remote smtp server

    ::

        $ autossh -M 25 -f remoteuser@remote.mach.in -L 25:127.0.0.1:25

    Here the ``-M 25`` tel autossh to watch the port 35 to check the
    connection is alive.

-   You can also use ssh as socks proxy you just launch

    ::

        $ ssh -D 4321 user@example.com

    and you get a socks proxy on port 4321 forwarding all traffic to
    example.com, you can browse the web as if you originate from
    example.com either to access a hidden lan or go thru a firewall. Of
    course you need a socks proxy enabled browser like firefox. You can
    use this socks with any socks-able client, but there are not many of
    them. So you can use a proxy relay a list of them is on the
    `Wikipedia SOCKS page <http://en.wikipedia.org/wiki/SOCKS>`_

-   Beginning with version 4.3, ssh has an option to do tunneling a tun
    device see:

    -   `tun-based VPN
        section <http://en.wikipedia.org/wiki/OpenSSH#tun-based_VPN>`_ of
        the `Openssh wikipedia
        page <http://en.wikipedia.org/wiki/OpenSSH>`_
    -   The manual of ssh, sshd, ssh-config (references above)
    -   `HOWTO VPN over SSH and
        tun <http://gentoo-wiki.com/HOWTO_VPN_over_SSH_and_tun>`_
    -   `Tunnels ethernet avec
        openssh <http://lea-linux.org/cached/index/Tunnels_ethernet_avec_openssh.html>`_

-   If you change user over ssh via su or sudo, you will no more find
    your X credentials. You can take as ``XAUTHORITY`` environment your
    original ``~/.Xauthority``, but it works only if the new user has
    access to this file. As it it not even true for root if your home is
    on a nfs file system, a better solution is to forward your
    credentials to the new user. A complete wrapper by François Gouget,
    `sux <http://fgouget.free.fr/sux/>`_ is available on many
    distribution. But when we don't have it at hand we can simply do:

    ::

        $ sudo -u <user> $SHELL -c "xauth add $(xauth list :${DISPLAY##*:}); <xprogram>"

Cipher Performances
-------------------
We find some tests in
`ssh speed tests
<http://www.damtp.cam.ac.uk/user/ejb48/sshspeedtests.html>`_
and
`OpenSSH ciphers performance benchmark
<http://blog.famzah.net/2010/06/11/openssh-ciphers-performance-benchmark/>`_.

I did some speed tests from my pc to my NAS 1GB connection.
I found that 3des is 2.5MB/s, the many aes are around 5MB/s,
blowfish and cast128 8MB/s, the many arcfour 12.5MB.

For arcfour we have to
`prefer arcfour128
<http://security.stackexchange.com/questions/26765/what-are-the-differences-between-the-arcfour-arcfour128-and-arcfour256-ciphers>`_




.. comment

   Local Variables:
   mode: rst
   ispell-local-dictionary: "english"
   End:
