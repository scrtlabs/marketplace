Data Providers
==============

Data Providers can publish data sources to Catalyst and get rewarded in
ENG tokens. Providers set the price of their data set. Subscribers can
pay providers directly through the Enigma Marketplace smart contract.
Engima does not interfere in the transactions.

Data Sources
~~~~~~~~~~~~
A data source can contain any data which can fit in a tabular format.
However, since data will be consumed by Catalyst for the purpose of
algorithmic trading, it must have some predictable properties. In
particular, each data source must contain these two columns:

* Date: The date/time of the event corresponding to each row of the
  data source.
* Symbol: The symbol of the currency of market associated with each even.
  If an event involves more that one currency or market, multiple entries can
  be provided with a semicolon separator.

This sample data source contains market cap information:

.. image:: https://s3.amazonaws.com/enigmaco-docs/data-providers/csv-example-marketcap.png
    :align: center
    :alt: Marketcap Data Source Sample CSV


Data Frequency and Availability
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
A data frequency and availability must be provided when registering each data
source.

* Data frequency: How frequently do data events occur?
    * Strict daily: Publish one event each day at a set time.
    * Strict hourly: Publish one event each hour between a set range of
      minutes.
    * Strict minutely: Publish one event each minutes between a set range
      of seconds.
    * At least each x days: Publish one of more events every day.
    * At least each x hours: Publish one or more events each hour.
    * At lest each x minutes: Publish one of more events each minute.

* Data Availability: How far back are we getting data?
    * Historical range: The data range for historical data. Use multiple date
      ranges if the data does not follow a continuous timeframe.
        * Start: Date/time of the first data event of the range.
        * End: Date/time of the last data event of the range.
    * Ongoing: Will data events continue to be published at the specified
      frequency on an ongoing basis?
    * Scheduled holds: A list a dates for scheduled gaps in ongoing data.

Registering Data Sources
~~~~~~~~~~~~~~~~~~~~~~~~
To register a new data source, download and install the Catalyst client.
Then, use the `catalyst data register` command. In this example, data is
published multiple times per hour at a variable time.::

    $ catalyst data register
    Enter the Data Source name: Test
    Enter the data frequency [daily, hourly, minute]: hourly
    Can data be published every hour at a regular time? [default: Y]: N
    At a minimum, how many times per hour will data be published? [default: 1]: 5
    Does it include historical data [default: Y]? N
    Please unlock your wallet account to authorize signature.

    Signature authorized.
    New data source: Test successfully registered to publisher address: 0x627306090abab3a6e1400e9345bc60c78a8bef57

In the following example, data is published daily on a fixed schedule.::

    Enter the Data Source name: Test
    Enter the data frequency [daily, hourly, minute]: daily
    Can data be published every day at a regular time? [default: Y]: Y
    At what time will the data be published? [default: 0:00]: 0:00
    Does it include historical data [default: Y]? Y
    Enter the first event date of an historical data range: 2017-01-01
    Enter the last event date of the same range: 2017-12-31
    Do you want to add another historical data range? N
    Please unlock your wallet account to authorize signature.

    Signature authorized.
    New data source: Test successfully registered to publisher address: 0x627306090abab3a6e1400e9345bc60c78a8bef57

Publishing Historical Data
~~~~~~~~~~~~~~~~~~~~~~~~~~
To upload data in an registered data source, use the `catalyst data publish`
command.::

    $ catalyst data publish --data-source=Test --datadir=~/test-data/

Upon execution, Catalyst will automatically identify, validate and upload
the data in all CSV files immediately inside the specified `datadir`. It
will create a `published` subdirectory in which to move each file after
successfully published.

For illustration, here is our `test-data` directory before running the
`catalyst data publish` command.

.. image:: https://s3.amazonaws.com/enigmaco-docs/data-providers/marketplace-publish-folders-before.png
    :align: center
    :alt: Test Data Source Folder Before Publish

Here is the same directory after successfully publishing the data.

.. image:: https://s3.amazonaws.com/enigmaco-docs/data-providers/marketplace-publish-folders-after.png
    :align: center
    :alt: Test Data Source Folder After Publish


