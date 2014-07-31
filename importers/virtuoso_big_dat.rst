How to manually load big datasets in a virtuoso system
======================================================

Introducing big datasets (30Mb or more) in the Virtuoso dataset could be a troublesome task, not as well documented in the official support as it should. In the context of our application, there are three ways to try it:

 - Automatically, using the capabilities of the Receiver module when introducing a raw dataset in our system. This option is the slowest one in computation terms and also consumes a lot of resources of the machine. Depending on the parameters of the machine and the size and structure of the dataset the process may take hours to be completed. All the data incorporated to LandPortal has been introduced in the system using this procedure, with the exception of some World Bank’s datasets. The problems found to make us deciding to not use this with World Bank was a lack of resources in the machine used for executing the process of data importation, in addition to the time that a single try took before failing (more than one day).
 - Manually, using the Conductor web interface of Virtuoso, once you have a dataset in a valid rdf format (Receiver module could be used for it). This is the easiest way, but it looks that it is not possible to handle this when the datasets are too big. An attempt to upload a big file will result in a time-out. I also found that there could be problems when you try to upload several datasets not so big but in a short period of time.
 - Manually, using console utilities, once you have a dataset in a valid rdf format (Receiver module could be used for it). This is the safest way and the one we use with World Bank troublesome datasets. 

The instructions used for doing the task of option three were mainly taken from the next three links:
 - http://virtuoso.openlinksw.com/dataspace/doc/dav/wiki/Main/VirtBulkRDFLoader
 - http://virtuoso.openlinksw.com/dataspace/doc/dav/wiki/Main/VirtBulkRDFLoaderExampleDbpedia
 - https://github.com/openlink/virtuoso-opensource

Anyway, we will put all this info together. We will explain step by step how to incorporate big datasets into virtuoso database, once you have those datasets in a valid rdf format.


 - Step 1. Put the dataset in some of the folders specified by DirsAllowed configuration option. This file can be found in /usr/local/var/lib/virtuoso/db/virtuoso.ini. Official doc indicate that you can add a custom directory to DirsAllowed or use an already specified one. We choose the second option and put our datasets in an already specified directory (/usr/local/var/lib/virtuoso/db).
 
 - Step 2. For each dataset you have to create in the same directory a file with the same name of the dataset + “.graph”. Example: if you have a dataset “example.rdf”, you must place in the same folder a file called “example.rdf.graph”. This new file should contain the name of the graph where the triplets should be introduced. In our content, the content of this file was http://book.landportal.org. Alternatively, official doc indicate that you could create a file called “global.graph” instead of a file for each dataset, and it is supposed to affect to every dataset in the folder. Anyway, we get the result by creating a “.graph” for each dataset.
 You may have problems if you specify a graph that does not exist yet in the database (we did). So you may create the graph using the web interface at first place before doing this task.

 - Step 3. Execute the the sql utility command line. In our system, this is placed in /usr/local/bin/isql. The order should look like this: “sudo isql 1111”. That command will enter the sql console with the role of database admin (it will ask you for the password).

 - Step 4. Introduce you datasets in a loading list to incorporate them to the database. That should be done by executing in the sql command line an order such as “ld_dir(‘path/to/directory’,’name_of_the_dataset.rdf’,’name_of_the_graph’);”. An example of a valid order would be “ld_dir(‘/usr/local/var/lib/virtuoso/db’, ‘dataset1.rdf’, ‘http://book.landportal.org’);”. Be careful when specifying folder’s path. You must not put a slash at the end of the path, or the process will fail.

 - Step 5. Check that you datasets have been successfully put in the load list. For that, you must execute this in the sql command line: “select * from DB.DBA.load_list;”. The path of your dataset should appear in the resulting list, with a flag “0”. That means that the dataset is ready to be incorporated, but it is not still in the system. For future queries like that, flag “1” means that the dataset is being processed and flag “2” that the dataset had already been incorporated.

 - Step 6. Introduce the dataset in the system by running “rdf_loader_run();”. That may take a few seconds. Once the process has ended, if you execute again the query of step 5 your datasets should appear with flag “2”.

 - Step 7. Make the changes effective. At this point, the datasets are in the system, but they belong to a transaction still uncommitted. You must execute “checkpoint;” in the sql command line to commit the changes.


You can get out of the sql command line by executing “exit;”.  Now, your datasets should be available for queries though the sparql endpoint.
