# Pre-requisites

The are several pre-requisites for running the Db2 Shift
program itself, as well as requirements related to the Db2
database and the target Db2 system. The Db2 Shift program
will analyze the source database and prevent any shifts from
occurring that will not work because of the target database
environment. 

An additional analysis feature is provided
which will determine if any INSTANCE parameters on the
source system need to be moved to the TARGET system. The
database can be shifted without the corresponding target
INSTANCE settings being modified, but it may affect the
behavior of the database when it is running in the new
environment.


## Target Users

The Db2 Shift program has been designed for experienced DBAs
or casual users through the GUI interface. It is more likely
that the user interface will be used for analysis and
initial testing while production use will use the command
line version. 

The user requires intimate knowledge of
current INSTANCE settings to make sure the containerized
environment has sufficient resources to run the workloads.
Further details of what is required to run the Db2 Shift
program is described below.

## Db2 Shift Program Requirements

The Db2 Shift program is a single executable Linux program
that can be installed in any directory. The only requirement
is that the program is marked as executable and is
accessible to the Db2 INSTANCE owner.

The program itself is self-contained and does not require
any additional libraries to run. The program can be removed
by simply deleting the file. When the program executes, it
will generate temporary files that are used during the shift
process. It is recommended that the program be run from
within a directory so that all files generated can be easily
found.

The program itself requires only 8M of space and sufficient
disk space for the log and control files that are generated.

The following Linux environments have been tested as source
systems:

* Linux X64, CentOS (7,8), CentOS Stream, Red Hat (7,8) Ubuntu 18.04, 20.04, SUSE 11.4
* Linux PPC (PowerPC) Little Endian has been tested as a target location

## Connectivity Requirements

The shifting of a database from one environment to another
requires connectivity between the servers. The process by
which Db2 Shift moves data requires a connection to either
an OpenShift/Kubernetes/CP4D cluster, a server-less ssh
connection, or a local connection.

When shifting data to OpenShift, Kubernetes, or CP4D, the
Db2 Shift program will require the following:

* A suitable client for the Cluster software

    * OpenShift CLI (oc) for OpenShift and Cloud Pak for Data shifts
    * Kuberbetes CLI (kubectl) for all Kubernetes clusters
    * SSH serverless connection to the target system
    * Local connection does not require a client but is only available for deployment of cloned databases

* OpenShift Version 3.11, 4.x 
* Cloud Pak for Data 3.5, 4.x
* Kubernetes Version 1.19
* A connection between the Source and Target servers 
* Sufficient disk space in /tmp or other directory (for Clone operations only)

Note that some Linux distributions do not support the
OpenShift CLI. For instance, only the Kubernetes CLI is
available for SUSE 11.4 environments. Customers wanting to
shift from a SUSE 11.4 environment to CP4D or OpenShift will
need to use the Clone option and then copy the clone
directory to a client that does support the OC client.

## Userid and Authentication Requirements

The Db2 Shift operation must take place under the userid of
the INSTANCE owner of the database being shifted.
Traditionally this userid has been db2inst1 but may be
different in your environment. The user must be logged in as
the INSTANCE owner and must have access to the db2shift
command.

The instance owner must also have ssh server-less
connectivity to the TARGET system if a Db2-to-Db2 instance
shift is being performed. The ssh connection is not required
when shifting into a containerized (OpenShift, Kubernetes,
CP4D) environment.

To access the TARGET pod in a cluster, the user must have
authenticated to OpenShift or Kubernetes and have access to
the POD that Db2 is running on. For OpenShift environments,
authentication is done either through a userid and password
or a token. An example of connecting with a userid/password:

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

The target database should be created before running the Db2
Shift command. The Db2 Shift command cannot create the
database for you in containerized environments (OpenShift,
Kubernetes, CP4D). You must use the CP4D console to create a
new database, or the Db2u Operators on OpenShift or
Kubernetes to create and deploy the Db2 database. For
traditional Db2 installations, the database would be created
using the CREATE DATABASE command.

The new database name should be the same as the database
being shifted. This is a strict requirement for CP4D
databases, but not necessary for the other environments. Db2
Shift will take the existing database and clone it into the
target database name.

For shifting or cloning a database to a traditional Db2
Instance, Db2 Shift will create the database on your behalf
if you specify the create database flag. Otherwise, the
program will stop execution because the target database will
not be found.

Db2 databases created in clustered environments are always
created at the latest level (11.5.7+). For traditional Db2
instances created using the db2install command, a minimum
level of 11.5 should be used since prior versions are now
considered out of service.

In addition, the nodes of an MPP configuration must match
that of the SOURCE database and the memory, disk, and cores
should match the SOURCE database requirements. 

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
* Text Extender 

There are a few configuration settings which cannot be shifted:

* Only databases created with automatic storage are supported
* The system contains external procedures which are not in the standard Db2 library - these will need to be manually recreated and catalogued
* The LOGARCHMETH1/2 setting only supports DISK as a target in Db2u
* The database encryption keys will be moved to the new location, but if the target already has encrypted databases then you will need to manually migrate the encryption key to the target location

## Instance Owner and DBADM and SECADM Userid

Special care must be taken when moving a database into Cloud Pak for Data or into any Db2U deployment. The Db2U POD always uses db2inst1 as the instance owner. If your instance owner is different (i.e., db2inst2), the db2inst1 user at the target will not have any privileges on this database. You will not be able to manage the database.

In addition, for Cloud Pak for Data environments, your database name on CP4D must match the source database name. If the database names are mismatched, the Data Management Console and CP4D services will be unable to detect and manage the database.

If you are shifting a database to Cloud Pak for Data or a Db2u POD, and your source INSTANCE userid is not db2inst1, you must execute the following SQL commands from a userid that has SECADM authority:
```
GRANT SECADM ON DATABASE TO USER db2inst1
GRANT DBADM  ON DATABASE TO USER db2inst1 
```

The `db2inst1` userid does not need to exist in the
Operating system to grant these privileges to the userid. If
you are moving the database to a traditional Db2 instance,
you could always create a new userid at the OS level to
manage the database, but this option is not available in a
container.