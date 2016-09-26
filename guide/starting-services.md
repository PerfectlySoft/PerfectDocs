<<<<<<< HEAD
# Ubuntu 16.04: 作为系统服务以root用户启动

完成编译的Swift可执行二进制文件可以随时根据需要在Ubuntu上执行；但是对于大多数服务器而言，如Web服务器这样的服务器端应用程序而言，最好以系统服务的方式安装，这样操作系统就可以自动启动并监控这些程序。

Ubuntu 16.04 用 `systemd` 命令来管理系统服务，本章用于解释如何设置和使用 `systemd`。

一旦Swift可执行文件包编译并部署到服务器上之后，请在`/etc/systemd/system/`目录下创建一个`.service` 文件。

该文件基本内容样式如下（请自行替换值）：
=======
# Ubuntu 16.04: Starting Services at System Boot

A compiled Swift binary can be run on demand on Ubuntu, however for common server side applications such as API servers, it is best to have the startup and monitoring of state performed by the system automatically.

Ubuntu 16.04 leverages `systemd` to manage services, and this guide explains how to set up and enable a binary to run under `systemd`.

Once your Swift binary is placed on the server, you will need to create a `.service` file in the `/etc/systemd/system/` directory.

The contents of the file will be, at minimum, the following (substituting the obvious values):
>>>>>>> PerfectlySoft/master

```
[Unit]
Description=XXX API Server

[Service]
Type=simple
ExecStart= /path/to/binary/APIServer
Restart=always
PIDFile=/var/run/apiserver.pid

[Install]
WantedBy=multi-user.target
```

<<<<<<< HEAD
文件保存后请设置该文件权限：
=======
Once this file has been saved, set the permissions:
>>>>>>> PerfectlySoft/master

```
chmod +x /etc/systemd/system/apiserver.service
chmod 755 /etc/systemd/system/apiserver.service
```
<<<<<<< HEAD
下面的命令能够激活（enable）并启动服务：
=======
Then enable, and start the service:
>>>>>>> PerfectlySoft/master

```
sudo systemctl enable apiserver.service
sudo systemctl start apiserver.service
```

<<<<<<< HEAD
关于 `systemd`命令的更多信息请参考[使用 systemd 命令管理服务](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Managing_Services_with_systemd-Services.html)
=======
Further detail about `systemd` can be found at [Managing Services with systemd](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Managing_Services_with_systemd-Services.html)
>>>>>>> PerfectlySoft/master
