## Introduction

Researchers are submitting workloads via The Run:AI CLI, Kubeflow or similar. To streamline resource allocation and create prioritize, Run:AI introduced the concept of __Projects__. Projects are quota entities that associate a project name with GPU allocation and preferences. 

A researcher submitting a workload needs to associate a project with a workload request. The Run:AI scheduler will compare the request against the current allocations and the project and determine whether the workload can be allocated resources or whether it should remain in a pending state.

Administartors manage Projects as detailed [here](Working-with-Projects.md)

At some organization, Projects may not be enough, this is because:

* There are simply too many individual entities that are attached with a quota.
* There are organizational quotas at a higher level. 


## Departments

__Departments__ are a second hierarchy of resource allocation:

* A Project is associated with a single Department.
* A Department, like a Project is associated with a Quota. 
* A Department quota supercedes a Project quota. 

### Overquota behavior

Consider an example from an academic use case: the Computer Science department and the GeoPhysics department have each purchased 10 DGXs with 80 GPUs, totalling a cluster of 160 GPUs. The two departmentes do not mind sharing GPUs as long as they always get their 80 GPUs when they truly need it. As such, there could be many Projects in the GeoPhysics departments, totalling a quota of 100 GPUs, but anything above 80 GPUs will be considered by the Run:AI scheduler as overquota 


## Creating and Managing Departments 

### Enable Departments

Departments are disabled by default. To start working with departments:

* Go to Settings | General
* Enable Departments 

Once departments are enabled, the menu will have a new item named "Departments" 

<img src="../img/department-menu.png" alt="department-menu" width="200"/>


Under __Departments__ there will be a single Department named __default__. All projects created before the Department feature was enabled will belong to the __default__ department


### Adding Departments

You can add new Departments by pressing the __Add New Department__ at the top right of the Department view.

<img src="../img/new-department.png" alt="new-department" width="400"/>

Add the department name and a quota allocation

### Assigning Projects to Departments

Under __Projects__ edit an existing project, you will see a new __Department__ drop down with which you can associate a project with a department
