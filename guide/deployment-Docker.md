# Docker Deployment

This guide contains steps which will allow you to derive your Docker image for your Swift + Perfect application. This particular guide is for deriving from the *perfect2-swift20160620-ubuntu1510* image, which contains:

* Ubuntu 15.10
* Swift development snapshot 2016-06-20 (prior to Swift 4 being released)
* Perfect 3

You'll need Docker to be available on your system:

* If you have an Ubuntu system, you can use:
`apt-get install docker`

* If you have a macOS system, you can consult these instructions:
[https://docs.docker.com/engine/installation/mac/](https://docs.docker.com/engine/installation/mac/)

* If you have a Microsoft Windows system, you can consult these instructions:
[https://docs.docker.com/engine/installation/windows/](https://docs.docker.com/engine/installation/windows/)

* For other Linux systems, you can consult these instructions:
[https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)

Once you have Docker available, you can fetch the perfect2-swift20160620-ubuntu1510 repository with the following command:

```
docker pull perfectlysoft/perfect2-swift20160620-ubuntu1510
```

Now we'll derive from that Docker image and fetch your application into the derived image. For the sake of demonstration, we'll pretend that your application is available via GitHub, at:

```
https://github.com/PerfectlySoft/PerfectTemplate
```

We need to start a shell inside the Docker container:

```
docker run -t -i perfectlysoft/perfect2-swift20160620-ubuntu1510 bash
```

Now our shell is running inside the Docker container.  Venture into the `/usr/src/` directory:

```
cd /usr/src/
```

Clone your application's Git repository:

```
git clone https://github.com/PerfectlySoft/PerfectTemplate
```

Venture inside your application's repository:

```
cd PerfectTemplate/
```

Your Docker container will require internet access.  Build your application:

```
swift build
```

Once built, your application will be available inside the `.build/debug/` directory.  You can copy it to somewhere more convenient, such as the `/root/` directory:

```
cp .build/debug/PerfectTemplate /root/
```

Now check and make a note of the hostname of your container:

```
hostname
```

For example, the hostname might be 2babae7df6fe.  Now exit the container:

```
exit
```

Capture your Swift + Perfect application with its image, using the hostname you'd made a note of:

```
docker commit -m "My application" -a "My Name" 2babae7df6fe myapp
```

(You can use your own application-name instead of "My application", your own name or company-name instead of "My Name", or a Docker image-name instead of "myapp".)

Now you can confirm that your image has been created:

```
docker images
```

Deploy your application like this:

```
docker run -d -p 0.0.0.0:8080:8181 myapp /root/PerfectTemplate
```

What the above command does is to have TCP port `8080` on your host system (`0.0.0.0`) redirect to TCP port 8181 in the Docker container (based on your myapp image) that's running your application (`/root/PerfectTemplate`). In this example, the PerfectTemplate program listens on TCP port `8181`.  You can test your application with the cURL command:

```
curl http://127.0.0.1:8080
```

