# Ubuntu 16.04: Starting Services at System Boot

A compiled Swift binary can be run on demand on Ubuntu, however for common server side applications such as API servers, it is best to have the startup and monitoring of state performed by the system automatically.

Ubuntu 16.04 leverages `systemd` to manage services, and this guide explains how to set up and enable a binary to run under `systemd`.

Once your Swift binary is placed on the server, you will need to create a `.service` file in the `/etc/systemd/system/` directory.

The contents of the file will be, at minimum, the following (substituting the obvious values):

```
[Unit]
Description=XXX API Server

[Service]
Type=simple
WorkingDirectory=/path/to/binary/
ExecStart=/path/to/binary/APIServer
Restart=always
PIDFile=/var/run/apiserver.pid

[Install]
WantedBy=multi-user.target
```

Once this file has been saved, set the permissions:

```
chmod +x /etc/systemd/system/apiserver.service
chmod 755 /etc/systemd/system/apiserver.service
```
Then enable, and start the service:

```
sudo systemctl enable apiserver.service
sudo systemctl start apiserver.service
```

Further detail about `systemd` can be found at [Managing Services with systemd](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Managing_Services_with_systemd-Services.html)