Smart Contract
==============

The Marketplace registration and the subscription data are stored in a smart contract. 

The subscription prices to data sources are monthly fees.

The contract is publicly available and can be used by any other contract in the network.


Registration
~~~~~~~~~~~~
In order to become a data curator, one would need to register with the smart contract. 

The registration process will require the following details:

Datasource name
**********************
In the smart contract, we represent the data sources with unique names. 

The name could be any type of string as long as it doesn't exist in the contract registry yet and its length is less than 32 bytes.

It could be something like: "Bitcoin order book Poloniex".

Datasource price
**********************
The price in ENG Tokens for a monthly subscription.

Datasource owner:
**********************
The owner of the data source to whom payouts will be made for using the data source.

The owner is also the one who can change the price, de-activate listing etc.


Subscription
~~~~~~~~~~~~
In order to use a data source, one would need to subscribe.
During the subscription process a subscriber is paying the data curator.
The payment is delegated via the Marketplace contract.

The subscription process is built out of 3 steps: 

1. The subscriber approve the Marketplace contract as a spender with the amount that is required for the subscription.
2. Thge subscriber register() to the Marketplace contract. 
3. The Marketplace contract deduct the amount and uses trasferFrom() method to pay the data curator.

.. image:: http://ethereumisrael.org/wp-content/uploads/2018/01/SubscriptionProtocolDiagram.png
    :align: center
    :alt: Enigma Data Marketplace subscription

The Marketplace contract functionality
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 

subscribe(bytes32 sourceName)
********************************************
Used to subscribe to a Data source after approving the Marketplace contract in the ENG token as a spender.
The price should be bigger or equal to the desired Datasource price.

register(bytes32 sourceName, uint price, address owner)
******************************************************************
Use for registering a data source.

checkAddressSubscription(address subscriber, bytes32 dataName)
******************************************************************
Get some subscription details in the order: 
[address subscriber, uint price, uint startTime, uint endTime, bool expired]	

.. image:: http://ethereumisrael.org/wp-content/uploads/2018/01/MarketPlaceContractUML-1.png
    :align: center
    :alt: Enigma Data Marketplace contract UML

