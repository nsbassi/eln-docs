# Table of Contents
* Introduction
   * Process Flow
* Getting Started
   * Authenticate
   * Get Assigned Projects
   * Get Experiments
   * Add attachment to Experiment
   
# Introduction
This application exposes certain endpoints as Rest services to interact with Agile PLM application. The End points have been created to enable integration between Agile PLM and ELN.

## Process Flow
The following figure depicts the process flow that this integration is targeting to address. 
![flow.png](https://github.com/nsbassi/eln-docs/raw/master/flow.png)
A Project Manager will setup project in PLM by initiating the same from template and assign required roles to team members.

Project Manager or Group Leaders will then create experiments under specific tasks in PLM and allocate the same to Scientists in their team.

Scientists will initiate a new Experiment in ELN and select Project, Task and PLM Experiment Number against which they intend to create experiment in ELN. Certain information related to the experiment will be pulled from PLM and mapped to specific fields of new Experiment being created in ELN.

The experiment will then be performed and the conclusions recorded in ELN. A human readable formatted output of the experiment will then be sent over to PLM as a PDF File.

# Getting Started
To perform operations in PLM, ELN application needs to first authenticate and then use the token returned by authentication request for making any subsequent requests. The following figure depicts typical sequence of interactions between ELN and PLM.
![intg.png](https://github.com/nsbassi/eln-docs/raw/master/intg.png)

## Authenticate
ELN needs to make a request to `/authenticate` endpoint passing user id and password as parameters. 
```
curl -d userId=admin -d password=tartan -X POST http://localhost:8080/authenticate
```
If the authentication is successful `success` flag in response will be set to `true` and an authorisation `token` will be included in the response.
```
{
   "token" : "e115ba98-8df0-4da9-a9d4-94d7a8a1be01",
   "expiry" : "27-Jun-17 01:28:24",
   "success" : true,
   "started" : "27-Jun-17 01:08:24"
}
```
In case the authentication fails an error message will be included in the response depicting reason for failure.
```
{
   "exception" : "com.agile.api.APIException",
   "message" : "Invalid username or password",
   "timestamp" : 1498509649171,
   "path" : "/authenticate",
   "status" : 500,
   "error" : "Internal Server Error"
}
```
## Get Assigned Projects
ELN needs to make a request to `/projects` endpoint passing user id and authorisation token returned by `/authenticate` request to retrieve list of projects assigned to the user. 
```
curl -d userId=admin -d token=e115ba98-8df0-4da9-a9d4-94d7a8a1be01 -X POST http://localhost:8080/projects
```
If the request is successful response will be a JSON array containing list of projects assigned to the specified user.
```
[
   {
      "number" : "PROJECT0000002",
      "id" : 31,
      "description" : "Template for XX Project",
      "name" : "CP Template"
   },
   {
      "name" : "SRF160002-PD-001",
      "description" : "XYZ",
      "id" : 17360,
      "number" : "PROJECT0000165"
   },
   {
      "id" : 58,
      "number" : "PROJECT0000003",
      "description" : "Template for Sample Preparation Project",
      "name" : "Sample Preparation Template"
   }
]
```
In case the request fails an error message providing reason for failure will be included in the response.
```
{
   "exception" : "java.lang.Exception",
   "message" : "Not Authenticated. Operation failed.",
   "path" : "/projects",
   "timestamp" : 1498509486723,
   "error" : "Internal Server Error",
   "status" : 500
}
```
## Get Assigned Tasks
ELN needs to make a request to `/tasks` endpoint passing user id, authorisation `token` and `number` of project selected from the list returned by `/projects` request to retrieve list of tasks assigned to the user under the selected project. 
```
curl -d userId=admin -d token=e115ba98-8df0-4da9-a9d4-94d7a8a1be01 -d prjNum=PROJECT0000165 -X POST http://localhost:8080/tasks
```
If the request is successful response will be a JSON array containing list of tasks assigned to the specified user.
```
[
   {
      "id" : 17395,
      "name" : "MOC Studies",
      "description" : "",
      "number" : "PH01762"
   },
   {
      "name" : "Process Optimization",
      "id" : 17394,
      "number" : "PH01761",
      "description" : ""
   }
]
```
In case the request fails an error message providing reason for failure will be included in the response.
```
{
   "exception" : "java.lang.Exception",
   "message" : "Not Authenticated. Operation failed.",
   "path" : "/tasks",
   "timestamp" : 1498509486723,
   "error" : "Internal Server Error",
   "status" : 500
}
```
## Get Experiments
ELN needs to make a request to `/experiments` endpoint passing authorisation `token` and `number` of task selected from the list returned by `/tasks` request to retrieve list of experiments created under the task in PLM. 
```
curl -d tskNum=PH01761 -d token=e115ba98-8df0-4da9-a9d4-94d7a8a1be01 -X POST http://localhost:8080/experiments
```
If the request is successful response will be a JSON array containing list of experiments created under the specified task.
```
[
   {
      "id" : "6071926",
      "number" : "D02943",
      "category" : "Process Optimization Studies",
      "product" : "",
      "objective" : "",
      "files" : []
   }
]
```
In case the request fails an error message providing reason for failure will be included in the response.
```
{
   "exception" : "java.lang.Exception",
   "message" : "Not Authenticated. Operation failed.",
   "path" : "/experiments",
   "timestamp" : 1498509486723,
   "error" : "Internal Server Error",
   "status" : 500
}
```
## Add attachment to Experiment
To add attachment to an experiment in PLM, ELN needs to make a POST request to `/experiment` endpoint and submit a multipart form containing experiment number, authorisation token and file to be uploaded.
```
curl -F file=@/tmp/sample.pdf -F expNum=D02943 -F token=e115ba98-8df0-4da9-a9d4-94d7a8a1be01 -X POST http://localhost:8080/experiment
```
If the request is successful experiment object will be returned in the response. The `files` field representing list of files attached to the experiment will be updated to include the details of the recently uploaded file.
```
{
   "id" : "6071926",
   "product" : "",
   "files" : [
      "{rowId=6072376, fileId=6072327, file=sample.pdf}"
   ],
   "number" : "D02943",
   "objective" : "",
   "category" : "Process Optimization Studies"
}
```
In case the request fails an error message providing reason for failure will be included in the response.
```
{
   "error" : "Internal Server Error",
   "message" : "Experiment with number D029433 not found in PLM",
   "path" : "/experiment",
   "exception" : "java.lang.Exception",
   "timestamp" : 1498509367883,
   "status" : 500
}
```
