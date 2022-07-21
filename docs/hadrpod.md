# Initialize HADR between Source and Target POD

This Db2 Shift option will take a source and destination (POD)
database and start the HADR service between them. The Db2u pod must have been created 
with the following setting during the shift step.

Syntax: `--hadr`
![Mode](img/field_move_hadr.png)

The panel requires the following information:

* The source database name and server
* The destination POD and server details

The syntax for initiating the HADR connection between two Db2 servers is:

<pre><code class="language-bash">db2shift

    Required Options     

    --mode=hadr_setup  
    --dest-type=POD
    --oc or --kubectl                                                                                       
    --source-dbname=flights
    --source-hadr-host=some.server.com
    --source-hadr-port=3700  
    --dest-server=c-demo-db2u-0
    --dest-hadr-host=oc.server.com
    --dest-hadr-port=3700

    Optional Settings
    
    --dest-namespace=db2u
</code></pre> 

The panel that provides this capability:

![ShiftPOD](img/c2c_hadr_pod.png)

## Mode Option

Syntax: `--mode=hadr_setup`

The HADR option is used to initialize and start HADR between a source and target server. This step
would be run after the database has been shifted to the new location. This option is 
not used for the initial HADR setup of the target system. The target database must be
created with the `--hadr` option in order for it be placed into the correct mode for
HADR communication. The `--hadr` is enabled for pods with the Move Database for HADR
option in the menu.
![Mode](img/field_move_hadr.png)
 
## Target Client (Instance to POD)

Syntax: `--oc`, `--kubectl`
![OC Client](img/field_oc_k8s.png)

The pod client for a deploy (clone) operation must be 
supplied as part of the Db2 Shift command. Only one of the following 
clients must be used:

* `--oc` OpenShift Destination
* `--kubectl` Kubernetes Destination

If the client is Kubernetes (`--kubectl`) or OpenShift (`--oc`), 
the program requires that the appropriate `kubectl` or `oc` client 
has been installed locally and that the namespace or project has already been specified.

## Source Database

Syntax: `--source-database=""`
![Source Database](img/field_source_database.png)

The source database is the name of the database that you want to setup HADR
with on the source and destination servers. Note that the database name 
must be same at the both locations. 

**Alert!**
</br>
If you are shifting a database to Cloud Pak for Data, then the cloned database must must have `db2inst1` as the INSTANCE userid. See the section on Cloning a database to make sure that the proper authorities have been
granted to the `db2inst1` userid.

## HADR Source and Destination Server

Syntax: `--source-hadr-server=""`, `--dest-hadr-server=""`
![HADR Source Server](img/field_hadr_source_server.png)
![HADR Target Server](img/field_hadr_target_server.png)

For HADR setup, the Db2 Shift command requires the IP or symbolic
name of the source server that will be used in an HADR setup. This 
server is referred to as the primary server. 

If the destination server is a standard Db2 instance, use the IP address of the server. If the destination 
target is an OpenShift or Kubernetes cluster, use the address of the load balancer.

## HADR ports

Syntax: `--source-hadr-port=#`, `--dest-hadr-port=#`
![HADR Source Port](img/field_hadr_source_port.png)
![HADR Target Port](img/field_hadr_target_port.png)

HADR communicates over a port which is different than the Db2
instance. You must supply the source and destination port numbers
that Db2 will communicate between the HADR servers. The default
port number is 3700 for HADR communications, but verify the value. The target
port number will also be required.

## Destination Server (POD)

Syntax: `--dest-server="pod_name"`
![Target Server](img/field_pod_name.png)

For deployments to OpenShift, Kubernetes, or CP4D, you must supply the name
of the POD that Db2U is running in. The OpenShift or Kubernetes client should
be used to connect to the target namespace or project before issuing the 
Db2 Shift command. 

## Destination Pod Namespace or Project

Syntax: `--dest-namespace=""`, `--dest-project=""`
![Namespace](img/field_namespace.png)

In Kubernetes deployments, the location of a pod is associated with 
a namespace, while in OpenShift deployments, the pod is associated with
a project.

When authenticating to a Kubernetes or OpenShift environment, it is 
recommended that the local client be connected to the project or 
namespace that the Db2U pod is running in. 

If you do not supply a namespace or project value, the Db2 Shift program
will assume that you are already connected to that project. If this is not
the case, the program will stop with an error when it attempts to find the 
pod. 

To have Db2 Shift connect to the appropriate project or namespace, 
supply the value of the namespace or project using this option.