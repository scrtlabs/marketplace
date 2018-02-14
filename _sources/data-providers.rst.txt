Data Providers
==============

As a data provider, you can publish your data sets to the Enigma Data
Marketplace and get rewarded in ENG tokens. As a provider, you set the price of 
your data sets for monthly subscriptions. The Enigma Data Marketplace is 
governed by a :doc:`smart contract<smart-contract>` that brokers all the 
transactions in the marketplace.


Data Sets
~~~~~~~~~
A data set may contain any data which fits in a tabular format.
However, since data will be consumed by Catalyst for the purpose of
algorithmic trading, it must have some predictable properties. All data sets 
must be structured as time series, which must contain these two columns:

* Date: The date/time of the event corresponding to each row of the
  data set.
* Symbol: The symbol of the currency of the market associated with each event.
  If an event involves more that one currency or market, multiple entries can
  be provided with a semicolon separator.

Here are the applicable data type conventions:

* String: String (or text) data must surrounded with double-quotes
* Date/Time: Both date and date/time types are supported. The date fields
  should be represented as a *String* following the *ISO 8601* format using the
  appropriate level of precision. Valid examples include:

    * "2017-11-01" or "2017-11-1" (for November 1st, 2017)
    * "2017-12-14 23:00" (for December 14th, 2017 at 11:00PM)

* Numbers: Numbers should be provided as integer or floats without quotes.
  Use as many decimals as necessary.

This sample data set contains market cap information:

.. raw:: html

    <table border="1" cellpadding="2" cellspacing="1" width=656 style='table-layout:fixed;width:492pt; font-size:0.85em'>
     <tr height='21' style='font-weight:bold'>
      <td style='width:180pt' bgcolor='#cccccc' align='center'>date</td>
      <td style='width:55pt'  bgcolor='#cccccc' align='center'>symbol</td>
      <td style='width:100pt' bgcolor='#cccccc' align='center'>day_volume_usd</td>
      <td style='width:100pt' bgcolor='#cccccc' align='center'>available_supply</td>
      <td style='width:100pt' bgcolor='#cccccc' align='center'>market_cap_usd</td>
     </tr>
     <tr height=21 style='height:16.0pt'>
      <td height=21 style='height:16.0pt'>2018-01-22 00:42:00+00:00</td>
      <td>btc</td>
      <td align=right>10010200064</td>
      <td align=right>16817962</td>
      <td align=right>197321785344</td>
     </tr>
     <tr height=21 style='height:16.0pt'>
      <td height=21 style='height:16.0pt'>2018-01-22 00:42:00+00:00</td>
      <td>eth</td>
      <td align=right>3380800000</td>
      <td align=right>97128896</td>
      <td align=right>103240245248</td>
     </tr>
     <tr height=21 style='height:16.0pt'>
      <td height=21 style='height:16.0pt'>2018-01-22 00:42:00+00:00</td>
      <td>xrp</td>
      <td align=right>2935330048</td>
      <td align=right>38739144704</td>
      <td align=right>54674878464</td>
     </tr>
     <tr height=21 style='height:16.0pt'>
      <td height=21 style='height:16.0pt'>2018-01-22 00:42:00+00:00</td>
      <td>bch</td>
      <td align=right>737806976</td>
      <td align=right>16924476</td>
      <td align=right>30479964160</td>
     </tr>
     <tr height=21 style='height:16.0pt'>
      <td height=21 style='height:16.0pt'>2018-01-22 00:42:00+00:00</td>
      <td>ada</td>
      <td align=right>822998976</td>
      <td align=right>25927069696</td>
      <td align=right>15821813760</td>
     </tr>
     <tr height=21 style='height:16.0pt'>
      <td height=21 style='height:16.0pt'>2018-01-22 00:42:00+00:00</td>
      <td>ltc</td>
      <td align=right>465356000</td>
      <td align=right>54861960</td>
      <td align=right>10571514880</td>
     </tr>
     <tr height=21 style='height:16.0pt'>
      <td height=21 style='height:16.0pt'>2018-01-22 00:42:00+00:00</td>
      <td>xem</td>
      <td align=right>99484704</td>
      <td align=right>8999999488</td>
      <td align=right>9715949568</td>
     </tr>
     <tr height=21 style='height:16.0pt'>
      <td height=21 style='height:16.0pt'>2018-01-22 00:42:00+00:00</td>
      <td>neo</td>
      <td align=right>355087008</td>
      <td align=right>65000000</td>
      <td align=right>8795799552</td>
     </tr>
    </table>

|
|

Data Frequency and Availability
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Data frequency and availability associated fields must be provided when 
registering each data source, as per the scheme outlined below:

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
Then, use the ``catalyst marketplace register`` command. In this example, data is
published multiple times per hour at a variable time:

.. code-block:: bash

    $ catalyst marketplace register
    Enter the name of the dataset to register: test
    Enter the price for a monthly subscription to this dataset in ENG: 10
    Enter the data frequency [daily, hourly, minute]: daily
    Does it include historical data? [default: Y]: 
    Doest it include live data? [default: Y]: 

Publishing Historical Data
~~~~~~~~~~~~~~~~~~~~~~~~~~
To upload data in an registered data set, use the `catalyst publish-data`
command:

.. code-block:: bash

    $ catalyst marketplace publish --dataset=test --datadir=~/test-data/

Upon execution, Catalyst will automatically identify, validate and upload
the data in all CSV files directly inside the specified `datadir`. It will not
scan recursively. 

The file naming convention is inconsequential; Catalyst will process any
file with a CSV extension. As long as the data is correctly represented, it
can be contained in one file or split across multiple files.

On error, Catalyst simply stops and displays the error in the standard output.
It does not roll-back the files already published.

Publishing Live Data
~~~~~~~~~~~~~~~~~~~~
Publishing live data works similarly to publishing historical data
except that Catalyst will watch the `datadir` and try to publish new data in
new or modified CSV files. To publish live data, simply add a the `watch`
parameter to the 'publish-data` command:

.. code-block:: bash

    $ catalyst marketplace publish --dataset=test --datadir=~/test-data/ --watch


