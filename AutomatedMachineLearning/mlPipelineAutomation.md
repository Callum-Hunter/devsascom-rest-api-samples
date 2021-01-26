# Machine Learning Pipeline Automation

The Machine Learning Pipeline Automation API provides a set of endpoints
that automates training predictive models. Business users can use these models to make business decisions.


Leveraging statistics and machine learning expertise at SAS, the Machine Learning Pipeline Automation API takes modeling-ready data, and does the following: 

*  executes data pre-processing steps (including optimal
transformations and imputations, variable selection and feature engineering)
*  generates data analysis pipelines
*  trains algorithms
*  optimizes models that can be used to predict outcomes or support business decisions

Once initiated, this automated process requires no further user input, which
removes manual and repetitive tasks from the business user workflow.

The service runs multiple steps of pre-processing, modeling, post-processing (ensembling),
and evaluates top models. The result is a dynamic, automated machine learning project that is transparent to the user.
The project is accessible from SAS Model Studio, with model assessment and interpretability reports. The accuracy backed by the leading machine learning libraries helps
empower businesses to make decisions in an agile environment. No statistics expertise is required.

## API Request Examples Grouped by Object Type

*  [Entry point](#entry-point)
*  [Creating automation projects](#creating-automation-projects)
*  [Querying all automation projects](#querying-all-automation-projects)
*  [Querying individual automation projects](#querying-individual-automation-projects)
*  [Updating automation projects](#updating-automation-projects)
*  [Retraining an automation project](#retraining-an-automation-project)
*  [Querying automation project state](#querying-automation-project-state)
*  [Updating automation project state](#updating-automation-project-state)
*  [Deleting automation projects](#deleting-automation-projects)
*  [Propagating deletion of an automation project](#propagating-deletion-of-an-automation-project)
*  [Champion model](#champion-model)

#### <a name='entry-point'>Entry Point</a>
An entry point to the service. The response to this GET request provides the links for users
to create an automation project and to query existing projects.

##### Request
```
GET /mlPipelineAutomation HTTP/1.1
Accept: application/vnd.sas.api+json
```

##### Response
```
{
    "version": 1,
    "links": [
        {
            "method": "GET",
            "rel": "collection",
            "href": "/mlPipelineAutomation/projects",
            "uri": "/mlPipelineAutomation/projects",
            "type": "application/vnd.sas.collection",
            "itemType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "POST",
            "rel": "createProject",
            "href": "/mlPipelineAutomation/projects",
            "uri": "/mlPipelineAutomation/projects",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        }
    ]
}
```

#### <a name='creating-automation-projects'>Creating Automation Projects</a>
A user begins using the service by creating an automation project. The Machine Learning Pipeline Automation
API uses the following attributes: user-supplied project
name, data table URI and target variable, and other attributes. The service creates an underlying analytics (VDMML) project, executes multiple steps of pre-processing, builds a data analysis
pipeline, and runs the pipeline to completion. The entire process is fully automated without further user intervention.

The request body is structured in three sections.

- Key information of the automation project
  - id (Automation project ID (automatically created by the service, read-only))
  - name (Name of the project (a random string is appended to ensure uniqueness))
  - description (Description of the project)
  - type (Project type (only "predictive" type is supported))
  - dataTableUri (Data table URI)
  - state (Project state)
  - pipelineBuildMethod (The pipeline build method (either "automatic" or "template". Default is "automatic".))
- settings (Automation project settings)
  - A properties bag through which a user can pass arbitrary key/value pairs, regardless if they are
    used. The properties currently used are as follows.
    - applyGlobalMetadata (a flag to indicate whether to apply global metadata during project creation.
      Default is set to true.)
    - autoRun (a flag to indicate whether to automatically start pipeline run at the time of
      project creation. Default is set to true.)
    - numberOfModels (a positive integer between 1 and 10 to indicate maximum number of top models to
      return. Default is set to 5)
    - maxModelingTime (the maximum time, in minutes, to use for modeling. Input is valid in icrements
      of 0.1. Use 0 for unbounded. Default is set to 0.0)
    - modelingMode (The modeling strategy that the automation project will use to select a champion
      model. Currently only the default value of 'Standard' is supported.)
    - locale (the locale to use for translating content. If this is left blank on the initial creation
      request, the value of the Accept-Language header will be used to populate the value. If there is
      no Accept-Language header, the default locale of 'en' will be used. This is intended to be
      read-only, and it is recommended that the Accept-Language header be used to populate this.)
- analyticsProjectAttributes (Analytics project attributes)
  - analyticsProjectId (Analytics project ID (automatically created by the service, read-only))
  - targetVariable (Target variable (required))
  - targetEventLevel (Target event level (optional))
  - classSelectionStatistic (a string to indicate class selection statistic, dependent upon the type
    of target variable. For 'BINARY' and 'NOMINAL' target variable types, the accepted values are
    listed below, where the default is set to 'ks'. The 'ORDINAL' target variable type is not
    supported.)
      - ase: Average squared error
      - acc: Accuracy
      - c: Area under curve (C statistic)
      - capturedResponse: Captured response
      - cumulativeCapturedResponse: Cumulative captured response
      - cumulativeLift: Cumulative lift
      - f1: F1 score
      - fdr: False discovery rate
      - fpr: False positive rate
      - gain: Gain
      - gini: Gini
      - ks: Kolmogorov-Smirnov statistic
      - lift: Lift
      - misclassificationEvent: Misclassification (Event)
      - mce: Misclassification (MCE)
      - mcll: Multiclass log loss
      - ks2: ROC separation
      - rase: Root average squared error
      - misclassificationCutoff: Misclassification at cutoff
  - intervalSelectionStatistic (a string to indicate interval selection statistic, dependent upon
    the type of target variable. For 'INTERVAL' target variable types, the accepted values are
    listed below, where the default is set to 'ase'. This field is ignored for all other types,
    like 'BINARY', 'NOMINAL', or 'ORDINAL'.)
      - ase: Average squared error
      - rase: Root average squared error
      - rmae: Root mean absolute error
      - rmsle: Root mean squared logarithmic error
  - partitionEnabled (A flag indicating whether partitioning is enabled. The default is true.
  - selectionDepth (The selection depth.  Default is 5)
    - 5
    - 10
    - 15
    - 20
  - selectionPartition (The selection partition. The default is 'default')
    - default
    - train
    - validate
    - test
  - overrideClassificationCutoffEnabled (Flag to indicate whether the default classification cutoff
    is overridden by the selected value. Default is false)
  - overrideClassificationCutoffValue (The override value (percentage) to use if
    overrideClassificationCutoffEnabled is true. This is a double in the range of 0.000001 to 0.999999
    with a default value of 0.5)
  - cutoffPercentage (The cutoff percentage. Default is 50)
    - 5
    - 10
    - 15
    - 20
    - 25
    - 30
    - 35
    - 40
    - 45
    - 50
    - 55
    - 60
    - 65
    - 70
    - 75
    - 80
    - 85
    - 90
    - 95
  - numberOfCutoffValues (The number of cutoff values. The default is 20)
    - 10
    - 20
    - 50
    - 100
    - 500
    - 1000
  - samplingEnabled (The sampling strategy. Default is AUTO)
    - YES
    - NO
    - AUTO
  - samplingPercentage (The sampling percentage. This is a double from 1-99. Default is 50.0)

##### Request for Project with an Automatically Generated Pipeline
```
POST /mlPipelineAutomation/projects HTTP/1.1
Accept: application/vnd.sas.analytics.ml.pipeline.automation.project+json

{
    "name": "Test Project Creation",
    "dataTableUri": "/dataTables/dataSources/cas~fs~cas-shared-default~fs~Public/tables/APITESTHMEQ",
    "type": "predictive",
    "pipelineBuildMethod": "automatic",
    "analyticsProjectAttributes": {
        "targetVariable" : "BAD",
        "partitionEnabled" : true,
        "targetEventLevel" : "1"
    },
    "settings": {
        "applyGlobalMetadata" : false
    }
}
```

##### Response
```
202 Accepted
Content-Type: application/vnd.sas.analytics.ml.pipeline.automation.project+json

{
{
    "creationTimeStamp": "2021-01-13T18:36:53.323214Z",
    "createdBy": "emduser1",
    "modifiedTimeStamp": "2021-01-13T18:36:53.323198Z",
    "modifiedBy": "emduser1",
    "revision": 0,
    "id": "71d81bd8-a20f-47ff-afda-afac6fb74f03",
    "name": "Test Project Creation (By MLPA Yuko3)",
    "description": "Model Automation Test Project",
    "links": [
        {
            "method": "GET",
            "rel": "up",
            "href": "/mlPipelineAutomation/projects",
            "uri": "/mlPipelineAutomation/projects",
            "type": "application/vnd.sas.collection",
            "itemType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "GET",
            "rel": "self",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "PUT",
            "rel": "update",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "DELETE",
            "rel": "delete",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03"
        },
        {
            "method": "DELETE",
            "rel": "propagateDelete",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?propagate=true",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?propagate=true"
        },
        {
            "method": "GET",
            "rel": "state",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state",
            "type": "text/plain"
        },
        {
            "method": "PUT",
            "rel": "updateState",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state?value={value}",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state?value={value}",
            "responseType": "text/plain"
        }
    ],
    "version": 2,
    "dataTableUri": "/dataTables/dataSources/cas~fs~cas-shared-default~fs~Public/tables/APITESTHMEQ",
    "type": "predictive",
    "state": "pending",
    "settings": {
        "applyGlobalMetadata": false,
        "autoRun": true,
        "locale": "en",
        "maxModelingTime": 0.0,
        "modelingMode": "Standard",
        "numberOfModels": 5
    },
    "analyticsProjectAttributes": {
        "analyticsProjectId": "193bd3f8-356e-4c7d-9830-72584a7b8c22",
        "targetVariable": "BAD",
        "targetEventLevel": "1",
        "partitionEnabled": true,
        "overrideClassificationCutoffEnabled": false,
        "samplingEnabled": "AUTO",
        "samplingPercentage": 50.0,
        "intervalSelectionStatistic": "ase",
        "classSelectionStatistic": "ks",
        "selectionDepth": 10,
        "selectionPartition": "default",
        "overrideClassificationCutoffValue": 0.5,
        "cutoffPercentage": 50,
        "numberOfCutoffValues": 20
    },
    "championModel": {},
    "pipelineBuildMethod": "automatic"
}
```

##### Request for Project with Pipeline Built from a Pre-defined Template

```
POST /mlPipelineAutomation/projects HTTP/1.1
Accept: application/vnd.sas.analytics.ml.pipeline.automation.project+json

{
    "name": "Test Project Creation",
    "dataTableUri": "/dataTables/dataSources/cas~fs~cas-shared-default~fs~Public/tables/APITESTHMEQ",
    "type": "predictive",
    "pipelineBuildMethod": "template",
    "analyticsProjectAttributes": {
        "targetVariable" : "BAD",
        "partitionEnabled" : true,
        "targetEventLevel" : "1"
    },
    "settings": {
        "applyGlobalMetadata" : false
    },
    "links": [
            {
                "method": "GET",
                "rel": "initialPipelineTemplate",
                "href": "/analyticsGateway/pipelineTemplates/dm.advancedbinarytargetautotunepl.template",
                "uri": "/analyticsGateway/pipelineTemplates/dm.advancedbinarytargetautotunepl.template",
                "type": "application/vnd.sas.analytics.pipeline.template"
            }
    ]
}
```

##### Response
```
202 Accepted
Content-Type: application/vnd.sas.analytics.ml.pipeline.automation.project+json

{
    "creationTimeStamp": "2021-01-13T18:40:39.249377Z",
    "createdBy": "emduser1",
    "modifiedTimeStamp": "2021-01-13T18:40:39.249366Z",
    "modifiedBy": "emduser1",
    "revision": 0,
    "id": "65c6508f-cf37-4789-8bdc-eb458712cdb6",
    "name": "Test Project Creation (By MLPA bRGEp)",
    "links": [
        {
            "method": "GET",
            "rel": "up",
            "href": "/mlPipelineAutomation/projects",
            "uri": "/mlPipelineAutomation/projects",
            "type": "application/vnd.sas.collection",
            "itemType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "GET",
            "rel": "self",
            "href": "/mlPipelineAutomation/projects/65c6508f-cf37-4789-8bdc-eb458712cdb6",
            "uri": "/mlPipelineAutomation/projects/65c6508f-cf37-4789-8bdc-eb458712cdb6",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "PUT",
            "rel": "update",
            "href": "/mlPipelineAutomation/projects/65c6508f-cf37-4789-8bdc-eb458712cdb6",
            "uri": "/mlPipelineAutomation/projects/65c6508f-cf37-4789-8bdc-eb458712cdb6",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "DELETE",
            "rel": "delete",
            "href": "/mlPipelineAutomation/projects/65c6508f-cf37-4789-8bdc-eb458712cdb6",
            "uri": "/mlPipelineAutomation/projects/65c6508f-cf37-4789-8bdc-eb458712cdb6"
        },
        {
            "method": "DELETE",
            "rel": "propagateDelete",
            "href": "/mlPipelineAutomation/projects/65c6508f-cf37-4789-8bdc-eb458712cdb6?propagate=true",
            "uri": "/mlPipelineAutomation/projects/65c6508f-cf37-4789-8bdc-eb458712cdb6?propagate=true"
        },
        {
            "method": "GET",
            "rel": "state",
            "href": "/mlPipelineAutomation/projects/65c6508f-cf37-4789-8bdc-eb458712cdb6/state",
            "uri": "/mlPipelineAutomation/projects/65c6508f-cf37-4789-8bdc-eb458712cdb6/state",
            "type": "text/plain"
        },
        {
            "method": "PUT",
            "rel": "updateState",
            "href": "/mlPipelineAutomation/projects/65c6508f-cf37-4789-8bdc-eb458712cdb6/state?value={value}",
            "uri": "/mlPipelineAutomation/projects/65c6508f-cf37-4789-8bdc-eb458712cdb6/state?value={value}",
            "responseType": "text/plain"
        }
    ],
    "version": 2,
    "dataTableUri": "/dataTables/dataSources/cas~fs~cas-shared-default~fs~Public/tables/HMEQ",
    "type": "predictive",
    "state": "pending",
    "settings": {
        "applyGlobalMetadata": false,
        "autoRun": true,
        "locale": "en",
        "maxModelingTime": 0.0,
        "modelingMode": "Standard",
        "numberOfModels": 5
    },
    "analyticsProjectAttributes": {
        "analyticsProjectId": "7ac1860e-7cf2-4fa3-a037-cfbe10fca352",
        "targetVariable": "BAD",
        "targetEventLevel": "1",
        "partitionEnabled": true,
        "overrideClassificationCutoffEnabled": false,
        "samplingEnabled": "AUTO",
        "samplingPercentage": 50.0,
        "intervalSelectionStatistic": "ase",
        "classSelectionStatistic": "ks",
        "selectionDepth": 10,
        "selectionPartition": "default",
        "overrideClassificationCutoffValue": 0.5,
        "cutoffPercentage": 50,
        "numberOfCutoffValues": 20
    },
    "championModel": {},
    "pipelineBuildMethod": "template"
}
```

#### <a name='querying-all-automation-projects'>Querying All Automation Projects</a>
A GET request sent to the `/mlPipelineAutomation/projects` endpoint makes the service query its database and
return a collection of existing automation projects. The parameters for this query are as follows:

| Parameter | Format          | Description                                              | Example
|-----------|:---------------:|----------------------------------------------------------|----------
| start     | Int             | The starting index in the database to search from.       | start=42
| limit     | Int             | The max number of projects to return.                    | limit=10
| sortBy    | \<key>:\<value> | Upon which parameter the results should be sorted.       | sortBy=projectId:descending
| filter    | Expression      | The filter to apply to the query results.                | filter=eq(projectId, 'example-value')

##### Request
```
GET /mlPipelineAutomation/projects?start=0&limit=2&sortBy=projectId:ascending&filter=eq(projectId, 'exampleId') HTTP/1.1
Accept: application/vnd.sas.collection+json
```

#### <a name='querying-individual-automation-projects'>Querying Individual Automation Projects</a>
The response is similar to the [Creating Automation Projects](#creating-automation-projects)
section. The response includes links to various operations on the project, such as starting or stopping the project, updating project settings/attributes, deleting the project, and so on.

##### Request
```
GET /mlPipelineAutomation/projects/981738b3-10b4-48ce-8911-17b1132b7992 HTTP/1.1
Accept: application/vnd.sas.analytics.ml.pipeline.automation.project+json
```

#### <a name='updating-automation-projects'>Updating Automation Projects</a>
This API is solely used for updating a project’s values such as name and description.
The retrain action is used to re-run a project with a different data URI and/or target variable.

This API does not support the PATCH operation to update the automation project with changes only. According to
standard, the user must enter the full project info in this PUT request, whether or not the parameters are to be changed.

List of parameters that can be updated.

- name
- description
- dataTableUri
- settings
- analyticsProjectAttributes (all analytics attributes can be changed, except analytics project ID)

Note that updating any of these fields via this endpoint will not result in the project being re-run. The retrain action
should be used to modify settings, attributes, and data.

List of parameters that cannot be updated. Any changes are ignored.

- id
- type
- analyticsProjectId
- common fields associated with all revision tracking resources
  - creationTimestamp
  - modifiedTimestamp
  - createdBy
  - modifiedBy
  - version

List of parameters that cannot be updated. Any changes cause 4xx error responses to be sent
back to the client.

- state (error code 409)
- revision (error code 412)

##### Request
```
PUT /mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03 HTTP/1.1
If-Match: W/"1610563552539418000"
Content-Type: application/vnd.sas.analytics.ml.pipeline.automation.project+json
Accept: application/vnd.sas.analytics.ml.pipeline.automation.project+json

{
    "creationTimeStamp": "2021-01-13T18:36:53.323214Z",
    "createdBy": "emduser1",
    "modifiedTimeStamp": "2021-01-13T18:45:52.539418Z",
    "modifiedBy": "emduser1",
    "revision": 10,
    "id": "71d81bd8-a20f-47ff-afda-afac6fb74f03",
    "name": "Modified Project Name",
    "description": "Modified Project Description",
    "links": [
        {
            "method": "GET",
            "rel": "up",
            "href": "/mlPipelineAutomation/projects",
            "uri": "/mlPipelineAutomation/projects",
            "type": "application/vnd.sas.collection",
            "itemType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "GET",
            "rel": "self",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "PUT",
            "rel": "update",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "DELETE",
            "rel": "delete",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03"
        },
        {
            "method": "DELETE",
            "rel": "propagateDelete",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?propagate=true",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?propagate=true"
        },
        {
            "method": "GET",
            "rel": "state",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state",
            "type": "text/plain"
        },
        {
            "method": "PUT",
            "rel": "updateState",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state?value={value}",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state?value={value}",
            "responseType": "text/plain"
        },
        {
            "method": "PUT",
            "rel": "retrainProject",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project",
            "responseType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "PUT",
            "rel": "retrainProjectReplacePipelines",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject&replacePreviousPipelines=true",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject&replacePreviousPipelines=true",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project",
            "responseType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        }
    ],
    "version": 2,
    "dataTableUri": "/dataTables/dataSources/cas~fs~cas-shared-default~fs~Public/tables/APITESTHMEQ",
    "type": "predictive",
    "state": "completed",
    "settings": {
        "applyGlobalMetadata": false,
        "autoRun": true,
        "locale": "en",
        "maxModelingTime": 0.0,
        "modelingMode": "Standard",
        "numberOfModels": 5
    },
    "analyticsProjectAttributes": {
        "analyticsProjectId": "193bd3f8-356e-4c7d-9830-72584a7b8c22",
        "pipelineId": "859959b8-8c21-4688-b825-ef1003fee7ee",
        "targetVariable": "BAD",
        "targetEventLevel": "1",
        "partitionEnabled": true,
        "overrideClassificationCutoffEnabled": false,
        "samplingEnabled": "AUTO",
        "samplingPercentage": 50.0,
        "intervalSelectionStatistic": "ase",
        "classSelectionStatistic": "ks",
        "selectionDepth": 10,
        "selectionPartition": "default",
        "overrideClassificationCutoffValue": 0.5,
        "cutoffPercentage": 50,
        "numberOfCutoffValues": 20
    },
    "championModel": {
        "name": "Forest (2)",
        "links": [
            {
                "method": "GET",
                "rel": "championModel",
                "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel",
                "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel",
                "type": "application/vnd.sas.analytics.ml.pipeline.automation.project.champion.model.report"
            },
            {
                "method": "PUT",
                "rel": "registerChampionModel",
                "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel?action=register",
                "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel?action=register"
            },
            {
                "method": "PUT",
                "rel": "publishChampionModel",
                "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel?action=publish",
                "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel?action=publish"
            },
            {
                "method": "POST",
                "rel": "scoreData",
                "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel/scoreData",
                "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel/scoreData",
                "type": "application/vnd.sas.analytics.ml.pipeline.automation.score.data.input",
                "responseType": "application/vnd.sas.analytics.ml.pipeline.automation.score.data.output"
            }
        ],
        "publishDestinations": [
            {
                "name": "maslocal",
                "description": "The default publishing destination for the Micro Analytic Score service.",
                "destinationType": "microAnalyticService"
            }
        ]
    },
    "pipelineBuildMethod": "automatic"
}
```

##### Response
```
200 OK
Content-Type: application/vnd.sas.analytics.ml.pipeline.automation.project+json

{
    "creationTimeStamp": "2021-01-13T18:36:53.323214Z",
    "createdBy": "emduser1",
    "modifiedTimeStamp": "2021-01-19T20:38:01.330997Z",
    "modifiedBy": "emduser1",
    "revision": 11,
    "id": "71d81bd8-a20f-47ff-afda-afac6fb74f03",
    "name": "Modified Project Name",
    "description": "Modified Project Description",
    "links": [
        {
            "method": "GET",
            "rel": "up",
            "href": "/mlPipelineAutomation/projects",
            "uri": "/mlPipelineAutomation/projects",
            "type": "application/vnd.sas.collection",
            "itemType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "GET",
            "rel": "self",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "PUT",
            "rel": "update",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "DELETE",
            "rel": "delete",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03"
        },
        {
            "method": "DELETE",
            "rel": "propagateDelete",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?propagate=true",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?propagate=true"
        },
        {
            "method": "GET",
            "rel": "state",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state",
            "type": "text/plain"
        },
        {
            "method": "PUT",
            "rel": "updateState",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state?value={value}",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state?value={value}",
            "responseType": "text/plain"
        },
        {
            "method": "PUT",
            "rel": "retrainProject",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project",
            "responseType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "PUT",
            "rel": "retrainProjectReplacePipelines",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject&replacePreviousPipelines=true",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject&replacePreviousPipelines=true",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project",
            "responseType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        }
    ],
    "version": 2,
    "dataTableUri": "/dataTables/dataSources/cas~fs~cas-shared-default~fs~Public/tables/HMEQ",
    "type": "predictive",
    "state": "completed",
    "settings": {
        "applyGlobalMetadata": false,
        "autoRun": true,
        "locale": "en",
        "maxModelingTime": 0.0,
        "modelingMode": "Standard",
        "numberOfModels": 5
    },
    "analyticsProjectAttributes": {
        "analyticsProjectId": "193bd3f8-356e-4c7d-9830-72584a7b8c22",
        "pipelineId": "859959b8-8c21-4688-b825-ef1003fee7ee",
        "targetVariable": "BAD",
        "targetEventLevel": "1",
        "partitionEnabled": true,
        "overrideClassificationCutoffEnabled": false,
        "samplingEnabled": "AUTO",
        "samplingPercentage": 50.0,
        "intervalSelectionStatistic": "ase",
        "classSelectionStatistic": "ks",
        "selectionDepth": 10,
        "selectionPartition": "default",
        "overrideClassificationCutoffValue": 0.5,
        "cutoffPercentage": 50,
        "numberOfCutoffValues": 20
    },
    "championModel": {
        "name": "Forest (2)",
        "links": [
            {
                "method": "GET",
                "rel": "championModel",
                "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel",
                "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel",
                "type": "application/vnd.sas.analytics.ml.pipeline.automation.project.champion.model.report"
            },
            {
                "method": "PUT",
                "rel": "registerChampionModel",
                "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel?action=register",
                "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel?action=register"
            },
            {
                "method": "PUT",
                "rel": "publishChampionModel",
                "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel?action=publish",
                "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel?action=publish"
            },
            {
                "method": "POST",
                "rel": "scoreData",
                "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel/scoreData",
                "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel/scoreData",
                "type": "application/vnd.sas.analytics.ml.pipeline.automation.score.data.input",
                "responseType": "application/vnd.sas.analytics.ml.pipeline.automation.score.data.output"
            }
        ],
        "publishDestinations": [
            {
                "name": "maslocal",
                "description": "The default publishing destination for the Micro Analytic Score service.",
                "destinationType": "microAnalyticService"
            }
        ]
    },
    "pipelineBuildMethod": "automatic"
}

```

#### <a name='retraining-an-automation-project'>Retraining an Automation Project</a>
To retrain an automation project with changed parameters, use the retrainProject endpoint. By default, the service generates an additional automated pipeline. This behavior can be overwritten with an optional query parameter
`replacePreviousPipelines`. When set to true, the parameter instructs the service to remove all previous
automatically generated pipelines before creating a new pipeline.

##### Request
```
PUT /mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject HTTP/1.1
If-Match: W/"1611088681330997000"
Content-Type: application/vnd.sas.analytics.ml.pipeline.automation.project+json
Accept: application/vnd.sas.analytics.ml.pipeline.automation.project+json

{
    "creationTimeStamp": "2021-01-13T18:36:53.323214Z",
    "createdBy": "emduser1",
    "modifiedTimeStamp": "2021-01-19T20:38:01.330997Z",
    "modifiedBy": "emduser1",
    "revision": 11,
    "id": "71d81bd8-a20f-47ff-afda-afac6fb74f03",
    "name": "Modified Project Name",
    "description": "Modified Project Description",
    "links": [
        {
            "method": "GET",
            "rel": "up",
            "href": "/mlPipelineAutomation/projects",
            "uri": "/mlPipelineAutomation/projects",
            "type": "application/vnd.sas.collection",
            "itemType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "GET",
            "rel": "self",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "PUT",
            "rel": "update",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "DELETE",
            "rel": "delete",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03"
        },
        {
            "method": "DELETE",
            "rel": "propagateDelete",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?propagate=true",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?propagate=true"
        },
        {
            "method": "GET",
            "rel": "state",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state",
            "type": "text/plain"
        },
        {
            "method": "PUT",
            "rel": "updateState",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state?value={value}",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state?value={value}",
            "responseType": "text/plain"
        },
        {
            "method": "PUT",
            "rel": "retrainProject",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project",
            "responseType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "PUT",
            "rel": "retrainProjectReplacePipelines",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject&replacePreviousPipelines=true",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?action=retrainProject&replacePreviousPipelines=true",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project",
            "responseType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        }
    ],
    "version": 2,
    "dataTableUri": "/dataTables/dataSources/cas~fs~cas-shared-default~fs~Public/tables/HMEQ",
    "type": "predictive",
    "state": "completed",
    "settings": {
        "applyGlobalMetadata": false,
        "autoRun": true,
        "locale": "en",
        "maxModelingTime": 0.0,
        "modelingMode": "Standard",
        "numberOfModels": 5
    },
    "analyticsProjectAttributes": {
        "analyticsProjectId": "193bd3f8-356e-4c7d-9830-72584a7b8c22",
        "pipelineId": "859959b8-8c21-4688-b825-ef1003fee7ee",
        "targetVariable": "BAD",
        "targetEventLevel": "1",
        "partitionEnabled": true,
        "overrideClassificationCutoffEnabled": false,
        "samplingEnabled": "AUTO",
        "samplingPercentage": 50.0,
        "intervalSelectionStatistic": "ase",
        "classSelectionStatistic": "ks",
        "selectionDepth": 10,
        "selectionPartition": "default",
        "overrideClassificationCutoffValue": 0.5,
        "cutoffPercentage": 50,
        "numberOfCutoffValues": 20
    },
    "championModel": {
        "name": "Forest (2)",
        "links": [
            {
                "method": "GET",
                "rel": "championModel",
                "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel",
                "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel",
                "type": "application/vnd.sas.analytics.ml.pipeline.automation.project.champion.model.report"
            },
            {
                "method": "PUT",
                "rel": "registerChampionModel",
                "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel?action=register",
                "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel?action=register"
            },
            {
                "method": "PUT",
                "rel": "publishChampionModel",
                "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel?action=publish",
                "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel?action=publish"
            },
            {
                "method": "POST",
                "rel": "scoreData",
                "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel/scoreData",
                "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel/scoreData",
                "type": "application/vnd.sas.analytics.ml.pipeline.automation.score.data.input",
                "responseType": "application/vnd.sas.analytics.ml.pipeline.automation.score.data.output"
            }
        ],
        "publishDestinations": [
            {
                "name": "maslocal",
                "description": "The default publishing destination for the Micro Analytic Score service.",
                "destinationType": "microAnalyticService"
            }
        ]
    },
    "pipelineBuildMethod": "automatic"
}
```

##### Response
```
202 Accepted
Content-Type: application/vnd.sas.analytics.ml.pipeline.automation.project+json

{
    "creationTimeStamp": "2021-01-13T18:36:53.323214Z",
    "createdBy": "emduser1",
    "modifiedTimeStamp": "2021-01-19T20:41:19.736495Z",
    "modifiedBy": "emduser1",
    "revision": 13,
    "id": "71d81bd8-a20f-47ff-afda-afac6fb74f03",
    "name": "Modified Project Name",
    "description": "Modified Project Description",
    "links": [
        {
            "method": "GET",
            "rel": "up",
            "href": "/mlPipelineAutomation/projects",
            "uri": "/mlPipelineAutomation/projects",
            "type": "application/vnd.sas.collection",
            "itemType": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "GET",
            "rel": "self",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "PUT",
            "rel": "update",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "type": "application/vnd.sas.analytics.ml.pipeline.automation.project"
        },
        {
            "method": "DELETE",
            "rel": "delete",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03"
        },
        {
            "method": "DELETE",
            "rel": "propagateDelete",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?propagate=true",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03?propagate=true"
        },
        {
            "method": "GET",
            "rel": "state",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state",
            "type": "text/plain"
        },
        {
            "method": "PUT",
            "rel": "updateState",
            "href": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state?value={value}",
            "uri": "/mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state?value={value}",
            "responseType": "text/plain"
        }
    ],
    "version": 2,
    "dataTableUri": "/dataTables/dataSources/cas~fs~cas-shared-default~fs~Public/tables/HMEQ",
    "type": "predictive",
    "state": "retraining",
    "settings": {
        "applyGlobalMetadata": false,
        "autoRun": true,
        "locale": "en",
        "maxModelingTime": 0.0,
        "modelingMode": "Standard",
        "numberOfModels": 5,
        "replacePreviousPipelines": false
    },
    "analyticsProjectAttributes": {
        "analyticsProjectId": "193bd3f8-356e-4c7d-9830-72584a7b8c22",
        "pipelineId": "859959b8-8c21-4688-b825-ef1003fee7ee",
        "targetVariable": "BAD",
        "targetEventLevel": "1",
        "partitionEnabled": true,
        "overrideClassificationCutoffEnabled": false,
        "samplingEnabled": "AUTO",
        "samplingPercentage": 50.0,
        "intervalSelectionStatistic": "ase",
        "classSelectionStatistic": "ks",
        "selectionDepth": 10,
        "selectionPartition": "default",
        "overrideClassificationCutoffValue": 0.5,
        "cutoffPercentage": 50,
        "numberOfCutoffValues": 20
    },
    "championModel": {},
    "pipelineBuildMethod": "automatic"
}
```

#### <a name='querying-automation-project-state'>Querying Automation Project State</a>
Automation project state can be one of these enum values.

- pending: indicates that the underlying analytics project was created but has not been run yet.
- preparing: indicates that the underlying analytics project was created and the metadata is being updated.
- waiting: indicates that the underlying analytics project metadata was updated.
- ready: indicates that the underlying analytics project is preparing and ready to submit CASL code.
- modeling: indicates that models are being composed and compared on CASL server.
- constructingPipeline: indicates that pipelines are being built on CASL server.
- runningPipeline: indicates that pipelines are being run on CASL server.
- quiescing: indicates that the user request to quiesce the project modeling has been received.
- quiesced: indicates that project modeling is being stopped in response to the user's quiescing request.
- completed: indicates that the underlying analytics project has run to completion without errors.
- canceled: indicates that the underlying analytics project run was canceled by user.
- failed: indicates that the underlying analytics project has encountered errors during pipeline run and has stopped.
- oversampling: indicates that oversampling is being performed on the project.
- retraining: indicates that retraining is being performed on the project.

##### Request
```
GET /mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state HTTP/1.1
```

##### Response
```
200 OK
Content-Type: text/plain

completed
```

#### <a name='updating-automation-project-state'>Updating Automation Project State</a>

To stop/cancel an automation project from the modeling state, user can issue a PUT request with state "quiescing" if the project is in the "modeling"
state (or earlier). This will mark the project as 'quiescing' and allow the CASL job to finish processing and provide whatever results are
available at the time. The project will then transition to "quiesced" when the operation is complete and partial results will be available.

Since the application is event-driven, explicitly updating the state is not allowed except in very specific instances because
asynchronous jobs are running while the project is running. If the state is updated by the user, then the completion of the job
will then update the state again to a new value. For example, setting the state to canceling while the project is retraining will
have no long-term effect because when the retraining job is completed, the project will transition to completed.

##### Request
```
PUT /mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/state?value=quiescing HTTP/1.1
```

##### Response
```
202 Accepted
Content-Type: text/plain

quiescing
```

#### <a name='deleting-automation-projects'>Deleting Automation Projects</a>
When it is not needed anymore, the automation project can be deleted with the request below. By
default, this API deletes the automation project while keeping its associated analytics project.
To delete both projects, use this API
([Propagating Deletion of an Automation Project](#propagating-deletion-of-an-automation-project))

##### Request
```
DELETE /mlPipelineAutomation/projects/981738b3-10b4-48ce-8911-17b1132b7992 HTTP/1.1
```

#### <a name='propagating-deletion-of-an-automation-project'>Propagating Deletion of an Automation Project</a>
This API can be used to delete both automation and analytics project.

##### Request
```
DELETE /mlPipelineAutomation/projects/981738b3-10b4-48ce-8911-17b1132b7992?propagate=true HTTP/1.1
```

#### <a name='champion-model'>Champion Model</a>

When a project completes, champion model information is added to the project body, and operations are available to be performed on the champion model

##### Retrieving Champion Model Information

###### Request
```
GET /mlPipelineAutomation/projects/71d81bd8-a20f-47ff-afda-afac6fb74f03/championModel HTTP/1.1
Accept: application/vnd.sas.analytics.ml.pipeline.automation.project.champion.model.report
```
###### Response
200 OK
```
[A collection of reports for the champion model]
```

##### Registering Champion Model with SAS Model Manager
```
PUT /mlPipelineAutomation/projects/981738b3-10b4-48ce-8911-17b1132b7992/championModel?action=register HTTP/1.1
```

##### Publishing Champion Model
```
PUT /mlPipelineAutomation/projects/981738b3-10b4-48ce-8911-17b1132b7992/championModel?action=publish&destination=maslocal HTTP/1.1
```

##### Score Data

Score the champion model data. Fields in the score data input type can be used for either individual or bulk scoring.

- scoreType (Individual or bulk)
- destinationName (The name of the destination where the champion model is published)
- inputDataTableURI (The data table to score when scoreType is set to bulk)
- scoreOutputCaslibURI (The output caslib for the score outputs table when the scoreType is set to bulk)
- scoreOutputTableName (The score output table name to generate when the scoreType is set to bulk)
- inputs (a list of inputs to score when the scoreType is set to individual)

```
POST /mlPipelineAutomation/projects/981738b3-10b4-48ce-8911-17b1132b7992/championModel/scoreData HTTP/1.1
Content-Type: application/vnd.sas.analytics.ml.pipeline.automation.score.data.input+json
Accept: application/vnd.sas.analytics.ml.pipeline.automation.score.data.output+json

{
    "scoreType": "Individual",
    "destinationName": "maslocal",
    "inputs": [
        {
            "name": "CLAGE",
            "value": 300
        },
        {
            "name": "DEBTINC",
            "value": 24.5
        },
        ...
        {
            "name": "JOB",
            "value": "Other"
        }
    ]
}
```

version 2, last updated 19 Jan, 2021
