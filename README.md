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
3. ```.pipeline.yml.gke.template``` for Google Cloud

The templates are similar as they are made up of the same ```steps``` only differ in the ```steps``` in charge for provisioning the Kubernetes cluster plus in case of Azure the deployment needs your Azure storage account and access key. You will have to add these as secrets, please see below. This is needed for event logging and Spark History Server. In case of Amazon we use Instance Profile access.

## Prerequisites for Amazon

* An instance of the Banzai Cloud Control Plane needs to be running and accessible
* Create an S3 bucket for persisting Spark event logs, so that they can be accessed by the Spark History Server. You will have to set the name of this bucket container in the example `yml` as [[your-s3-bucket]].
* The example dataset should be available for the cluster; it needs to be downloaded from the above mentioned location and uploaded to your s3 bucket, then update our example notebook ```sf-police-incidents-aws.json``` replacing [[your-bucket-name]].

## Prerequisites for Azure

* An instance of the Banzai Cloud Control Plane needs to be running and accessible
* The following resources are needed on the Azure cloud:

    - `Resource Group` in one of the locations
    - `Storage Account`, take note of your access key for the `Storage Account`, you will have to set [[your-storage-account-name]] and [[your-storage-account-access-key]] as `secrets` in later steps.
    - `Blob Service` for persisting Spark event logs so that hey can be accessed by the Spark History Server. You will have to set the name of this Blob container in the example `yml` as [[your-blob-container]].


* The data needs to be downloaded from the above mentioned location (our smaller data set is also available [here](https://s3.amazonaws.com/lp-deps-test/data/Police_Department_Incidents.csv)) and uploaded to WASB. Create a separate `Blob Service` in the same `Storage Account` created in previous step and upload the data file. Update our example notebook ```sf-police-incidents-azure.json``` replacing [[your-blob-container]], [[your-azure-storage-account]] values with yours.

## Prerequisites for Google Cloud

* An instance of the Banzai Cloud Control Plane needs to be running and accessible
* The following resources are needed on Google Cloud:

    - `Project` You'll have to enter the ID of your project as [[your-gke-project-id]] in example `yml`
    - `Storage bucket` for persisting Spark event logs so that hey can be accessed by the Spark History Server. You will have to set the name of this bucket in the example `yml` as [[your-gs-bucket]].


* The data needs to be downloaded from the above mentioned location (our smaller data set is also available [here](https://s3.amazonaws.com/lp-deps-test/data/Police_Department_Incidents.csv)) and uploaded to Google Storage. Please create a separate `Storage Bucket` for this.
Update our example notebook ```sf-police-incidents-gke.json``` replacing [[your-gs-bucket]] value.

## Steps required to hook in into the [Banzai Pipeline](https://github.com/banzaicloud/pipeline) CI/CD workflow

In order for a project to be part of a Banzai Pipeline CI/CD workflow it must contain a specific configuration file: ```.pipeline.yml``` in it's root folder.

> In short: the configuration file contains the steps the project needs to go through the workflow from provisioning the environment, building the code, running tests to being deployed and executed along with project specific variables (eg.:credentials, program arguments, etc needed to assemble the deployment). We reference this file as the CI/CD flow descriptor

* depending on the chosen cloud provider, rename one of the templates to [.pipeline.yml](.pipeline.yml).
Update the below properties, depending on cloud type.

  ##### Amazon

  - [[your-cluster-name]]
  - [[your-s3-bucket]]

  ##### Azure

  - [[your-cluster-name]]
  - [[your-azure-cluster-location]]
  - [[your-azure-resource-group]]
  - [[your-blob-container]]

  ##### Google Cloud

  - [[your-gke-cluster-name]]
  - [[your-gke-project-id]]
  - [[your-gs-bucket]]


* navigate to the CI/CD user interface (that usually runs on the Banzai Cloud Control plane instance)
* enable the project build from the list of available repositories
* add the following secrets to the build:

```
PLUGIN_ENDPOINT = [control-plane]/pipeline/api/v1
PLUGIN_TOKEN = "oauthToken"
```

*Credentials for Azure Blob Storage access*

    PLUGIN_AZURE_STORAGE_ACCOUNT = "[[your-storage-account-name]]"
    PLUGIN_AZURE_STORAGE_ACCOUNT_ACCESS_KEY = "[[your-storage-account-access-key]]"


The project is configured now for the Banzai Cloud CI/CD flow. On each commit to the repository a new flow will be triggered. You can check the progress on the CI/CD user interface.


> If you'd like to hook in your own notebook into the Banzai Pipeline Workflow:
> - add your notebook to the repository:
> _zeppelin-pdi-example/my-notebook.json_

> - change the configuration file to point to it (see the marked lines below)

```yml
run:
  ....
  zeppelin_notebook_name: "my-notebook.json" # <---- change this
  zeppelin_notebook_file_path: "my-notebook.json" # <---- change this
  ....
```

For further reference check [this](https://github.com/banzaicloud/drone-plugin-zeppelin-client) and
[this](https://github.com/banzaicloud/pipeline/blob/master/docs/pipeline-howto.md)
