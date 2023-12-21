## Fixing the SSH error “Authentication failed, permission denied” on your EC2 instance the easy way!

### Overview

This solution uses AWS CloudShell to automate the process of resetting permissions and ownership of important files in your EC2 instance.

The script runs the following commands via Cloud-init user data:

```
cat /etc/ssh/sshd_config
cat /var/log/secure | grep -i ssh | tail -50
ls -ld /etc/ssh /home /home/USERNAME /home/USERNAME/.ssh
ls -l /etc/ssh /home/USERNAME/.ssh
journalctl -n 100 --no-pager -r -u sshd
chmod 755 /etc/ssh
chmod 644 /etc/ssh/*
chmod 600 /etc/ssh/ssh_host_*_key /home/USERNAME/.ssh/authorized_keys
chmod 700 /home/USERNAME/.ssh
chown -R root:root /etc/ssh
chown -R USERNAME:USERNAME /home/USERNAME/.ssh
```

The USERNAME field will be substituted with the username you enter when you execute the script.

The chmod and chown commands are what do the bulk of the work. The first lot of commands will output data to /var/log/cloud-init-output.log which can be useful if you want to know what state these files and the SSH service in general were in before the changes were made.

When you've regained access to the instance, the script will stop the instance again and remove the recovery user data. If you had any user data prior to the execution of this script, it will be placed back in before the instance is started for the final time.

### Prerequisites

The AMI you used to launch your instance should have Cloud-Init installed. If not, i'm afraid this solution isn't going to work for you, nor are most solutions that involve using cloud-init. You should contact AWS Premium Support if you need further assistance as that's beyond the scope of this article.

#### IAM Permissions

As long as you have the ability to run Describe API calls against your instance, and have the ability to stop and start the instance you should be fine. If you need a list of the exact permissions you need, here they are:

 - ec2:DescribeInstanceStatus
 - ec2:DescribeInstanceAttribute
 - ec2:ModifyInstanceAttribute
 - ec2:StopInstances
 - ec2:StartInstances

You will also need the AWSCloudShellFullAccess IAM policy if you do not have full admin access.

Finally, you need the following information as you'll be prompted for it during the execution of the recovery script:

1. The user that needs to be fixed. This will either be ec2-user, ubuntu, centos etc. The default is ec2-user.
2. The EC2 instance ID of the instance that we need to fix. Enter the instance ID ( i-xxxxxxxxx ) and not the name of the instance.
3. The AWS region in which the instance is located. The default is the same region you launched AWS CloudShell in.

#### WARNING

Please note, your instance will be stopped during this process. If your instance has a public IP and that IP is not an Elastic IP then your public IP '''WILL CHANGE'''.

Also, if your instance utilises instance store volumes, data on these volumes will be destroyed as part of the stop/start process.

Finally, I take no responsibility for anything that happens to your instance as a result of running this script. 

#### Launch AWS CloudShell

From the AWS Management Console, you can launch CloudShell by choosing one of the following options:

1. On the navigation bar, choose the CloudShell icon.
2. In the Search box, type “CloudShell”, and then choose CloudShell.
3. In the Recently visited widget, choose CloudShell.
4. Choose CloudShell on the Console Toolbar, on the lower left of the console.

When the command prompt displays, the shell is ready for interaction.

#### Execute the script

When you run the script below, follow the prompts.

```
[cloudshell-user@ip-10-134-32-74 ~]$ bash -e <(curl -s -o - https://raw.githubusercontent.com/iitggithub/aws/master/fix_ssh_permissions.sh)
```

Sample output:

```
[cloudshell-user@ip-10-134-32-74 ~]$ bash -e <(curl -s -o - https://raw.githubusercontent.com/iitggithub/aws/master/fix_ssh_permissions.sh)
Enter the SSH username (Default: 'ec2-user'): 
Enter the EC2 instance ID ie i-xxxxxxxxxxxxxxxxx: i-xxxxxxxxxxxxxxxxxx
Enter the AWS region where the instance is located (Default: 'us-east-1'): 
WARNING! By executing this script, you will stop your instance!
If your instance has a public IP, but not an Elastic IP, you will lose it.
If your instance has instance store volumes, and you have data on them, you will lose that data.

You must also have the following IAM permissions:
 - ec2:DescribeInstanceStatus
 - ec2:DescribeInstanceAttribute
 - ec2:ModifyInstanceAttribute
 - ec2:StopInstances
 - ec2:StartInstances

Press any key to when ready to begin...

Content-Type: multipart/mixed; boundary="//"
MIME-Version: 1.0

--//
Content-Type: text/cloud-config; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cloud-config.txt"

#cloud-config
cloud_final_modules:
- [scripts-user, always]

--//
Content-Type: text/x-shellscript; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="userdata.txt"

#!/bin/bash
cat /etc/ssh/sshd_config
cat /var/log/secure | grep -i ssh | tail -50
ls -ld /etc/ssh /home /home/ec2-user /home/ec2-user/.ssh
ls -l /etc/ssh /home/ec2-user/.ssh
journalctl -n 100 --no-pager -r -u sshd
chmod 755 /etc/ssh
chmod 644 /etc/ssh/*
chmod 600 /etc/ssh/ssh_host_*_key /home/ec2-user/.ssh/authorized_keys
chmod 700 /home/ec2-user/.ssh
chown -R root:root /etc/ssh
chown -R ec2-user:ec2-user /home/ec2-user/.ssh
--//

Finished generating user data...
base64 encoding user data...
Waiting for instance to enter the stopped state...
{
    "StoppingInstances": [
        {
            "CurrentState": {
                "Code": 64,
                "Name": "stopping"
            },
            "InstanceId": "i-xxxxxxxxxxxxxxxxxx",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}
...........
Backing up any existing user data to userdata_original.txt...
Inserting recover user data...
Starting instance i-xxxxxxxxxxxxxxxxxx...
{
    "StartingInstances": [
        {
            "CurrentState": {
                "Code": 0,
                "Name": "pending"
            },
            "InstanceId": "i-xxxxxxxxxxxxxxxxxx",
            "PreviousState": {
                "Code": 80,
                "Name": "stopped"
            }
        }
    ]
}
Waiting for instance to enter the running state...
.

Instance is now in the running state. Please check SSH access now.

Should I remove the user data changes now? y/n: y
Waiting for instance to enter the stopped state...
{
    "StoppingInstances": [
        {
            "CurrentState": {
                "Code": 64,
                "Name": "stopping"
            },
            "InstanceId": "i-xxxxxxxxxxxxxxxxxx",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}
............
Removing recovery user data...
inserting old user data...
Starting instance i-xxxxxxxxxxxxxxxxxx...
{
    "StartingInstances": [
        {
            "CurrentState": {
                "Code": 0,
                "Name": "pending"
            },
            "InstanceId": "i-xxxxxxxxxxxxxxxxxx",
            "PreviousState": {
                "Code": 80,
                "Name": "stopped"
            }
        }
    ]
}
Waiting for instance to enter the running state...
.

Instance is now in the running state once again.
Script execution complete.
[cloudshell-user@ip-10-134-32-74 ~]$
```

### What do I do if this doesn't fix it?

The script also dumps a lot of SSH related data points that can be used for troubleshooting. You may be able to see them in the EC2 Instance System Log or in /var/log/cloud-init-output.log (if you can extract it from the instance).

If you don't know how to troubleshoot from this point, I would recommend raising a support case with AWS Premium Support.

Thanks for reading everyone!
