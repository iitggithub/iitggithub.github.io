## Deploy an iPerf Speed Testing Server with Web Interface

Monday, 28 May 2018

Sometimes you just need to run network performance tests between two hosts. While there are tools available for running speed tests such as iPerf, they're often annoying to setup, confusing to use, and require console access to the server itself in order to test.

Here's where the IITG Web-based iPerf containers come into play. With this guide, you can setup iPerf servers in multiple locations and allow network support staff to perform network testing without the need to give them console access to the server itself.

Let's get started!

#### Installation

Install web interface, iperf-servers and iperf command:

```
$ curl -L https://raw.githubusercontent.com/iitggithub/iperf-web/master/install.sh | bash
```

#### Configure iperf-web

If the URL you will be using to access the iperf web interface does not match the fully qualfied domain name of your docker container host, make sure you set FQDN_SERVER_NAME to something more meaningful.

##### Set a variable to contain the hostname

```
$ export FQDN_SERVER_NAME="`hostname`"
```

By default iperf-web exposes port 80. That's not necessary since we'll use a self signed certificate and HTTPS later on. You can manually edit the /data/iperf-web/docker-compose.yml file to disable exposing port 80 to the outside world or just recreate from scratch using the code below

##### Reconfigure iperf-web docker-compose.yml file

```
$ cat | sudo tee /data/iperf-web/docker-compose.yml <<EOF
server:
image: iitgdocker/iperf-web:latest
volumes:
- /var/run/docker.sock:/var/run/docker.sock
- /data/iperf-web/images:/var/www/html/images
environment:
- VIRTUAL_HOST=${FQDN_SERVER_NAME}
EOF
```

#### Configure An Nginx Web Server

Assuming you don't already run your own webserver on the host, we'll use a docker container for that too. This one will automatically regenerate and reload nginx whenever a compatible container is detected. This is based on jwilders nginx proxy. You can find more detailed information on how to use it here: [https://github.com/jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy).

##### Create a self-signed SSL certificate (Optional)

```
$ mkdir -p /data/nginx/certs 
$ openssl req -new -newkey rsa:2048 -nodes -out /data/nginx/certs/${FQDN_SERVER_NAME}.crt -keyout /data/nginx/certs/${FQDN_SERVER_NAME}.key -subj "/C=/ST=/L=/O=IITG Blog/CN=${FQDN_SERVER_NAME}"
```

##### Install Your Own Company Logo (Optional)

```
$ mkdir -p /data/nginx/images
$ cp <path_to_my_logo>.png /data/nginx/images
```

##### Install the nginx docker compose file

```
$ cat | tee /data/nginx/docker-compose.yml <<EOF
proxy:
image: jwilder/nginx-proxy:latest
ports:
- "80:80"
- "443:443"
volumes:
- /var/run/docker.sock:/tmp/docker.sock
- /data/nginx/certs:/etc/nginx/certs
EOF
```

##### Install the systemd service file

If you can make use of a systemd service file, here's the one you'll need for the nginx proxy

```
$ cat | sudo tee /usr/lib/systemd/system/docker-nginx.service <<EOF
[Unit]
Description=Nginx web proxy
After=docker.service
[Service]
Conflicts=shutdown.target
StartLimitInterval=0
Restart=always
TimeoutStartSec=0
Restart=on-failure
WorkingDirectory=/data/nginx
ExecStartPre=-/usr/local/bin/docker-compose stop
ExecStartPre=-/usr/local/bin/docker-compose pull
ExecStart=/usr/local/bin/docker-compose up
ExecStop=-/usr/local/bin/docker-compose stop
[Install]
WantedBy=multi-user.target
EOF
```

##### Start the nginx proxy

```
$ sudo systemctl start docker-nginx
$ sudo systemctl enable docker-nginx
```

##### Restart the iperf-web server

```
$ sudo systemctl restart docker-iperf-web
```
