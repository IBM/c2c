# Db2 Shift Installation

## Installation

The Db2 Shift program (`db2shift`) is a single executable
image that can be run directly on Linux. This is a bundled
application which means that it contains files and settings
that are part of the executable code. From a user
perspective, no additional software is required to run it.

The program does not create any directories on your system,
but it will generate files during the execution of a shift.
A best practice would be to create a new directory only for
the purposes of the running the Db2 Shift code.

The Db2 Shift program is bundled into a single zip file
which contains version of the code for different Linux
distributions (there may be more than those listed). 

* CentOS 6, Red Hat 6 - db2shift-centos6
* CentOS 7, Red Hat 7 - db2shift-centos7
* CentOS 8, CentOS Stream, Red Hat 8 - db2shift-centos8
* Ubuntu 18.04 - db2shift-ubuntu18
* Ubuntu 20.04 - db2shift-ubuntu20
* SUSE 11.4 - db2shift-suse11
* Power Linux - db2shift-powerLE

**Note**: The current Technical Preview site provides the
ability to download individual versions of the Db2 Shift
program rather than a bundled zip file.

Rename the Db2 Shift version that is suitable for your
environment to db2shift:

```
mv db2shift-centos7 db2shift
```

Place the program into its own directory and make sure that it has the execution bit set.

```
chmod +x ./db2shift
```
Additional versions may be added depending on feedback from users. 

The program will work in command mode for all environments.
If you attempt to use the UI mode with the incorrect OS
version, the screen may look distorted.

![Bad Version](img/c2c_bad_version.png)

If your display looks like this, then you will have the
wrong version installed. If you are running on a different
platform than those listed above, please contact us
directly. See Appendix B for details.

## Program Download

The program is currently in Technical Preview and is available from the following download site:

* [Db2 Click to Containerize Technical Preview](https://ibm.biz/c2cdownload)

You will need an IBM userid to register and gain access to the program.
 


