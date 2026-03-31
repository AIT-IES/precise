# The PreCISE Common Data Format

The following rules should be followed for constructing a data file:

1. Data and metadata are combined in a zip archive.
2. The zip archive contains an XML file named `manifest.xml` which provides information and metadata about all data contained in the zip file.
3. Data (tabular as well as key‐value data) is stored in CSV files which are likewise contained in the zip archive.
   Data files may have arbitrary names (except for the file extension which must be .csv).
   No convention is defined for the distribution of data between files; i.e. a two‐dimensional table with 10 columns may be stored as one file, five files of two columns each etc.
   The manifest file is used to describe the data in each file.
4. Each CSV file must follow the format described in RFC 4180 [1], see also the appendix, with the following characteristics:
   + The header line (RFC 4180 section 2 rule 3) is mandatory.
   + String data must always be enclosed in double quotes (RFC 4180 section 2 rules 5 and 6) in order to proactively prevent a number of common conversion problems.
   + Strings containing double quotes must be escaped as described in RFC 4180 section 2 rule 7.
5. The use of language‐specific and diacritical characters (Ü, æ, á etc.) is a common source of conversion issues and should be avoided as far as possible in data files and manifest.
   Ideally, these should be transcribed to the English alphabet in a consistent manner. (Example: Århusgade ‐> Aarhusgade, Köstendorf ‐> Koestendorf etc.)
6. Regardless of local conventions, a dot (“.”) must be used as a decimal point separator in order to avoid confusion with the comma used as a field separator.
   (Example: one half is 0.5 instead of 0,5).
7. Missing or invalid data (NaN, blank fields etc.) are to be represented as empty fields in the CSV files, i.e. without a space between the separators. (Example: 1,2,,,,6).
8. All timestamps must be provided in the UTC time zone to avoid discontinuities caused by daylight saving, geographical distance etc.
   They must be written either in POSIXlt (epoch time) or POSIXct (calendar time) format:

   + Milliseconds elapsed since 01/01/1970, 00:00:00.000 i.e. the Unix Epoch.
     Seconds since the Epoch are supported by virtually all programming environments, including Matlab.
     The conversion to milliseconds is trivial; some programming languages (e.g. Java) already represent Epoch time in milliseconds.
     Example: Dec 1st, 2017, 13:00:00.000 central European time corresponds to an Epoch time of 1512129600000.
   + The Internet Time format as specified by RFC 3339 [2] and ISO 8601 [3].
     Note that RFC 3339 allows the capital “T” separating date and time to be replaced with a space (“ “) to improve readability.
     Example: Dec 1st, 2017, 13:00:00.000 is written as “2017‐12‐01 12:00:00.000” (note the 1 hour timezone difference between CET and UTC).

## Manifest file

The XML manifest file should have the following overall structure:

``` XML
<?xml version='1.0' encoding='UTF-8'?>
<dataset name="..." author="...">
<description>
[...]
</description>
<timeseries name="..." file="..." time-column="1">
[...]
</timeseries>
<table name="..." file="...">
[...]
</table>
<keyvalue name="..." file="...">
</keyvalue>
</dataset>
```

Exactly one `<dataset>` tag is required at the highest level of the XML tree.
The general concept assumes that only related data is distributed in the same zip archive; multiple zip archives should be created for multiple unrelated data sets.

At the second level, `<description>`, `<timeseries>`, `<table>` and `<keyvalue>` sections are possible.
Exactly one `<description>` tag is mandatory.
It encloses text which describes the origin and purpose of the entire dataset in a human‐readable form.
The three other tags enclose descriptions of different forms of tabular data:

+ A `<table>` is a generic two‐dimensional dataset in row/column form.
  (One‐dimensional datasets are expressed as two‐dimensional sets with a single column).
+ A `<timeseries>` is a special kind of table in which one column carries timestamps to which all column entries in the same row refer.
+ A `<keyvalue>` (key‐value set) is another special kind of table with exactly two columns.
  In each row, the entry in the first column is a key, and the entry in the second column is the corresponding value.
  This is a useful storage format for e.g. parameter lists.

The XML sections for the three tabular types are discussed in the following.

### \<table\> section

These sections have the following general form:

``` XML
<table name="..." file="...">
<description>
[...]
</description>
<column name="..." column="..." datatype="...">
<unit>[...]</unit>
<abs-accuracy>[...]</abs-accuracy>
<rel-accuracy>[...]</rel-accuracy>
<rawdata>[...]</rawdata>
<source>[...]</source>
<description>
[...]
</description>
<min-value>[...]</min-value>
<max-value>[...]</max-value>
</column>
<column name="..." column="..." datatype="...">
<description>
[...]
</description>
<enum-keys>
<key name="..." value="..."/>
</enum-keys>
</column>
</table>
```

Exactly one `<description>` tag is mandatory for the outer level `<table>` tag; however, the `<description>` tags for the individual columns are optional.
Each <table> may contain an unlimited number of `<column>` entries; however, all columns have to be found in the same CSV file (file attribute).

All subtags under each `<column>` tag are optional (unit, description, min‐value, max‐value, enum‐keys, source, rawdata, abs‐accuracy, and rel‐accuracy) while the column attributes are mandatory:
Each column must be given a name, the (1‐based) column index in the CSV file must be provided and the data type of the column must be indicated.
Permitted data types are integers (*int*), floating point numbers (*float*), boolean values (*bool*), enumerated values (*enum*) or general strings (*string*).
For enumerated values, an `<enum‐keys>` section should be provided while integers and floating point numbers should provide a unit and the valid range of values (min‐value/max‐value) to allow cross‐checks.

Additional information about the origin of the data may be added using the `<source>` tag.
`<rawdata>` is an optional boolean flag which indicates whether the data has been postprocessed (*no*) or originates directly from a measurement (*yes*).
If the relative and/or absolute accuracy of a measurement is known, the `<rel‐accuracy>` (in p.u.) and/or `<abs‐accuracy>` (in the same unit as the data) tags can be used.

### \<timeseries\> section

These sections are similar to the `<table>` form:

``` XML
<timeseries name="..." file="..." time-column="...">
<description>
[...]
</description>
<isochronous>[...]</isochronous>
<resolution>[...]</resolution> <!--Resolution in seconds between samples -->
<column name="..." column="..." datatype="...">
<unit>[...]</unit>
<abs-accuracy>[...]</abs-accuracy>
<rel-accuracy>[...]</rel-accuracy>
<rawdata>[...]</rawdata>
<source>[...]</source>
<description>
[...]
</description>
<min-value>[...]</min-value>
<max-value>[...]</max-value>
</column>
<column name="..." column="..." datatype="...">
<enum-keys>
<key name="..." value="..."/>
</enum-keys>
<description>
[...]
</description>
</column>
</timeseries>
```

The main difference is the mandatory <time‐column> attribute at the outer level which specifies the (1‐based) column index in the CSV file.
One level below, two additional mandatory tags must be provided:
The boolean `<isochronous>` tag states whether the table rows are equidistant in time.
If the data is isochronous, `<resolution>` specifies the time delta between rows.
`<resolution>` is to be omitted for non‐isochronous data.

### \<keyvalue\> section

These sections are very simple due to the fixed format of key‐value data:

``` XML
<keyvalue name="..." file="...">
</keyvalue>
```

## Example manifest file

``` XML
<?xml version='1.0' encoding='UTF-8'?>
<dataset name="testdata" author="Oliver Gehrke, DTU">
<description>
This is a test data set to illustrate the structure of the manifest.xml file.
</description>
<timeseries name="Weather data" file="weather.csv" time-column="1">
<description>
This is a timeseries of weather data.
</description>
<isochronous>yes</isochronous>
<resolution>5</resolution> <!--Resolution in seconds between samples -->
<column name="windspeed" column="2" datatype="float">
<unit>m/s</unit>
<abs-accuracy>0.2</abs-accuracy>
<rel-accuracy>0.015</rel-accuracy>
<rawdata>no</rawdata>
<source>Meteorology mast, DTU Risoe campus</source>
<description>
Windspeed measured by cup anemometer 10m over terrain
</description>
<min-value>0</min-value>
<max-value>100</max-value>
</column>
<column name="validity" column="3" datatype="enum">
<enum-keys>
<key name="unknown" value="0"/>
<key name="valid" value="1"/>
<key name="invalid" value="2"/>
</enum-keys>
<description>
Validity of the measured data, based on the self-diagnosis of the data acquisition system
</description>
</column>
</timeseries>
<table name="Reactive power characteristics" file="q_1.csv">
<description>
This is a P-vs-Q mapping table of the reactive power characteristics of a wind turbine.
Intermediate values must be interpolated linearly between the nearest known points.
</description>
<column name="P" column="1" datatype="float">
<unit>kW</unit>
<description>
Active power measured at terminals, generator counting system
</description>
<min-value>-2</min-value>
<max-value>15</max-value>
</column>
<column name="Q" column="2" datatype="float">
<unit>kVAr</unit>
<description>
Reactive power measured at terminals, generator counting system
</description>
<min-value>-10</min-value>
<max-value>10</max-value>
</column>
</table>
<keyvalue name="gridcode" file="gridcode.csv">
</keyvalue>
</dataset>
```

## References

<table>
<colgroup>
<col style="width: 3%" />
<col style="width: 96%" />
</colgroup>
<tbody>
<tr>
<td>[1]</td>
<td>Y. Shafranovic, “RFC 4180: Common Format and MIME Type for Comma‐Separated Values (CSV) File,” Internet Engineering Task Force (IETF), 2005.</td>
</tr>
<tr>
<td>[2]</td>
<td>G. K. a. C. Newman, “RFC 3339: Date and Time on the Internet: Timestamps,” Internet Engineering Task Force (IETF), 2002.</td>
</tr>
<tr>
<td>[3]</td>
<td>ISO, “ISO 8601:2000: Representation of dates and times,” International Organization for Standardization (ISO), 2000.</td>
</tr>
</tbody>
</table>


## Funding acknowledgement

<img alt="European Flag" src="https://upload.wikimedia.org/wikipedia/commons/thumb/b/b7/Flag_of_Europe.svg/330px-Flag_of_Europe.svg.png" align="left" style="margin-right: 10px" height="57"/> This development has been supported by the [SmILES] project of the European Union’s research and innovation programme Horizon 2020 under the grant agreement ID 730936.

[SmILES]: https://cordis.europa.eu/project/id/730936
