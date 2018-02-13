# Zeppelin Police Department Incidents data example

This repository's scope is to present an example of how to hook an arbitrary Zeppelin notebook project into the Banzai [Pipeline](https://github.com/banzaicloud/pipeline) CD/CI workflow.

The project contains a simple Zeppelin notebook that operates on the San Francisco Police Department Incidents dataset. (The format and the description of the dataset is available
[here](https://data.sfgov.org/Public-Safety/Police-Department-Incidents/tmnf-yvry "SFData") ).

The notebook is a `json` file exported from Zeppelin. It's recommended to be edited with the Zeppelin notebook editor as the exported json contains a lot of "noise".

The notebook in this project is made up of a few `paragraphs` that:

- configure the spark context
- load and parse the dataset
- render a map for the data to be displayed on
- a few paragraphs that execute select operations with different API calls and display the data

There are two CI/CD flow descriptor templates provided so far, one per cloud provider:  

1. ```.pipeline.yml.aws.template``` for Amazon
2. ```.pipeline.yml.azure.template``` for Azure

The templates are similar as they are made up of the same ```steps``` only differ in the ```steps``` in charge for provisioning the kubernetes cluster plus in case of Azure the deployment needs your Azure storage account and access key. You will have to add these as secrets, please see below. This is needed for event logging and Spark History Server. In case of Amazon we use Instance Profile access.

## Prerequisites for Amazon

* An instance of the  Banzai Cloud Control Plane needs to be running and accessible
* Create an S3 bucket and a folder for persisting Spark event logs, so that they can be accessed by the Spark History Server. (In our example we named these to *spark-k8-logs* and *eventLog*, replace these with your appropriate values)
* The example dataset should be available for the cluster; it needs to be downloaded from the above mentioned location and uploaded to your s3 bucket

## Prerequisites for Azure

* An instance of the Banzai Cloud Control Plane needs to be running and accessible
* The following resources are needed on the Azure cloud:
 a resource group in one of the locations, a `Storage Account` and a `Blob Service` and a folder for persisting Spark event logs so that hey can be accessed by the Spark History Server. (In our example we named these to *spark-k8-logs* and *eventLog* respectively, replace these with your appropriate values. Also take note of your access key for the `Storage Account`, you will have to set provide these to the `steps` as `secrets`).
* The data needs to be downloaded from the above mentioned location (our smaller data set is also available [here](https://s3.amazonaws.com/lp-deps-test/data/Police_Department_Incidents.csv)) and uploaded to WASB. Please create a separate `Blob Service` for this.
In our example notebook ```.pipeline.yml.azure.template``` we named this to *pdidata* and our storage account to *sparklogstore*, replace these with yours if you are using different names.

## Configurations required to hook in into the [Banzai Pipeline](https://github.com/banzaicloud/pipeline) CI/CD workflow

In order for a project to be part of a Banzai Pipeline CI/CD workflow it must contain a specific configuration file: ```.pipeline.yml``` in it's root folder.

> In short: the configuration file contains the steps the project needs to go through the workflow from provisioning the environment, building the code, running tests to being deployed and executed along with project specific variables (eg.:credentials, program arguments, etc needed to assemble the deployment). We reference this file as the CI/CD flow descriptor

Depending on the chosen cloud provider, rename one of the templates to [.pipeline.yml](.pipeline.yml).
Update the below properties, depending on cloud type.

### Amazon

Replace s3 bucket name and folder name with yours:

- pipeline.install_spark_history_server.deployment_values.app.``logDirectory``
- pipeline.install_zeppelin.deployment_values.zeppelin.sparkSubmitOptions.``eventLogDirectory``

### Azure

Replace resource group name with yours:

- pipeline.create_cluster.``azure_resource_group``

Replace Blob container name and folder name with yours:

- pipeline.install_spark_history_server.deployment_values.app.``logDirectory``
- pipeline.install_zeppelin.deployment_values.zeppelin.sparkSubmitOptions.``eventLogDirectory``


## The following secrets need to be set on the CI user interface

### The endpoint for the Pipeline API

    plugin_endpoint: http://<host/ip>/pipeline/api/v1

The pipeline runs on the Banzai Cloud Control Plane, so this value should point to the hostname/ip of the machine hosting it.

### Credentials for the Pipeline API

    plugin_user: <admin>
    plugin_password: <example>

These credentials specify the user of the Pipeline API.

### Credentials for zeppelin

    plugin_zeppelin_username
    plugin_zeppelin_password

These credentials authenticate the Zeppelin user.

### Credentials for Azure Blob Storage

    azure_storage_account
    azure_storage_account_access_key

These credentials are needed for Azure Blob Storage access.

> If you'd like to hook in your own notebook into the Banzai Pipeline Workflow:
> - add your notebook to the repository:
> _zeppelin-pdi-example/my-notebook.json_

> - change the configuration file to point to it (see the marked lines below)

```yml
run:
  ....
  zeppelin_notebook_name: "my-notebook.json" # <---- change this
  zeppelin_notebook_file_path: "my-notebook.json" #<---- change this
  ....
```

For further reference check [this](https://github.com/banzaicloud/drone-plugin-zeppelin-client) and
[this](https://github.com/banzaicloud/pipeline/blob/master/docs/pipeline-howto.md)
