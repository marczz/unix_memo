GnuPG Memo
==========
References:
    -    `Using the GNU Privacy Guard
         <http://www.gnupg.org/documentation/manuals/gnupg/>`_
    -    `Invoking GPG
         <http://www.gnupg.org/documentation/manuals/gnupg/Invoking-GPG.html>`_
         is an online version of the gpg2(1) manual.

Gpg commands by category
------------------------

.. contents:: `Operations`
   :depth: 2
   :local:

For all commands a running
`gpg-agent <http://www.gnupg.org/documentation/manuals/gnupg/Invoking-GPG_002dAGENT.html>`__
avoid to enter again your passphrase in the same session.

list of keys
~~~~~~~~~~~~

::

    gpg --list-keys
    gpg --list-keys Luc

to add the fingerprints::

    gpg --fingerprint
    gpg --fingerprint Luc

signing
~~~~~~~

::

    gpg --sign message.txt

in this case the result ``message.txt.gpg`` is compressed and not
legible.

To make a clear signature::

    gpg --clearsign message.txt

for a a detached signature in ``/tmp/message.txt.sig``::

    gpg --detach-sign /tmp/message.txt

now a detached and ascii armored signature in ``/tmp/message.txt.asc``::

    gpg --detach-sign --armor /tmp/message.txt


Verifying a signature
~~~~~~~~~~~~~~~~~~~~~

If you have a file ``message.txt`` and the detached signature
``/tmp/message.txt.asc``::

    gpg --verify message.txt.asc

If the signature name does not match the file name::

   gpg --verify message.txt.asc renamedfile.txt

For a signed or clear-signed message use::

    gpg --verify message.txt.gpg

if it is also crypted the ``--decrypt`` operation decrypt *and*
verify the signature.


Encrypting
~~~~~~~~~~

::

    gpg --recipient Luc  --encrypt /tmp/message.txt

produces ``/tmp/message.txt.gpg`` ``Luc`` is **a part** of the Id of the
recipient, if ambiguous or absent you are asked for the recipient.

*Note that you* `don't need a full uid
<http://www.gnupg.org/documentation/manuals/gnupg/Specify-a-User-ID.html>`_
of a key or subkey, any non ambiguous part will do. You can also use
the pub key itself.

To encrypt to stdout with the key associated wit pub key
``A897396C``::

    gpg --recipient A897396C --output - /tmp/message.txt

The same operation using fingerprints::

    gpg --recipient 123434343434343C3434343434343734349A3434 --output - /tmp/message.txt

If you have configured ``~/.gnupg/gpg.conf`` with a
``default-recipient-self`` and set a ``default-key``::

    gpg --encrypt /tmp/message.txt

encrypt with your own key. If these defaults are missing it will
ask for a recipient.

To encrypt and *armor* in the *ASCII-armored text*
``/tmp/message.txt.asc`` use::

    gpg --recipient Luc  --armor --encrypt /tmp/message.txt``

To encrypt and sign::

    gpg --recipient Luc --sign --armor --encrypt /tmp/message.txt


To create an encrypted archive with your default key::

    tar -vcz dir1 dir2 file1 | gpg --encrypt --output archive.tgz.gpg

And you extract the tar archive with::

     gpg --decrypt archive.tgz.gpg | tar -zx

Even if gpg is most often used for
:wikipedia:`public key cryptography <Public-key_cryptography>`
you can use it for encoding with a :wikipedia:`symmetric key
<Symmetric-key_algorithm>`. In this case GnuPG will ask for a
passphrase, and the  passphrase verification. The
default gnupg encryption algorithm is :wikipedia:`CAST-128`,
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

Decrypting:
~~~~~~~~~~~
::

    gpg --decrypt /tmp/message.txt.asc
    gpg --decrypt --output /tmp/message.txt /tmp/message.txt.asc

available ciphers
~~~~~~~~~~~~~~~~~

List version, available cipher algorithms and compression methods
::

    gpg --version


receiving keys
~~~~~~~~~~~~~~

::

    gpg --recv-keys --keyserver hkp://subkeys.pgp.net 0xC9C40C31

server can be omitted to use the one in ``~/.gnupg/gpg.conf``

refreshing keys
~~~~~~~~~~~~~~~
::

    gpg --refresh-keys --keyserver hkp://subkeys.pgp.net

or with default server::

    gpg --refresh-keys

creating a key
~~~~~~~~~~~~~~
::

    gpg --gen-key

you should then create a revocation certificate with::

    gpg --ouput revoke.asc --gen-revoke FE8512E1

and put it in a secure place.

Exporting a key
~~~~~~~~~~~~~~~

To export the public keys in binary format to ``/tmp/keyring``::

    gpg --output /tmp/keyring --export

To export Luc public key in ascii for sending by mail::

    gpg --export --armor Luc

Publish a key on a keyserver (mandatory key id)::

    gpg --keyserver keys.gnupg.net --send-key FE8512E1

importing a key
~~~~~~~~~~~~~~~
::

    gpg --import colleague.asc

To import from the default keyserver when you now the key ID::

    gpg --recv-keys FE8512E1 12345FED

Or choose a key by name regexp::

    gpg --search-keys somebody

If there are multiple strings matching ``somebody`` gpg
will present you a menu to choose one specific key".

Editing your keys
~~~~~~~~~~~~~~~~~
::

    gpg --edit-key me@example.com
    gpg --edit-key FE8512E1


present a menu with many key management related tasks, you get a
list with ``help``, among which:

+----------+--------------------------------+
|key       |select subkey                   |
+----------+--------------------------------+
|adduid    |add a user ID                   |
+----------+--------------------------------+
|deluid    |delete selected user IDs        |
+----------+--------------------------------+
|addkey    |add a subkey                    |
+----------+--------------------------------+
|delkey    |delete selected subkeys         |
+----------+--------------------------------+
|revkey    |revoke key or selected subkeys  |
+----------+--------------------------------+
|expire    |change the expiration date      |
+----------+--------------------------------+
|passwd    |change the passphrase           |
+----------+--------------------------------+
