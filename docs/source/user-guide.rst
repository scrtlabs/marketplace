User Guide
==========

Catalyst users may subscribe to data sets from the Catalyst Marketplace
and use them in their trading algorithms. Depending on their properties,
data sets may be usable for backtesting as well as live trading.

Subscribe
~~~~~~~~~
The Catalyst Marketplace enables data providers to publish data sets
directly to subscribers. All published data is validated and hosted on the
Catalyst Marketplace. Your subscription payment is protected by the Catalyst
Marketplace smart contract. If a provider stops delivering data as per their
agreement, the subscriber will be automatically refunded.

The following command subscribes to the Marketcap data set::

    $ catalyst subscribe-data --dataset=Marketcap
    Subscribing to the Marketcap data set published by Enigma.

    Terms:
        * New data published once per day at exactly 0:00
        * Daily historical data available between:
            * 2017-01-01 to 2018-01-21
        * Subscription price 1ENG/month
        * Subscribing for 1 month: 2017-01-22 to 2017-02-22
        * Today's Deposit: 1ENG
        * The entire 1ENG deposit will be transferred to Enigma on 2017-02-22
          upon completion of these terms.

    Agree to subscribe to Marketcap for 1ENG? [N] Y
    Please unlock your wallet account to authorize signature.

    Signature authorized.
    0x627306090abab3a6e1400e9345bc60c78a8bef57 successfully subscribed to
    Marketcap for 1ENG

Ingest
~~~~~~
