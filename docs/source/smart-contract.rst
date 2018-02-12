Smart Contract
==============

The Enigma Data Marketplace is governed by a smart contract deployed on the 
Ethereum network that is operated directly with Enigma tokens (ENG). The smart 
contract allows data curators to register new data sets, and allows users to 
subscribe to existing data sets. The subscription period defaults to 
*one month*, and prices for all data sets are designated accordingly as monthly 
costs.

The Enigma Data Marketplace contract is publicly available and can be used by 
any other contract in the network. The smart contract stores each of the data 
sets metadata (owner, price, number of subscriptions,...) and information about
each subscription to any data set (price, start time and end time).

Registration
~~~~~~~~~~~~

Registration is the process through which data providers can offer their 
datasets on the Data Marketplace. Data providers are paid for their dataset 
after the ``subscription period`` ends for any given subscriber, through a
``withdraw`` transaction.

In order to make a new data set available in the Marketplace, the data provider
needs to register with the smart contract by sending a ``register`` transaction 
with the following details:

* **Datasource name**

	Data source names are strings that must be unique, and will be converted
	to lowercase. The name can be any string as long as it doesn't exist in the 
	contract registry already and its length is less than 32 bytes.

* **Datasource price**

	The price is designated in ENG Tokens for a monthly subscription.

* **Datasource owner:**
	
	The address of the owner of the data source. This address is where the 
	payments from the users' subscriptions to this data source will be sent to.
	The owner's address also allows for changing the monthly price (does not 
	affect active subscriptions, only new subscriptions and renewals), and 
	deactivating a dataset.

* **Withdrawing owner's tokens:**
	The withdraw protocol.

.. image:: http://ethereumisrael.org/wp-content/uploads/2018/02/calcWithdraw.png
    :align: center
    :alt: Enigma Data Marketplace withdraw

Subscription
~~~~~~~~~~~~

The subscribers are the users in the Data Marketplace that pay for data. Every 
payment is first made to the ``Marketplace`` contract and is held until the 
Subscription is expired or the Data curator is punished (should their data stop
being available). In case the data provider was punished then and only then, 
the subscriber will able to ask for a ``refund``. The refund is prorated on the
fraction of the monthly subscription when the data was available, and will be 
deposited back to the subscriber's address.

In order to access an existing dataset, a user needs to subscribe to the 
desired dataset through a two-step process: 

1. The subscriber approves the Enigma Data Marketplace contract as a 
   spender in the Enigma ERC20 contract on their behalf for the
   specified amount (the monthly subscription for that dataset).

2. Once the Marketplace contract has been approved, a user can send a 
   subscription transaction that includes the desired dataset name. The
   price is specified in the dataset (stored in the marketplace contract), and
   the amount is transferred on behalf of the user (after the pre-approval 
   from step #1 above succeeds).

.. image:: http://ethereumisrael.org/wp-content/uploads/2018/01/SubscriptionProtocolDiagram.png
    :align: center
    :alt: Enigma Data Marketplace subscription

* **Refund subscriber in case the data provider is punished:**
	
.. image:: http://ethereumisrael.org/wp-content/uploads/2018/02/calcRefund.png
    :align: center
    :alt: Enigma Data Marketplace withdraw

The Marketplace contract main functionality
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The full API documentation can be found in the Github repository inside the 
``IBasicMarketplace.sol``, ``IMarketplace.sol`` and ``IRecoverableMarketplace.sol`` contracts at:
`<https://github.com/enigmampc/smart_contract_marketplace/blob/master/contracts/IBasicMarketplace.sol>`_
`<https://github.com/enigmampc/smart_contract_marketplace/blob/master/contracts/IMarketplace.sol>`_
`<https://github.com/enigmampc/smart_contract_marketplace/blob/master/contracts/IRecoverableMarketplace.sol>`_

Enigma's Data Marketplace provides the following functionality.
 
Subscriber related functions 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

subscribe(bytes32 sourceName):bool
***********************************

Subscribes to a data source after approving the marketplace contract in 
the ENG token contract as a spender on behalf of the subscriber. The amount
of ENG available on the subscriber's address should be equal or larger than the
price of the desired dataset.

refundSubscriber(bytes32 dataName):bool
***************************************
The following method will create a refund transaction and return the relative 
amount back to the subscriber if the data curator was punished sometime during 
the subscription time.
Refund is calculated using that formula:

.. code-block:: bash

	PRICE - [(punishTime- startTime) * PRICE / (endTime - startTime)]

Data curator related functions 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

register(bytes32 sourceName, uint price, address owner):bool
*************************************************************

Registers a new data source. ``sourceName`` must be unique, and ``price`` is
designated in ENG tokens for a monthly subscription.

withdrawProvider(bytes32 dataName):bool
***************************************

Will create a withdraw transactions and `transfer` all the `ENG` tokens to the 
owner address for each subscription that belongs to the data curator and not 
paid yet. It will ignore subscriptions that are not expired yet and calculate 
the relative time amount in case the curator is punished with that formula:

.. code-block:: bash

	(punishTime- startTime) * PRICE / (endTime - startTime)

updateDataSourcePrice(bytes32 dataName, uint256 newPrice):bool
**************************************************************

Update the price of a given dataset, effective for all future subscriptions 
and can be used only by the `data owner`.

changeDataSourceActivityStatus(bytes32 dataName,bool _isActive):bool
********************************************************************

Update the ``dataset`` activity status, effective for all future subscriptions. 
``_isActive=false`` means that the data source is offline for some reason.  
Defaults to ``True``.

General public functions 
~~~~~~~~~~~~~~~~~~~~~~~~

getDataProviderInfo(bytes32 dataName):Provider
**********************************************

Given a dataset name, returns the following details of the provider's status:

* address owner 
* uint256 price
* uint256 volume
* uint256 subscriptionsNumber
* bool isProvider
* bool isActive
* bool isPunished

checkAddressSubscription(address subscriber, bytes32 dataName)
**************************************************************

Given a subscriber address and an existing dataset, returns the following 
details of the user's subscription to that dataset:
 
* address subscriber
* bytes32 dataName
* uint price
* uint startTime
* uint endTime
* bool unExpired
* bool paid
* bool punishedProvider
* bool isOrder

getAllProviders():bytes32[]
******************************

Returns a full list of all the dataset names in hex format. The first value is 
`0x0` a name representing the beginning of the list. 

getWithdrawAmount(bytes32 dataName)
***********************************
Accessible by anyone, calculate the available withdraw amount for a given 
dataset.

getRefundAmount(address subscriber, bytes32 dataName):uint
**********************************************************

Accessible by anyone, calculate the available refund amount by a subscriber 
to a given dataset.

isActiveDataSource(bytes32 dataName):bool
*****************************************

Indicates whether the dataset is active.


isExpiredSubscription(address _subscriber, bytes32 _dataSourceName):bool
************************************************************************

Indicates whether a subscription is expired (True).


Marketplace owner only related functions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

setPunishProvider(bytes32 dataName, bool _isPunished):bool
**********************************************************

Set a punishment status transaction to a dataset. Can be set only by the 
Marketplace contract owner. If ``_isPunished`` is ``True`` the daset will be 
punished.

Marketplace functionallity with loops outside the chain
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sometimes, calculations and loops want to be done outside the chain.

refundSubscriberAt(bytes32 dataName, uint256 _index):bool
**********************************************************
Allowed only by the subscriber otherwise will throw.

getRefundAmountAt(bytes32 dataName,uint256 _index):uint
**********************************************************
Get the refund of a subscription amount from a data curator

getSubscriptionsSize(bytes32 dataName):uint
**********************************************************
Get the number of subscriptions given a data curator.

checkSubscriptionAt(bytes32 dataName, uint256 _index)
**********************************************************
Get subscriptions information in the following order: 

* address subscriber
* bytes32 dataSourceName
* uint price
* uint startTime
* uint endTime
* bool isUnExpired
* bool isPaid
* bool isPunishedProvider
* bool isOrder

isExpiredSubscriptionAt(bytes32 dataName,uint256 _index):bool
**************************************************************
``True`` = Expired, ``False`` = Not expired.

withrawProviderAt(bytes32 dataName, uint256 _index):bool
**********************************************************
Allowed only be the data curator otherwise throws.

getWithdrawAmountAt(bytes32 dataName, uint256 _index):uint
***********************************************************
Get the withdraw amount of some data curator at subscription 'i'.

getProviderNamesSize():uint
**********************************************************
Get the number of all the existing data curators.

getNameAt(uint256 _index):bytes32
**********************************************************
Get the ``name=id`` of a data curator at index (0,...,size);



The Marketplace contract Event types
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

event Registered(address indexed dataOwner, bytes32 indexed dataSourceName, uint price, bool success);
******************************************************************************************************

.. code-block:: javascript

	@dev When data provider finishes registration in the contract
	@param dataOwner the owner of the data
	@param dataSourceName the new name registred
	@param price the price for subscription
	@param true if registred successfully


event SubscriptionDeposited(address indexed from, address indexed to, uint256 value);
*************************************************************************************

.. code-block:: javascript

	@dev an event that indicates that someone has paid the Marketplace contract subscription (before that data is updated in the contract)
	@param from who paid
	@param to the data source owner
	@param value the value that was transfered


event Subscribed(address indexed subscriber,bytes32 indexed dataSourceName, address indexed dataOwner, uint price, bool success);
*********************************************************************************************************************************

.. code-block:: javascript

	@dev an event fired every time subscription has finished (AFTER succssfull payment AND data update).
	@param subscriber who subscribed
	@param dataSourceName the data source name
	@param dataOwner the owner of the data source
	@param price the price paid for subscription
	@param success true if subscribed successfully

event TransferToProvider(address indexed dataOwner, bytes32 indexed dataSourceName, uint256 amount);
****************************************************************************************************

.. code-block:: javascript

	@dev triggerd upon a token transfer to provider (before finishing state update)
	@param dataOwner - the owner that got paid 
	@param dataSourceName - the name of the data source
	@param amount - the amount transferd from the marketplace contract to the provider

event ProviderWithdraw(address indexed dataOwner, bytes32 indexed dataSourceName, uint amount);
***********************************************************************************************

.. code-block:: javascript

	@dev triggerd when the provider finished the withdraw (TransferToProvider event + state update)
	@param dataOwner - the owner that withdrawed
	@param dataSourceName - name of the data source
	@param amount - the amount withdrawed

event ProviderPunishStatus(address indexed dataOwner, bytes32 indexed dataSourceName, bool isPunished);
*******************************************************************************************************

.. code-block:: javascript

	@dev triggerd when punishment status changed (true = punished, false = not punished)
	@param dataOwner - the provider 
	@param dataSourceName - the name of the data source 
	@param isPunished - the status current status AFTER the change

event SubscriberRefund(address indexed subscriber,bytes32 indexed dataSourceName, uint256 refundAmount);
********************************************************************************************************

.. code-block:: javascript

	@dev triggerd when the subscriber got a refund (punished provider)
	@param subscriber - the refunded address
	@param dataSourceName - name of the data source
	@param amount - the amount of the refund

event PriceUpdate(address indexed editor, bytes32 indexed dataSourceName, uint256 newPrice);
********************************************************************************************

.. code-block:: javascript

	@dev triggerd uppon a price change of an existing data source
	@param editor the owner that changed the price
	@param dataSourceName the data source that has changed
	@param newPrice the new price.


event ActivityUpdate(address indexed editor, bytes32 indexed dataSourceName, bool newStatus);
*********************************************************************************************

.. code-block:: javascript

	@dev triggerd upon a change in the state of a data source availablity.
	@param editor who changed the activity state
	@param dataSourceName which dataSource changed. 
	@param newStatus true = active, false = not active (cannot be sold)


Source code
~~~~~~~~~~~

The soure code for the smart contract is available at: 
`<https://github.com/enigmampc/smart_contract_marketplace/blob/master/contracts/Marketplace.sol>`_

