# Pre-requisites

The are a number of pre-requisites for running the Db2 Shift program itself, 
as well as requirements related to the Db2 database and the target Db2
system. The Db2 Shift program will analyze the source database and prevent
any shifts from occuring that will not work because of the target database
environment.

An additional analysis feature is also provided which will determine if 
any INSTANCE parameters on the source system need to be moved to the
TARGET system. The database can be shifted without the corresponding
target INSTANCE settings being modified, but it may affect the behavior of the
database when it is running in the new environment.

## Target Users

The Db2 Shift program has been designed for experienced DBAs or casual users through the GUI interface. It is more
likely that the user interface will be used for analysis and initial testing while production use will more likely use the command line version. 

The user requires intimate knowledge of current INSTANCE settings to make sure the containerized 
environment has sufficient resources to run the workloads. Further details of what is required to run the Db2 Shift program is 
described below.


## Db2 Shift Program Requirements

The Db2 Shift program is a single executable Linux program that can be 
installed in any directory. The only requirement is that the program is marked as executable 
and is accessible to the Db2 INSTANCE owner.

The program itself is self-contained and does not require any additional 
libraries to run. The program can be removed by simply deleting the file. When
the program executes, it will generate temporary files that are used during the
shift process. It is recommended that the program be run from within a directory
so that all files generated can be easily found.

The program itself requires only 8M of space and sufficient disk space for the
log and control files that are generated. 

The following Linux environments have been tested as source systems:

* Linux X64, CentOS (7,8), CentOS Stream, Redhat (7,8) Ubuntu 18.04, 20.04
* SUSE has not been verified
* Linux PPC (PowerPC) Little Endian has been tested as a target location

## Connectivity Requirements

The shifting of a database from one environment to another requires connectivity between
the servers. The process by which Db2 Shift moves data requires a connecion to either 
an OpenShift/Kubernetes/CP4D cluster, a server-less ssh connection, or a local connection.

When shifting data to OpenShift, Kubernetes, or CP4D, the Db2 Shift program will
require the following:

* A suitable client for the Cluster software

    * OpenShift CLI (oc) for OpenShift and Cloud Pak for Data shifts
    * Kuberbetes CLI (kubectl) for all Kubernetes clusters
    * SSH serverless connection to the target system
    * Local connection does not require a client but is only available for deployment of cloned databases
    <br/>
</br>

* OpenShift Version 3.11, 4.x 
* Cloud Pak for Data 3.5, 4.x
* Kubernetes Version 1.19
* A connection between the Source and Target servers 
* Sufficient disk space in /tmp or other directory (for Clone operations only)

## Userid and Authentication Requirements

The Db2 Shift operation must take place under the userid of the INSTANCE owner of the database
being shifted. Traditionally this userid has been `db2inst1` but may be different in your environment.
The user must be logged in as the INSTANCE owner and must have access to the `db2shift` command.

The instance owner must also have `ssh` server-less connectivity to the TARGET system if a 
Db2 to Db2 instance shift is being performed. The `ssh` connection is not required when shifting
into a containerized (OpenShift, Kubernetes, CP4D) environment.

To access the TARGET pod in a cluster, the user must have authenticated to OpenShift or Kubernetes and
have access to the POD that Db2 is running on. For OpenShift environments, authentication is 
done either through a userid and password or a token. An example of connecting with a userid/password:

`oc login -u ocadmin -p ocadmin`

Shifting to Cloud Pak for Data does not require a CP4D userid. The OpenShift userid or token for the 
underlying cluster is required to authenticate for the Db2 Shift operation.

Kubernetes installations will require the use of a profile (Config) to connect to the cluster. 

For both OpenShift and Kubernetes shift operations, the namespace (Kubernetes) or project (OpenShift) name
will be required. In the case of CP4D the namespace is usually `cp4d` but it depends on how the cluster
was deployed. In OpenShift and Kubernetes, the default namespace/project must be set **prior** to 
running the Db2 Shift command. You can override the namespace and project in the command but setting
the namespace prior to executing the script will minimize errors with Db2u pods not being found.

## Target Database Requirements

The target database must be created before running the Db2 Shift command. The Db2 Shift command 
cannot create the database for you. You must use the CP4D console to create a new database, or the
Db2u Operators on OpenShift or Kubernetes to create and deploy the Db2 database. For traditional Db2
installations, the database would be created using the `db2setup` or `db2_install` program.

The new database name should be the same as the database being shifted. This is a strict requirement
for CP4D databases, but not necessary for the the other environments. Db2 Shift will take the existing
database and rename it to the target database name.

Db2 databases created in clustered environments are always created at the latest level (11.5.7+). For
databases that are created using the `db2install` command, a minimum level of 11.5 should used since 
prior versions are now considered out of service.

In addition, the nodes of an MPP configuration must match that of the SOURCE database and the 
memory, disk, and cores **should** match the SOURCE database requirements. 


## Source Database Requirements

The source database can be moved as long as the following conditions are met:

* The database resides on a Linux server (X64 or PowerLinux LE)
* The database was created with Automatic storage
* The database is an OLTP, SMP, or MPP system
* Row or Column mode storage, including encrypted databases
* Mirror, Archive, and Overflow Logs
* User Defined Functions/Procedures located in the SQL lib Directory
* Db2 Version 10.5, 11.1, or 11.5 servers can be moved and upgraded at the same time

The following features are not currently supported:

* pureScale Feature (Not available yet in the Db2u container) 
* Spatial, Text Extender 

There are a few configuration settings which cannot be shifted:

* Only databases created with automatic storage are supported
* The system contains external procedures which are not in the standard Db2 library - these will need to be manually recreated and catalogued
* The LOGARCHMETH1/2 setting only supports DISK as a target in Db2u
* The database encryption keys will be moved to the new location, but if the target already has encrypted databases then you will need to manually migrate the encryption key to the target location


## Cloud Pak for Data Requirement

If you are shifting a database to Cloud Pak for Data, and your INSTANCE userid is **not** db2inst1, you 
must execute the following SQL commands from a userid that has SECADM authority:

```
GRANT SECADM ON DATABASE TO db2inst1
GRANT DBADM  ON DATABASE TO db2inst1 
```
The `db2inst1` userid does not need to exist in the Operating system in order to grant these privileges to the userid. This
requirement does not apply to other Db2 Shift environments. If `db2inst1` is not defined as a `SECADM` and `DBADM` user in the
database, the Data Management Console feature of CP4D will not be able to access the database nor will it be able to monitor it.
