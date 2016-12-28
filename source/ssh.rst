.. _ssh_section:

SSH
===
..  highlight:: shell-session


ssh memo.
---------
Fo ssh commands examples see  :ref:`ssh commands <ssh_commands>`
in the :ref:`linux_command_memo`.

ssh escapes.
~~~~~~~~~~~~
From :bsdman:`escape characters in the ssh manual
<ssh#ESCAPE_CHARACTERS>`


+---------+-------------------------------------+
| ``~.``  | Disconnect.                         |
+---------+-------------------------------------+
| ``~^Z`` | Background ssh.                     |
+---------+-------------------------------------+
| ``~#``  | List forwarded connections.         |
+---------+-------------------------------------+
| ``~&``  | Background ssh at logout. [#]_      |
+---------+-------------------------------------+
| ``~?``  | list escape characters.             |
+---------+-------------------------------------+
| ``~B``  | Send a BREAK to the remote system   |
+---------+-------------------------------------+
| ``~C``  | Open command line. [#]_.            |
+---------+-------------------------------------+
| ``~R``  | Request rekeying of the connection. |
+---------+-------------------------------------+
| ``~V``  | Decrease the log verbosity.         |
+---------+-------------------------------------+
| ``~v``  | Increase the log verbosity.         |
+---------+-------------------------------------+

.. [#] when waiting for forwarded connection / X11 sessions to terminate.
.. [#] currently this allows the addition or cancelation of port forwardings using the
       ``-L``, ``-R`` and ``-D``; or ``-KL`` ``-KR``, ``-KD``; ``-h`` for help.

       ``!command`` execute a local command if the ``PermitLocalCommand``
       option is enabled in :bsdman:`ssh_config`.

ssh keys.
---------

Key encryption
~~~~~~~~~~~~~~

SSH protocol 2 supports
:wikipedia:`DSA
<https://en.wikipedia.org/wiki/Digital_Signature_Algorithm>`,
:wikipedia:`RSA
<https://en.wikipedia.org/wiki/RSA_(cryptosystem)>`
:wikipedia:`ECDSA
<https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm>`,
:wikipedia:`Ed25519 <https://en.wikipedia.org/wiki/EdDSA>`
keys, the two last being instance of
:wikipedia:`Curve algorithm <https://en.wikipedia.org/wiki/Curve25519>`;
protocol 1 only supports RSA keys.

DSA has vulnerabilities and is deprecated in openssh 7.0,
there are `concerns about the security of ECDSA
<https://git.libssh.org/projects/libssh.git/tree/doc/curve25519-sha256@libssh.org.txt#n4>`_
and it is supposed that NSA could have put backdoors in this
algorithm, as Ed25519 is also technically superior we can always
prefer it.

The more portable key is RSA, Ed25519 will give you the best security
and performance but requires recent versions of client & server,
Ed25519 and ECDSA are not supported by gnome keyring as of March 2016.

`SSH implementation comparison: hostkey
<http://ssh-comparison.quendi.de/comparison/hostkey.html>`
give the support of key algorithm for most of ssh software.
ssh-RSA is required to be supported by ssh RFC, so is always present
:wikipedia:`ECDSA
<https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm>`
is widely present; but
:wikipedia:`SSH-Ed25519 <https://en.wikipedia.org/wiki/EdDSA>`
is only supported by OpenSSH, and few other softare like
the windows clients :wikipedia:`PuTTY` and
`smartFTP <https://www.smartftp.com/>`_, the iOS and Android client
`TinyTerm <http://www.censoft.com/products/mobile/>`_, and the linux
tiny client `TinySSH <https://tinyssh.org/index.html>`_.

You  can also find a list of `Things that use Ed25519
<https://ianix.com/pub/ed25519-deployment.html>` including a list of
ssh software

Even if Ed25519 is both secure and fast, most often for ssh what
matter is the ref:`cipher performance` not the authentication speed.

Generating a key pair
~~~~~~~~~~~~~~~~~~~~~

To generate a RSA key with default keysize of 2048::

  $ ssh-keygen

The ``-b`` option allow to choose an other key size but as state the
`Gnupg FAQ <https://www.gnupg.org/faq/gnupg-faq.html#no_default_of_rsa4096>`_
*Once you move past RSA-2048, you’re really not gaining very much*
and you loose the portability.

If you use Ed25519 all keys are 256 bits.

You can consult a list of `Summary of keylength recommendations of
well-known security organizations <https://www.keylength.com/>`_

If you wantto explore thie keylength topic you have first to
understand why `symmetric cryptography have smaller key than
asymmetric cryptography
<https://blog.cloudflare.com/why-are-some-keys-small/>`_.
You can also look in the `Référentiel Général de Sécurité
version 2.0 <http://www.ssi.gouv.fr/uploads/2015/01/RGS_v-2-0_B1.pdf>`_.


If you really want a stronger key you can use Ed25519 with::

  $ ssh-keygen -t ed25519

But it is a good choice only to communicate with recent OpenSSH
servers, older version and some other ssh servers don't support it,
there is a list of `Things that use Ed25519
<https://ianix.com/pub/ed25519-deployment.html>` including a list of
ssh software, note that as far as april 2016 the windows popular
client PutTTY support Ed25519 in its snapshot version.

.. _new key format:

The ed2519 are stored in a new format that implement a
:wikipedia:`Key derivation function` using many bcrypt rounds to
make more difficult rainbow table attacks. This new format is
the default for ed2519 and can be requested for other keys by adding
the option ``-o``::

  $ ssh-keygen -o -f ~/.ssh/myspecialid_rsa

See :ref:`below <bcrypt_private_key>` for details on this new format.

To know what keys are supported by your ssh software issue::

  $ ssh -Q cipher

It is not advisable to have a key without password since any one that
get access to your private key can will be able to assume your
identity on any SSH server. Nevertherless if I never use as main key a
key without password, it can be acceptable to have a secondary key
that allow unattended connections if you make sure that only the
appropriate daemon can use it, by using :ref:`a proper authorized-keys
entry like shown below <authorized-keys>`.

Modyfying a key
~~~~~~~~~~~~~~~

To change the passphrase of an existing key::

  $ ssh-keygen -f ~/.ssh/id_rsa -p

To get the public key from the private one::

  $ ssh-keygen -f ~/.ssh/id_rsa -y


Key formats
~~~~~~~~~~~

To convert a public key to PEM format::

  $ ssh-keygen -e -m PEM -f ~/.ssh/id_rsa.pub >id_rsa_PEM.pub

It works also with the private key as input, but the output is only
the public key::

    $ ssh-keygen -e -m PEM -f ~/.ssh/id_rsa >id_rsa_PEM.pub

You can also give to ``-m`` the format ``RFC4716`` to have a SSH2
public key or ``PKCS8`` to have an openssl compatible
:wikipedia:`PKCS8 <PKCS>` key.

Refs: :bsdman:`ssh-keygen`, :bsdman:`openssl`

.. _bcrypt_private_key:

You can convert your old key to `new key format`_ by::

  $ ssh-keygen -o -p -a 64 -f id_rsa

The ``-a`` give the number of bcrypt rounds, and default to 16, the
bigger they are the longer is the password verification time, and the
stronger the protection to brute-force password cracking. As example
adding to the agent with ``ssh-add`` a private RSA 256 bytes on my
laptop gives a time of 0.004s (too small to be truly significative)
but with a default of 16 rounds encryption 0.292s i.e 73 time longer,
a 100 rounds encryption 1.616s 404 times longer, a 1000 rounds
encryption it is 16.172 seconds 4176 longer, it means that a rainbow
table attack will try one table entry for the encrypted format in the
same time than 4000 entries with the unencrypted format.

Of course a slower decrypting could be annoying if you wait for each
ssh-connection, but if you use the agent, and still more if you have
:ref:`keychain<keychain_prog>` or :ref:`envoy<envoy_prog>`.
You have to wait only once.


To recognize the formats of your key you can look at the head comment
of the key block.

For an RSA password less key ::

  -----BEGIN RSA PRIVATE KEY-----
  (base64 blurb)

For a RSA encrypted ssh old format  ::

  -----BEGIN RSA PRIVATE KEY-----
  Proc-Type: 4,ENCRYPTED
  DEK-Info: AES-128-CBC,227...
  (base64 blurb)

For the new format ::

  -----BEGIN OPENSSH PRIVATE KEY-----
  (base64 blurb)

.. _authorized-keys:

authorized-keys.
~~~~~~~~~~~~~~~~

-   The file ``authorized-keys`` protocol 2 public key consist of:
    options, keytype, base64-encoded key, comment. Where options are
    separated by a comma
-   You can secure ssh when using a key without passphrase by putting
    **options** in your authorized_keys file. Options allow you to
    restrict to some clients, limit port forwarding, or force the use of
    a predefined command. The options are listed in the
    :bsdman:`SSHRC section of sshd man page <sshd#SSHRC>` that
    also gives some examples like

    ..  code-block:: cfg

        # Comments allowed at start of line
        ssh-rsa AAAAB3Nza...LiPk== user@example.net
        from="*.sales.example.net,!pc.sales.example.net" ssh-rsa AAAAB2...19Q== john@example.net
        command="dump /home",no-pty,no-port-forwarding ssh-dss   AAAAC3...51R== example.net
        permitopen="192.0.2.1:80",permitopen="192.0.2.2:25" ssh-dss  AAAAB5...21S==
        tunnel="0",command="sh /etc/netstart tun0" ssh-rsa AAAA...==  jane@example.net


copying the key to a remote server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
You can use :bsdman:`ssh-copy-id` to copy the file to the remote server::

  $ ssh-copy-id -i ~/.ssh/mykeyid_rsa.pub username@remote-server.org

If you omit the id it will add all your keys to the remote server,
either the keys returned bi ``ssh-add -L``, if nothing is in your
agent it will use the most recent file that matches: ``~/.ssh/id*.pub``.
When using the ssh-agent key, :bsdman:`ssh-copy-id` will loose your
comment. When you have multiple keys the comment is very usefull to
remember the key role, so it is better to always givr the key file
with the ``-i`` option.

It is allowed but not recommended to specify the port or other options
with ssh-copylike this::

  $ ssh-copy-id -i ~/.ssh/mykeyid_rsa.pub -p 27654 -o 'X11Forward=Yes' username@remote-server.org

But is is always better to put these option in  :bsdman:`ssh_config`.

We can also manually copy the key, if we can ssh to the server by::

  $ cat ~/.ssh/mykeyid_rsa.pub | ssh username@remote-server.org \
  'sh -c "cat >> ~/.ssh/authorized_key; chmod 0600  ~/.ssh/authorized_key"

which is similar to the previous ``ssh-copy``.

If you have not yet an ssh access to the server, you can copy the key
by any mean like ftp, webdav, shared cloud ... to the server, if the
transport media is not protected it is more secure to encrypt it
during the transport with gpg or symetric encryption; the on the
server::

  $ mkdir ~/.ssh
  $ chmod 700 ~/.ssh
  $ cat /path/of/mykeyid_rsa.pub >> ~/.ssh/authorized_keys
  $ rm /path/of/mykeyid_rsa.pub
  $ chmod 600 ~/.ssh/authorized_keys


Gnome Keyring
~~~~~~~~~~~~~

Gnome Keyring is a daemon that keeps user's security credentials,
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


ssh agent.
----------
An SSH agent is a program which caches your decrypted private keys and
provides them to SSH client programs on your behalf.

Launching ssh-agent.
~~~~~~~~~~~~~~~~~~~~

On Debian the ``ssh-agent`` is launched in the ancestors of your X session
by ``/etc/X11/Xsession`` so it should run in your X session.

``ssh-agent`` export two environments variables ``SSH_AUTH_SOCK`` the
socket path, and ``SSH_AGENT_PID`` the pid of the process, so you
can check a running instance with:
::

  $ [ $SSH_AUTH_SOCK ] && echo "socket $SSH_AUTH_SOCK" && ps u $SSH_AGENT_PID

If it is not running you can launch it by::

  $ eval $(ssh-agent)

In Debian default you have no ssh-agent session when in a console
session, or connected from a remote site.

You can launch it from your profile, if it is not yet present.

You may use `a more elaborate script
<http://mah.everybody.org/docs/ssh>`_ to ensure you are launching an
unique agent session for your user on the computer.

In the way used by default by Debian, if it is not yet done you can launch
it as a parent process of a daemon with::

  $ ssh-agent startx

or adding to your .xinitrc::

  eval $(ssh-agent)

It is also possible to `start it as a systemd user service
<https://wiki.archlinux.org/index.php/SSH_keys#Start_ssh-agent_with_systemd_user>`_
and you will have a global ssh-agent for your global user session,
whatever it run X or not.


``ssh-agent`` can be replaced by ``gpg-agent`` that can act as an
agent both for gpg keys and ssh keys if it is run with the argument
``--enable-ssh-support`` you can then launch it like set `in the manual
<https://www.gnupg.org/documentation/manuals/gnupg/Agent-Examples.html#Agent-Examples>`_
::

    unset SSH_AGENT_PID
    if [ "${gnupg_SSH_AUTH_SOCK_by:-0}" -ne $$ ]; then
      export SSH_AUTH_SOCK="/run/user/$UID/gnupg/S.gpg-agent.ssh"
    fi

in the same way used for ssh you can prefer to
`start gpg-agent with systemd user
<https://wiki.archlinux.org/index.php/GnuPG#Start_gpg-agent_with_systemd_user>`_

 Refs: :bsdman:`ssh-agent`, `gpg-agent
 <https://www.gnupg.org/documentation/manuals/gnupg/Invoking-GPG_002dAGENT.html>`_

Using ssh-agent.
~~~~~~~~~~~~~~~~

You can list the cached keys::

  $ ssh-add -l
  2048 SHA256:4135dff81d9eff01f2319078995c06ab05feccc0S28 /home/user/.ssh/id_rsa (RSA)

Add a key with::

  $ ssh-add /path/of/key

Remove all keys from cache by::

  $ ssh-add -D

Refs: :bsdman:`ssh-add`

ssh agent forwarding.
~~~~~~~~~~~~~~~~~~~~~

To get agent forwarding we must have the option ``ForwardAgent``
set, it is not recommended to set it globally because
users with the ability to bypass file permissions on the remote host
socket ``$SSH_AUTH_SOCK`` can access the local agent
through the forwarded connection.

You can either do it when required by::

  $ ssh -oForwardAgent=true user@example.com

or use the short option ``-A``::

  $ ssh -A user@example.com

or if you want to always forward agent to a specific server you trust,
you can put in ``~/.ssh/config``::

  Host example.com
    ForwardAgent yes

in any case you can check your have forwarder your agent by looking at
the value of ``$SSH_AUTH_SOCK`` which should be defined::

  $ ssh -oForwardAgent=true user@example.com
  Linux server 3.2.62-1  ...
  ....
  $ echo "$SSH_AUTH_SOCK"
  /tmp/ssh-4TjiNKqsGf/agent.3737

Refs: :bsdman:`ssh`

Forwarding to a sudo session.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are logged in a machine A with a ssh-agent running and holding
your key, and you ssh to a machine B with agent forwarding in your B
session you can still use your key to log in to a server C.

Now suppose you do a sudo  you loose the agent because SSH_AUTH_SOCK
is not exported, so you can no longer ssh to C even if your
user key is authorized.

You can preserve your agent by using::

  $ sudo -i SSH_AUTH_SOCK=$SSH_AUTH_SOCK

or if you want to use su::

  $sudo SSH_AUTH_SOCK=$SSH_AUTH_SOCK su -p -l

Note than when using ``su`` the option ``-p`` preserve the environment
that as yet be reset by ``sudo`` except  SSH_AUTH_SOCK=$SSH_AUTH_SOCK.

If you want to do it for all your sudo sessions you could add to your
``/etc/sudoers``::

  Defaults    env_keep+=SSH_AUTH_SOCK

This method may not work for an other user than root because it does
not have the rights to read ``$SSH_AUTH_SOCK``, you have to add it
either by adding it to your group and ensuring thet the group has
read-write access, or using acl like::

  $ setfacl -m otheruser:x   $(dirname "$SSH_AUTH_SOCK")
  $ setfacl -m otheruser:rwx "$SSH_AUTH_SOCK"
  $ sudo su - otheruser

Refs: :man:`su`, :man:`sudo`, man:`setfacl`

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

..  code-block:: aconf

    Host *
    ControlPath ~/.ssh/sshsocket-%r@%h:%p

then you can set first a master connection by adding the option
``-M`` to your ssh command. The following connections will use the
same control socket. and will not ask for any authentication If you
don't want to use ``-M`` you can put in your ssh config

.. code-block:: aconf

    Host *
    ControlMaster auto

you can also use ``ask`` to be asked if you want to reuse an existing
connection and ``autoask`` to combine both options

If you use ``ControlMaster`` you need to specify
``-o ControlMaster=no`` when using ssh to do ssh tunneling.

  $ ssh -Y example.com

when your goal is to open an X11 application on the server you can
use::

  $ ssh -X -f example.com xprog

ssh will open the remote session, letting you enter your credentials,
then background before command execution.before command execution.

You may want to allow automatic X11 forwarding to trusted servers,
you can do it by putting in your ``~/.ssh/config``::

  Host example.com
    ForwardX11 yes
    ForwardX11Trusted yes

Note that to be able to forward connection you the server should have
in its  :bsdman:`sshd_config` ``X11Forwarding yes`` and the
default is ``no``, and ``AllowTcpForwarding``, ``X11UseLocalhost`` set to
``yes`` which is the default. In some case you may want to change also
``X11DisplayOffset``. A basic Xorg configuration including ``xauth``
should also be present on the remote server, but it does not imply
that the remote server has a display.

Refs: :bsdman:`ssh manual - X11 forwarding section
<ssh#X11_FORWARDING>`, :bsdman:`sshd_config(5)<sshd_config>`,
:bsdman:`ssh_config(5)<ssh_config>`.

.. _keychain_prog:

Keychain
~~~~~~~~

While :bsdman:`ssh-agent`
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

.. _envoy_prog:

Envoy
~~~~~

`Envoy <https://github.com/vodik/envoy>`_ (GPL)
is a ssh/gpg-agent wrapper leveraging cgroups and
systemd/socket activation with functionalities similar to
keychain, but done in c. It takes advantage of cgroups and systemd.

-   `ArchWiki: Envoy
    <https://wiki.archlinux.org/index.php/SSH_keys#envoy>`_

But now envoy can be considered as obsolete as its author Simon
Gomizelj find simpler and better to just wrap gpg-agent in a service,
and replace ssh-agent by gpg-agent.

It has now set up a small new project `gpg-exec
<https://github.com/vodik/gpg-tools>`_ to support this policy.


Ssh port forwarding
-------------------

-   ssh port forwarding and tunneling is explained in the
    :bsdman:`Tcp forwarding section
    <ssh#TCP_FORWARDING>`
    and :bsdman:`X11 forwarding section
    <ssh#X11_FORWARDING>`
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

Keeping session alive
---------------------
You can work either on the server side or the client side.

For the client you can set the configuration option
``ServerAliveInterval`` which is an intervall after wich a ssh
keepalive message is sent to the server, the default is 0.

It work in combination with ``ServerAliveCountMax`` which is the max
number of such message sent, the default value is 3.
If you have set ``ServerAliveInterval`` to 30 you send at most 3
messages every 30s.

If the option ``BatchMode`` is ``yes`` then  ``ServerAliveInterval``
will be set to 300 seconds.

On the Server side by default ``ClientAliveInterval`` is 0 which means
that the server does not send keep alive message to the client.

If you set ``ClientAliveInterval 300`` and ``ClientAliveCountMax 12``
(default is 3) you send to the inactive client a keep alive message
each 5mn during 2 hours.

autossh
~~~~~~~
`autossh <http://www.harding.motd.ca/autossh/>`_ (modified BSD) is a
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
         ssh_command='autossh -M 0' username@example.com:/\
     /mnt/remote


mosh
~~~~
`mosh <http://mosh.mit.edu/>`_ (GPL with OpenSSL exceptions) is a
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


.. _ssh_ciphers:

Cipher Performances
-------------------
The list of supported symmetric **cipher**, supported message integrity
codes (**MAC**), key exchange algorithms (**KEX**), and **key** types
are displayed by using the ``-Q`` option::

  $ ssh -Q cipher

the result may contain :wikipedia:`aes <aes>`,
:wikipedia:`triple DES <triple DES>` *superseded by aes*,
:wikipedia:`blowfish <blowfish>`, :wikipedia:`cast128 <cast128>`,
:wikipedia:`arcfour <RC4>` also spelled :wikipedia:`RC4 <RC4>`,
:wikipedia:`chacha20 <Salsa20#ChaCha_variant>`, ...


:wikipedia:`Arcfour <RC4>` is now known to be vulnerable  to some complex
attacks, so it should not be used in exposed situations; but the speed
of arcfour let him stand as a good candidate on firewalled local area
networks *when chacha20 is still unavailable*.

Note that :wikipedia:`chacha20 <Salsa20#ChaCha_variant>` is a fast
and secure algorithm, see the :ref:`speed tests<ssh_speed_tests>` below.

.. _cipher_compatibility:

Note that you can only use it if the server allow this cipher
otherwise you will get an answer like::

  $ ssh -c arcfour128 server.example.com
  no matching cipher found: client arcfour128 \
  server aes25.

`SSH Implementation Comparison: Ciphers
<http://ssh-comparison.quendi.de/comparison/cipher.html>`_ shows what
cipher is supported by each ssh software, :wikipedia:`Arcfour <RC4>`
is still suported by many server and clients, while
:wikipedia:`chacha20 <Salsa20#ChaCha_variant>`
is only available in OpenSSH,  :wikipedia:`PuTTY` and
`TinySSH`_.


.. _ssh_speed_tests:

For *chacha20-poly1305*
there are a `CloudFare page showing the improvement on https
<https://blog.cloudflare.com/do-the-chacha-better-mobile-performance-with-cryptography/>`_
when opting for  *chacha20-poly1305* encryption.

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

  tar -cf- src | ssh -q -c chacha20-poly1305@openssh.com lanhost tar -xf- -Cdest

As set :ref:`above <ssh_ciphers>` we should replace
``chacha20-poly1305@openssh.com`` with ``arcfour128`` whenever it is
unavailable.

sshd config
-----------

AllowUsers
~~~~~~~~~~

To restrict to some users and hosts the ssh access, we can use the
directives *Allowusers*, *AllowGroups*, *DenyUsers*, *DenyGroups*.

*Allowusers* can use patterns that takes the form *USER@HOST* to
restrict to some user on specific hosts.

Example:
..  code-block:: aconf

    AllowUsers john root@119.20.143.62 root@119.20.143.116
          maint@119.20.143.*

Match directive examples
~~~~~~~~~~~~~~~~~~~~~~~~

*Match* deirectives are more powerfull than the *Allowusers*,
*AllowGroups*, *DenyUsers*, *DenyGroups* directive but need more care
to setup properly.

An example of overriding settings on a per-user basis
from the sshd configuration example in the *openssh* package:

..  code-block:: aconf

    Match User anoncvs
           X11Forwarding no
           AllowTcpForwarding no
           PermitTTY no
           ForceCommand cvs server

and older examples previously posted by Darren Tucker:

..  code-block:: squid

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

I use it to detect local subnets like:

..  code-block:: aconf

    # faster ciphers for lan
    Match exec "local_ip %h"
         Ciphers chacha20-poly1305@openssh.com,arcfour128,blowfish-cbc,aes128-ctr
    Match exec "local_ip --local '^119\.20\.143' %h"
         Ciphers chacha20-poly1305@openssh.com,arcfour128,blowfish-cbc,aes128-ctr

here local ip is a python function that match the ip associated with
an hostname:

..  code-block:: python

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
we can check it with the ``-v`` *verbose* option:

..  code-block:: console

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
|:man:`gpg-agent`                 |Authentication agent for both gpg and ssh.               |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`ssh-add`                |Tool which adds keys to in the above agent.              |
+---------------------------------+---------------------------------------------------------+
|:bsdman:`ssh-copy-id`            |copy your pub key to a remote server                     |
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
    `SSH Keys <https://wiki.archlinux.org/index.php/SSH_keys>`_,
    `Sshguard <https://wiki.archlinux.org/index.php/Sshguard>`_ *daemon
    that protects SSH and other services against brute-force attacts*.
-   `Red Hat Entreprise System Administrator's Guide - Chapter 9
    OpenSSH
    <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7-Beta/html/System_Administrators_Guide/ch-OpenSSH.html>`_
-   `Matt Taggart: Good practices for using ssh
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
    :bsdman:`CERTICATES section of ssh-keygen(1)
    `<ssh-keygen#x434552544946494341544553>`.
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



..  comment

    TODO: include if needed developped themes or references to
    [[https://wiki.archlinux.org/index.php/SSH_keys#Choosing_the_type_of_encryption][SSH keys - ArchWiki]] ,
    [[https://wiki.archlinux.org/index.php/Secure_Shell#X11_forwarding][Secure Shell - ArchWiki]],
    [[https://developer.github.com/guides/using-ssh-agent-forwarding/][Using SSH Agent Forwarding | GitHub Developer Guide]],
    [[https://wiki.archlinux.org/index.php/GNOME/Keyring][GNOME/Keyring - ArchWiki]],
    [[http://tartarus.org/~simon/putty-snapshots/htmldoc/][PuTTY User Manual]],
    [[https://ianix.com/index.html][IANIX Documents]],
    [[https://ianix.com/pub/browser-privacy-handbook.html][The browser privacy handbook]],
    [[https://stribika.github.io/2015/01/04/secure-secure-shell.html][Secure Secure Shell]],
    [[https://git.libssh.org/projects/libssh.git/tree/doc/curve25519-sha256@libssh.org.txt#n4][projects/libssh.git - libssh shared repository]],
    many pages in [[https://en.wikibooks.org/wiki/Category:OpenSSH][Category:OpenSSH - Wikibooks]],

..  comment

    Local Variables:
    mode: rst
    ispell-local-dictionary: "english"
    End:
