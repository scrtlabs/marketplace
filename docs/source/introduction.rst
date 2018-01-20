Introduction
============

At a high level, Enigma creates a decentralized data marketplace that allows 
people, companies and organizations to contribute data (we call these data 
curators), which users of the system can then subscribe to and consume. In 
other words, the data marketplace acts as a single gateway to a network of 
databases, owned collectively by data curators who supply the content. The 
marketplace is decentralized and not owned by any single party. Instead, 
listing and subscribing to a data source is managed on-chain, including rewards
and penalties, which are exchanged using the ENG token.

The data itself, its storage and transmission, lives off-chain. In that sense, 
the blockchain acts as the controller of the network (as was illustrated in 
the `Enigma whitepaper <http://enigma.co/enigma_full.pdf>`_), while the 
off-chain network handles everything else. In particular, the off-chain 
component handles everything from routing requests, verifying permissions, 
parsing and forwarding queries to the source (e.g. a database, a CSV file), 
and finally — routing the result back in the peer-to-peer network to the 
requesting client (encrypted with the client’s private key).

.. image:: https://s3.amazonaws.com/enigmaco-docs/enigma-data-marketplace.png
    :align: center
    :alt: Enigma Data Marketplace

The figure above provides more details on how the data marketplace functions. 
Before we go into detail, let us provide a 30,000 feet explanation.

Registering a new data-set starts by submitting a new data curation script, 
which then sends a transaction to the blockchain that all nodes track, 
announcing a new data set has been made available. This new resource has a 
permanent address that clients can subscribe to on-chain, while paying the 
specified fee. Once subscribed, clients can consume the data in their 
applications, by signing a query request and broadcasting it over the off-chain
network of peers.

Fairness in the system is guaranteed mostly through economic incentives — good
datasets will have more subscribers, which pay more to their data curators and
are ranked higher in the system. Data sources can have a ‘try before you buy’ 
option, which help attract new subscribers. If a data set is then shown to not 
be useful, most people would unsubscribe and the source would be further 
reduced in ranking. In addition, some provable offenses (e.g., a data curator 
going offline) can be penalized.

Below, we will walk you through an end-to-end example of how to use the data 
marketplace, while pointing out important technical aspects. We assume for 
simplicity that all nodes run an enigma node — a full peer-to-peer node that 
connects to the off-chain Enigma network, creating the data marketplace, as 
well as the host blockchain where the on-chain management is done. For the 
foreseeable future, our plan is to connect our network to Ethereum. At the time
of writing, it is the most secure chain that also provides us with the 
smart-contract functionality that we require.

Registration
~~~~~~~~~~~~

Naturally, the first step in the process is to upload a data set to the 
marketplace and register it. Under the hood, this is somewhat of an involved 
procedure, which is a mix of on-chain and off-chain operations:

1. The data curator, through their off-chain client, submits a *cron*-like job 
   with the data curation script and details on how frequently to execute the 
   job (e.g., minutely, hourly, daily). This ensures the data always remains 
   up-to-date. In addition, the client needs to specify some metadata such as 
   the name of the database, where it is physically stored, and which nodes 
   execute the job (more on this later).

2. A *register* transaction is sent to the blockchain, signed by the data 
   curator. The transaction needs to include in its payload the name of the 
   data-set; a price to subscribe (we’ll assume for simplicity these are monthly 
   subscriptions); a deposit the data curator locks in as collateral; payout 
   addresses of nodes that assist the curator (e.g., in providing computing and
   storage resources); and a hash of all meta-data that is stored off-chain.

3. All meta-data concerning the new data-set is broadcast as a signed message to
   all nodes in the enigma off-chain network, and stored in a shared, 
   distributed hash table (DHT). To ensure data integrity and network 
   robustness, we plan to use some revision of `PBFT <https://dl.acm.org/citation.cfm?id=296824>`_ 
   and `S/Kademlia <http://www.tm.uka.de/doc/SKademlia_2007.pdf>`_. This should
   present a good enough solution against malicious coalitions of certain size
   (between ⅓–½ of the nodes, depending on the implementation). We expect this
   to be sufficient given that like in Bitcoin, nodes have an implicit incentive
   to remain honest in order to earn *data mining* rewards.

While all Enigma nodes (not including thin clients) must be a part of the 
Enigma network — which means they are both connected to the host blockchain and
the off-chain network, there are two other (optional) roles they can serve. The
first is acting as a *worker node*, which executes cron jobs that aggregate the
data. The second role serves as a storage and query engine for the data. This is
known as a *persistence node*.

Persistence nodes are meant to be especially thin and modular. They are simply
wrappers that can parse data queries coming from the enigma network and route
them to a more traditional database engine and storage unit. To begin with, we
plan to build support for a single common underlying database. Over time, we
hope to enable a fully distributed database that lives inside of our network.

Subscription
~~~~~~~~~~~~

Subscribing (or unsubscribing) to a data-set happens completely on-chain. To 
make discovery easy, data sources are ranked according to a simple public metric:

.. image:: https://s3.amazonaws.com/enigmaco-docs/rank-formula.png
    :align: center
    :alt: Enigma Data Marketplace

Once a user finds a data-set that they like and wish to subscribe to, they send
a subscription transaction specifying the unique path of the data-set, as well
as the required payment for it. This transaction can be automated in the client
so that it is repeated every month. In fact, this will likely be the default
behavior. If the payment is valid, the smart-contract adds the signer’s address
to the list of subscribers.

Consumption
~~~~~~~~~~~

As opposed to subscription, consumption happens completely off-chain. A user who
wishes to query a specific data-set, for example — <curator_address>/coinbase_data, 
can broadcast a message requesting it. Nodes in the off-chain network propagate
the message, until it reaches a persistence node having direct access to the 
data. The persistence node then parses the request, verifying that the user is
subscribed to the data on-chain. If the user has the appropriate permission, the
node obtains the data (potentially after issuing the parsed query on behalf of 
the user), encrypts it with the public-key of the user and propagates the 
message back in the network until it reaches its destination.

Incentives
~~~~~~~~~~
A leading principle in the marketplace is that data curators are in charge of 
the quality and availability of their own data. This greatly simplifies the 
incentive mechanism and ensures users of the system enjoy better service. As 
mentioned before, Data curators need to stake tokens on every data-set they 
share. If the data goes offline, or is otherwise corrupt (addressing this is 
beyond the scope of this post), they will have to pay it from their deposit. 
Such dishonest actions are likely to reduce the number of subscribers to the 
data, further decreasing the payoff of the data curator. With the assumption 
that curators are rational agents who wish to maximize their utility, this 
ensures their best strategy is to provide quality data and guarantee its 
availability.

While data curators can construct and serve their data through a local node 
acting as a worker and persistence node, they may prefer to outsource these 
operations to other nodes. This allows data curators to go offline after 
registering a new data-set. Worker and persistence nodes are specified as 
additional payout addresses in the registration transaction (which could be 
updated by the data curator at any time). This ensures that these nodes remain 
incentivised to do good work. If the curator wishes to keep all the rewards to 
herself, she can specify her own node(s) as the worker/persistence address.

Data privacy
~~~~~~~~~~~~
In some cases, data that is sold in the system would need to remain confidential
even at use. At Enigma, we performed a great deal of research around computing 
over encrypted data, and our goal is to gradually introduce these ideas into our
protocol.

Initially, we plan to introduce relatively fast deterministic and 
order-preserving encryption mechanisms, that will enable data curators to 
encrypt their data at the source. This presents a fairly good trade-off in 
practice, that provides adequate security guarantees, although not perfect, as 
they don’t satisfy indistinguishability against adaptive chosen-plaintext 
attacks. Later on, we plan to gradually introduce more complicated ideas that 
provide perfect security. We refer you to our `whitepaper <http://enigma.co/enigma_full.pdf>`_
to learn more.
