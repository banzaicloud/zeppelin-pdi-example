# Zeppelin Police Department Incidents data example

This repository's scope is to present an example of how to hook an arbitrary Zeppelin notebook project into the Banzai [Pipeline](https://github.com/banzaicloud/pipeline) CD/CI workflow.

The project contains a simple Zeppelin notebook that operates on the San Francisco Police Department Incidents dataset. (The format and the description of the dataset is available
[here](https://data.sfgov.org/Public-Safety/Police-Department-Incidents/tmnf-yvry "SFData") ).

The notebook is a `json` file exported from Zeppelin. It's recommended to be edited with the Zeppelin notebook editor as the exported json contains a lot of "noise".

The notebook in this project is made up of a few `paragraphs` that:
- dowload dependencies required to access S3
- configure the spark context
- load and parse the dataset
- render a map for the data to be displayed on
- a few paragraphs that execute select operations with different API calls and display the data

## Prerequisites

* A Banzai Cloud Control Plane needs to be running and accessible
* The data needs to be downloaded from the above mentioned location and uploaded to an arbitrary S3 bucket, made publicly readable


## Configurations required to hook in into the [Banzai Pipeline](https://github.com/banzaicloud/pipeline) CI/CD workflow

In order for a project to be part of a Banzai Pipeline CI/CD workflow it must contain a specific configuration file: ```.pipeline.yml``` in its root folder.

> In short: the configuration file contains the steps the project needs to go through the workflow from provisioning the environment, building the code, running tests to being deployed and executed along with project specific variables (eg.:credentials, program arguments, etc needed to assemble the execution command - `spark-submit` in this case).

The current configuration comes with the location of the dataset pointing to a publicly readable S3 bucket (Warning: This may change!). Edit the '.pipeline.yml' file and change the value of the 'spark_app_args', '--dataPath' argument to point to the proper location of your dataset:

```yml
...
spark_app_args: --dataPath s3a://<your-bucket>/<your-pdi-data>.csv
```
The configuration file for this project is [.pipeline.yml](.pipeline.yml)

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

> If you'd like to hook in your own notebook into the Banzai Pipeline Workflow:
> - add your notebook to the repository:
```
zeppelin-pdi-example/my-notebook.json

```
> - change the configuration file to point to it (see the mareked lines ebelow)


```yml
run:
  ....
  zeppelin_notebook_name: "my-notebook.json" # <---- change this
  zeppelin_notebook_file_path: "my-notebook.json" #<---- change this
  ....
```

For further reference check [this](https://github.com/banzaicloud/drone-plugin-zeppelin-client)
