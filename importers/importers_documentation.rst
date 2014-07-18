LandPortal Importers Documentation
==================================

.. sectnum::
.. contents:: Index

General
-------
What is an importer?
^^^^^^^^^^^^^^^^^^^^
An importer is an independent module with the goal of tracking data from an organization/datasource and introducing them in the LandPortal DataBase. In order to do that, it has to download/request the info from the target source and convert it to a xml with a concrete format. Once it has that xml built, it has to send it to the Receiver module in LandPortal Architecture.

Restrictions:
 - Language: You can develop an importer in any language, there is no restriction about this. Anyway, it is recommended to build importers in python so you will have some support with modules/libraries developed in python to do some typical and common operations easily and efficiently.
 - IDs (identifiers): The produced xml contains a set of observations, slices and info about license use, datasource, organization, indicators, etc. Most of this entities, such as observations, must have an unique ID in the system, and it is the importer module which assigns the identifiers to them. So, there must be a defined strategy to generate the identifiers. More on this in Section "IDs assignation". If the importer produces repeated IDs in entities such as observations, the receiver will reject the xml. On the other hand, the importer needs to ensure that other entities such as indicator always have the same ID. 
 - Write access: All the developed importers need to write some intermediate files while executing, and probably the new ones will also need it. So, the importer must be executed with write permissions in its directory.
 - Executions modes: Every developed importer should have two execution modes: one for tracking every data of the source and other for tracking only those that had not been processed yet. 


Stages of execution
^^^^^^^^^^^^^^^^^^^

The main goal of the importer is to produce an xml with the data of the source and send it to the Receiver module. To reach that goal there are some typical stages. The code do not necessarily have to be divided in modules/packages thinking in this stages, but it is highly recommended to produce homogeneous and maintainable importers.

 - Downloading data: the first part is getting the data from the source. This data may come in several formats: xml, csv, xml, json, different request to an API or even through the appliance of scrapping techniques. It is preferable to separate this stage from the others. If the data from the source can be easily stored in a single local file (or few files), the code will result more maintainable. There will be cases where this can result difficult or unproductive (too many API request, scrap).
 - Parsing the data and converting it to intermediate objects: despite of the fact that the goal of the importer is to produce an xml file with the source's data, it is preferable to build plain model objects containing the data before, as an intermediate step. Generating objects such as Observation, Country, Slice, etc., makes it easier to understand and recognize them as xml nodes, the programs are easier to maintain and debug. Every python importers convert the source's data into a group of common entities. Those entities are transformed later into "xml" by a common module.
 - Getting country's ISO3: The observations in the xml should be related to a country/region. In the final xml, this area should be identified with an ISO3 for countries or a UN_CODE for regions. But the usual scenario is that we can't find those codes between the source's data. Some sources give us a short name, some others a long name, an unordered name, a name in other language, an ISO2, other type of code... i.e.: Paraguay could be identified of many forms, such as "Paraguay", "the Republic of Paraguay", "The Republic of Paraguay", "Paraguay, the Republic of", "FAOSTAT_CODE = 169",... The importer should have a mechanism to turn whatever the source uses to identify countries into an ISO3/UN_CODE, ensuring that no observation is lost or assigned to a wrong country. Python importers count on a module called "ContryReconciler" to do the work.
 - Generating XML and sending it to the Receiver: That's the final step every importer must achieve. At this point we should have a group of linked model objects that represents all the information contained in a dataset. Now, we have to translate it into xml and send them with a reference of the original source, to the ReceiverModule, which will incorporate to the system the new information. Python importers count on a module called "ModelToXml" to do all that work.


IDs assignation
^^^^^^^^^^^^^^

As it has been said, there are many entities within the domain that should have an unique ID. In this section we will enumerate them all as well as the way to form its id:

 - Organization id: Those ids are usually generated joining the string "ORG" with a little chain which should be representative of the organization. eg: WorldBank -> WB 

From now on, every id is formed using a string compound of three letters, the organization chain and an integer number that may be static or not.

 - Datasource id: Those ones are the result of putting together the string "SOU", the organization's representative chain and a given number, which should be statically assigned to every datasource, so it will not change between data imports.

 - Dataset id: Same as datasource, but in this case, the string used is "DAT" and the numeric part is not statically assigned, but calculated when the importer is working, based in previous executions.
 
 - Indicator id: In this case, the id will be formed using the same method as datasource, static numeric indicator included and using "IND" instead of "SOU".
 
 - Slice id: As well as dataset, this is not assigned statically and the string used is "SLI" instead of "DAT". 
 
 - Observation id: Same as slice, but using "OBS" as the representative string.
 
 
Executing importers
^^^^^^^^^^^^^^^^^^^

Until now, every importer has been developed in Python, which means executing it requires having Python (2.7) installed in the computer, as well as all the needed modules: CountryReconciler, ModelEntities and the ModelToXml (all of then are in the repository).

In order to make the importers work, every modules should be placed in the same folder (importers are configured to look for the external modules there). Alternatively, you can add the modules' directories path to the system variable "PythonPath". Finally you should use any terminal to place yourself on the root folder of the importer, and execute the "main.py" file located there.

Also every importer has a configuration ini file, with a "historical mode" parameter, located generally under a "Translator" section, this parameter reflects the mode in which the importer will be executed. Being True means the importer will search for every available data without concerning about dates, whereas if False only data dated after the "historical year" parameter will be tracked.
If you are interested in the other sections or parameters, please have a look at this section "Configuration files".


Importers in python
^^^^^^^^^^^^^^^^^^^
Until now, every importer has been developed in Python, following a similar schema:

 - /data: If needed, this folder contains the files from which the data are extracted (xml, csv, json, ...)
 - /es/weso/: Is the source code folder, inside there will be different packages depending on the importer, usually here will be placed the importer class itself, and some utility classes like file readers, api callers, etc.
 - /files: Contains the Python ini files, that will be explained forward.
 
As it was said in the previous section, in order to execute the importers, there is a "main.py" file, located in the root directory of the module that will make the magic happens, this files are always similar (completely equals in some importers) and run the next actions:

 1. Configure log file 
 2. Load and read ini file
 3. Run the importer class
 4. Update ini file

Having understood this, now a little explanation of the utilities modules and configuration files structure will continue. This modules are completly optional, but is highly recommended to give them a try if you are developing a Python importer

Configuration files
"""""""""""""""""""
In general, every information that is not directly downloaded from the sources in each execution but has to be produced by the importer, is stored in the configuration file. i.e.:
 - Name and description of the organization.
 - All the indicators information, in several languages.
 - Type of license.
 - Etc.

Each piece of information is related to a section with the name of an entity of the model. Example: every data referred to indicators (name, translations,...) will be placed in a section "INDICATOR" of the configuration.ini file. The same with LICENSE, DATASET, DATASOURCE, ORGANIZATION and USER.

This file also contains some specific fields that some concrete importers need to do their task, such as an url, an API pattern or a relative path calculated from the root directory of the importer. However, these fields are not always used and are different for each importer module, so they will be mentioned in other sections.

This is an example_ of a real config file for UNDP importer.

.. _example: https://github.com/weso/landportal-importers/blob/master/UNDPExtractor/files/configuration.ini


Land portal entities
""""""""""""""""""""
This package contains representations of the different entities in the model, plain objects with all the needed attributes and few or none logic. Also, some of them has default values assigned to match default entities.

In most cases, every parameter that the "__init__" accept is needed to build a complete object, despite of "None" is specified as default value. This have been done only for not force the programmers to pass every argument when building the object. Most of the times it is easier to assign the parameter after calling the init, because of the internal complexity of some of the objects passed.

Country reconciler
""""""""""""""""""
As it was said before, countries are represented in several ways. This module, relying in an excel file which contains the countries and every possible representation, will build the country object based in the entities module from whatever value you provide (name, iso2, iso3, ...).

It has a public interface with methods called getCountryByXXX, where XXX can be iso3, iso2, en_sname, faostat_code... All the methods return a Country object as it is defined in the model, and that object is always the same for a concrete instance of CountryReconciler. Example: if you invoke many times getCountryByIso2("ES") and one time getCountryByIso3("ESP"), the calls will always return the same instance, that represents "Spain".

ModelToXML
""""""""""
The most important action of an importer is building the XML file that will be sent to the receiver. This module, relying into the Entities module, will parse all information into a valid XML file.

Its init method may be a bit complex. We will explain the meaning and expected content of its params:
 - dataset: object dataset of the model. Through it, all the rest of the objects needed to represent the source's info are reachable.
 - import_process: string that represent the original format of the data: csv, excell, api... the module has a set of public constants: XML, XLS, CSV, API, JSON, SCRAP. You should use one of them.
 - user: user object built while parsing the original source. You have to pass it because is no reachable from dataset.
 - path_to_original_file: several options depending on the nature of the source:

     + A local path to a single file form where the data was parsed.
     + A list of local files from where the data was parsed.
     + The base URL of an API, if no local file was stored.
     + The base URL of a web page, if scrapping techniques were used.

 - indicator_relations: list of objects IndicatorRelation, as they are represented in the model. It is really uncommon to have indicator relations so, by default, this param is None.
 
The schema describing the xml's structure could be accesed here_, and this xml_ would be an example of a final xml produced by the IFPRI importer.
.. _here: https://raw.githubusercontent.com/weso/landportalDoc/gh-pages/interfaces/xml/sample.xml
.. _schema describing the xml's structure: https://github.com/weso/landportalDoc/blob/gh-pages/interfaces/xml/landportalDataset.xsd

FAO Agricultural Censues
""""""""""""""""""""""""
FAO organization has four different importers, depending on where we have find the data. This one in particular handles three different excel files, which represents the same indicators through time. Every excel file has a different format which made impossible the development of an homogeneous importer, driving to this situation:

In the configuration.ini file there are the next properties:
 - file_names: Points what files there have to been parser, separated by commas.
 - data_range_rows_file_name: Indicates the valid rows of the excel file separated by '-'.
 - data_range_cols_file_name: Indicated the valid cols of the excel file separated by '-'.
 
Those properties are needed because the excel files contain a lot of metadata, which is useless for the importer.

The excel reader, as many others importers, load a data matrix with the required cells of the file, making it easier to the importer to work with them.

Due to the differences between files, all of them has a customize method in which are specified the columns for every indicator, but at the end all this methods rely in a last one that is the same for every file.

FAO Food Security
"""""""""""""""""
This particular one, relies in a single excel file, but with lot of sheets. That is why every indicator has a sheet name assigned in the config file.

It works as the previous one, loading all the data in a matrix, but with the difference of having only one method to parse every possible sheet, as all of them have the same format.

FAO Gender
""""""""""
We can download data through a REST API that gives back data in XML format, but this importer can be really troublestone because of the quality of this information: It is poor and it looks quite unstable. It is highly possible that the way in which data is presented could change in the future.

Despite of the xml is valid and well-formed, all the useful info (date, observation, indicator) is given in a single node mixed with HTML tags, sometimes encoded, sometimes not. Also, depending on the consulted country in each petition, the result cames in different languages (it usually comes in the official language of the consulted country).

For instance, if you put this URL in your browser, you will obtain data for Spain: http://data.fao.org/developers/api/landrights/xml-country?topic=sls&version=1.0&country=ESP

As you can see, all the results are written in Spanish, and there are a lot of useless characters referred to style, html, etc.

Currently, the importer can manage all this to produce clean info, but several things had to be beard in mind:

* If several languages are available, there will be more than one <lang> node under <country> node. We don't have to care about language, just take a random one (the first). In <subtopic>, the attribute "code" is constant across different languages, and the node.text that we have to parse does not contain meaningful words.
* The content in the indicator subnode "GCI" is not coherent with the rest of the <subtopic> nodes: dates, numbers, special elements,.... but it is not a problem. Only four indicator are requested, and GCI is not one of them.
* It looks that there are two ways to say "No data available" inside a subtopic. Containing the text "N/A" or containing the text "N.D.".
* Dates are placed between "[]". It looks that there can be single years or intervals. i.e.: [2009] or [2008-2010].
* The concrete data could be hard to parse, because: 

 - It is mixed with everything: estrange chars, dates, ranks, html notation...
 - When numbers are bigger than 999, they try to make a graphical separation with blank spaces between every 3 digits. i.e.: 111 222 333 means the number 111222333.

 So, to parse it we will have to:
  + Remove text between "[]" (dates)
  + Remove text between "()" (rank)
  + Remove text between "<>" (html notations)
  + Remove text between "&" and ";" (special html chars)
  + After all these things, remove every white space in the resulting chain.

 Following that steps we should obtain a string parseable to a number that represents the observation value.

* If we send a petition for a country non-stored we will obtain an xml such as the next:

 <country iso3="SOMECOUNTRY" name="">No records found</country>

* 3 ways to identify it, that looks constant:

  - empty attribute name
  - only a node, not children
  - node.text = No records found.

 Probably the safest option to determine if we have data or not, and maybe even the fastest, is the second one. Whit no children there is no info, not mattering the rest of the content. And thinking in computing terms, we would only have to check the existence or not of children. Similar to check if name is empty, but better that checking the third option.

The way to filter old data is specifying the first valid year in the configuration.ini file, with the value of the property "first_valid_year". Anyway, the importer will deduce that value in every execution, to be prepared for the next time it has to be executed.  

FAO Stat
""""""""
The way in which importer works is:

* It downloads a huge CSV file containing all the info available of Faostat database related to "Resources Land".
* It converts each line of the csv into an intermediate object that represents it.
* It filters this list of objects, removing all those that:

 - They contain data of indicators that had not been requested.
 - They refers to countries or regions that are not in the official list of countries.
 - They contain observations that has been already incorporated to the system (when the importer is not executing in historical mode).

* It turns that intermediate objects in objects of the common model.

It looks that the file that contains the entire database has a name that does not depend on dates, so it could be possible that in the next time that the importer need to execute the Download url may not change. If it does, the new URL must be specified in "zip_url", in the configuration.ini file. 

The importer expects an URL pointing to a zip that contains a single CSV file.

You may notice that the log of this importers produces many warnings. In general, it is not something to worry about: it is the normal behavior of the importer. The algorithm downloads the entire database of Fao Stat and filters unneeded data. In that process, many observations referred to unsupported regions are detected as "unknown countries" and registered in the log as it. But that observations should not be present in the final system.

Foncier
"""""""

The importer obtains the data through a REST API that gives back XML content. The content is easily parseable, and the API has a coherent pattern. This importer must be manually configured in order to know the years to query. 

This is the pattern to make a petition to the API: "http://www.observatoire-foncier.mg/xml-api.php?year={YEAR}&month={MONTH}"

The importer will give to MONTH values between 1 and 12 and to YEAR values between "first_year" and "last_year", and will send a request with every combination. Those values can be set in the configuration.ini file.

The param "last_checked_year" in hte same file is used for executing in non historical mode: Only the observations with a date higher than this value will be taken into account. This value can be manually configured, but the importer will deduce it in every execution.

IFPRI
"""""

This importer downloads several xls files and parse it. The problem is that the URL pattern to request these files is coherent, but the internal format of the files is not.

The changes between the files looks minimal, but they are really troublesome to produce an unified parsing algorithm. Main reasons:

 * Before the data sometimes there are comments, sometimes not.
 * An observation sometimes is contained in a single column, sometimes in two.
 * Indicator's names have different names across the different files.
 * Indicators' data has different width (different number of columns) across different files.
 * Sometimes white columns between indicators appear, sometimes not.
 * There are non numeric values referring to a numeric indeterminate quantity ("<5").

The strategy followed in this case is not trivial. The importer parse the files making as less assumptions as possible, in order to be ready to manage future changes, and turns the xsl data into an intermediate objects. Then, it transforms these objects into ones of the common model.

However, if this tendency continues in future updates, it could happen that the importer could not manage the new xls format. In this case, it will be necessary to produce a new algorithm or to adapt the current one.

The pattern of the available files to download can be found in the config file and its name and value are: url_pattern = http://www.ifpri.org/sites/default/files/ghi{year}datam.xlsx

Currently, only 2012 and 2013 are available. In case of new datasets, the filed "available_years" in the config file should be modified, adding the new year with a comma, in order to substitute {year} in the url pattern by this new value.

Example: the current "available_years" value is:

available_years = 2012,2013

If a new dataset of 2014 is published, then we should actualize "available_years" as follows:

available_years = 2012,2013,2014


LandMatrix
""""""""""
This importer downloads the entire landmatrix database in xml format, but the observations have a complex nature, hard to fit in the general model. A node of information in the landmatrix does not refer to a concrete country, with a concrete indicator and a single value in a concrete date. The indicators built are aggregations of values, such as "total number of deals in some sector".
The python project of the importer includes a file "strategy.txt" where the indicators are detailed.

The date assigned to each observation is the highest date found when parsing a value that belongs to its aggregate. For this reason, when executing again the importer to incorporate new data, it may be something to consider removing from the system all the old observation and run the program in historical mode.

The importer is, as all of the importers, prepared to ignore data with a date lower than the value specified in the "first_valid_year" filed (that can be manually set or automatically calculated by the importer) of the configuration.ini file. However, doing this, we will get observations of data aggregates between the specified date and the current one, menawhile the old observations are aggregates of every available date. The meaning will not be consistent. By removing all the old observations we will obtain new ones that are aggregates of the old and the new values.

OECD
""""
This source may have problems to incorporate new data or may not: it entirely depends on the criteria when offering new data of its owners. Anyway, it looks it will be problematic. Currently, all the data are tracked in json format, but not all the data is stored in the same dataset. Data referred to 2009 or older, is stored in a a database. Newer data is stored in a different one.
So, to download it, despite of there are a powerful API, you have to do petitions to different databases using this API.

The databases are represented by an identifier that you should use when making the request, but that identifier is not built with a pattern: the old database's ID is GID2, meanwhile the one of the new database is GIDDB2012. So there is no way to predict the name of the new database, and it has to be set manually.

However, this is a minor problem. This importer retrieve data in json format, but the most annoying thing about it is that it does it with a different internal structure across the different databases. The algorithm of the importer is a bit complex because it must reconcile all this changes between datasets to treat it in a homogeneous ways.

To incorporate new data, first of all, a new general query with must be added after a character "|" in the field "querys" of the configuration file, situated in "es/weso/oecdextractor/configutation/data_sources.ini". It will probably look like the other queries that you can find there, but code identification and database id could change. If the structure of the new database does not change compared with GID2, GIDDB2012, the importer should work. In any other case, the algorithm should be refined.


RAW
"""
This is a special importer, cause it hasn't been developed for any particular organization.

This one handles a default excel file, that can be fulfilled by the user, with the needed data. Every indicator that wants to be imported must be in a different file and placed under the data folder.

In the configuration.ini file, it should appear the organizations represented by the files (may be more than one) as properties under the section [ORGANIZATIONS], and as value the different file names, separated by commas.

The importer will generate an organization object for every property placed under the [ORGANIZATIONS] section and will load them with the indicators specified in the files. As in any time, more files could be added for the same organization, the importer also generated a custom ini file for them, saving the indicator id, the generated datasets, and the number of generated observations for this particular organization.

Transparency International
""""""""""""""""""""""""""
The same case that with the FAO Agricultural Censues importer, with the difference that this one is adapted to the new model in which every indicator represents a section under the ini file.

Like the other importer, there is a need to know the valid range of rows and columns for data extraction, but in this case, every indicator has its own. Also as there are more than one sheet in the files, it's necessary to know the sheet from which the data will be extracted. Again, as the files are totally different is needed more than one function to transform the data into observations, based on the rows-columns where the data is presented.

UNDP
""""
The most particular thing with this importer is that it was thought to be able to download and transform several sections of UNDP data. The source is organized in topics, such as "Health", "Education" or "Human Development Index Trends". The last one is the only one that has been requested so, at the end, the importer only process this dataset. Anyway, the algorithm was thought to treat more than one dataset in an homogeneous way, downloading it in xml format.

If case you want to amplify this importer to track the rest of the data, most of the work will be done by uncomment a line in the file "files/undp_table_names". In that file all the name of the available datasets and the URL to download it in xml are stored.
Each line of that archive is expected to be formed by a table name, a '\t' char and a URL to download data in xml format. Lines that start with "#" are ignored. Most of the work to incorporate a new dataset will be done just uncommenting (removing "#") the line that contains the target information. There would also be needed to incorporate info about the new indicators in the "files/configuration.ini" and code in the application to build an indicator object with this new data.

If the source is actualized with new data, the importer may not need any changes to work, but it entirely depends on if the owners decide to change the url where the data is available or not. If the url (and the structure of the xml) does not change, the importer will work and will be able to distinguish between old and new data.

In the configuration.ini file the field "first_valid_year" indicates the earliest date in which data will be taken in account when executing in non historical mode. This value can be set manually, but the importer automatically calculates it after every execution.



World Health Organization
"""""""""""""""""""""""""
In this case, the World Health organization provides a lot of ways to download its data (csv, excel, etc.) and in different formats (codes, text and both). We are using the verbose ones, which provides both text and codes, so it makes easier to add new indicators.

As those files are downloaded from an endpoint and that endpoint is different for every indicator, an indicator endpoint must be passed through the configuration file and transformed into an URL. So, we will find the next fields in the configuration.ini file:

 - URL pattern: It is used to compound the URL with the indicator endpoint values provided.
 - Indicator: It is a code used by the WHO to identify its indicators.
 - Profile: Points the mode in which the file will be downloaded (empty for code, 'text' for text and 'verbose' for both of them).
 - Countries: It may be 'COUNTRY:\*' which means all countries are requested or a list of countries with the format 'COUNTRY:XXX;COUNTRY:YYY' being XXX and YYY the ISO3 codes of the countries.
 - Regions: Same as country but with 'REGION:' instead of 'COUNTRY:'

Once the endpoint is compound, the file is downloaded with name given in the indicator section and the data is extracted from it by the importer.

World Bank
""""""""""
This is the slowest and the biggest of the importers. It works directly with the WorldBank API, which requires as parameters the indicator and the country you are looking for, which means that for every indicator there have to be done an equivalent to the number of countries calls (in our case 256), multiplying it by the number of indicators (33) and you will have a lot of calls to a free API. On the other hand, it makes quite easy to add a new indicator to parse, the only thing that is need to be done is to add the new indicator to an existing datasource, or, in the case the indicator doesn't belong to the existing ones, add a new datasource under the corresponding section.

Within the configuration.ini file there are two URLs, one of them is used to retrieve a list of 256 countries from the WorldBank API, that will be used then to compound the second URL, which needs both the country code and an indicator ID (those ids are WorldBank custom and need to be consulted in the web).

What this importer does is:

 1. Generates a country list relying in the Country reconciler and the iso codes obtained from the WorldBank API.
 2. Loads all the datasources specified in the ini file.
 3. For every datasource loads all the indicators under the corresponding section.
 4. Makes a call to the API compounding the url with the country code and the indicator code and transform the response to several observations (one for every year contained in the response).

Importers in any other language
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
There hasn't been developed importers in any other language than Python, but it's possible. The mayor drawback you will have is to adapt the utilities modules provide in Python to the new language, or ensuring that you implement an algorithm that does the same work without using them. The restrictions you have to bear in mind are:

 * You must ensure that there are not mistakes when you associate an observation with a country: no losses of observations and no observations with wrong countries. Those countries must be identified in the final xml with their official iso3 code (or un_code for regions).
 * You must manage ids properly: 
  - A concrete indicator should always have the same id in different executions. The same for datasources.
  - The id of indicators, datasets, observations, etc, should contain a reference to its organization. See "ID Assignation".
  - There can not be two entities with the same id, nor an entity with different ids in different executions.
 * Your final xml should fit in the xsd schema provided in this repository (interfaces/xml/landportalDataset.xsd)
 * If you introduce of a datasource that belongs to an organization already present in the system, you should use the same info (name, descriptions, image) that is being used in the rest of the importers.
 * The Receiver module expects a request with the next fields:
  - xml: xml content with all the info.
  - file: content of the original local file of which the information was taken, or base URL of the API/web page consulted.
  - api_key: security API KEY to send a request. Currently, the receiver uses the one associated with an special user. Other importers could use a different key.


Implementation advices
^^^^^^^^^^^^^^^^^^^^^^
If you are reading this section, then you have in mind to develop your own importer. Before you do that consider to use the RAW importer, cause the only action you will need to do is fulfill an excel file. If you feel like it has no use fulfilling the file, here are some advices that will help your path during the development process.

 - Try to develop in an object oriented methodology, the domain of the data is really huge, and having some objects to rely in will be useful.
 - In case you are using different files, try to make them as similar as possible. You don't want to end developing an importer with a function to read and parse every file differently.
 - If you are considering calling an API, take into account the time it will take to the importer to retrieve all the data (sometimes it's easier to locate and download the files from whom the API extracts the data). 
 - Sometimes you will find a data source in a web page, but you won't be able to locate the data, API, etc. which can lead you to think about scraping the web... it's possible, but not recommended (if you really want to use data from a source you should contact with the providers).
