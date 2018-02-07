.. image:: https://s3.amazonaws.com/enigmaco-docs/enigma-marketplace.png
    :target: https://enigmampc.github.io/marketplace
    :align: center
    :alt: Enigma | Marketplace

|
|

Overview
========

The Enigma Data Marketplace is the platform layer that lays in between the 
protocol and application layers of the Enigma network. It provides the 
decentralized and secure data infrastructure on top of which applications can be
built, like `Catalyst <https://enigmampc.github.io/catalyst>`_.

The Data Marketplace is currently in Phase 1 of its development. The current
implementation includes the on-chain portion, dealing with contextual 
information about data-sets, namespaces, and subscriptions. The on-chain logic 
is coded in smart contracts deployed on the Ethereum network, and is operated 
directly with Enigma tokens (ENG). In this first implementation, all data sets 
are provided off-chain by several providers, and their storage is managed 
independently from the logic embedded in the smart contract. 

The first implementation of the data marketplace is designed for integration 
with Catalyst on the application layer. Data sets are consumed as premium data 
inputs to trading algorithms that derive trading signals from them in the quest
to improve performance.

**Phase 2** includes the first implementation of an off-chain network, which will 
run on a federated network of known nodes (a-la sidechains). This could also be 
considered the MVP of the Enigma protocol — one that operates under a restricted
threat model. **Phase 3** is where things get really interesting, and we launch our 
open off-chain network where anyone can become a node and provide storage and 
computational resources in return for ENG tokens. Finally, **Phase 4** is where we 
introduce important privacy features for protecting data in the network. This 
would allow extending the usability of our data protocol to practically all 
kinds of data streams, including those that include personally identifiable 
information (PII).

Learn more at our `Documentation Website <https://enigmampc.github.io/marketplace/>`_.