Troubleshooting
===============

If you get errors like either one of these::

  File "(...)/virtualenv/lib/python3.6/site-packages/eth_keyfile/keyfile.py", line 7, in <module>
    from Crypto.Protocol.KDF import scrypt
  ImportError: cannot import name 'scrypt'

.. code:: bash

  File "(...)/virtualenv/lib/python3.6/site-packages/eth_keyfile/keyfile.py", line 5, in <module>
    from Crypto import Random
  ImportError: cannot import name 'Random'

Make sure you don't have package ``PyCrypto`` installed, which is not 
being maintained currently::

	pip uninstall pycrypto

and it that was the case, reinstall package ``pycryptodome``::

	pip uninstall pycryptodome; pip install pycryptodome