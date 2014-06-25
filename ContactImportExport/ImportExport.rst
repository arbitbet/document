Import/Export contacts in OroCRM
====================================

**OroImportExportBundle** intended for import / export entities in the Oro Platform. **OroImportExportBundle** uses **OroBatchBundle** 
to organize execution of import/export operations. Any import / export operation is a **Job**. Job is abstract by itself, 
it doesn't know specific details of what is going on during it's execution. Job is process that included **Steps**. 
Job can be configured with execution context and executed by client.

Each step aggregates three crucial components:

* **Reader**
* **Processor**
* **Writer**

Each of them doesn't know of each other. Step uses the reader to read data from source. After reader give data to the processor 
that to change data and give it to writer. Writer save data to other source.

Looks how to work import / export in the **OroCRMContactBundle**. Contact bundle uses **OroImportExportBundle**. You can see 
extended classes from **OroImportExportBundle** in the OroCRM/Bundle/ContactBundle/ImportExport and configuration 
in the ContactBundle\Resources\config\importexport.yml. Import / export has thre operation: import entity to CSV file, 
validate import data and export entity to CSV file, you can look it in the batch jobs 
configuration file  Oro/Bundle/ImportExportBundle/Resources/config/batch_jobs.yml.This configuration is used by **OroBatchBundle**.

Import
------

Import contacts can be done in three user steps (each of them is job).

At the first step user fill out the form with source file that he want to import and submit it. See controller action
OroImportExportBundle:ImportExport:importForm (route "oro_importexport_import_form"), this action require parameter
"entity" which is a class name of entity that will be imported.

At the second step import validation is triggered. See controller action OroImportExportBundle:ImportExport:importValidate
(route "oro_importexport_import_validate"). As a result a user will see all actions that will be performed by import and
errors that were occurred. Records with errors can't be imported but errors not blocks valid records.

At the last step import is processed. See controller action OroImportExportBundle:ImportExport:importProcess
(route "oro_importexport_import_process"). Please see how it works below.

Import operation consists one step, please look configuration batch job for import:

.. code-block:: yaml

    # Oro/Bundle/ImportExportBundle/Resources/config/batch_jobs.yml
    connector:
        name: oro_importexport
        jobs:
            entity_import_from_csv:
                title: "Entity Import from CSV"
                type: import
                steps:
                    import:
                        title:     import
                        class:     Oro\Bundle\BatchBundle\Step\ItemStep
                        services:
                            reader:    oro_importexport.reader.csv
                            processor: oro_importexport.processor.import_delegate
                            writer:    oro_importexport.writer.entity
                        parameters: ~

**OroBatchBunlde** has ``Oro\Bundle\BatchBundle\Step\ItemStep`` class that execute step in the job. It is method doExecute() 
that uses ``Oro\Bundle\BatchBundle\Step\StepExecutor`` class and method execute(). Important! Method processes(read and process) by 
one item in the loop. If source is empty loop breaks and write all items, after step is done.

So when you import data (for example for contacts) Reader ``Oro\Bundle\ImportExportBundle\Reader\CsvFileReader`` class gets one row 
from CSV file and return dimensional array $data of fields to Processor ``Oro\Bundle\ImportExportBundle\Processor\ImportProcessor``. 
Reader uses for it method read(). Import processor uses method process() to convert item and serialize result. 
Data Converter ``Oro\Bundle\ImportExportBundle\Converter\ConfigurableTableDataConverter`` class uses method convertToImportFormat($item) 
to convert dimensional array to complex array. After it processor receive one object with all related objects 
and collections of objects from Serializer ``Oro\Bundle\ImportExportBundle\Serializer\Serializer`` class. 
Than processor change object with Strategy ``Oro\Bundle\ImportExportBundle\Strategy\Import\ConfigurableAddOrReplaceStrategy`` class. 
Than processor return object to Writer ``Oro\Bundle\ImportExportBundle\Writer\EntityWriter`` class. Writer stores array of objects 
using method write(array $items).

Export
------



