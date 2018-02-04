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

Registration is the process that data curators need to do to particiapte in the Marketplace. In order to protect the ``subscribers``, a data curator will be able to ``withdraw`` the funds only after the ``subscription period`` has ended. 

In order to make a new data set available in the Marketplace, the data curator
needs to register with the smart contract by sending a ``register`` transaction with
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

* **Withdrawing owner's tokens:**
	The withdraw protocol.

.. image:: http://ethereumisrael.org/wp-content/uploads/2018/02/calcWithdraw.png
    :align: center
    :alt: Enigma Data Marketplace withdraw

Subscription
~~~~~~~~~~~~

The subscribers are the ones paying for the data. Every payment is first made to the ``Marketplace`` contract and is held until the Subscription is expired or the Data curator is punished.
In case the Data curator was punished then and only then, the subscriber will able to ask for a ``refund``. The refund is calculated by the relative time already used and will be deposited back to the subscriber's address.

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

* **Refund subscriber incase the data curator is punished:**
	
.. image:: http://ethereumisrael.org/wp-content/uploads/2018/02/calcRefund.png
    :align: center
    :alt: Enigma Data Marketplace withdraw

The Marketplace contract main functionality
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The full API documentation can be found in the Github repository inside the ``IBasicMarketplace.sol`` and ``IMarketplace.sol`` at:
`<https://github.com/enigmampc/smart_contract_marketplace/blob/master/contracts/IBasicMarketplace.sol>`_
`<https://github.com/enigmampc/smart_contract_marketplace/blob/master/contracts/IMarketplace.sol>`_

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
* bytes32 dataName
* uint price
* uint startTime
* uint endTime
* bool unExpired
* bool paid
* bool punishedProvider
* bool isOrder

getDataProviderInfo(bytes32 dataName)
**************************************************************

Given a data curator name, returns the following details of the provider's status

* address owner 
* uint256 price
* uint256 volume
* uint256 subscriptionsNumber
* bool isProvider
* bool isActive
* bool isPunished

getAllProviders()
**************************************************************

Returns a full list of all the data curators names, the first value is `0x0` a name representing the beginning of the list. 

refundSubscriber(bytes32 dataName)
**************************************************************
The following method will create a refund transaction and return the relative amount back to the subscriber if the data curator was punished sometime during the subscription time.
Refund is calculated using that formula:
``PRICE - [(punishTime- startTime) * PRICE / (endTime - startTime)]``.


withdrawProvider(bytes32 dataName)
**************************************************************

Will create a withdraw transactions and `transfer` all the `ENG` tokens to the owner address for each subscription that belongs to the data curator and not paid yet.
It will ignore subscriptions that are not expired yet and calculate the relative time amount in case the curator is punished with that formula:
``(punishTime- startTime) * PRICE / (endTime - startTime)``.

getWithdrawAmount(bytes32 dataName)
**************************************************************
Accessible by anyone, calculate the available withdraw amount of a data curator.


getRefundAmount(address subscriber, bytes32 dataName)
**************************************************************

Accessible by anyone, calculate the available refund amount of a subscriber to some data set.




Source code
~~~~~~~~~~~

The soure code for the smart contract is available at: 
`<https://github.com/enigmampc/smart_contract_marketplace/blob/master/contracts/Marketplace.sol>`_

