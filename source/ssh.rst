.. _ssh_section:

SSH
===
..  highlight:: shell-session


ssh memo
--------
Fo ssh commands examples see  :ref:`ssh commands <ssh_commands>`
in the :ref:`linux_command_memo`.

-   To get the public key from the private one::

      $ openssl rsa -in rsa_key.priv -pubout

authorized-keys
~~~~~~~~~~~~~~~

-   The file ``authorized-keys`` protocol 2 public key consist of:
    options, keytype, base64-encoded key, comment. Where options are
    separated by a comma
-   You can secure ssh when using a key without passphrase by putting
    **options** in your authorized_keys file. Options allow you to
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



Connection sharing
~~~~~~~~~~~~~~~~~~

You can enable connection sharing over a single network connection
by setting ``ControlMaster`` to ``yes``. **ssh** will listen for
connections on a control socket specified using the ``ControlPath``
argument.

These feature are described in the
:bsdman:`ssh_config(5) manual page <ssh_config>` under the
``ControlMaster``, ``ControlPath`` and ``ControlPersist`` options.

You can fix the control path of your connections by putting in
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

If you use ``ControlMaster`` you need to specify
``-o ControlMaster=no`` when using ssh to do ssh tunneling.

Ssh port forwarding
-------------------

-   ssh port forwarding and tunneling is explained in the
    `Tcp forwarding section
    <http://www.openbsd.org/cgi-bin/man.cgi?query=ssh#TCP+FORWARDING>`_
    and `X11 forwarding section
    <http://www.openbsd.org/cgi-bin/man.cgi?query=ssh#X11+FORWARDING>`_
    of the man page, `SSH Port Forwarding
    <http://www.symantec.com/connect/articles/ssh-port-forwarding>`_
    by Brian Hatch see also `Compressed-TCP HOWTO
    <http://en.tldp.org/HOWTO/Compressed-TCP.html>`_ by Sebastian
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

.. _ssh_ciphers:

Cipher Performances
-------------------
The list of supported symmetric **cipher**, supported message integrity
codes (**MAC**), key exchange algorithms (**KEX**), and **key** types
are displayed by using the ``-Q`` option::

  ssh -Q cipher

the result may contain :wikipedia:`aes <aes>`,
:wikipedia:`triple DES <triple DES>` *superseded by aes*,
:wikipedia:`blowfish <blowfish>`, :wikipedia:`cast128 <cast128>`,
:wikipedia:`arcfour <RC4>` also spelled :wikipedia:`RC4 <RC4>`,
:wikipedia:`chacha20 <Salsa20#ChaCha_variant>`, ...


:wikipedia:`Arcfour <RC4>` is now known to be vulnerable  to some complex
attacks, so it should not be used in exposed situations; but the speed
of arcfour let him stand as a good candidate on firewalled local area
networks *when chacha20 is still unavailable*.

.. _cipher_compatibility:

Note that you can only use it if the server allow this cipher
otherwise you will get an answer of::

  $ ssh -c arcfour128 server.example.com
  no matching cipher found: client arcfour128 \
  server aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes128-ctr


We find some tests in the articles
`ssh speed tests
<http://www.damtp.cam.ac.uk/user/ejb48/sshspeedtests.html>`_ that test
ssh between two pentiums
and
`OpenSSH ciphers performance benchmark
<http://blog.famzah.net/2010/06/11/openssh-ciphers-performance-benchmark/>`_
that ssh from a pentium to an arm computer.

As you will see below *aes256* is very fast on Pentium, but may be
quite slow on arm computers, it is why it is more important to choose
your cipher for speed when transferring from or to an arm computer,
when it does not involve security risks.

I did also some ssh speed tests from my pc (pentium 1.70GHz) to three arm
computers with a 1Gb ethernet connection *only 100Mb/s for
raspberry*. The arm computers are: a NAS with armv5l 1.2GHz, a raspberry  ARM11 armv6,
0.7GHz, a banana pi armv7h, cortex-A7 2 cores, 1GHz.


+-----------+----------+----------+---------+---------+---------+----------+
| processor |aes256-ctr|aes128-ctr| 3des    |blowfish |arcfour  |chacha20  |
+===========+==========+==========+=========+=========+=========+==========+
|armv5l     |4.8MB/s   |5.9MB/s   |2.5MB/s  |8MB/s    |12.5MB/s |          |
+-----------+----------+----------+---------+---------+---------+----------+
|armv6      |4.4MB/s   | 4.8MB/s  |1.7MB/s  |5.0MB/s  |5.6MB/s  |          |
+-----------+----------+----------+---------+---------+---------+----------+
|armv7h x 2 |5.9MB/s   |8.3MB/s   |         |         |         |12.5MB/s  |
+-----------+----------+----------+---------+---------+---------+----------+


For *arcfour* we have to
`prefer arcfour128
<http://security.stackexchange.com/questions/26765/what-are-the-differences-between-the-arcfour-arcfour128-and-arcfour256-ciphers>`_,
I repeated the test on raspberry, with the same result,I don't
understand such poor
performance for a cipher whose main quality is the speed, more it
contradict the following test done with openssl.

For extra security when there is no *chacha20* support
on wan we can use :wikipedia:`blowfish
<Blowfish_(cipher)>` for a quick cypher, stronger than
:wikipedia:`RC4`, but the tests above show that the gain is minor on
most architectures.

The transfer time is the result of five  operations , reading,transfer
proper, decoding, writing when ethernet link is fast, and we use a
fast storage *for the test I use tmpfs* the encoding
capabilities of the processors are crucial.

Of course from the five links the weaker is encryption/decryption on
the arm computer, to better isolate this element I tested
encryption/decryption of a 10MB random bytes file on three processors.

I used as command::

  $ time openssl enc -e -aes-256-ctr -out /dev/null -in /tmp/testdata -k mypasswd
  $ time openssl enc -d -aes-256-ctr -out /dev/null -in /tmp/testdata.enc -k mypasswd


+------------+------------+------------+----------+----------+----------+----------+
|cipher      |pentium enc |pentium dec |armv6 enc |armv6 dec |armv7h enc|armv7h dec|
+============+============+============+==========+==========+==========+==========+
|aes-256-ctr |0.5s        |0.6s        |9.8s      |9.9s      | 6.7s     |6.5s      |
+------------+------------+------------+----------+----------+----------+----------+
|des3        |7.6s        |7.4         |46.7s     |46.6      | 22.6s    | 22.4s    |
+------------+------------+------------+----------+----------+----------+----------+
|blowfish    |1.9s        |1.6s        |9.8s      |9.7s      |5.6s      | 5.9s     |
+------------+------------+------------+----------+----------+----------+----------+
|rc4         |0.3s        |0.3s        |3.3s      |3.3s      |2.4s      |2.7s      |
+------------+------------+------------+----------+----------+----------+----------+
|chacha20    |            |            |          |          |          |          |
+------------+------------+------------+----------+----------+----------+----------+

So there is hardly any reason on Pentium to use an other crypto than
*aes-256* that is said very secure. Des3 is 14 times slower than
*aes-256*,  even *blowfish* is slower than
*aes-256* and
the gain of *arcfour* is not worth the loss of security.

Compared to Pentium the encryption
time of *aes-256* is multiplied by 20 on armv6 and 13 on armv7. Even
here the gain of *blowfish* is nul or low, but arcfour is three time
faster than *aes-256*.

I will add the results of *chacha20* when I get on these computers an
openssl newer than 1.02 which is needed for
*chacha20* support.


.. _chacha20_cipher:

The new :wikipedia:`chacha20 <Salsa20#ChaCha_variant>` has also be
conceived `to replace RC4 and be fast on devices that don’t have
AES hardware acceleration
<http://googleonlinesecurity.blogspot.fr/2014/04/speeding-up-and-strengthening-https.html>`_
the previous paper contains also some speed tests.

More details on this new cipher in the
`ietf draft
<https://tools.ietf.org/id/draft-agl-tls-chacha20poly1305-01.html>`_.

When it is available it should replace weaker *RC4* and *blowfish* which
can now be considered as outdated.

.. _ssh_file_transfer:

File transfer on a quick link
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The `ssh speed tests
<http://www.damtp.cam.ac.uk/user/ejb48/sshspeedtests.html>`_
article point out that for file transfer we get a better gain, no by
only choosing a proper cipher, but mainly by using the appropriate
method.

This article compare *scp*, *tar over ssh*, *rsync*, *sshfs* when
transferring compressible or incompressible data. He shows *tar over
ssh* without compression at 100MB/S while scp at 10MB/s and sshfs at
4MB/s.

In this test with a gigabit connection, compression of the tar or scp
decrease the speed; of course it would be no longer true with slow
links, but even then we must care that bzip2 is too slow to be used
for on-the-fly compression.

The main conclusion is that to transfer a big directory on a fast lan the
better is::

  tar -cf- src | ssh -q -c arcfour128 lanhost tar -xf- -Cdest

As set :ref:`above <chacha20_cipher>` we should replace ``arcfour128`` with
``chacha20-poly1305@openssh.com`` whenever it is available.

sshd config
-----------

AllowUsers
~~~~~~~~~~

To restrict to some users and hosts the ssh access, we can use the
directives *Allowusers*, *AllowGroups*, *DenyUsers*, *DenyGroups*.

*Allowusers* can use patterns that takes the form *USER@HOST* to
restrict to some user on specific hosts.

Example::

  AllowUsers john root@119.20.143.62 root@119.20.143.116
          maint@119.20.143.*

Match directive examples
~~~~~~~~~~~~~~~~~~~~~~~~

*Match* deirectives are more powerfull than the *Allowusers*,
*AllowGroups*, *DenyUsers*, *DenyGroups* directive but need more care
to setup properly.

An example of overriding settings on a per-user basis
from the sshd configuration example in the *openssh* package::

    Match User anoncvs
           X11Forwarding no
           AllowTcpForwarding no
           PermitTTY no
           ForceCommand cvs server

and older examples previously posted by Darren Tucker
::

    # allow anyone to authenticate normally from the local net
    Match Address 192.168.0.0/24
            RequiredAuthentications default

    # allow admins from the dmz with pubkey and password
    Match Group admins Address 1.2.3.0/24
            RequiredAuthentications publickey,password

    # deny untrusted and local users from any other net
    Match Group untrusted,lusers
            RequiredAuthentications deny

    # anyone else gets normal behaviour
    Match all
            RequiredAuthentications default

    There's also some potential for other things too:

    Match User anoncvs
            PermitTcpForwarding no

    Match Group nosftp
            Subsystem sftp /bin/false

Testing new configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

If we administer a server where the only access is through ssh we
should be very careful when changing sshd configuration, or we can be
locked out with no way to get in.

I use to test my configuration on the server with::

  $ /usr/sbin/sshd -p 10000 -f /etc/ssh/sshd_config.new -d

which I test on a client with::

  $ ssh -p 10000 -vvv server.example.com


ssh config
----------

Match directive
~~~~~~~~~~~~~~~

The match directive is available also for the client since 6.4.

I use it to detect local subnets like::

    # faster ciphers for lan
    Match exec "local_ip %h"
         Ciphers chacha20-poly1305@openssh.com,arcfour128,blowfish-cbc,aes128-ctr
    Match exec "local_ip --local '^119\.20\.143' %h"
         Ciphers chacha20-poly1305@openssh.com,arcfour128,blowfish-cbc,aes128-ctr

here local ip is a python function that match the ip associated with
an hostname::

    import socket
    import re
    import sys
    private_re = r'^192\.168\.\d\d?\d?\.\d\d?\d?$'
    private_re += '|' + r'10\.\d\d?\d?\.\d\d?\d?\.\d\d?\d?$'
    private_re += '|'  + r'172\.(?:1[0-6]|2\d|3[0-1])\.\d\d?\d?.\d\d?\d?$'

    def check_local(local_re, hostname):
        local = re.compile(local_re)
        hostip = socket.gethostbyname(hostname)
        return local.match(hostip)

    def main():
        import argparse
        parser = argparse.ArgumentParser(description='Match local ips.')
        parser.add_argument('hostname', help='hostname or ip')
        parser.add_argument('--local', dest='local_re', default=private_re)
        args = parser.parse_args()
        raise SystemExit(0 if check_local(args.local_re, args.hostname) else 1)

    if __name__ == '__main__':
        main()

With these settings when I target a local subnet my settings are used,
we can check it with the ``-v`` *verbose* option::

    OpenSSH_6.5, OpenSSL 1.0.1f 6 Jan 2014
    debug1: Reading configuration data /home/marc/.ssh/config
    debug1: Executing command: 'local_ip 119.20.143.62'
    debug1: permanently_drop_suid: 1206
    debug1: Executing command: 'local_ip --local '^119\\.20\\.143' 119.20.143.62'
    debug1: permanently_drop_suid: 1206
    debug1: /home/marc/.ssh/config line 11: matched 'exec "local_ip --local '^119\\.20\\.143' 119.20.143.62"'
    .....
    debug1: SSH2_MSG_KEXINIT sent
    debug1: SSH2_MSG_KEXINIT received
    debug1: kex: server->client arcfour128 hmac-md5 none
    debug1: kex: client->server arcfour128 hmac-md5 none

Note that if you use some special cipher for a client, you should make
sure that your list include one
:ref:`server compatible <cipher_compatibility>` cipher, it is why the
well known `aes128-ctr` is included above, as a server may want to
disable less secure cipher, the defaults of openssh 6.7 do not allow
arcfour or blowfish, it does allow *chacha20* but it is unknown by older
releases and most alternate servers.

If you administer an openssh server you can
tune your ciphers, in accordance with your security and speed needs.

When connecting to a small server like
:wikipedia:`Dropbear <Dropbear_(software)>` the choice of ciphers,
MACs and key exchange algorithms is limited.

Dropbear can only support AES128, AES256, 3DES, TWOFISH256,
TWOFISH128, BLOWFISH *disabled ny default*;
look at `options.h in source tree
<https://github.com/mkj/dropbear/blob/master/options.h>`_ for details.

When dropbear is `built for a small server
<https://github.com/mkj/dropbear/blob/5cf83a7212c0f353e7367766cc4bbf349e83ff0b/SMALL>`_
some of these ciphers may be disabled.

ssh debugging
-------------

-   A usual and easy problem are the permissions on your home
    directory, .ssh directory, and the authorized_keys file.  Your
    home directory should be writable only by you, ``~/.ssh`` should
    be 700, all the keys and ``authorized_keys`` should be 600.  On
    the client this is the easier problem, because your client clearly
    signal this error, it is less obvious for ``authorized_keys`` on
    the server side.
-   On ssh client side you can add a ``-v`` option to your ssh
    command add more ``-v`` for more detailed debug
-   To see authentification problems on the server tail the
    authentication log: ``less +H /var/log/auth.log``, and the
    sshd.service: ``journalctl -f -u ssh.service``.
-   On the server run sshd in debug mode on a distinct port ex:
    ``/usr/sbin/sshd -d -p 2222``


Fish
----

Fish is the acronym for Files transferred over shell protocol, it is a
protocol to use SSH or RSH and Unix utilities like ls, cat or dd to
transfer files. The protocol was designed for Midnight Commander and can
also be used by `lftp <http://lftp.yar.ru/lftp-man.html>`_ and by KDE
:wikipedia:`KIO` kioslave.

The fish protocol reference is
`midnight commander: README.fish
<https://github.com/MidnightCommander/mc/blob/master/src/vfs/fish/helpers/README.fish>`_
it is also explained in `Wikipedia: Files transferred over
shell protocol <http://en.wikipedia.org/wiki/Files_transferrer_over_shell_protocol>`_.

You can use fish when the remote host does not provide a sftp service,
as it is often the case with with dropbear *(because an openssl sftp
is needed to run sftp with dropbear)* and on servers where sftp is not
enabled.
You need only a full ssh access to the remote host as fish requires a
full rsh or ssh shell on the remote side.

SSH References
--------------

-  Introduction:
   Wikipedia: :wikipedia:`Secure Shell`,
   :wikipedia:`OpenSSH`, :wikipedia:`SSh tunnel`.

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
|:bsdman:`ssh-keygen`             |Key generation tool, include use of certificates         |
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
    that protects SSH and other services against brute-force attacts*.
-   ` Matt Taggart: Good practices for using ssh
    <http://lackof.org/taggart/hacking/ssh/>`_ explains basic security
    rule to use ssh **client**.
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
-   Van Emery: `Useful OpenSSL
    Tricks <http://www.vanemery.com/Linux/Apache/openSSL.html>`_, `X over
    SSH <http://www.vanemery.com/Linux/XoverSSH/X-over-SSH2.html>`_
-   The eecs departement of berkeley has some `quick text help
    files <http://inst.eecs.berkeley.edu/usr/pub/>`_ among with
    `ssh.help <http://inst.eecs.berkeley.edu/usr/pub/ssh.help>`_ and
    `ssh-agent.help <http://inst.eecs.berkeley.edu/usr/pub/ssh-agent.help>`_.
-   OpenSSH certificates are not so well known, the reference is the
    `CERTICATES section of ssh-keygen(1)
    <http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man1/ssh-keygen.1?query=ssh-keygen#x434552544946494341544553>`_
    they are distinct and simpler than X.509 certificates used in ssl
    and allow client and servers to authenticate in a simpler and more
    reliable wy than user/host keys.

    There are some tutorials on this subject:
    `DigitalOcean: How To Create an SSH CA to Validate Hosts and
    Clients
    <https://www.digitalocean.com/community/tutorials/how-to-create-an-ssh-ca-to-validate-hosts-and-clients-with-ubuntu>`_,
    `Blargh: OpenSSH certificates tutorial
    <http://blog.habets.pp.se/2011/07/OpenSSH-certificates>`_,
    `Using a CA with SSH <http://www.lorier.net/docs/ssh-ca>`_.
-   While `ssh-agent
    <http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-agent>`_
    is a daemon that cache your decrypted private keys during your
    session `Keychain <http://www.funtoo.org/wiki/Keychain>`_ is a
    front-end to ssh-agent, allowing you to have one long-running
    ssh-agent process per system, rather than one per login session.
    Keychain was `introduced by Daniel Robins in 2001
    <http://www.ibm.com/developerworks/linux/library/l-keyc2/>`_ for
    Gentoo *Keychain has evolved since this article*, It is now available
    in most distributions.

    -   `Gentoo Guide: Keychain
        <http://www.gentoo.org/doc/en/keychain-guide.xml>`_.
    -   `ArchWiki: Keychain
        <https://wiki.archlinux.org/index.php/SSH_keys#Keychain>`_
    -   `man: keychain(1) <http://man.cx/keychain(1)>`_

-   `Envoy <https://github.com/vodik/envoy>`_ (GPL)
    is a ssh/gpg-agent wrapper leveraging cgroups and
    systemd/socket activation with functionalities similar to
    keychain, but done in c, takes advantage of cgroups and systemd.
-   Gnome Keyring is a daemon that keeps user's security credentials,
    such as user names and passwords encrypted in a keyring file in the
    user's home folder. The default keyring uses the login password for
    encryption.

    -   `ArchLinux: Gnome Keyring
        <https://wiki.archlinux.org/index.php/GNOME_Keyring>`_
        describe also how to `use it without gnome
        <https://wiki.archlinux.org/index.php/GNOME_Keyring#Use_without_GNOME.2C_and_without_a_display_manager>`_.
    -   `mozilla-gnome-keyring
        <https://github.com/infinity0/mozilla-gnome-keyring>`_
        is a mozilla extension to replace the default password manager in
        Firefox and Thunderbird and store passwords and form logins
        in gnome-keyring. The Debian package is named
        *xul-ext-gnome-keyring*.

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
    rules. Mosh cannot forward ssh-agent nor X11.

    -  :wikipedia:`mosh`
    -  `Mosh usage <https://mosh.mit.edu/#usage>`_, `info
       <https://mosh.mit.edu/#techinfo>`_
       and `FAQ <https://mosh.mit.edu/#faq>`_.
    -  `GitHub: keithw/mosh source repository
       <https://github.com/keithw/mosh>`_.
    -  `ArchWiki:
       autossh <https://wiki.archlinux.org/index.php/Secure_Shell#Autossh_-_automatically_restarts_SSH_sessions_and_tunnels>`_
    -  Mosh has a chrome plugin and an `android client JuiceSSH
       <https://play.google.com/store/apps/details?id=com.sonelli.juicessh>`.

.. comment

   Local Variables:
   mode: rst
   ispell-local-dictionary: "english"
   End:
