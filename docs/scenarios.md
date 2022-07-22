# Shift Scenarios

There are 8 scenarios covered in this section. Each scenario represents a typical use of the Db2 Shift program.
The scenarios can be summarized as:

* Shift a Db2 database to OpenShift, Kubernetes or CP4D
* Shift a Db2 database to another Db2 instance
* Create a Cloned copy of the Db2 database for later deployment
* Deploy a clone into an OpenShift, Kubernetes or CP4D container
* Deploy a clone into another Db2 instance
* Initialize HADR between Source and Target POD 
* Initialize HADR between Source and Target Instance
* Initialize DMC and LDAP Authentication for CP4D
* Copy Cloned Databases to a POD

All scenarios assume that you are currently connected to the instance
that has the Db2 database and have authenticated to OpenShift or Kubernetes
before issuing the command. 

The following sections describe the different Db2 Shift
scenarios and what settings are required to run each one.
Each scenario includes information on what fields are
required and those that are optional. More details on the
settings are discussed in the chapters dedicated to these
scenarios. Use this section to determine what settings are
required before attempting to do a shift operation.

### Db2 Shift Command

To invoke the Db2 shift command use the format `./db2shift` if the program is installed in a local
directory. If it is stored in the `/bin` directory then you can refer to it without the 
directory prefix `db2shift`. 

The program has three modes of operations:

* If the program is started without any parameters, it will go into full screen display mode and
prompt you for the action you wish to take. You can use this prompt mode to generate any 
Db2 Shift command. 
</p>

* If the `--help` option is used, the program will display a menu of help topics on the use
of the Db2 Shift program.
</p>

* Otherwise, the Db2 Shift program will attempt to run the shift based on the settings provided.

### Db2 Containerization to OpenShift, Kubernetes, or CP4D

Containerizing an existing Db2 database on an on-premise system to OpenShift, Kubernetes, or CP4D 
is the most common scenario for using Db2 Shift. In order to move a Db2 database to a POD you 
will require the following information:

* Source Database details
* Destination location
* Type of Containerization environment
* Shift Options

The target of the Db2 Shift operation can be OpenShift, a Kubernetes cluster, or Cloud Pak for Data. 

More details can be found in the [Db2 Containerization to OpenShift, Kubernetes, or CP4D](shiftpod.md) section.

### Db2 Shift to another Db2 Instance

This format of the Db2 Shift will take an existing Db2 database on an on-premise system,
and shift it to another traditional Db2 system hosted on another on-premise
server or cloud virtual machine. This does not containerize Db2! The Db2 Shift command 
requires the following information:

* Source Database details
* Destination location
* Shift Options

The db2shift program assumes that you are currently connected to the instance
that has the Db2 database and have ssh connectivity to the target server.

More details can be found in the [Db2 Shift to another Db2 Instance](shiftinstance.md) section.

### Clone an Existing Database

The Db2 Shift clone option is used to take an existing Db2 database that is currently
on-premise, and clone it into a directory. This cloned database can then
be transported to another server and be deployed at that location.

The advantage of cloning is that the destination does not need to be 
connected to the source location and the deployment of the clone can
be done at a more convenient time.

The Db2 Shift program requires the following information:

* Source Database details
* Clone Options

The destination details are not required to clone a database.

More details can be found in the [Clone an Existing Database](clone.md) section.

### Clone Deployment to OpenShift, Kubernetes, or CP4D

This Db2 Shift option will take a database clone and deploy it into a Db2u pod running on 
OpenShift, Kubernetes or CP4D.

The panel requires the following information:

* Database name and clone location
* Destination POD details
* Type of Containerization environment
* Shift Options

More details can be found in the [Clone Deployment to OpenShift, Kubernetes, or CP4D](deploypod.md) section.

### Clone Deployment to Another Db2 Instance

This Db2 Shift option will take a database clone and deploy it into another 
Db2 instance running natively (in any environment including Cloud VMs).

The panel requires the following information:

* Database name and clone location
* Destination instance details
* Shift Options

More details can be found in the [Clone Deployment to Another Db2 Instance](deployinstance.md) section.

### Initialize HADR between Source and Target POD

This Db2 Shift option will take a source and destination (POD)
database and start the HADR service between them. The Db2u pod must have been created with the following setting
during the shift step. Note that a destination HADR port is not required since it is automatically
generated for you by the Db2 Shift program.

The panel requires the following information:

* The source database name and server
* The destination POD and server details

More details can be found in the [Initialize HADR between Source and Target POD](hadrpod.md) section.
 
### Initialize HADR between Source and Target Instance

This menu is similar to the previous one where the HADR service is setup between the source
Db2 database and another Db2 instance. The Db2 database on the target system must have been
created with one of the following settings during the shift step.

The panel requires the following information:

* The source database name and server
* The destination server details

More details can be found in the [Initialize HADR between Source and Target Instance](hadrinstance.md) section.
 
### Initialize DMC and LDAP Authentication for CP4D

When shifting a database to a CP4D platform or LDAP-secured environment, the Db2 Shift program will automatically
add the appropriate userids to the LDAP service and reset the Data Management Console (DMC) so 
that it recognizes the database.

If you choose to shift a database and set it up as an HADR secondary, the LDAP and DMC setup
cannot be performed until **after** you have converted the secondary POD into the primary
server. Only when the POD is the primary server can it be initialized to support LDAP and DMC
services on IBM Cloud Pak for Data. 

More details can be found in the [Initialize DMC and LDAP Authentication for CP4D](ldapdmc.md) section.

### Clone Copy

This Db2 Shift command provides a feature that allows a user to copy an existing
database clone copy to a POD, or to retrieve a database clone from a POD. 

Once a database clone has been generated, the copy can be moved to any location and then 
deployed locally. This option provides a convenient way of copying the database using 
Db2 Shift without having to use OpenShift or Kubernetes commands.

The panel requires the following information:

* Type of copy (From Source to Target or Target to Source)
* Source cloned database directory
* Target cloned database directory
* The destination POD and server details

More details can be found in the [Clone Copy](clonecopy.md) section.

