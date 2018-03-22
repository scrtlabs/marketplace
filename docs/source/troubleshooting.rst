Troubleshooting
===============

Getting Help
~~~~~~~~~~~~

If you experience problems using the Data Marketplace, you can seek additional
help through the following channels:

- Join our `Catalyst Forum <https://catalyst.enigma.co/>`_, and browse a variety
  of topics and conversations around common issues that others face when using
  Catalyst and the Data Marketplace, and how to resolve them. And join the 
  conversation!

- Join our `Discord community <https://discord.gg/SJK32GY>`_, and head over 
  the #marketplace channel where many other users (as well as the project 
  developers) hang out, and can assist you with your particular issue. The 
  more descriptive and the more information you can provide, the easiest will 
  be for others to help you out.

- Report the problem you are experiencing on our 
  `GitHub repository <https://github.com/enigmampc/catalyst/issues>`_ 
  following the guidelines provided therein. Before you do so, take a moment 
  to browse through all `previous reported issues
  <https://github.com/enigmampc/catalyst/issues?utf8=%E2%9C%93&q=is%3Aissue>`_ 
  in the likely case that someone else experienced that same issue before, 
  and you get a hint on how to solve it.


Known Issues
~~~~~~~~~~~~

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