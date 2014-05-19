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
 - Language: You can develop an importer in any language, there is no restriction about this. Anyway, it is recommended to built importers in python so you will have some support with modules/libraries developed in python to do some typical and common operations easily and efficiently.
 - IDs (identifiers): The produced xml contains a set of observations, slices and info about license use, datasource, organization, indicators, etc. Most of this entities, such as observations, must have an unique ID in the system, and it is the importer module which asigns the identifiers to them. So, there must be a defined strategy to generate the indentifiers. More on this in Section "IDs asignation". If the importer produces repeated IDs in entities such as observations, the receiver will reject the xml. On the other hand, the importer needs to ensure that other entities such as indicator always have the same ID. 
 - Write acces: All the developed importers need to write some intermediate files while executing, and chances are that the new ones will also need it. So, the importer must be executed with write acces in its directory.
 - Executions modes: Every developed importer should have two execution modes: one for tracking every data of the source and other for tracking only those that had not been processed yet. 


Stages of execution
^^^^^^^^^^^^^^^^^^^

The main goal of the importer is to produce an xml with the data of the source and send it to the Receiver. To reach that goal there are some typical stages. The code do not necessarily have to be divided in modules/packages thinking in this stages, but it is highly recomended to produce homogenous and mantenible importers.

 - Downloading data: the first part is getting the data from the source. This data may come in several formats: xml, csv, xml, json, different request to an API or even through scrapping technics. It is preferable to separate this stage from the others. If the data from the source can be easily stored in a single local file (or few files), the code will result more mantenible. There will be cases where this can result difficult or improductive (too many API request, scrap)
 - Parsing the data and converting it to intermediate objects: despite of the fact that the goal of the importer is to produce an xml file with the source's data, it is preferable to build plain model objects containing the data before, as an intermediate step. Generating objects such as Observation, Country, Slice, etc., makes it easier to understand and recognize them as xml nodes, the programs are easier to maintain and debug. Every python importers convert the source's data into a group of common entities. Those entities are transformed later into "xml" by a common module.
 - Getting country's ISO3: The observations in the xml should be related to a country/region. In the final xml, this area should be identified with an ISO3 for countries or a UN_CODE for regions. But the usual scenario is that we can't find those codes between the source's data. Some sources give us a short name, some others a long name, an unordered name, a name in other language, an ISO2, other type of code... i.e.: Paraguay could be identified of many forms, such as "Paraguay", "the Republic of Paraguay", "The Republic of Paraguay", "Paraguay, the Republic of", "FAOSTAT_CODE = 169",... The importer should have a mechanism to turn whatever the source uses to identify countries into an ISO3/UN_CODE, ensuring that no observation is lost or reasigned to a wrong country. Python importers count with a module to do the work.
 - Generating XML and sending it to the Receiver: That's the final step every importer must achieve, if you have follow our advices, you will have multiple objects indentified, so the work should be easier, also if you are developing the importer in Python, there is a custom module that will help you transforming datasources directly to the xml. In order to know more about the xml format have a look at this section "XML format"


IDs asignation
^^^^^^^^^^^^^^

As it has been said, there are many entities within the domain, that should have an unique ID. In this section we will enumerate them all as well as the way to form its id

 - Organization id: Those ids are usually generated concatenating the string "ORG" plus a little chain which should be representative of the organization. eg: WorldBank -> WB 

From now on, every id is formed using a string compound of three letters, the organization chain and integer that may be static or not.

 - Datasource id: Those ones, are the result of putting together the string "SOU", the organization's representative chain and a given number, which should be statically assigned to every datasource, so it won't change between data imports.

 - Dataset id: Same as datasource, but in this case, the string used is "DAT" and the numeric part is not statically assigned, but calculated when the importer is working, based in previous executions.
 
 - Indicator id: In this case, the id will be formed using the same method as datasource, static numeric indicator included and using "INT" instead of "SOU".
 
 - Slice id: As well as dataset, this is not assigned statically and, of course, the string used is "SLI" instead of "DAT". 
 
 - Observation id: Same as slice, but using "OBS" as the object string.
 
 
Executing importers
^^^^^^^^^^^^^^^^^^^

Until now, every importer has been developed in Python, which means executing it requires having Python installed in the computer, as well as all the needed modules such the CountryReconciler, the ModelEntities and the ModelToXml (which can be downloaded from this repository).

In order to make it work, every module should be in the same folder (importers are configured to track them there) or, alternatively you can add the directories path to "PythonPath". Finally you should use any terminal to place yourself on the importer folder, and execute the "main.py" file located there.

Also every importer has a configuration ini file, with a "historical mode" parameter, located generally under a "Translator" section, this parameter reflects the mode in which the importer will be executed. Being True means the importer will search for every available data without concerning about dates, whereas if False only data dated after the "historical year" parameter will be tracked.
If you are interested in the other sections or parameters, please have a look at this section "SECTION".


Importers in python
^^^^^^^^^^^^^^^^^^^
Until now, every importer has been developed in Python, following a similar schema:

 - /data: If needed, this folder contains the files from which the data are extracted (xml, csv, json, ...)
 - /es/weso//: Is the source code folder, inside there will be different packages depending on the importer, usually here will be placed the importer class itself, and some utility classes like file readers, api callers, etc.
 - /files: Contains the Python ini files, that will be explained forward.
 
As it was said in the previous section, in order to execute the importers, there is a "main.py" file, located in the root directory of the module that will make the magic happens, this files are always similar (completly equals in some importers) and run the next actions:

 1. Configure log file 
 2. Load and read ini file
 3. Run the importer class
 4. Update ini file

Having understood this, now a little explanation of the utilities modules will continue, this modules are completly optional, but is highly recommended to give them a try if you are developing a Python importer

Land portal entities
""""""""""""""""""""
This package is as useful as easy to understand. It contains object representations of the different entities in the model, so it's easier to compound them when parsing data. Also, some of them has default values assigned to match default entities.

Country reconciler
""""""""""""""""""
As it was said before, countries are represented in several ways, this module, relying in an excel file, which contains the countries and every possible representation, will build the country object based in the entities module from whatever value you provide (name, iso2, iso3, ...)

ModelToXML
""""""""""
The most important action of an importer, is building the XML file that will be send to the receiver. This module, relying into the Entities module, will parse all information into a valid XML file.

So, now that you know the utilites modules, and more or less how an importer works, it's time to see them deeply, and as I said, as every importer has it's differences, here will be listed everyone explaining (in the case they are) why they are special.

FAO Agricultural Censues
""""""""""""""""""""""""
FAO organization has four different importers, depending on where we have find the data, this one in particular handles three different excel files, which represents the same indicators through time. Every excel file has a different format which made impossible the development of an heterogenous importer, driving to this situation:

In the configuration.ini file there are the next properties:
 - file_names: Points what files there have to been parser, separated by commas.
 - data_range_rows_file_name: Indicates the valid rows of the excel file separated by '-'
 - data_range_cols_file_name: Indicated the valid cols of the excel file separated by '-'
 
Those properties are needed because the excel files contain a lot of metadata, which is useless for the importer.

The excel reader, as many others importers, load a data matrix with the required cells of the file, making it easier to the importer to work with them.

The importer itself, in this case and due to the differences between files, all of them has a customize method in which are specified the columns for every indicator, but at the end all this methods rely in a last one that is the same for every file.

FAO Food Security
"""""""""""""""""
This particular one, relies in a single excel file, but with lof of sheets, that is why every indicator has a sheet name assigned in the config file.

It works as the previous one, loading all the data in a matrix, but with the difference of having only one method to parse every possible sheet, as all of them have the same format.

FAO Gender
""""""""""
FAO Stat
""""""""
Foncier
"""""""
IPFRI
"""""
LandMatrix
""""""""""
OECD
""""
RAW
"""
This is a special one, cause it hasn't been developed for any particular organization.

This one handles a default excel file, that can be fulfilled by the user, with the needed data. Every indicator that wants to be imported must be in a different file and placed under the data folder.

In the configuration.ini file, there should appear the organizations represented by the files (may be more than one) as properties under the section [ORGANIZATIONS], and as value the different file names, separated by commas.

The importer will generate and organization object for every property placed under the [ORGANIZATIONS] section and will load them with the indicators specified in the files. As in any time, more files could be added for the same organization, the importer also generated a custom ini file for them, saving the indicator id, the generated datasets, and the number of generated observations for this particular organization.

Transparency International
""""""""""""""""""""""""""
The same case that with the FAO Agricultural Censues importer, with the difference that this one is adapted to the new model in which every indicator represents a section under the ini file.

Like the other importer, there is a need to know the valid range of rows and columns for data extraction, but in this case, every indicator has its own. Also as there are more than one sheet in the files, it's needed to know the sheet from which the data will be extracted. Again, as the files are totally different is needed more than one function to transform the data into observations, based on the rows-columns where the data is presented.

UNDP
""""
World Health Organization
"""""""""""""""""""""""""
In this case, the World Health organization provides a lot of ways to download its data (csv, excel, etc.) and in different formats (codes, text and both). We are using the verbose ones, which provides both text and codes, so it maked easier to add new indicators.

As those files, are downloaded from and endpoint and it's different for every indicator and indicator endpoint must be passed through the configuration file and transformed into an URL, which is why there are the next properties:

 - URL pattern: Is used to compound the URL with the indicator endpoint values provided.
 - Indicator: Is a code used by the WHO to identify its indicators
 - Profile: Points the mode in which the file will be downloaded (empty for code, 'text' for text and 'verbose' for both of them)
 - Countries: It may be 'COUNTRY:*' which means all countries are requested or a list of countries with this format 'COUNTRY:XXX;COUNTRY:YYY' being XXX and YYY the ISO3 codes of the countries.
 - Regions: Same as country but with 'REGION:' instead of 'COUNTRY:'

Once the endpoint is compound the file is downloaded with name given in the indicator section and the data is extracted from it by the importer.

World Bank
""""""""""
This is the slowest and the biggest of the importers. It works directly with the WorldBank API, which requires as data the indicator and the country you are looking for, which means that for every indicator there have to be done an equivalent to the number of countries calls (in our case 256), multiply it by the number of indicators (33) and you will have a lot of calls to a free API. On the other hand, it makes quite easy to add a new indicator to parse, the only thing that is need to be done is to add the new indicator to an existing datasource, or, in the case the indicator doesn't belong to the existing ones, add a new datasource under the corresponding section.

Within the configuration.ini file there are two URLs, one of them is used to retrieve a list of 256 countries from the WorldBank API, that will be used then to compound the second URL, which needs both the country code and an indicator ID (those ids are WorldBank custom and need to be consulted in the web).

What this importer does is:

 1. Generates a country list relying in the Country reconciler and the iso codes obtained from the WorldBank API.
 2. Loads all the datasources specified in the ini file.
 3. For every datasource loads all the indicators under the corresponding section.
 4. Makes a call to the API compounding the url with the country code and the indicator code and transform the response to several observations (one for every year contained in the response).

Importers in any other language
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
There hasn't been developed importers in any other language than Python, but of course, it's possible. The mayor drawback you will have is to adapt the utilities modules provide in Python to the new language.

Implementation advices
^^^^^^^^^^^^^^^^^^^^^^
If you are reading this section, then you have in mind to develop your own importer. Before you do that consider to use the RAW importer, cause the only action you will need to do is fullfill an excel file. If you feel like it has no use fulfilling the file, here are some advices that will help your path during the development process.

 - Try to develop in an object oriented methodology, the domain of the data is really huge, and having some objects to rely in will be usefull.
 - In case you are using different files, try to make them as similar as possible. You don't want to end developing an importer with a function to read and parse every file differently.
 - If you are considering calling an API, take into account the time it will take to the importer to retrieve all the data (sometimes it's easier to locate and download the files from whom the API extracts the data). 
 - Sometimes you will find a data source in a web page, but you won't be able to locate the data, API, etc. which can lead you to think about scraping the web... it's possible, but not recommended (if you really want to use data from a source you should contact with the providers)
