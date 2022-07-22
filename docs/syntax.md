# Db2 Shift Syntax

The Db2 Shift command has two modes of operation:

* Command Line mode - A set of parameters are passed to the program and executed
* Interactive mode - An interactive User Interface is used to supply the parameters required to execute a command

The interactive mode provides the ability to generate all of the commands and includes the following additional features:

* Ability to save scripts for future use
* Compare source and destination target databases to check for compatibility
* Review Log files for any shift errors
* Extensive help information for input fields and Db2 Shift scenarios

The interactive mode is invoked when the Db2 Shift command is executed with no parameters, or
if either the following options are included on the command line:

* `--help` – Request detailed help information on running the Db2 Shift command
* `--mono` – Use a monochrome display format (White on Black) and non-Unicode character set when using the UI
* `--accept` – Accept license agreement (no license prompt) 
* `--version` – Display Version information
* `--shiftpod`, `--shiftdb2`, `--clone`, `--deploypod`, `--deploydb2`, `--hadrpod`, `--hadrdb2` – Direct link to the Db2 Shift scenario found on the main menu
* `--logs` – View logs from last shift execution

Details of how to navigate the UI system are found in the [User Inteface Overview](gui.md) section.

## Syntax Summary

The following is a list of all options that are available with the Db2 Shift command:

<pre><code class="language-bash">db2shift

   Operation 
   --mode=[all,move,clone,apply_clone,sec_and_monitor,hadr_setup,
           push_clone,pull_clone]

   UI Mode Only
   --help
   --mono
   --accept

   Client
   --ssh,--local,--oc,--kubectl

   Source Details
   --source-database="sample"
   --source-owner="db2inst1"

   Destination Details
   --dest-database="bludb"
   --dest-owner="db2inst1"
   --dest-server="c-demo-db2u-0" or "userid@ip.address"
   --dest-namespace="db2u", --dest-project="db2u"
   --overrides="instance_memory 3468896"

   Clone and Copy Details
    --clone-dir="/tmp/clone"
    --local-dir="/tmp/clone"
    --dest-dir="/tmp/clone"

   HADR Options
   --hadr
   --source-hadr-server="some.server.com"
   --dest-hadr-server="other.server.com"
   --source-hadr-port=3700
   --dest-hadr-port=3700

   Meta Data Generation
   --blank-slate=[True|False], --gen-settings
   --verify-only

   Synchronization Mode
   --sync=[start_sync, rerun_sync, finish_sync]   

   Database Status 
   --online, --offline

   Performance Options
   --threads=[1..8]
   --compression=[0..9]

   Function Library Support
   --exclude-functions
</code></pre>

Details for each one of these options can be found in the next section or by clicking on one of the following links.

* [Mode Option](reference.md#mode-option)
* [Target Client Instance](reference.md#target-client-instance-to-instance)
* [Target Client POD](reference.md#target-client-instance-to-pod)
* [Source Database](reference.md#source-database)
* [Source Owner](reference.md#source-or-instance-owner)
* [Destination Database](reference.md#destination-database)
* [Destination Owner](reference.md#destination-owner)
* [Destination Server POD](reference.md#destination-server-pod)
* [Destination Server Instance](reference.md#destination-server-instance)
* [Destination Pod Namespace or Project](reference.md#destination-pod-namespace-or-project)
* [Clone Directory](reference.md#clone-directory)
* [Local Directory](reference.md#source-clone-directory)
* [Target Directory](reference.md#target-clone-directory)
* [HADR Setup](reference.md#hadr-setup)
* [HADR Source or Destination Server](reference.md#hadr-source-or-destination-server)
* [HADR Ports](reference.md#hadr-ports)
* [Metadata Generation](reference.md#metadata-generation)
* [Synchronization Options](reference.md#synchronization-options)
* [Online/Offline Mode](reference.md#online-or-offline-move)
* [Threading](reference.md#threading)
* [Compression](reference.md#compression)
* [Stored Procedures](reference.md#stored-procedures-and-functions)
* [Overrides](reference.md#overrides)