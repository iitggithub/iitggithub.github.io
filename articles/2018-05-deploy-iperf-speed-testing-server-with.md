## Deploy an iPerf Speed Testing Server with Web Interface

Sometimes you just need to run network tests between two hosts. While there are tools available such as ping, traceroute, iperf3 etc, they're often annoying to install, confusing to use and require access to servers themselves. There's got to be a better way!

Inspired by the website [https://digwebinterface.com/](https://digwebinterface.com/), I decided to create the IITG Iperf-Web docker container. An Alpine-based Python 3 Flask web server which allows you to execute iperf, iperf3, dig, nslookup, netcat, ping, and traceroute commands and view their output rom a web interface so no server access is required.

It's deployed as a docker container for easy setup but if you know what you're doing, all of the files are available in the iperf-web github repository.

#### Why is it called iperf-web?

Originally it was only supposed to be a web interface for iperf-web before requests were made to expand it to encompass other tools. The name was kept the same, but now there's more features!

#### Pre-requisites

To follow along with this tutorial, you will need to [https://docs.docker.com/engine/install/](install the Docker engine) and have docker running.

You will also need the appropriate sudo permissions or access to the root user account.

#### Installation

Pull down the iperf-web docker container from the Docker Hub and run it!

```
sudo docker run -d --restart=always -name iperf-web -p 5000:5000 iitgdocker/iperf-web
```

The command will run a docker container using the docker image iitgdocker/iperf-web. It exposes the containers TCP port 5000 which the flask app runs on to TCP port 5000 on the docker host (-p 5000:5000), runs the container non-interactively in daemon mode (-d) and will automatically try to restart the container if anything happens to it (--restart=always).

That's it! It's ready to use. Simply visit the site in your browser on the correct TCP ie http://192.168.1.10:5000 and you'll be greeted with a web interface similar to what's shown below:

![Overview](https://github.com/iitggithub/iperf-web/blob/master/overview.png?raw=true)

#### How To Save Server Connection Details

If you have a list of servers that you regularly perform testing against or you plan to deploy multiple iperf-web servers and perform testing between them, you can save the connection details in a list. You can then select a configuration and it will prefill the fields for that test type.

![Server Configuration Details](https://github.com/iitggithub/iperf-web/blob/master/NTU_Server_Details.png?raw=true)

1\. Mount the config directory

You need to mount a directory from your docker host into the iperf-web docker container. This allows you to edit the config.json file from the docker host and ensure that the contents of the file persists between container restarts.

In the example below, we mount the /opt/iperf-web/config directory on the docker host to the /app/config directory in the docker container.

```
sudo mkdir -p /opt/iperf-web/config
sudo docker run -d --restart=always -name iperf-web -p 5000:5000 -v /opt/iperf-web/config:/app/config iitgdocker/iperf-web
```

2\. Configure the config.json file according to your requirements

Note: An example has been provided in the config directory called config_example.json. You don't have to include every field name, just the fields you wish to change.

You will need to know the name of the test ie dig, iperf, mtr, nc, nslookup, ping, traceroute etc and the name of each of the fields you wish to prefill.

Because the test types and field names may change at any time, I recommend reviewing the [https://github.com/iitggithub/iperf-web/blob/master/templates/index.html](https://github.com/iitggithub/iperf-web/blob/master/templates/index.html) file directly.

```
{
    "dig": [
        {
            "name": "ineedmyip.com",
            "dig_target": "ineedmyip.com",
            "dig_parameters": "+short"
        },
        {
            "name": "ineedmyip.com reverse DNS",
            "dig_target": "140.82.49.8",
            "dig_parameters": "-x"
        }
    ],
    "iperf": [
        {
            "name": "Server 1",
            "iperf_version": "3",
            "iperf_target": "192.168.1.111",
            "iperf_port": "5201",
            "iperf_conn_type": "TCP",
            "iperf_timeout": "10",
            "iperf_parameters": "--format m"
        },
        {
            "name": "Server 2",
            "iperf_version": "2",
            "iperf_target": "192.168.1.222",
            "iperf_port": "5201",
            "iperf_conn_type": "UDP",
            "iperf_timeout": "15",
            "iperf_parameters": "--bandwidth 10M"
        }
    ],
    "nc": [
        {
            "name": "ineedmyip.com SSH port 22",
            "nc_target": "ineedmyip.com",
            "nc_port": "22"
        }
    ],
    "ping": [
        {
            "name": "8.8.8.8",
            "ping_target": "8.8.8.8",
            "ping_count": "4"
        }
    ]
}
```

3\. Restart the container if it's already running

Note: This is required to make sure the container uses the updated image

```
sudo docker pull iitgdocker/iperf-web:latest
sudo docker stop iperf-web
sudo docker rm iperf-web
sudo docker run -d --restart=always -name iperf-web -p 5000:5000 -v /opt/iperf-web/config:/app/config iitgdocker/iperf-web
```

Using the configuration above, a dropdown box will appear when the Dig, Iperf or Ping test types are selected and allow you to select the appropriate configure based on the configuration name. Selecting a configration will automatically prefill those fields.

#### Environment Variables

##### IPERF\_WEB\_PORT

Sets the TCP port the python flask app listens on.

Possible values: number between 1025 and 65535

Default: 5000

##### IPERF\_WEB\_DEBUG\_MODE

Enables Flask app debug mode

Possible values: True, False

Default: False

#### Further Reading, comments, issues etc

More detailed documentation is available on the [iperf-web github repo](https://github.com/iitggithub/iperf-web).
