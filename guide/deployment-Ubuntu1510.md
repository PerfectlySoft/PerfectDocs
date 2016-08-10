# Producing an Ubuntu 15.10 Base Image for Swift 3 and Perfect 2

This guide will share steps that you can use to produce an Ubuntu 15.10 environment which is suitable for use with the Swift 3 programming language and with PerfectlySoft's Perfect 2 application framework.

## Running as the Root User

Begin by installing, deploying or otherwise utilizing an Ubuntu 15.10 system.  For ease of use, you may use the following command to perform the steps as the **root** user.  You'll need to be using an administrative account and you'll need its password.  If you choose to skip this step, you'll need to prefix many of the steps in this guide with the **sudo** command.

`  sudo su`

## Running a Screen Session

If you are connected to the Ubuntu system via **SSH**, you might even wish to establish a **screen** session for these steps, in case you're disconnected.  This section is not mandatory, however; you can skip to the next section of steps.  To establish a **screen** session called "perfect," you'd do:

`  screen -S perfect`

If **screen** isn't installed, you'll need to install it:

`  apt-get install screen`

To intentionally disconnect from a **screen** session, you can use **Control-A, D**.  To reconnect to the intentionally disconnected **screen** session:

`  screen -r perfect`

If you were unintentionally disconnected from your **screen** session, you might need to use:

`  screen -D -R perfect`

## Install Some Dependencies via APT

You can install the majority of dependencies for Swift 3 and for Perfect 2 via the APT system:

`  apt-get install make git clang libicu-dev libmysqlclient-dev libpq-dev sqlite3 libsqlite3-dev apache2-dev pkg-config libssl-dev libsasl2-dev libcurl4-openssl-dev uuid-dev wget`

## Install MongoDB from Source Code

Download the MongoDB source-code:

`  cd /usr/src/  
  wget https://github.com/mongodb/mongo-c-driver/releases/download/1.3.5/mongo-c-driver-1.3.5.tar.gz`

Decompress the MongoDB source-code:

`  gunzip mongo-c-driver-1.3.5.tar.gz`

Extract the MongoDB source-code:

`  tar -xvf mongo-c-driver-1.3.5.tar`

Remove the MongoDB source-code archive:

`  rm mongo-c-driver-1.3.5.tar`

Configure the MongoDB source-code for compilation:

`  cd mongo-c-driver-1.3.5/  
  ./configure --enable-sasl=yes`

Compile MongoDB:

`  make`

Install MongoDB:

`  make install`

## Finished

You now have an environment suitable for Swift 3 and Perfect 2.  Congratulations!

## Trying It Out

If you wish to install a "development snapshot" of Swift before Swift 3 has been released, you can download one:

`  cd /usr/src/  
  wget https://swift.org/builds/swift-3.0-preview-3/ubuntu1510/swift-3.0-PREVIEW-3/swift-3.0-PREVIEW-3-ubuntu15.10.tar.gz`

And install it:

`  gunzip < swift-3.0-PREVIEW-3-ubuntu15.10.tar.gz | tar -C / -xv --strip-components 1`

And remove the compressed archive:

`  rm swift-3.0-PREVIEW-3-ubuntu15.10.tar.gz`

Now you can copy the Perfect 2 template project:

`  git clone https://github.com/PerfectlySoft/PerfectTemplate`

And build it:

`  cd PerfectTemplate  
  git checkout d13e8dd8eb4868fea36468758604fe05a48b9aa2  
  swift build`

Once built, this application will be available inside the `.build/debug/` directory.  You can copy it to somewhere more convenient, such as the `/root/` directory:

`  cp .build/debug/PerfectTemplate /root/`

Now you can run this application from the `/root/` directory.  Be careful: If you are still using the **root** user, the application will have the **root** user's privileges!

`  /root/PerfectTemplate --port 80`

You can test this application from a different shell with the `curl` command:

`  curl http://127.0.0.1`

## The End

If you find an issue, please report it at:

[http://jira.perfect.org:8080/servicedesk/customer/portal/1](http://jira.perfect.org:8080/servicedesk/customer/portal/1)
