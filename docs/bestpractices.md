# Shift Considerations

There are several things you should consider before
performing a Db2 Shift. Many of these recommendations
are evolving and updates will be added to this document on a
regular basis.

## Database Migration

The Db2 Shift command will migrate a database from Version
10.5 and 11.1 to the latest version of Db2 (11.5). Databases
that are at older release levels will need to be migrated to
10.5 or 11.1 before using the Db2 Shift utility.

When performing a Db2 Shift on an older release, the
database must be shut down or DEACTIVATED before a shift can
be performed. All Db2 upgrades require that the database be
in a fully consistent state for the upgrade to take place.
The utility will fail if the database was in a WRITE SUSPEND
state.

If the Db2 Shift utility is run against a database which
requires migration, it will check to see if `--offline` or
`--online` was supplied in the command line or in the user
interface. If `--offline` was set, the Db2 Shift utility will
shift the files assuming the database is offline. If the
database was not offline, the upgrade at the target will
fail because the control information in the database files
will not be consistent.

If `--online` was selected, and a migration is required, Db2
will force the database into a DEACTIVATED state. This will
cause a database outage and potentially rollback
transactions that were in progress. Make sure that the
database does not have any workloads running against it if
you decide to shift when using the `--online` mode. Note that
having a database in DEACTIVATED state is not a guarantee
that no one can connect to the database.

The other option to reduce outage time for a database is to
use the clone option. A clone copy can be used for an
upgrade and the database can be write suspended when using
this mode of operation.

In summary, when shifting a database that needs an upgrade,
using one of the following techniques:

* Shut down the database and use the `--offline` flag
* Use the `--online` flag and let Db2 DEACTIVATE the database
* Clone the database (and deploy later) using the `--online` flag
 
## Online Versus Offline Mode

Db2 Shift provides two options when dealing with the state
of a database during the shift process:

* `--online` mode - the database is online while the shift is taking place
* `--offline` mode - the database has been shutdown

### Online Mode

Online mode is the default value for all shifts. While this
can be used for shifting databases that require an upgrade,
the recommendation is to use `--offline` mode instead.

When shifting a database in `--online` mode, the file copy
step will not affect the status of the database. Workloads
can continue to run while the copying is taking place. Note
that the buffer pools may be affected by the reading process
so there may be lower cache utilization while the utility is
running.

During the last step of the shift process, Db2 Shift will
search for any updates that may have been applied to the
database after the initial copying was done. To ensure the
integrity of the data being copied, the database will be
placed into a WRITE SUSPEND mode. This last step should only
take a few seconds and the database will be WRITE RESUMED
when the final copy is done.

When the database is suspended, all read activities will
continue. New connections will not be permitted, but any
applications that are currently running will be allowed to
continue. Those transactions that are updating records will
be temporarily "paused" while the copying is done.

Once the copy is complete, a suspended application will
finish their transaction. Online mode provides the least
disruption to an active database but is not recommended for
databases that being upgrade.

### Offline Mode

Offline mode should be used for database migrations. If it
is not used, the Db2 Shift utility will DEACTIVATE the
database before continuing.

Offline mode indicates to Db2 Shift that the database has
either been DEACTIVATED or the instance has been shut down.
If you want to use offline mode, you must first gather the
control information while the database is online. To do
this, either through the command, or the UI, select
`--blank-slate=true` `--verify`.

The `--blank-slate=true` option will gather the source and
target control information that is required to do a shift.
If the source database was already offline, this information
would not available.

Adding the `--verify` option will check that all the settings
are correct between the source and target and stop execution
(i.e., the shift will not occur).

After your shut down the source database you would use the
option `--blank-slate=false`. This will force Db2 Shift to use
the existing control information that you generated in the
previous step. At this point the database will be a
consistent state and the upgrade step will proceed at the
target location.

## Instance Ownership

If you are moving a database that was created with an
instance owner that is not found on the target system, you
must make sure to add the instance owner at the target into
the current database before shifting the database.

Example: Database CQW was created by instance owner
`db2inst2`. This database is then cloned to another instance
that has `db2inst1` as the instance owner. When `db2inst1`
attempts to access the database, they will be unable to
manage it because they do not have `DBADM` or `SECADM`
privileges on the tables created by `db2inst2`.

There are two ways to solve this. One is to create the
userid `db2inst2` at the destination location for a
traditional instance. You would then be able to connect to
the database as `db2inst2` and grant `DBADM` and `SECADM` back to
`db2inst1`. However, this approach will not work with Db2U and
CP4D installations because there is no mechanism available
within the POD to create a new OS-level userid.

The second (and easier) method is to add the destination
instance owner to the source database.

```
GRANT SECADM ON DATABASE TO USER db2inst1
GRANT DBADM  ON DATABASE TO USER db2inst1
```

The `db2inst1` userid does not need to exist in the Operating
system to grant these privileges to the userid. Once the
database is shifted to the new location, the instance owner
`db2inst1` will have full access to the database.

## Target Db2 Environment

The Db2 Shift program can clone an existing Db2 database into two environments:

* Db2U Pod running on OpenShift, Kubernetes, Cloud Pak for Data
* Db2 Instance on premise, on a virtual machine (on premise or Cloud)

### Cloning into a Containerized Environment

For instances where you are shifting to a Db2U POD, the POD must already exist and the
database must have been created. Depending on what you plan to do with the target
database, the database name must be:

* The same as the SOURCE database if:

    * You are connecting the source and target with HADR
    * You are cloning the database into Cloud Pak for Data
    * You want to keep the same database name

* Any valid database name

If you move a database (i.e. SAMPLE) into a Db2U POD database called DB2OLTP, the
contents of the database SAMPLE will end up in DB2OLTP. In other words, the SAMPLE
database is shifted and renamed to DB2OLTP. If you want to have the same database name,
then you must create the database with the same name.

### Cloning into a Traditional Db2 Instance

When cloning a Db2 database into a traditional instance, the Db2 database can already
exist, or you can ask that the database be generated for you at shift time. The Db2
instance must be available and running on the server.

Syntax: `--dest-create-db`
![Target Database](img/field_force_create.png)

To create the database at shift time, make sure the select the `Force Database Creation`
setting in the UI menu or use `--dest-db-create` flag in the command line. The Db2 Shift program
will attempt to create the database in a directory structure under `/db2_portable` on the
target system.

When cloning into a traditional Db2 instance, similar rules apply to the database name.
The database name must be:

* The same as the SOURCE database if:

    * You are connecting the source and target with HADR
    * You want to keep the same database name

* Any valid database name

## Database Storage Relocation

When shifting a database to a POD or to another Db2 instance, the storage
paths will need to be modified in order for Db2 to run. The
`dft_paths.cfg` file contains information on where the
Db2 Shift utility will place database objects on the destination server. If
the `dft_paths.cfg` file does not already exist in the directory where the
Db2 Shift utility is running, the file will be created during the first
execution of the utility.

The `dft_paths.cfg` file contains 2 sets of paths, one for DB2u Pod/Container
deployments and the other for standard Instance installs
classified as server-type `other`.

You can alter the type “other” paths in the file as long as you do not change the
`DFT_STORAGE_PATH` variable. The `{}` characters in a path signifies where `$HOME`
will be substituted into the string. If the `{}` characters are removed, the absolute
path will be used.

A copy of the file is shown below. The format of the file is:

```
PATH | Directory | type
```

Where:

* Path Type - The type of path that the directory represents
* Directory - The directory that will be used for the path type
* Target - `pod` for Db2U deployments and `other` for Db2 instance deployments


```
MIRRORLOG_PATH|/mnt/bludata0/db2/mirrorlog|pod
FAILARCHIVE_PATH|/mnt/bludata0/db2/failarchive|pod
LOGARCHMETH1|/mnt/bludata0/db2/archive_log|pod
LOGARCHMETH2|/mnt/bludata0/db2/archive_log2|pod
OVERFLOWLOG_PATH|/mnt/bludata0/db2/overflowlog|pod
DFT_STORAGE_PATH|/mnt/bludata0/db2/databases/|pod
OTHER_STORAGE_PATH|/mnt/bludata0/db2/databases/SPATH|pod
OTHER_LOG_PATH|/mnt/bludata0/db2/dblogs|pod
MIRRORLOG_PATH|{}/db2_portable/mirror_logs/mirrorlog|other
FAILARCHIVE_PATH|{}/db2_portable/failarchive|other
LOGARCHMETH1|{}/db2_portable/archive_log|other
LOGARCHMETH2|{}/db2_portable/archive_log_mirror|other
OVERFLOWLOG_PATH|{}/db2_portable/overflow_logs/overflowlog|other
DFT_STORAGE_PATH|{}|other
OTHER_LOG_PATH|{}/db2_portable/primary_logs/dblogs|other
NEW_DB_ON|/db2_portable/databases|other
OTHER_STORAGE_PATH|{}/db2_portable/alt_store_paths/SPATH|other
```

You must update this file **prior** to running a shift operation. The Db2 Shift utility
will use the existing `dft_paths.cfg` file to relocate the directories into the
target location.

**Note**: The `dft_path.cfg` is setup to transfer multiple databases and their paths under 
the `/$HOME/db2_portable` directory.

The following paths, as defined in `dft_paths.cfg` 
can be modified using the `mount` utility on Linux to point them to a different directory.

```
MIRRORLOG_PATH|{}/db2_portable/mirror_logs/
FAILARCHIVE_PATH|{}/db2_portable/failarchive/
LOGARCHMETH1|{}/db2_portable/archive_log/
LOGARCHMETH2|{}/db2_portable/archive_log_mirror/
OVERFLOWLOG_PATH|{}/db2_portable/overflow_logs/
OTHER_LOG_PATH|{}/db2_portable/primary_logs/
```
Note that the mount path depends on the path type e.g. `MIRRORLOG_PATH` 
is not necessarily at the end of the path defined in the default config. 
That is because of the unique naming requirement for paths and the assumption 
that paths need to be multi-tenant capable.

For instance:

* Mirror log path: `MIRRORLOG_PATH = {}/db2_portable/mirror_logs/mirrorlog`
* Mount path for db2inst1 alternate mirrored logs volume: `/home/db2inst1/db2_portable/mirror_logs`

If we were to change the config file simply to point at the
mirror logs path using an alternate absolute path, your
config file entry would look like:

```
MIRRORLOG_PATH|/mnt/mirror_logs/mirrorlog|other
```

For your absolute path you would need to create the directory
prior to the shift :

```
mkdir /mnt/mirror_logs
```

**Note**: The Db2 Shift tool creates the `dft_paths.cfg` on startup if
it does not already exist. If you need to recreate the
file with default values, simply delete the file. If the file
already exists in the directory, it will not be recreated by
the tool.

**Note**: `DFT_STORAGE_PATH` and `NEW_DB_ON` paths should not
be changed. `NEW_DB_ON` controls where any database created
by the tool goes on the target instance, and this path is
always relative to the `$HOME` directory for the instance. If
you want the database created elsewhere, manually create it
first before running the tool. The other path changes,
as per above, will be observed. 

For alternate Storage paths you can mount the volume at a
lower level but you must know how many paths will be
included. This can be determined by looking at the `new0.cfg`
file after a generate settings run.

Syntax: `--blank-slate=true`, `--gen-settings`
![Meta Data](img/field_settings_only.png)

The `--gen-settings` (Generate Settings) option is used in
conjunction with `--blank-slate`. The use of
`--gen-settings` will prevent Db2 Shift from continuing
execution after the metadata files have been created.

Storage can also be set up by
MLN or NODE.

```
OTHER_STORAGE_PATH|{}/db2_portable/alt_store_paths/SPATH[1..N]/NODE000X
```

For Example:

```
/home/db2inst2/db2_portable/alt_store_paths/SPATH1/db2inst2/NODE0000
/home/db2inst2/db2_portable/alt_store_paths/SPATH1/db2inst2/NODE0001
/home/db2inst2/db2_portable/alt_store_paths/SPATH1/db2inst2/NODE0002
/home/db2inst2/db2_portable/alt_store_paths/SPATH1/db2inst2/NODE0003
/home/db2inst2/db2_portable/alt_store_paths/SPATH2/db2inst2/NODE0000
/home/db2inst2/db2_portable/alt_store_paths/SPATH2/db2inst2/NODE0001
/home/db2inst2/db2_portable/alt_store_paths/SPATH2/db2inst2/NODE0002
/home/db2inst2/db2_portable/alt_store_paths/SPATH2/db2inst2/NODE0003
/home/db2inst2/db2_portable/alt_store_paths/SPATH3/db2inst2/NODE0000
/home/db2inst2/db2_portable/alt_store_paths/SPATH3/db2inst2/NODE0001
/home/db2inst2/db2_portable/alt_store_paths/SPATH3/db2inst2/NODE0002
/home/db2inst2/db2_portable/alt_store_paths/SPATH3/db2inst2/NODE0003
```

You will have to pre-create the paths at the target server using this pattern
prior to shifting the database into it.

If you apply the mount at a lower point in the directory
structure, it will likely be deleted in the Db2 Shift process.

**Note**: No changes to the `dft_paths.cfg` are
supported for moves to containers which are identified with the 
`pod` keyword at the end of a control line.

## Encrypted Databases

The Db2 Shift utility can only shift encryption keys that are available
locally to the database. If your database uses an enterprise key manager then
you will need to register that key manager at the target pod or instance.

If you are shifting to another Db2 instance or pod that contains multiple
databases, you cannot shift the key file. During a shift, the target key file is
completely replaced which would remove any keys being used by existing
databases at the target location. If you want to shift the database to the new
location, you will have to extract the current key and reimport it into the target
key manager.

## Database, Instance, and Environment Settings

The Db2 Shift utility will maintain all database settings when the
database is shifted to a different Db2 instance or pod. However, the
Instance and Environment settings are not moved.

The Db2 Shift provides an option to override the target instance settings.
If a particular setting needs to be updated, you can provide the value as
part of the UI or command line. An example of a setting that you may want to
update during a shift is the `INSTANCE_MEMORY` and the `INTRA_PARALLEL` settings.

You should ensure that any change you make at the instance level are reflected in the
Db2U POD configuration in the event the Db2U code is replaced.

Environment settings are not handled by the Db2 Shift program. You must ensure that the
instance or Db2U POD have the appropriate environment variables set before you shift
the database. For instance, if you are shifting a Db2 database that has Oracle
compatibility turned on, you must create an empty Db2 database with this setting
turned ON before shifting the original database. There is no way to set the
compatibility flag after the database has been created.

## Threads and Compression

One area that is still evolving is the use of threads and compression when shifting a database into a cloud environment. The Db2 Shift tool has been set to use default settings that give the best performance in networks that have 1Gb/s throughput. These settings will need to be adjusted based on your network and machine characteristics.
The following are some general observations from early testing.

### Threads

Using a higher number of threads is useful in situations
where the Db2 database has multiple paths defined for a
storage group. If the database was created with a single
storage path, the threads are competing for I/O on the same
device.

Increasing the number of threads will not necessarily result
in more throughput. Testing has shown that 4 threads is a
good starting point for parallelism, with small incremental
benefits as you increase the number. You need to balance the
CPU usage by the Shift utility and the impact on workloads
running on the server.

If the Db2 Shift command is being used for gradual shifts
(initial, refresh), the number of threads should be set to 1
to reduce the overhead on the server. Once the final shift
is run you may want to increase the value to minimize the
suspend time of the database.

If there is no contention on the server then the maximum
thread count should be 1 less than the number of VPCs of the
server or 8, whichever is less. For a cloud instance that is
defined with 4 cores, 8 VPCs (SMTx2), the thread count
should be a maximum of 7 (8 VPCs - 1).  For larger servers,
a maximum of 8 threads can be used.  

### Test Scenario

The Db2 Shift command was run from a standard Db2 on Linux instance
and shifting the database to a Kubernetes
cluster with Db2U installed. The database size was
approximately 50Gb with 260+M rows of transactional data.

The best elapsed time was 237 seconds (131s copy time only).
Increasing the number of threads did not make a difference
to the total elapsed time. Examining the CPU usage, the
average utilization of the threads was almost identical
between 4 and 8 threads.

The network throughput appears to track with the CPU usage,
demonstrating that the CPUs are busy transmitting
as much data onto the network as possible.

The results showed that the maximum throughput was capped by
the network limit of 5Gb/s. The network throughput was:

* 1 thread was 1240 Mb/sec
* 4 threads were 4890 Mb/sec

A CPU thread has a limit on how much data it can push onto
the network. By running a test on a single thread, you can
determine how many cores you can effectively use during a
Shift run. Dividing the network capacity by the single core
performance will determine the optimal number of threads to
use.

Example:

* Throughput of one thread is approximately 1.2Gb/s
* Network limit is 5Gb/s
* 5/1.2 is approximately 4 threads

This result can also be used to determine your total copy time:

* Database Size/(Network Limit/8) = elapsed time
* 50 GB/(5Gbs/8) = 50GB/(.625GBs) = 80s with ideal conditions
* Tests results were 50GB/(4.89Gbs/8) = 128s ideal (131 observed)

**Note**: These test results were run in a laboratory environment
with ideal conditions and
may not be applicable to your installation. You are advised to test
the throughput on your own system to determine the performance of the
Db2 Shift utility.

### Compression

Compression will significantly reduce throughput if you are
CPU constrained. If the database is encrypted, uses
compression, or has a high ratio of numeric values over
character values, adding compression will not help
throughput. In some cases, this may significantly reduce
throughput.

The recommendation is to use compression only if you have a
very slow network (<1Gbps). For these types of situations,
the compression may result in less data being transferred
which will reduce the load on the network. Aside from this
situation, leaving compression at 0 is the best strategy for
initial Db2 Shift tests.
