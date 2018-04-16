# Producing an Ubuntu Base Image for Swift 4 and Perfect 3

This guide will share steps that you can use to produce an Ubuntu Linux environment, which is suitable for use with the Swift 4 programming language, and with PerfectlySoft's Perfect 3 application framework.

## Running as the Root User

Begin by installing, deploying, or otherwise utilizing an Ubuntu system. For ease of use, you may use the following command to perform the steps as the **root** user. You'll need to be using an administrative account, and you'll need its password. If you choose to skip this step, you'll need to prefix many of the steps in this guide with the **sudo** command.

```
sudo su
```

## Running a Screen Session

If you are connected to the Ubuntu system via **SSH**, you might wish to establish a **screen** session for these steps in case you're disconnected.  This section is not mandatory. You can skip to the next section of steps. To establish a **screen** session called "perfect," use:

```
screen -S perfect
```

If **screen** isn't installed, you'll need to install it:

```
apt-get install screen
```

To intentionally disconnect from a **screen** session, you can use **Control-A, D**. To reconnect to the intentionally disconnected **screen** session:

```
screen -r perfect
```

If you were unintentionally disconnected from your **screen** session, you might need to use:

```
screen -D -R perfect
```

## Install Some Dependencies via APT

You can install the majority of dependencies for Swift 4 and for Perfect 3 via a free installation script [Perfect Ubuntu](https://github.com/PerfectlySoft/Perfect-Ubuntu.git)

## Install MongoDB from Source Code

Download the MongoDB source code:

```
cd /usr/src/
wget https://github.com/mongodb/mongo-c-driver/releases/download/1.3.5/mongo-c-driver-1.3.5.tar.gz
```

Decompress the MongoDB source code:

```
gunzip mongo-c-driver-1.3.5.tar.gz
```

Extract the MongoDB source code:

```
tar -xvf mongo-c-driver-1.3.5.tar
```

Remove the MongoDB source code archive:

```
rm mongo-c-driver-1.3.5.tar
```

Configure the MongoDB source code for compilation:

```
cd mongo-c-driver-1.3.5/
./configure --enable-sasl=yes
```

Compile MongoDB:

```
make
```

Install MongoDB:

```
make install
```

## Finished

Congratulations! You now have an environment suitable for Swift 4 and Perfect 3.

## Trying It Out

Now you can copy the Perfect 3 template project:

```
git clone https://github.com/PerfectlySoft/PerfectTemplate
```

Then build & run:

```
cd PerfectTemplate
swift run
```

You can test this application from a different shell with the `curl` command:

```
curl http://127.0.0.1:8181
```
