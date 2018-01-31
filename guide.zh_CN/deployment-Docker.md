# Docker Deployment

本文介绍了如何为您的Swift + Perfect应用程序部署Docker镜像，特别是有代表性的 *perfect2-swift20160620-ubuntu1510* 镜像，内容包含：

* Ubuntu 15.10
* Swift 开发版本 016-06-20 (在Swift 4发行前的版本)
* Perfect 3

Docker 在您的系统下可以按照如下方式进行安装：

* 如果您使用的是Ubuntu系统，则请用下面的命令行进行安装：
`apt-get install docker`

* 如果您使用的是macOS系统，安装方法详见：
[Docker在mac上的安装方法](https://docs.docker.com/engine/installation/mac/)

* 如果您使用的是微软的Windows系统，详细安装方法请见：
[Docker的Windows安装指南](https://docs.docker.com/engine/installation/windows/)

* 对于其它Linux系统，详细安装方法请见：
[Docker在Linux系统上的安装指南](https://docs.docker.com/engine/installation/)

一旦Docker安装完成，您就可以用下列命令获取 perfect2-swift20160620-ubuntu1510 资源文件：

```
docker pull perfectlysoft/perfect2-swift20160620-ubuntu1510
```

显现可以从Docker镜像上继续开发新程序了。为了方便演示，我们假定您的项目放在GitHub上：

```
https://github.com/PerfectlySoft/PerfectTemplate
```

现在需要从Docker容器中启动终端控制台：

```
docker run -t -i perfectlysoft/perfect2-swift20160620-ubuntu1510 bash
```

现在终端控制台应该已经在Docker容器内开始运行。请进入 `/usr/src/` 目录：

```
cd /usr/src/
```

将您的应用程序从Git代码资源库中克隆：

```
git clone https://github.com/PerfectlySoft/PerfectTemplate
```

进入您的项目文件夹：

```
cd PerfectTemplate/
```

请注意Docker容器需要联网运行。用下面的命令行进行项目编译：

```
swift build
```

编译完成后，您的应用程序应该出现在 `.build/debug/` 目录下。现在可以把这个文件拷贝到任何需要的地方，比如 `/root/` 目录：

```
cp .build/debug/PerfectTemplate /root/
```

现在请检查您的容器主机名：

```
hostname
```

比如，主机名可能是 `2babae7df6fe`。随后退出容器：

```
exit
```

现在您的新Swift + Perfect项目程序镜像已经准备完毕，请使用对应的主机名推送到云端：

```
docker commit -m "My application" -a "My Name" 2babae7df6fe myapp
```

（您可以为项目应用程序自定名称，取代“My Application”，同时用您本人的名字、公司的名字代替“My Name”，并用一个Docker镜像名取代“myapp”）

现在可以确认您的镜像已经创建：

```
docker images
```

现在可以将您的应用程序部署为：

```
docker run -d -p 0.0.0.0:8080:8181 myapp /root/PerfectTemplate
```

上面的命令意思是将您本地主机（`0.0.0.0`）的TCP端口`8080` 重新定向到您的Docker容器`8181`端口（基于您自己的应用镜像）并运行您的应用程序（`/root/PerfectTemplate`）。在本例子中，PerfectTemplate模板程序监听TCP端口`8181`。您可以用下面的命令行进行测试：

```
curl http://127.0.0.1:8080
```
