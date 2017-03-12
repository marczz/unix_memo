.. _gnupg_memo:

GnuPG Memo
==========

There are two gnupg programs GnuPG 1.4 is the standalone,
non-modularized series, GnuPG 2.x is the new modularized version of
GnuPG supporting OpenPGP and S/MIME. GnuPG 2.1 introduce number of
`new characteristics
<https://www.gnupg.org/faq/whats-new-in-2.1.html>`_
including a new format for locally storing the public keys, and
dropping the file ``secring.gpg``. ``gpg`` 1.4 is the standalone
version of gpg it will stay for embedded and server usage, as it
brings less dependencies and smaller binaries.  For desktop we use
gpg2 and most distribution alias the command `gpg` to `gpg2`.


.. contents:: `Gpg commands by category`
   :depth: 2
   :local:

For all commands a running
`gpg-agent <http://www.gnupg.org/documentation/manuals/gnupg/Invoking-GPG_002dAGENT.html>`__
avoid to enter again your passphrase in the same session.

list of keys.
-------------

::

    gpg --list-keys
    gpg --list-keys Luc

to add the fingerprints::

    gpg --fingerprint
    gpg --fingerprint Luc

List only secret keys::

    gpg --list-secret-keys
    gpg --list-secret-keys 123456789D

*keys followed by # are not usable.*

The usage field use: **e** for *encrypt*, **s** for *sign*, **c** for *certify*
i.e. signing an other key, **a** for *authentication*  cf `doc/DETAILS
<http://git.gnupg.org/cgi-bin/gitweb.cgi?p=gnupg.git;a=blob_plain;f=doc/DETAILS>`_
installed with gnupg.

signing.
--------
To sign with your default key:
::

    gpg --sign message.txt

in this case the result ``message.txt.gpg`` is compressed and not
legible.

If you want to use an other private key
::

    gpg -u 1234ABC8 message.txt
    gpg -u 'gnu corp. inc'  message.txt

Here the string *'gnu corp. inc'* is a unique substring in the
private key uids.

To make a clear signature::

    gpg --clearsign message.txt

for a a detached signature in ``/tmp/message.txt.sig``::

    gpg --detach-sign /tmp/message.txt

now a detached and ascii armored signature in ``/tmp/message.txt.asc``::

    gpg --detach-sign --armor /tmp/message.txt



Verifying a signature.
----------------------

If you have a file ``message.txt`` and the detached signature
``/tmp/message.txt.asc``::

    gpg --verify message.txt.asc

If the signature name does not match the file name::

   gpg --verify message.txt.asc renamedfile.txt

For a signed or clear-signed message use::

    gpg --verify message.txt.gpg

if it is also crypted the ``--decrypt`` operation decrypt *and*
verify the signature.


Encrypting.
-----------

::

    gpg --recipient Luc  --encrypt /tmp/message.txt

produces ``/tmp/message.txt.gpg`` ``Luc`` is **a substring** of the Id of the
recipient, if ambiguous or absent you are asked for the recipient.

*Note that you* `don't need a full uid
<http://www.gnupg.org/documentation/manuals/gnupg/Specify-a-User-ID.html>`_
of a key or subkey, any non ambiguous part will do. You can also use
the pub key Id itself.

To encrypt to stdout with the key associated wit pub key
``A897396C``::

    gpg --recipient A897396C --output - /tmp/message.txt

The same operation using fingerprints::

    gpg --recipient 123434343434343C3434343434343734349A3434 --output - /tmp/message.txt

You can also use a word match::

    gpg --recipient '+Smith Frank Junior'

Will match a key uid containing *Frank Smith Junior*.

If you have configured ``~/.gnupg/gpg.conf`` with a
``default-recipient`` or ``default-recipient-self``::

    gpg --encrypt /tmp/message.txt

encrypt to the default recipient, if it is missing it will
ask for a recipient.

To encrypt and *armor* in the *ASCII-armored text*
``/tmp/message.txt.asc`` using the key set in configuration with
``default-key`` (or a single private key) use::

    gpg --recipient Luc  --armor --encrypt /tmp/message.txt``

To encrypt and sign with armored text::

    gpg --recipient Luc --sign --armor --encrypt /tmp/message.txt


To encrypt and sign choosing as recipient the key which have an uid
with an *exact* (not *substring*) mail address of
``luc.smith@gnu.org``, and a specific secret key::


    gpg --local-user 1122C3B8 --recipient '<luc.smith@gnu.org>' --output /tmp/doc.gpg \
    --encrypt --sign doc.txt

The recipient will use the ``--decrypt`` option to extract
*and verify the signature* of ``message.txt.asc`` or ``doc.gpg``.

To create an encrypted archive with your default key::

    tar -vcz dir1 dir2 file1 | gpg --encrypt --output archive.tgz.gpg

And you extract the tar archive with::

     gpg --decrypt archive.tgz.gpg | tar -zx

Even if gpg is most often used for
:wikipedia:`public key cryptography <Public-key_cryptography>`
you can use it for encoding with a :wikipedia:`symmetric key
<Symmetric-key_algorithm>`. In this case GnuPG will ask for a
passphrase, and the  passphrase verification. The
default gnupg encryption algorithm is :wikipedia:`CAST-128` also
called *CAST5*,
you can change it with ``--cipher-algo``. To encrypt with a
symmetric key use::

    gpg --symmetric /tmp/message.txt

To encrypt with a symmetric key and use the plain ASCII form of
output::

    gpg --symmetric  --armor /tmp/message.txt

If you have yet encrypted a file in binary format and you want to
transform in ascii::

    gpg  --output message.asc --enarmor message.gpg

To  encrypt with a symmetric key using AES256 algorithm::

    gpg --cipher-algo AES256 --symmetric /tmp/message.txt

Decrypting.
-----------
::

    gpg --decrypt /tmp/message.txt.asc
    gpg --decrypt --output /tmp/message.txt /tmp/message.txt.asc

available ciphers.
------------------

List version, available cipher algorithms and compression methods
::

    gpg --version


receiving keys.
---------------

::

    gpg --recv-keys --keyserver hkp://subkeys.pgp.net 0xC9C40C31

The server can be omitted to use the default one in
``~/.gnupg/gpg.conf``

refreshing keys.
----------------
::

    gpg --refresh-keys --keyserver hkp://subkeys.pgp.net

or with default server::

    gpg --refresh-keys

creating a key.
---------------
::

    gpg --gen-key

you should then create a revocation certificate with::

    gpg --ouput revoke.asc --gen-revoke FE8512E1

and put it in a secure place.

Exporting a key.
----------------

To export the public keys in binary format to ``/tmp/keyring``::

    gpg --output /tmp/keyring --export

To export Luc public key in ascii for sending by mail::

    gpg --export --armor Luc

Publish a key on a keyserver (mandatory key id)::

    gpg --keyserver keys.gnupg.net --send-key FE8512E1

If you need to export a secret key *for using on an other computer*::

    gpg --output /tmp/mygpgkey_sec.gpg --armor --export-secret-key  FE8512E1

The secrete key is a very sensible data, exporting in cleartext should
only be done on a secure computer, and the file must be shreded (
:man:`shred(1)`)  after use.

`shred` does not work on some filesystem like *brtfs*, if your */tmp/*
is a *tmpfs* file system, you are safe to use it but you have still the problem to
protect your file during transport and on the other computer.

You can better symmetric encrypt the exported private key::

    gpg --export-secret-key  FE8512E1 | \
    gpg --symmetric --armor --output  /tmp/mygpgkey_sec.asc

You are then asked for a password for symmetric encryption, and you
private key stay protected.

importing a key.
----------------
::

    gpg --import colleague.asc

To import from the default keyserver when you now the key ID::

    gpg --recv-keys FE8512E1 12345FED

Or choose a key by name regexp::

    gpg --search-keys somebody

If there are multiple strings matching ``somebody`` gpg
will present you a menu to choose one specific key".


To import a previously exported secret key::

    gpg --allow-secret-key-import --import /tmp/mygpgkey_sec.gpg

If you follow the advice to symetric encrypt the secret key::

    gpg --decrypt   /tmp/mygpgkey_sec.asc | gpg --allow-secret-key-import --import


Editing your keys
-----------------

To edit a key you have to select it by a substring of one of its IDs.
::

    gpg --edit-key me@example.com
    gpg --edit-key FE8512E1

present a menu with many key management related tasks, you get a
list with ``help``, among which:

+-----------------------------+--------------------------------+
|list                         |list subkeys and uid            |
+-----------------------------+--------------------------------+
|key                          |select subkey N                 |
+-----------------------------+--------------------------------+
|uid                          |select uid N                    |
+-----------------------------+--------------------------------+
|:ref:`adduid <uid_manage>`   |add a user ID                   |
|                             |                                |
+-----------------------------+--------------------------------+
|:ref:`deluid <uid_manage>`   |delete selected user IDs        |
|                             |                                |
+-----------------------------+--------------------------------+
|:ref:`revuid <uid_manage>`   |revoke selected user ID         |
|                             |                                |
+-----------------------------+--------------------------------+
|addkey                       |add a subkey                    |
+-----------------------------+--------------------------------+
|delkey                       |delete selected subkeys         |
+-----------------------------+--------------------------------+
|revkey                       |revoke key or selected subkeys  |
+-----------------------------+--------------------------------+
|expire                       |change the expiration date      |
+-----------------------------+--------------------------------+
|passwd                       |change the passphrase           |
+-----------------------------+--------------------------------+
|:ref:`showpref <pref_modify>`|show key preferences            |
|                             |                                |
+-----------------------------+--------------------------------+
|:ref:`setpref <pref_modify>` |change key preferences          |
|                             |                                |
+-----------------------------+--------------------------------+
|sign                         |sign a key                      |
+-----------------------------+--------------------------------+
|lsign                        |local sign a key                |
+-----------------------------+--------------------------------+
|:ref:`trust<trust>`          |change owner trust level        |
+-----------------------------+--------------------------------+
|save                         |save and quit                   |
+-----------------------------+--------------------------------+
|quit                         |ask for saving and quit         |
+-----------------------------+--------------------------------+


..  _uid_manage:

Deleting or Revoking UID
~~~~~~~~~~~~~~~~~~~~~~~~

Sometime you either change your mail address, or drop an old one, or
acquire a new one. An uid cannot be modified. You have to delete or
revoke the old uid, and create a new one.

For local keys you can delete components subkeys and uid, but when
your key is distributed, for instance when published on a key server,
it is ineffective and your old id will still be present on the
keyserver, and other people keyring, see
`GnuPg Manual: Adding and deleting key components
<https://www.gnupg.org/gph/en/manual/c235.html#AEN281https://www.gnupg.org/gph/en/manual/c235.html#AEN281>`_
for explanations.

so if your key is distributed you rather want to revoke old
components, and add new ones.

::

    gpg> list
    pub  2048R/1234567C created ......
    sub  2048R/9876543F created ....
    [ultimate] (1). Frank <frank.nick@mail.org>
    [ultimate] (2)  Frank <frank.oldnick@prevmail.org>
    gpg> uid 2
    .....
    [ultimate] (2)*  Frank <frank.oldnick@prevmail.org>
    gpg> revuid

..  _trust_vs_validity:

Trust and validity
~~~~~~~~~~~~~~~~~~
*Trust* is used to mean trust in a key's owner, and *validity* is
used to mean trust that a key belongs to the human associated with the
key ID. So a key is *Valid* if signed by trusted people, you can
manage the *trust* in the owners of your keyring by editing key trust,
and you can also *validate* a key by signing it, or by doing a private
(i.e. not exported and shown to other) *local signature*.


There are four trust levels: *unknown* this is the initial trust of a
newly imported key, *none* i.e. untrusted, *marginal* i.e. good trust
level, *full* i.e. as secure as your own key.

A key is considered valid if it meets two conditions:

1.  It has been signed by you or by one fully trusted key, or by three
    marginally trusted keys.

2.  The path length from your key down to the considered key is less
    or equal to five steps.

You can see an example of a marginally trusted but nevertheless not
valid key in the :ref:`next subsection<trust>`.

..  _trust:

Editing trust.
~~~~~~~~~~~~~~

You change the owner trust with the *trust* subcommand:

::

    gpg> trust
    pub  dsa1024/ECEC8BDAA6606D75
        created: 2004-12-09  expires: never       usage: SCA
        trust: unknown       validity: unknown
    sub  elg1024/D7671AF7DE8BAA37
        created: 2004-12-09  expires: never       usage: E
        [ unknown] (1). Albert Lebrazh <albert.Lebrazh@gmail.com>

    Please decide how far you trust this user to correctly verify other users' keys
    (by looking at passports, checking fingerprints from different sources, etc.)

    1 = I don't know or won't say
    2 = I do NOT trust
    3 = I trust marginally
    4 = I trust fully
    5 = I trust ultimately
    m = back to the main menu

    Your decision?

    3

    pub  dsa1024/ECEC8BDAA6606D75
        created: 2004-12-09  expires: never       usage: SCA
        trust: marginal      validity: unknown
    sub  elg1024/D7671AF7DE8BAA37
        created: 2004-12-09  expires: never       usage: E
        [ unknown] (1). Albert Lebrazh <albert.Lebrazh@gmail.com>
    Please note that the shown key validity is not necessarily correct
    unless you restart the program.

In this example nobody else than Albert Lebrazh has signed this key,
so even when I declare marginally trusting him as explained
:ref:`above<trust_vs_validity` , his key is still not
valid, this is confirmed when I try again to edit the key.

::

    gpg --edit-key ECEC8BDAA6606D75

    gpg: checking the trustdb
    gpg: marginals needed: 3  completes needed: 1  trust model: pgp

If I am sure than the key belongs to the user, I can sign it *or
lsign*; and it will become valid.

..  _pref_modify:

Changing Preferences
~~~~~~~~~~~~~~~~~~~~
Key preferences are list of preferred algorithm for ciphers, digest,
and compression.

If you have some old private key, it could have been created with a
set of preferrence that is no longer current.

The first versions of GnuPg used a default hash of SHA1, now
considerred as weak, and sha2 is preferred.

You can inspect your preferences and change them in the following way.
::

    gpg> showpref
         [ultimate] (1). Frank <frank.nick@mail.org>
         Cipher: AES256, AES192, AES, CAST5, 3DES
         Digest: SHA1, SHA256, RIPEMD160
         Compression: ZLIB, BZIP2, ZIP, Uncompressed
         Features: MDC, Keyserver no-modify
     gpg> setpref
         Set preference list to:
         Cipher: AES256, AES192, AES, CAST5, 3DES, IDEA
         Digest: SHA256, SHA1, SHA384, SHA512, SHA224
         ......
     Really update the preferences? (y/N) y
     You need a passphrase to unlock the secret key for ...
     .....
     gpg> pref
         [ultimate] (1). Frank <frank.nick@mail.org>
         Cipher: AES256, AES192, AES, CAST5, 3DES, IDEA
         Digest: SHA256, SHA1, SHA384, SHA512, SHA224
         ......

Here we have used pref *without argument* to reset the preferences to
the default, either the server wide default set by gnupg or if you
have set personal defaults in your configuration with
``default-preference-list``.

You can also set preferences only for this key, see more details in
`GnuPg Manual: Key Management
<https://www.gnupg.org/documentation/manuals/gnupg/OpenPGP-Key-Management.html>`_
in the *setpref* description.

Replacing old  dsa key by a new rsa one.
----------------------------------------
-  `Ana's blog: Creating a new GPG key
   <http://ekaia.org/blog/2009/05/10/creating-new-gpgkey/>`_
   included also in
   `keyring.debian.org - Creating a new GPG key
   <http://keyring.debian.org/creating-key.html>`_.
-  `Weblog for dkg: HOWTO prep for migration off of SHA-1 in
   OpenPGP <http://www.debian-administration.org/users/dkg/weblog/48>`_

To summarize the process:

-   Create a new key, using 2048-bit RSA
-   Generate revocation certificate for the new key
-   Add necessary uids
-   Sign your new key with your old one.
-   Revoke no longer used uid from the old key.
-   If all uid are to be revoked, create a new one specifying
    *in the comment*, that the other key is to be used now.
-   Publish both keys.
-   Ask trusted people that use and certificated the old key to
    certificate the new one.
-   Issue a new certification for users keys that you certified with
    the old key, and are still current.


References
----------

-   `Wikipedia: GNU Privacy Guard
    <http://en.wikipedia.org/wiki/GNU_Privacy_Guard>`_
-   `Using the GNU Privacy Guard
    <http://www.gnupg.org/documentation/manuals/gnupg/>`_
-   `GnuPG Home Page
    <http://www.gnupg.org/>`_

    -   `Invoking GPG
        <http://www.gnupg.org/documentation/manuals/gnupg/Invoking-GPG.html>`_
        is an online version of the gpg2(1) manual.
    -   `The GNU Privacy
        Handbook <http://www.gnupg.org/gph/en/manual.html>`_, ( `french
        translation <http://www.gnupg.org/gph/fr/manual.html>`_ )

        -   `GnuPG manual
            <http://www.gnupg.org/documentation/manuals/gnupg/>`_
            This manual documents how to use the GNU Privacy Guard system as
            well as the administration and the architecture.
        -   `Faq <http://www.gnupg.org/documentation/faqs.html>`_
        -   `list of howtos
            <http://www.gnupg.org/documentation/howtos.en.html>`_

-   `GnuPG Gentoo User Guide <http://wiki.gentoo.org/wiki/GnuPG>`_
    how to install GnuPG, how to create your key pair, how to add keys
    to your keyring, how to submit your public key to a key server and
    how to sign, encrypt, verify or decode messages you send or
    receive, or local files.
-   `ArchWiki: GnuPG <https://wiki.archlinux.org/index.php/GnuPG>`_
    environment variables, configuration file, encrypt and decrypt,
    gpg-agent, pinentry, start gpg-agent with systemd user,
    unattended passphrase, keysigning Parties, smartcards, troubleshooting.
-   Ubuntu Documentation: `Gnu Privacy Guard Howto
    <https://help.ubuntu.com/community/GnuPrivacyGuardHowto>`_
    does not bring much on usage, but has a section on web of trust,,
    key signing and key backup.

    -   `Storing GPG Keys on an Encrypted USB Flash Drive
        <https://help.ubuntu.com/community/GPGKeyOnUSBDrive>`_
-   `OpenPGP Best Practices
    <https://help.riseup.net/en/security/message-security/openpgp/best-practices>`_
-  `gpg quickstart <http://www.madboa.com/geek/gpg-quickstart/>`__ by
   Paul Heinlein, is an up-to-date beginner how-to.
-  subkeys are a quite difficult feature of gnupg and not very well
   documented. You can read `Using multiple subkeys in
   GPG <http://blog.dest-unreach.be/wp-content/uploads/2009/04/pgp-subkeys.html>`__
   by Adrian von Bidder and `Debian
   Wiki:subkeys <https://wiki.debian.org/subkeys>`__.
-  `Philip Zimmermann Home page <http://www.philzimmermann.com/>`__
   Philip Zimmermann is the creator of PGP and distributes also Zfone,
   soft for encrypting voip telephony.


GPG tools
~~~~~~~~~
-   `GnuPg Helper Tools
    <http://www.gnupg.org/documentation/manuals/gnupg/Helper-Tools.html>`_
    contains *watchgnupg*, *gpgv*, *addgnupghome*, *gpgconf*,
    *applygnupgdefaults*, *gpgsm-gencert.sh*, *gpg-preset-passphrase*,
    *gpg-connect-agent*, *dirmngr-client*, *gpgparsemail*, *symcryptrun*,
    *gpg-zip*.
-   `gpgv2
    <https://www.gnupg.org/documentation/manuals/gnupg/gpgv.html>`_
    a stripped-down version of gpg which is only able to check signatures.
-   `Gpa <http://wald.intevation.org/projects/gpa/>`__ is a graphical
    user interface for GnuPG. GPA utilizes GTK+ and connects to GnuPG via
    the `GPGME
    library <http://www.gnupg.org/documentation/manuals/gpgme/>`__.
-   `gpg-preset-passphrase
    <https://www.gnupg.org/documentation/manuals/gnupg/gpg_002dpreset_002dpassphrase.html#gpg_002dpreset_002dpassphrase>`_
    Put a passphrase into the cache.
