Data Providers
==============

As a data publisher, you can publish your data sets to the Catalyst
Marketplace and get rewarded in ENG tokens. Providers set the price of their
data set. The Catalyst Marketplace brokers the transaction using
smart contract.

Data Sets
~~~~~~~~~
A data set may contain any data which fits in a tabular format.
However, since data will be consumed by Catalyst for the purpose of
algorithmic trading, it must have some predictable properties. In
particular, each data set must contain these two columns:

* Date: The date/time of the event corresponding to each row of the
  data set.
* Symbol: The symbol of the currency of market associated with each even.
  If an event involves more that one currency or market, multiple entries can
  be provided with a semicolon separator.

Here are the applicable data type conventions:

* String: String (or text) data must surrounded with double-quotes
* Date/Time: Both date and date/time types are supported. The date fields
  should be represented as String following the ISO 8601 format using the
  appropriate level of precision. Valid examples include:
    * "2017-11-01" or "2017-11-1" (for November 1st, 2017)
    * "2017-12-14 23:00" (for December 14th, 2017 at 11:00PM)
* Numbers: Numbers should be provided as integer or floats without quotes.
  Use as many decimal as necessary.

This sample data set contains market cap information:

.. image:: https://s3.amazonaws.com/enigmaco-docs/data-providers/csv-example-marketcap.png
    :align: center
    :alt: Marketcap Data set Sample CSV


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

Registering Data Sets
~~~~~~~~~~~~~~~~~~~~~
To register a new data set, download and install the Catalyst client.
Then, use the `catalyst register-data` command. In this example, data is
published multiple times per hour at a variable time::

    $ catalyst register-data
    Enter the Data set name: Test
    Enter the data frequency [daily, hourly, minute]: hourly
    Can data be published every hour at a regular time? [default: Y]: N
    At a minimum, how many times per hour will data be published? [default: 1]: 5
    Does it include historical data [default: Y]? N
    Please unlock your wallet account to authorize signature.

    Signature authorized.
    New data set: Test successfully registered to publisher address: 0x627306090abab3a6e1400e9345bc60c78a8bef57

In the following example, data is published daily on a fixed schedule::

    $ catalyst register-data
    Enter the Data set name: Test
    Enter the data frequency [daily, hourly, minute]: daily
    Can data be published every day at a regular time? [default: Y]: Y
    At what time will the data be published? [default: 0:00]: 0:00
    Does it include historical data [default: Y]? Y
    Enter the first event date of the range: 2017-01-01
    Enter the last event date of the same range: 2017-12-31
    Do you want to add another historical date range? N
    Please unlock your wallet account to authorize signature.

    Signature authorized.
    New data set: Test successfully registered to publisher address: 0x627306090abab3a6e1400e9345bc60c78a8bef57

Publishing Historical Data
~~~~~~~~~~~~~~~~~~~~~~~~~~
To upload data in an registered data set, use the `catalyst publish-data`
command::

    $ catalyst publish-data --data-source=Test --datadir=~/test-data/

Upon execution, Catalyst will automatically identify, validate and upload
the data in all CSV files directly inside the specified `datadir`. It will not
scan recursively. It will create a `published` subdirectory in which to move
each file when successfully published.

For illustration, here is our `test-data` directory before running the
`publish-data` command.

.. image:: https://s3.amazonaws.com/enigmaco-docs/data-providers/marketplace-publish-folders-before.png
    :align: center
    :alt: Test Data set Folder Before Publish

Here is the same directory after successfully publishing the data.

.. image:: https://s3.amazonaws.com/enigmaco-docs/data-providers/marketplace-publish-folders-after.png
    :align: center
    :alt: Test Data set Folder After Publish

The file naming convention is inconsequential, Catalyst will process any
file with a CSV extension. As long as the data is correctly represented, it
can be contained in one file or split across multiple files.

On error, Catalyst simply stops and display the error in the standard output.
It does not roll-back the files already published.

Publishing Live Data
~~~~~~~~~~~~~~~~~~~~
Publishing live data works similarly to publishing historical data
except that Catalyst will watch the `datadir` and try to publish new data in
new or modified CSV files. To publish live data, simply add a the `watch`
parameter to the 'publish-data` command::

    $ catalyst publish-data --datasource=Test --datadir=~/test-data/ --watch


