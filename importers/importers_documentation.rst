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



Importers in any other language
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Implementation advices
^^^^^^^^^^^^^^^^^^^^^^

