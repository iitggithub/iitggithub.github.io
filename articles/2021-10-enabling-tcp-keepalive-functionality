## Enabling TCP Keepalive Functionality For Legacy Linux Applications

Wednesday, 27 October 2021

### Problem

You want to enable TCP keep alive functionality but the application either doesn't support or is being overriden by the application itself.

You may have tried (and failed) to configure this using the sysctl parameters mentioned below to no avail. As a result the connection eventually times out or is closed on its own.

The sysctl parameters you may have tried to configure are:

```
net.ipv4.tcp_keepalive_time = 60
net.ipv4.tcp_keepalive_intvl = 10
net.ipv4.tcp_keepalive_probes = 6
```

If you have tried to configure the above-mentioned parameters and you still aren't seeing TCP Keepalive functionality enabled then this article may be of use to you.

### Solution

You can install the libkeepalive library and using the LD_PRELOAD environment variable, instruct the application to load the library and enable TCP Keepalive functionality.

#### Quick Setup Guide

```
[ec2-user@ip-172-31-30-6 ~]$ curl -O libkeepalive-0.3.tar.gz http://prdownloads.sourceforge.net/libkeepalive/libkeepalive-0.3.tar.gz?download
[ec2-user@ip-172-31-30-6 ~]$ tar zxf libkeepalive-0.3.tar.gz
[ec2-user@ip-172-31-30-6 ~]$ cd libkeepalive-0.3
[ec2-user@ip-172-31-30-6 libkeepalive-0.3]$ make
[ec2-user@ip-172-31-30-6 libkeepalive-0.3]$ sudo cp libkeepalive.so /usr/lib64
[ec2-user@ip-172-31-30-6 libkeepalive-0.3]$ export LD_PRELOAD=/usr/lib64/libkeepalive.so
[ec2-user@ip-172-31-30-6 libkeepalive-0.3]$ export KEEPCNT=20
[ec2-user@ip-172-31-30-6 libkeepalive-0.3]$ export KEEPIDLE=75
[ec2-user@ip-172-31-30-6 libkeepalive-0.3]$ export KEEPINTVL=60
[ec2-user@ip-172-31-30-6 libkeepalive-0.3]$ /path/to/myapplication
```

#### How It Works

When /path/to/myapplication executes, the OS will preload the libkeepalive.so library enabling TCP keepalive functionality for newly created TCP sockets in accordance with the KEEPCNT, KEEPIDLE, and KEEPINTVL environment variables. I have tested this using the nc command to create a process that listens on TCP port 5000. Run the nc command via strace, and you'll see what happens when TCP Keepalives are not enabled:

```
[ec2-user@ip-172-31-30-6 ~]$ strace nc -l 5000 2>&1 | grep setsockopt
setsockopt(3, SOL_SOCKET, SO_REUSEADDR, [1], 4) = 0
setsockopt(3, SOL_IPV6, IPV6_V6ONLY, [1], 4) = 0
setsockopt(4, SOL_SOCKET, SO_REUSEADDR, [1], 4) = 0
^C
[ec2-user@ip-172-31-30-6 ~]$
```

We set that the SO_KEEPALIVE socket option has not been enabled nor have any of the other Keepalive related settings. Now let's review the difference when TCP Keepalives are enabled for the same command:

```
[ec2-user@ip-172-31-30-6 ~]$ export LD_PRELOAD=/usr/lib64/libkeepalive.so
[ec2-user@ip-172-31-30-6 ~]$ export KEEPCNT=20
[ec2-user@ip-172-31-30-6 ~]$ export KEEPIDLE=75
[ec2-user@ip-172-31-30-6 ~]$ export KEEPINTVL=60
[ec2-user@ip-172-31-30-6 ~]$ strace nc -l 5000 2>&1 | grep setsockopt
setsockopt(3, SOL_SOCKET, SO_KEEPALIVE, [1], 4) = 0
setsockopt(3, SOL_TCP, TCP_KEEPCNT, [20], 4) = 0
setsockopt(3, SOL_TCP, TCP_KEEPIDLE, [75], 4) = 0
setsockopt(3, SOL_TCP, TCP_KEEPINTVL, [60], 4) = 0
setsockopt(3, SOL_SOCKET, SO_REUSEADDR, [1], 4) = 0
setsockopt(3, SOL_IPV6, IPV6_V6ONLY, [1], 4) = 0
setsockopt(4, SOL_SOCKET, SO_KEEPALIVE, [1], 4) = 0
setsockopt(4, SOL_TCP, TCP_KEEPCNT, [20], 4) = 0
setsockopt(4, SOL_TCP, TCP_KEEPIDLE, [75], 4) = 0
setsockopt(4, SOL_TCP, TCP_KEEPINTVL, [60], 4) = 0
setsockopt(4, SOL_SOCKET, SO_REUSEADDR, [1], 4) = 0
^C
[ec2-user@ip-172-31-30-6 ~]$
```

After using telnet to connect to TCP port 5000, we can use the netstat (or ss) command and see that a Keepalive timer is being used which further confirms that we've enabled TCP Keepalive functionality successfully for the telnet session.

```
[ec2-user@ip-172-31-30-6 ~]$ sudo netstat -tnopea
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name     Timer
...
tcp        0      0 127.0.0.1:5000          127.0.0.1:36246         ESTABLISHED 1000       1886232    24629/nc             keepalive (71.16/0/0)
```
