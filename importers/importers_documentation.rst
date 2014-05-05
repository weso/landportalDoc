LandPortal Importers Documentation
==================================

.. sectnum::
.. contents:: Index

General
-------
What is an importer?
^^^^^^^^^^^^^^^^^^^^
An importer is an independient module with the goal of tracking data from a organization/datasource and introducing them in the LandPortal DataBase. To do that, it has to download/requets the info form the target source and convert it to a xml with a concrete format. Once it has that xml built, it has to send it to the module Receiver in LandPortal Architecture.

Restrictions:
 - Language: you can develop an importer in any languaje, there is no restrcition about this. Anyway, it is recommended to built importers in python. The only reason for this is that there are some support modules/libraries developed in python to do some tipical and common operations in an easy and efficient way.
 - IDs (identifiers): The produced xml contains a set of observations, slices and info about use license, datasource, organization, indicators, etc. Most of that entities, such as observations, must have an unique ID in the system, and it is the importer the module that asigns the identifiers to that entities. So, there must be a defined strategy to generate them. More on this in Section "IDs asignation". If the importer produce repeated IDs in entities such as observations, the receiver will reject the xml. On the other hand, the importer needs to ensure that other entities such as indicator always have the same ID. 
 - Write acces: All the developed importers needs to write some intermediate files while executing, and it is probably that the new ones will also need it. So, hte importer must execute with write acces in its directory.
 - Executions modes: every developed importer should have two execution modes: one for tracking every data of the source and other for tracking only those that had not been processed yet. 


Stages of execution
^^^^^^^^^^^^^^^^^^^

The main goal of the importer is to produce an xml with the data of the source and to send it to the Receiver. To reach that goal there are some typical stages. The code do not necessarily have to be divided in modules/packages thinking in this stages, but it is highly recomended to produce homogenous and mantenible importers.

 - Downloading data: the first part is getting the data from the source. This data can came in several formats: xml, csv, xml, json, different request to an API or even through scrapping technics. It is preferable to separate this stage from the others. If the data from the source can be easily stored in a single local file (or few files), the code will result more mantenible. There will be cases where this can result difficult or improductive (too many API request, scrap)
 
 - Parsing the data and converting it to intermediate objects: despite of the fact that the goal of the importer is to produce an xml file with the source's data, it is preferable to build plain model objects containing the data before, as an intermediate step. Generating objects such as Observation, Country, Slice, etc., easier to undesrtand and recognize than xml nodes, the programs are easier to maintain and debug. Every python importers convert the source's data into a group of common entities. Those entities are transformed later into "xml" by an common module.

 - Getting country's ISO3: The observations in the xml should be related to a country/region. In the final xml, this area should be identifies with an ISO3 for countries or a UN_CODE for regions. But the usual scenario is that we can't find those codes between the source's data. Some sources give us a short name, some others a long name, an unordered name, a name ion other idiom, an ISO2, other type of code... i.e.: Paraguay could be identified of many forms, such as "Paraguay", "the Republic of Paraguay", "The Republic of Paraguay", "Paraguay, the Republic of", "FAOSTAT_CODE = 169",... The importer should have a mechanism to turn whatever the source uses to identify countries into an ISO3/UN_CODE, ensuring that no observation is lost or reasigned to a wrong country. Python importers count with a module that do that work.

 - Generating XML and sending it to the Receiver:


IDs asignation
^^^^^^^^^^^^^^

Executing importers
^^^^^^^^^^^^^^^^^^^

Receiver's responses
^^^^^^^^^^^^^^^^^^^^

Importers in python
^^^^^^^^^^^^^^^^^^^

Importers in any other language
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Implementation advices
^^^^^^^^^^^^^^^^^^^^^^

