Configuration
=============

In order to use the Enigma Data Marketplace, you will need to configure Catalyst
to use an Ethereum account of yours. Please note that Catalyst will **NEVER 
handle your private key**, you only need to configure your public address.

The first time that you run any Catalyst command on the marketplace, you'll get
an error like the following::

    $ catalyst marketplace {command}
    MarketplacePubAddressEmpty:  Please enter your public address to use in the
    Data Marketplace in the following file: $HOME/.catalyst/data/marketplace/addresses.json 

This creates an empty JSON file in the above location for you to edit and 
specify which address you want to use. This is what the initial empty file looks
like::

    [
        {
            "pubAddr":"",
            "desc":""
        }
    ]

Which you would edit as follows using your public address::

    [
        {
            "pubAddr":"0x4A68d69d6BE0f7F429aBFb07eA98caE2093F6896",
            "desc":"My Hardware Wallet "
        }
    ]

where ``pubAddr`` is a required field where you set the Ethereum public address
that you want to use with Catalyst, and the ``desc`` field is optional, in case
you have multiple addresses and you want to easily distinguish between them. 
Should you want to configure multiple addresses, the file would like something 
like the example below::

    [
        {
            "pubAddr":"0x4A68d69d6BE0f7F429aBFb07eA98caE2093F6896",
            "desc":"My first address"
        },
        {
            "pubAddr":"0x3341436363A8343571a643dB3De216c8842D2dC4",
            "desc":"My second address"
        }
    ]

Now when you invoke any of the ``catalyst marketplace`` commands, if you have 
only configured one address (the most likely case), Catalyst will inform you 
that it will use that address for any transaction that you execute::

    $ catalyst marketplace {command}
    Using 0x4A68d69d6BE0f7F429aBFb07eA98caE2093F6896 for this transaction.
    ...

Instead, if you have configured multiple addresses, it will prompt you to choose
which one of them you want to use for the command you are executing::

    $ catalyst marketplace {command}
    0   0x4A68d69d6BE0f7F429aBFb07eA98caE2093F6896  My first address
    1   0x3341436363A8343571a643dB3De216c8842D2dC4  My second address
    Choose your address associated with this transaction: [default: 0] 




