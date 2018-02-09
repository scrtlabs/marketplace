Data Subscribers
================

Catalyst users may subscribe to data sets from the Enigma Data Marketplace
and use them in their trading algorithms. Depending on their properties,
data sets may be usable for backtesting as well as live trading.

Subscribe
~~~~~~~~~
The Enigma Data Marketplace enables data providers to publish data sets and make
them directly available to subscribers. All published data is validated and 
hosted on the Enigma Data Marketplace. Your subscription payment is protected 
by the :doc:`Enigma Data Marketplace Smart Contract<smart-contract>`. If a 
provider stops delivering data as per their agreement, the subscriber will be 
automatically refunded on a *pro rata* basis for the unfulfilled portion of 
their subscription.

The following command subscribes to the Marketcap data set:

.. code-block:: bash

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
The Catalyst ``ingest-data`` command makes the specified portion of a data set 
available locally for backtesting with Catalyst.

This commands ingests all the data available in the Marketcap data set.
If the data takes a long time to download and you only need a subset,
use the command parameters to filter by date:

.. code-block:: bash

    $ catalyst ingest-data --dataset=Marketcap
    Ingesting data set Marketcap...
    [=                                   ]  Ingesting daily data for Marketcap:    4%  00:02:43


The progress bar will dynamically update until completion.

Use in an Algorithm
~~~~~~~~~~~~~~~~~~~
Once you have subscribed to a data set, and ingested the required data if
appropriate, the data source is now usable in your algorithms.

This snippet of code below shows how to use our Marketcap data set in the
`initialize()` function of an algorithm to rank the top-five asset by
market cap:

.. code-block:: python

    def initialize(context):
        # Get the marketcap data
        df = get_data_source(
            'marketcap', start=context.datetime
        )  # type: pd.DataFrame

        # Keep only the 100 top currencies by market cap
        df = df.loc[df['market_cap_usd'].isin(df['market_cap_usd'].nlargest(100))]
        # Sort the currencies by market cap
        df.sort_values(by=['market_cap_usd'], ascending=True, inplace=True)

        # Prepare the standard output for a wide DataFrame
        set_print_settings()
        # Display the top 100 largest currencies
        print('the marketplace data:\n{}'.format(df))

        # Out of the 100 largest currencies, we want to trade the smallest
        # 5 markets with an Ether quote on the exchange of our algorithm
        quote_currency = 'eth'
        exchange = context.exchanges[next(iter(context.exchanges))]
        # Get a list of symbols only for assets tradable since the start
        # date of our algorithm.
        symbols = [a.symbol for a in exchange.assets
                   if a.start_date < context.datetime]

        # Initialize an asset list which we will populate with the 5 smallest
        # markets
        context.assets = []
        for currency, price in df['market_cap_usd'].iteritems():
            if len(context.assets) >= 5:
                break

            s = '{}_{}'.format(currency.decode('utf-8'), quote_currency)
            if s in symbols:
                context.assets.append(symbol(s))
