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

The following command subscribes to the ``marketcap`` data set:

.. code-block:: bash

    $ catalyst marketplace subscribe --dataset=marketcap
    
    The price for a monthly subscription to this dataset is 10 ENG
    Checking that the ENG balance in 0x..... is greater than 10 ENG... OK.
    Please confirm that you agree to pay 10 ENG for a monthly subscription to the dataset "marketcap" starting today. [default: Y] 
    Ready to subscribe to dataset marketcap.

Catalyst will then provide you with instructions to sign two different 
transactions to process your subscription to the dataset.

Ingest
~~~~~~
The Catalyst ``ingest-data`` command downloads and makes the given data set 
available locally for backtesting or live trading with Catalyst.

The following command ingests the ``marketcap`` data set:

.. code-block:: bash

    $ catalyst marketplace ingest --dataset=marketcap
    Starting download of dataset for ingestion...
    Dataset downloaded successfully. Processing dataset...
    INFO: Marketplace: Processing file 0 of 1
    INFO: Marketplace: Processing file 1 of 1

If you need to keep the data up to date while running a live algorithm, you can
schedule a job (``cron`` in Linux/MacOS) to run the above command at periodic 
intervals, and the live/paper trading algorithm will always retrieve the latest
data available on disk on every iteration of the algorithm.

Use in an Algorithm
~~~~~~~~~~~~~~~~~~~
Once you have subscribed to a data set, and ingested the required data if
appropriate, the data source is now usable in your algorithms.

This snippet of code below shows how to use our Marketcap data set in the
`initialize()` function of an algorithm to rank the top-five asset by
market cap:

.. code-block:: python

    from catalyst.api import get_dataset
    from catalyst.exchange.utils.stats_utils import set_print_settings

    def initialize(context):
        # Get the marketcap data
        df = get_dataset(
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
