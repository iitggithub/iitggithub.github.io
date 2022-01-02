## udev: renamed network interface eth0 to eth1

### Problem

You can't access your linux machine across the network as it's not responding to ping, and ports aren't open etc.

You've checked and your network interface no longer exists and you see the following message in dmesg:

```
udev: renamed network interface eth0 to eth1
```

or you may see something like:

```
ena 0000:00:05.0 eth1: renamed from eth0
```

The network interface has been renamed and as a result, the network fails to start, and the host isn't accessible.

### Solution

There are a couple of ways to solve this issue, but both will require rebooting the machine.

#### Short Term Solution

In the /etc/udev/rules.d directory, there is a udev rule file ending with "-persistent-net.rules". Usually this file will be prepended with a number (such as 70) which defines the order in which udev rules are processed. Delete the file, and when the OS is started again, the file will be generated from scratch and the network interface will not be renamed to eth1..

```
$ sudo rm -vf /etc/udev/rules.d/70-persistent-net.rules
$ sudo reboot
```

If you also see the following message in the console log and the network interface is attached as eth0, then you'll need to check the /etc/sysconfig/network-scripts (or the OS equivalent) to make sure that your network configuration scripts are named correctly ie. ifcfg-eth0 rather than ifcfg-eth1, and the DEVICE and NAME parameters match the device name (eth0).

```
Bringing up interface eth1: Determining IP information for eth1... done.
```

An example is shown below of what it's supposed to look like:

```
[ec2-user@ip-172-31-28-71 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-eth0 
DEVICE=eth0
BOOTPROTO=dhcp
ONBOOT=yes
TYPE=Ethernet
```

#### Long Term Solution

Depending on the operating system, in /lib/udev or /usr/lib/udev there is a bash script called "write_net_rules ". In this file, you'll find the section of code towards the bottom of the file which renames the network device if a rule already exists. The exact section of code i'm referring to is outlined below.

To stop the issue from re-occurring in future, hash out the portion of code below before proceeding to the next step. When you're done, it should look like this;

```
#else
#        # if a rule using the current name already exists, find a new name
#        if interface_name_taken; then
#                INTERFACE="$basename$(find_next_available "$basename[0-9]*")"
#                # prevent INTERFACE from being "eth" instead of "eth0"
#                [ "$INTERFACE" = "${INTERFACE%%[ \[\]0-9]*}" ] && INTERFACE=${INTERFACE}0
#                echo "INTERFACE_NEW=$INTERFACE"
#        fi
```

Once you've done that, you'll need to remove the persistent-net.rules file as per the "Short Term Solution" above.

#### Why Is This Happening?

When you launch a new machine from an existing image or snapshot, udev will have an existing rule for eth0. An example of this rule is shown below.

```
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="01:23:45:67:89:ab", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"
```

Simply put, when the conditions above are met, the device name is set to eth0.

When a new machine is launched which has not been patched, it launches with an network interface that has a different MAC address. Because of this, the new network interface will not match against the above-mentioned udev rule.

The write_net_rules script will then add a new entry to the persistent-net.rules file for the new network interface. However, since a rule already exists for eth0, udev changes the device name to a device name which isn't already "In use". The script will increase the interface number by one which effectively forces eth0 to be renamed to eth1 because it fails to match on the first rule, and succeeds on the second.

If the customer experiences this behaviour, you should see similar output to what's shown below in the persistent-net.rules file.

```
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="01:23:45:67:89:ab", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="ab:cd:ef:09:87:65", ATTR{type}=="1", KERNEL=="eth*", NAME="eth1"
```

As long as the second rule exists, the machine will always boot and rename eth0 to eth1.

Also if you're wondering, Amazon Linux 1/2 disables the above-mentioned functionality in the ec2-net-utils package. This can be seen here:

[https://github.com/aws/ec2-net-utils/blob/master/write_net_rules](https://github.com/aws/ec2-net-utils/blob/master/write_net_rules)

#### How Does Udev Rule Matching Work?

Let's take the following udev rule and break it down.

```
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="01:23:45:67:89:ab", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"
```

if a network device ( SUBSYSTEM=="net" ) is added ( ACTION=="add" ) to the system, and it's not a VLAN'd i.e eth0.130 or sub-interface i.e. eth0:0 ( DRIVERS=="?*" ) with a MAC address of 01:23:45:67:89:ab ( ATTR{address}=="01:23:45:67:89:ab" ), and it's the primary ethernet device ( ATTR{type}=="1" ), and the kernel name of the device begins with "eth" ( KERNEL=="eth*" ), set the name of the device to eth0 ( NAME="eth0" ).

If you want to read more about udev rules, there are far better explanations on the interwebs. I recommend the following online resources.

* https://linuxconfig.org/tutorial-on-how-to-write-basic-udev-rules-in-linux[](https://linuxconfig.org/tutorial-on-how-to-write-basic-udev-rules-in-linux)
* [http://www.linuxfromscratch.org/lfs/view/6.3/chapter07/network.html](http://www.linuxfromscratch.org/lfs/view/6.3/chapter07/network.html)
