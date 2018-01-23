Server-side Authentication
==========================

The first implementation of the Enigma Data Marketplace is designed to integrate 
with Catalyst on the application layer. Until the Enigma network is up and 
running, a temporary implementation of the functionality that will be provided
by the network in the future is offered by an authentication server hosted in 
the cloud.

When a user running Catalyst on their local machine requests to ingest a data 
set they are subscribed to, Catalyst routes that request to the authentication
server. The authentication scheme used between client and server is an 
implementation of digest access authentication using the user's private and 
public cryptographic keys, without the client ever handling their private key 
directly.

When the user requests access to a subscribed data set, their Catalyst client
opens a session with the authentication server. The server replies back with a
challenge that includes a ``nonce`` and an ``opaque`` as specified in 
`RFC 2617 <https://tools.ietf.org/html/rfc2617#section-3.2.1>`_. The client then
requests the ``nonce`` to be hashed and signed using the user's private key. 
The client replies to the authentication server with the signature of the 
hashed ``nonce`` and the ``opaque`` untouched. The server validates that the 
``nonce`` and ``opaque`` are valid, and verifies that the signature corresponds
with the user's public address. If successful, queries the smart contract to 
validate that the user's public address holds and active subscrition to the 
requested data set, and if successful responds with the requested data set back
to the client, thereby completing the request.

.. image:: https://s3.amazonaws.com/enigmaco-docs/digest-authentication.png
	:align: center
	:alt: Digest Authentication

All blockchain implementations, and Ethereum in particular, use public-key
cryptography in their addressing scheme. Each address on the blockchain is 
associated with a private and a public key, where the private key is secret, 
the public key can be mathematically derived from the private key, and the 
public address is a hash+checksum of the public key. It is critical that the 
private key is not leaked, otherwise whoever has access to the private key gains 
full control of its associated address.

Instead of Catalyst handling any private keys to authenticate requests, Catalyst
supports external wallets that handle the user's private keys, and can sign 
transactions on their behalf. Catalyst requests any of the supported wallets to
sign a transaction on behalf of the user (the wallet will require the user to
enter a password to unlock their account), and returns the signature to Catalyst
so that it can continue with the authentication process.

Signature Implementation
~~~~~~~~~~~~~~~~~~~~~~~~

The digest authentication scheme for Enigma's Data Marketplace uses an 
Ethereum-compatible signature implementation that adds a custom Ethereum 
message and length using the following structure:

.. code:: python

	message_to_sign(message):
		 msg = "\x19Ethereum Signed Message:\n" + str(len(message)) + message
		 return sha3(msg)

where the ``sha3()`` is Ethereum's implementation of the 
`KECCAK-256 <https://keccak.team/index.html>`_ cryptographic hash function, 
albeit it does not follow the 
`FIPS-202 <https://keccak.team/specifications.html#FIPS_202>`_ based standard 
(a.k.a SHA-3).

Supported Wallets
~~~~~~~~~~~~~~~~~

The wallets currently supported by Catalyst to access the Data Marketplace are:

* MyEtherWallet
* Parity
* Any other wallet that allows to sign transactions through the MyEtherWallet 
  online interface: 

  * MetaMask / Mist
  * Ledger Wallet
  * TREZOR
  * Digital Bitbox
  * Keystore / JSON File

Source Code
~~~~~~~~~~~

The relevant server-side code that handles the digest authentication described 
in this section is the following:

.. code:: python

	from ethereum.utils import ecrecover_to_pub, checksum_encode, sha3


	def hex_to_byte(hexStr):
	    """
	    Convert a string hex byte values into a byte string. The Hex Byte values
	    may or may not be space separated.
	    """
	    # The list comprehension implementation is fractionally slower in this case
	    #
	    #    hexStr = ''.join( hexStr.split(" ") )
	    #    return ''.join( ["%c" % chr( int ( hexStr[i:i+2],16 ) ) \
	    #                                   for i in range(0, len( hexStr ), 2) ] )
	    _bytes = []
	    hexStr = ''.join(hexStr.split(" "))
	    for i in range(0, len(hexStr), 2):
	        _bytes.append(chr(int(hexStr[i:i+2], 16)))
	    return ''.join(_bytes)


	def msg_to_sign(hexStr):
	    b = hex_to_byte(hexStr)
	    message = hex_to_byte(binascii.hexlify(
	                "\x19Ethereum Signed Message:\n{}".format(len(b))))
	    return message+b


	class CatalystAuth(HTTPDigestAuth):
	    def __init__(self, scheme=None, realm=None, use_ha1_pw=False):
	        super(CatalystAuth, self).__init__(scheme or 'Digest', realm)

	    def authenticate(self, auth, stored_password_or_ha1):
	        if(not auth or not auth.username or not auth.realm or not auth.uri
	           or not auth.nonce or not auth.response):
	                return False

	        if not(self.verify_nonce_callback(auth.nonce)) or \
	                not(self.verify_opaque_callback(auth.opaque)):
	            return False

	        v, r, s, wallet = [e for e in auth.response.split(',')]

	        if wallet == 'mew':
	            nonce = '0x{}'.format(auth.nonce)
	            message = "\x19Ethereum Signed Message:\n{}{}".format(
	                            len(nonce), nonce)
	        elif wallet == 'parity':
	            message = msg_to_sign(auth.nonce)
	        else:
	            return False  # Wallet not supported, fail silently

	        pub = ecrecover_to_pub(sha3(message), long(v), long(r), long(s))
	        pubAddr = checksum_encode(sha3(pub)[-20:])

	        return pubAddr.lower() == auth.username.lower()

The source code for the digest authentication server is available at:
https://github.com/enigmampc/marketplace-authentication 


