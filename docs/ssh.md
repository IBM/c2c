# Password-less SSH
 
The Db2 Shift utility requires the use of password-less SSH in order to shift a database between two
Db2 instances. SSH is not used when shifting a database to a Db2U instance in OpenShift, Kubernetes
or Cloud Pak for Data.

You must set up Password-less SSH between your source server
and target server in order to use Db2 Shift. The Db2 Shift
command issues commands to the target server that requires local 
access and proper authentication. Normally the the ssh command requires
a password to be supplied whenever you are connecting to a
server. This can become tedious if you are constantly
re-connecting to the server to issue a command.

From a Db2 Shift perspective, the program will never prompt for a password. Db2 Shift requires
that a password-less SSH connection is established between
the HOST and TARGET instances. If password-less ssh has not been
established, the utility will stop and issue an error message.

To create a password-less connect to another server, you must generate
an ssh key.

From a terminal connection on the source machine, issue the following command.
```
ssh-keygen -t rsa -b 4096 -C "db2shift@myserver.com"
```

The value in quotes `"db2shift@myserver.com"` can be any value you want. 
This command will generate a ssh key that will be used to
synchronize with the other server. Output from the command will be similar
to the following: 

```
Generating public/private rsa key pair.

Enter file in which to save the key (/home/db2inst1/.ssh/id_rsa): 
```

Use the default value (press Return). If the file exits, it
will prompt you if you want to override the file (yes).

```
/home/db2inst1/.ssh/id_rsa already exists.
Overwrite (y/n)? y

Created directory '/home/db2inst1/.ssh'.
Enter passphrase (empty for no passphrase): 
```
Leave this value blank for this and the next question.
```
Enter same passphrase again: 
Your identification has been saved in /home/db2inst1/.ssh/id_rsa.
Your public key has been saved in /home/db2inst1/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:LvzoiRx5NjnDVpfPC+nyyUCDd6PzAq4GBjGmbKEu+tU db2shift@skytap.com
The key's randomart image is:
+---[RSA 4096]----+
|                 |
|oo               |
|=o.              |
|+o     .   .     |
|o.    . S =      |
|..o  =.* = =     |
|o. .+.E.= o o    |
|.  o.*.Oo* o .   |
| ...+o+ .+* .    |
+----[SHA256]-----+ 
```
At this point the key has been generated and must be placed
into the destination server.  The command used to copy the
key to destination server is called `ssh-copy-id`. Issue the
`ssh-copy-id` command and include the userid and server name that
you want to create the connection with.
```
ssh-copy-id db2inst1@10.0.0.2

/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/db2inst1/.ssh/id_rsa.pub"

The authenticity of host '10.0.0.2 (10.0.0.2)' can't be established.
ECDSA key fingerprint is SHA256:+GJj8KYH4RS1zdaJ+McoZTCz493XdaP3M56u3uQOPKU.
ECDSA key fingerprint is MD5:43:b5:e8:b7:fd:83:7d:ac:cf:4c:44:c3:9d:57:6a:30.
Are you sure you want to continue connecting (yes/no)? 
```
Make sure to answer yes to continue connecting.
```
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
db2inst1@10.0.0.2's password: 
```
Enter the password of the target user.
```
Number of key(s) added: 1

Now try logging into the machine, with: "ssh 'db2inst1@10.0.0.2'" and check to make sure that only the key(s) you wanted were added.
```
At this point you have established a password-less
connection between the source and target servers.You will
need to repeat this same step for every target server you want 
to shift Db2 databases to.

Once you have done this you will be able to run the Db2 Shift command 
successfully between two Db2 instances.