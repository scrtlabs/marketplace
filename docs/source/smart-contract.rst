Smart Contract
==============

The Enigma Data Marketplace is governed by a smart contract deployed on the 
Ethereum network that is operated directly with Enigma tokens (ENG). The smart 
contract allows data curators to register new data sets, and allows users to 
subscribe to existing data sets. The subscription period defaults to 
*one month*, and prices for all data sets are designated accordigly as monthly 
costs.

The Enigma Data Marketplace contract is publicly available and can be used by 
any other contract in the network. The smart contract stores each of the data 
sets metadata (owner, price, number of subscriptions,...) and information about
each subscription to any data set (price, start time and end time).

Registration
~~~~~~~~~~~~

In order to make a new data set available in the marketplace, the data curator
needs to register with the smart contract by sending a register transaction with
the following details:

* **Datasource name**

	Data source names are strings that must be unique. The name can be any 
	string as long as it doesn't exist in the contract registry already and 
	its length is less than 32 bytes.

* **Datasource price**

	The price is designated in ENG Tokens for a monthly subscription.

* **Datasource owner:**
	
	The address of the owner of the data source. This address is where the 
	payments from the users' subscriptions to this data source will be sent to.
	The owner's address also allows for changing the price (does not effect
	active subscriptions, only new subscriptions and renewals), and deactivating
	a data set.

Subscription
~~~~~~~~~~~~

In order to access an existing data set, a user needs to subscribe to the 
desired data set through a two-step process: 

1. The subscriber approves the Enigma Data Marketplace contract as a 
   spender in the Enigma ERC20 contract on their behalf for the
   specified amount for that data set.

3. Once the Marketplace contract has been approved, a user can send a 
   subscription transaction that includes the desired data source name. The
   price is specified in the data set (stored in the marketplace contract), and
   the amount is transferred on behalf of the user (after the pre-approval 
   from step #1 above succeeds).

.. image:: http://ethereumisrael.org/wp-content/uploads/2018/01/SubscriptionProtocolDiagram.png
    :align: center
    :alt: Enigma Data Marketplace subscription

The MarketPlace contract functionality
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Enigma's Data Marketplace provides the following functionality.
 
subscribe(bytes32 sourceName)

*****************************
Subscribes to a data source after approving the marketplace contract in 
the ENG token contract as a spender on behalf of the subscriber. The amount
of ENG available on the subscriber's address should be equal or larger than the
price of the desired data set.


register(bytes32 sourceName, uint price, address owner)
*******************************************************
Registers a new data source. ``sourceName`` must be unique, and ``price`` is
designated in ENG Tokens for a monthly subscription.

checkAddressSubscription(address subscriber, bytes32 dataName)
**************************************************************
Given a subscriber address and an existing data set, returns the following 
details of the user's subscription to that data set
 
* address subscriber
* uint price
* uint startTime
* uint endTime
* bool expired

.. image:: http://ethereumisrael.org/wp-content/uploads/2018/01/MarketPlaceContractUML-1.png
    :align: center
    :alt: Enigma Data Marketplace contract UML

Source code
~~~~~~~~~~~

The soure code for the smart contract is available at: 
`<https://github.com/enigmampc/smart_contract_marketplace/blob/master/contracts/Marketplace.sol>`_

