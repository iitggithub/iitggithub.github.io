## How to fix chmod execute permissions

Wednesday, 27 October 2021

### Problem

You've run something like the following and accidentally removed the execute permission from /bin/chmod:

```
[ec2-user@ip-172-31-30-6 ~]$ sudo chmod -x /bin/chmod
[ec2-user@ip-172-31-30-6 ~]$ ls -l /bin/chmod
-rw-r--r-- 1 root root 54384 Jan 23  2020 /bin/chmod
...
[root@ip-172-31-30-6 ~]# /bin/chmod +x /usr/bin/netstat 
-bash: /bin/chmod: Permission denied
```

Now you can't execute chmod, to change the permissions on any files on the system including chmod itself. Below are a couple of ways to fix it.

### Solution

#### Use the ld.so and ld-linux.so\* dynamic loader to execute chmod

According to its [man page(https://linux.die.net/man/8/ld-linux)], "The programs ld.so and ld-linux.so* find and load the shared libraries needed by a program, prepare the program to run, and then run it.".

We can use this to execute chmod despite the fact it doesn't have execute permissions, and undo our mistake. Before doing so, we first need to find the ld linux binary. In Amazon Linux 2, I found ld.so under /usr/lib64/ld-2.26.so.

```
[ec2-user@ip-172-31-30-6 ~]$ sudo find /usr/lib64 -name "ld*.so*"
/usr/lib64/ld-2.26.so
/usr/lib64/ld-linux-x86-64.so.2
...
```

Now that we've found them we can use either one of them to execute chmod:

```
[ec2-user@ip-172-31-30-6 ~]$ ls -l /bin/chmod
-rw-r--r-- 1 root root 54384 Jan 23  2020 /bin/chmod
[ec2-user@ip-172-31-30-6 ~]$ sudo /usr/lib64/ld-2.26.so /bin/chmod +x /bin/chmod
```

Finally we verify that the issue is resolved and we can execute chmod to our hearts content:

```
[ec2-user@ip-172-31-30-6 ~]$ ls -l /bin/chmod
-rwxr-xr-x 1 root root 54384 Jan 23  2020 /bin/chmod
[ec2-user@ip-172-31-30-6 ~]$ sudo /bin/chmod +x /usr/bin/netstat
[ec2-user@ip-172-31-30-6 ~]$ 
```

#### Using Perl

Interestingly enough, perl has its chmod function [built in(https://perldoc.perl.org/functions/chmod)]. Why? I have no idea, but we can use it to fix the chmod binary.

An example is shown below:

```
[ec2-user@ip-172-31-30-6 ~]$ sudo chmod -x /bin/chmod
[ec2-user@ip-172-31-30-6 ~]$ ls -l /bin/chmod
-rw-r--r-- 1 root root 54384 Jan 23  2020 /bin/chmod
[ec2-user@ip-172-31-30-6 ~]$ sudo perl -e 'chmod(0755, "/bin/chmod")'
[ec2-user@ip-172-31-30-6 ~]$ ls -l /bin/chmod
-rwxr-xr-x 1 root root 54384 Jan 23  2020 /bin/chmod
[ec2-user@ip-172-31-30-6 ~]$ 
```

As you can see chmod has execute permissions once again.

#### Rsync the chmod binary from another server

If you have the ability to rsync /bin/chmod from another server you can use the following command as an example to pull the file. This will replace the existing chmod file, including file metadata (such as execute permissions).

```
[ec2-user@ip-172-31-30-6 ~]$ ls -l /bin/chmod
-rw-r--r-- 1 root root 54384 Jan 23  2020 /bin/chmod
[ec2-user@ip-172-31-30-6 ~]$ rsync -av SOURCE_SERVER:/bin/chmod /tmp/chmod
...
[ec2-user@ip-172-31-30-6 ~]$ sudo mv /tmp/chmod /bin/chmod
[ec2-user@ip-172-31-30-6 ~]$ ls -l /bin/chmod
-rwxr-xr-x 1 root root 54384 Jan 23  2020 /bin/chmod
[ec2-user@ip-172-31-30-6 ~]$ 
```

Note that you'll need to update SOURCE_SERVER with the IP address or DNS hostname of the source server.

When I have done this in the past, I've used the same OS i.e. Amazon Linux 2. I'm not sure if this would work if it were a completely different OS.

#### Making a copy and replacing its contents

This solutions requires making a copy of an existing binary which does have execute permissions, and then rsync'ing the contents of the existing broken chmod binary to our copied file before moving the copied file to replace the /bin/chmod that's broken. This one is probably better explained with an example.

```
[ec2-user@ip-172-31-30-6 ~]$ ls -l /bin/chmod
-rw-r--r-- 1 root root 54384 Jan 23  2020 /bin/chmod
[ec2-user@ip-172-31-30-6 ~]$ sudo cp /bin/chown /bin/chmod2
[ec2-user@ip-172-31-30-6 ~]$ sudo rsync /bin/chmod /bin/chmod2
[ec2-user@ip-172-31-30-6 ~]$ sudo /bin/chmod2 +x /bin/chmod
[ec2-user@ip-172-31-30-6 ~]$ sudo rm -f /bin/chmod2
[ec2-user@ip-172-31-30-6 ~]$ ls -l /bin/chmod
-rwxr-xr-x 1 root root 54384 Jan 23  2020 /bin/chmod
[ec2-user@ip-172-31-30-6 ~]$ 
```

#### Using a Live CD

Unfortunately, I can't go into much detail on this one as its dependant on what kind of OS you're running. Essentially you'd boot the machine with the Live CD, mount the old root volume to a temporary location, and use the Live CD's version of chmod to make your broken chmod executable once again.
