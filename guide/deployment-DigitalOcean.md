# Digital Ocean Deployment

This guide will walk you through setting up a Digital Ocean Droplet that will run your compiled Swift application. You can actually do all of these steps on any other Linux server running Ubuntu 15 or 16. Let's start...

First create your droplet, select Ubuntu 16 (if 15 is not available), and let it finish. Once the droplet is created you can ssh into it as root.
```
ssh root@YOUR_DROPLET_IP -v
```
This should use the private key you have created earlier and it should not ask you for a password or anything.

### Setting up the server - Part 1

The next step is creating a user to run all commands through and also run the final app since it's not that nice to do all as root. Since we're still connected as root we're going to skip the `sudo` commands since there's no point in using it. Please adjust the `USERNAME` and `USER_PASSWORD` variables in the command below.
```
useradd -d /home/USERNAME -m -s /bin/bash USERNAME
echo "USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

The last command will add your new `USERNAME` to the list of accounts allowed to run commands using `sudo`. The spectrum used here, `ALL`, is a bit unrestrictive and it is advised to limit the commands to just a few.

Once the user is created, you have two options: 1. Generate a password for this user and use the password when you want to connect to the server, or 2. Generate a SSH key and use that key to connect to the server - we're going to take a look at both options below (personally I prefer the SSH key over the password).

### Option 1 - Generate a password

Since we're still connected as root we're going to set the password for our user using
```
passwd USERNAME
```

Once that is done we can drop the connection as root (Ctrl-D) and reconnect as USERNAME.

### Option 2 - Generate a SSH key

This option is a bit more complicated to set up, but in the long run it will be easier to connect since we don't have to remember the password we've set or risk that the password isn't secure enough, etc...

There are again, two options here: 1. Use the same key as root, or 2. Generate a new key for USERNAME.

#### Sub-Option 1

To use the same key you've set up when the droplet was created you can just run
```
mkdir /home/USERNAME/.ssh
cp /root/.ssh/authorized_keys /home/USERNAME/.ssh
chown -R USERNAME.USERNAME /home/USERNAME/.ssh
```

#### Sub-Option 2

First we need to generate a new SSH key on our local environment - if you are running Windows as your OS, please google how you can do it and continue after this step. Remember, this step is to be done on your local environment and not on the droplet...
```
ssh-keygen -t rsa
```
The key generator will ask you where do you want to save the new key and what will be the key name. For this example, I have saved my key to `/Users/rb/.ssh/id_rsa_perfectapp` - We'll call this `LOCAL_KEY_PATH` from now on.

Once it's finished generating the key, you need to extract the contents of the public key and prepare to use it on the droplet by using
```
cat /Users/rb/.ssh/id_rsa_perfectapp.pub
```
which will result in something like this
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3Mzd0naYx59XfiObuSEphNTupfNqHGFSRExAl+WS0yiA8j8Kja964qTg0Bs2DUD9OoazaAIUTrt+GoQFUSlP37a4XqUxu4Pm8LyDrvx+AGEj4ylr5PdfG847x7n73ucoBSHuo3Ws6NLgfBhJ4U6R/mpFsc7GtYbaIISSZb/DUKchLA5ihpdST0Nv8G5LIS+HqgWfy9DQS1/w1RfxjXL6UvydDNyOklMCyWnpswyl19pZmW/JhvrTj1x3RM5Qb3p3EV3Xp4ZoJgY0X675ovr+uq1f7dumH1a5gqz+NeL+cThWc4B7SZTXUu+/0XIOL0485FgE+BwBg6CETDyt960L9 rb@Roberts-MacBook-Pro.local
```

Next, moving back to the droplet server, we need to set up the public key we've just created and set it up so the server can identify us with it.
```
su - USERNAME
mkdir .ssh
chmod 700 .ssh
echo "THE_PUBLIC_KEY_STRING" >> .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
exit
```
Long story short, we've switched to the new user's home folder, we've created the folder `.ssh` and inside of that folder we've created a file called `authorized_keys` in which we've pasted the content of the public key created earlier. The last two lines are going back up a level in the folder structure and change the permissions of the `.ssh` folder and its contents to the `USERNAME` user.

To test that everything works, and the key was added successfully, on your local environment run the following
```
ssh -i LOCAL_KEY_PATH USERNAME@YOUR_DROPLET_IP -v
```
You should be logged into your droplet now, under the USERNAME account.


### Setting up the server - Part 2

Now that we are not longer connected to the droplet as `root` we can continue setting it up and preparing it for running/compiling Swift code and the arrival of you app.

#### Updating packages installed by DigitalOcean
```
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
sudo dpkg-reconfigure locales
sudo apt-get update
sudo apt-get upgrade -y
```
This will configure the locales installed on the server, update the packages list to the latest available and upgrade (-y without asking you anything) the installed packages - it's always a good practice to have the latest stable versions up and running.

Next, lets installed some of the packages needed by Swift to compile, and a few other packages needed in the process. You can do this by running
```
sudo apt-get install make clang libicu-dev pkg-config libssl-dev libsasl2-dev libcurl4-openssl-dev uuid-dev git curl wget unzip -y
```

Next, we will install Swift so the whole system has access to the binaries. At the moment, the latest available version of Swift binaries/libraries is `Swift 4.0 GM Candidate` - If you are not sure what's the latest version, head on to https://swift.org/download/ and check there. Because we're trying to make Perfect2.0 compile on this server we need to use Swift3.

```
cd /usr/src
sudo wget https://swift.org/builds/swift-3.0-GM-CANDIDATE/ubuntu1510/swift-3.0-GM-CANDIDATE/swift-3.0-GM-CANDIDATE-ubuntu15.10.tar.gz
sudo gunzip < swift-3.0-GM-CANDIDATE-ubuntu15.10.tar.gz | sudo tar -C / -xv --strip-components 1
sudo rm -f swift-3.0-GM-CANDIDATE-ubuntu15.10.tar.gz
```

You can test if Swift was copied/installed correctly by running
```
swift --version
```
in any location on the server. You should get back something like
```
Swift version 3.0 (swift-3.0-GM-CANDIDATE)
Target: x86_64-unknown-linux-gnu
```

At this point we have another fork in the road - What database system (if needed) will we use?

### MySQL

If you are going to use a database provider as a service (RDS from Amazon, etc) you can run
```
sudo apt-get install libmysqlclient-dev -y
```
but if you also want to host the database on the same server, you can run
```
sudo apt-get install mysql-client libmysqlclient-dev -y
```

Next, you need to edit the file `mysqlclient.pc` located in the folder `/usr/lib/x86_64-linux-gnu/pkgconfig`. If you can't find the file there, please run a `find / -name mysqlclient.pc`. Once you find the file and edit it, you will need to remove any occurence of "-fno-omit-frame-pointer" and "-fabi-version=2".
```
sudo nano /usr/lib/x86_64-linux-gnu/pkgconfig/mysqlclient.pc
```

For example, my file looks like this:
```
prefix=/usr
includedir=${prefix}/include/mysql
libdir=${prefix}/lib/x86_64-linux-gnu

Name: mysqlclient
Description: MySQL client library
Version: 20.3.0
Cflags: -I${includedir} -fabi-version=2 -fno-omit-frame-pointer
Libs: -L${libdir} -lmysqlclient
Libs.private: -lpthread -lz -lm -lrt -ldl
```

and after the edit, the file looks like this:
```
prefix=/usr
includedir=${prefix}/include/mysql
libdir=${prefix}/lib/x86_64-linux-gnu

Name: mysqlclient
Description: MySQL client library
Version: 20.3.0
Cflags: -I${includedir}
Libs: -L${libdir} -lmysqlclient
Libs.private: -lpthread -lz -lm -lrt -ldl
```

If you chose to host your database on the same server as the Swift app, you will have to do a few more steps in order to configure the MySQL server.

```
sudo mysql_secure_installation
```
This will prompt you for the root password you created in step one. You can press ENTER to accept the defaults for all the subsequent questions, with the exception of the one that asks if you'd like to change the root password.

Next, we'll initialize the MySQL data directory, which is where MySQL stores its data. How you do this depends on which version of MySQL you're running. You can check your version of MySQL with the following command.

```
mysql --version
```
You'll see some output like this:
```
mysql  Ver 14.14 Distrib 5.7.11, for Linux (x86_64) using EditLine wrapper
```
If you're using a version of MySQL earlier than 5.7.6, you should initialize the data directory by running `mysql_install_db`.
```
sudo mysql_install_db
```

Please note that in MySQL 5.6, you might get an error that says FATAL ERROR: Could not find my-default.cnf. If you do, copy the `/usr/share/my.cnf` configuration file into the location that `mysql_install_db` expects, then rerun it.
```
sudo cp /etc/mysql/my.cnf /usr/share/mysql/my-default.cnf
sudo mysql_install_db
```
This is due to some changes made in MySQL 5.6 and a minor error in the APT package.

The mysql_install_db command is deprecated as of MySQL 5.7.6. If you're using version 5.7.6 or later, you should use `mysqld --initialize` instead.

However, if you installed version 5.7 from the Debian distribution, the data directory was initialized automatically, so you don't have to do anything. If you try running the command anyway, you'll see the following error:
```
2016-03-07T20:11:15.998193Z 0 [ERROR] --initialize specified but the data directory has files in it. Aborting.
```

### Sequel3
Coming soon...

### PostgreSQL
First Install docker on your Digital Ocean Linux box (this is for the standard DO 16.04 Ubuntu image )
``` sudo apt-get install docker.io ```

Now that docker is installed we need to go about installing the postgres docker container.  for that we’ll need a directory to put the database in.  On an abstract level, using a docker image of postgres is like a black box, commands from CLI go into the docker container (running separate from the rest of the OS, then pop out in where ever you direct the container to put the database.  in this case, we are going to put it in a new /postgres folder 

But before making that folder lets build a systemd service.  That'll make it so that if our server ever goes down, etc, this service will automatically start it running again.

Create the service and jump into the new .service file using Visual Instrument (or preferred editor): ```sudo vi /etc/systemd/system/perfect-postgres.service```

Now that you are in the editor, copy paste the following (replace SERVICE_NAME with your desired name for the service (I used perfect-postgres), USER with your current username (set above), MYSECRETPASSWORD with-- you guessed it --a secure password, DB_NAME with your desired database name and please don't use underscores in your names-- dashes are fine.): 
```
[Service] 
ExecStart=/usr/bin/docker run --rm -p 5432:5432 --name SERVICE_NAME -e POSTGRES_USER=USER -e POSTGRES_PASSWORD=MYSECRETPASSWORD -e POSTGRES_DB=DB_NAME -v /postgres:/var/lib/postgresql/data postgres
Restart=always  

[Install]
WantedBy=multi-user.target  
```
NOTE: The ```Restart=always``` is the part that makes it so that if the server ever restarts, so will postgres/ the database.  

Great work.  Now (if in ```vi```) press the ESC button to get out of INSERT mode and then type ```:wq```, the : for command, w for save and q for quit.

Now that the service is set up we need to refresh the systemd startup list so that it will have our new service on it:
```sudo systemctl daemon-reload```
and then enable our service (use your service name):
```sudo systemctl enable SERVICE_NAME ```
this should show you something that looks like this: 
```Created symlink from /etc/systemd/system/multi-user.target.wants/perfect-postgres.service to /etc/systemd/system/perfect-postgres.service.
```
Great, let's start the service: ```sudo systemctl start perfect-postgres ```
And check out some of its logs to make sure it is working via journalctl  (systemd's logs): 
```sudo journalctl -f -u perfect-postgres```
Cool.  Now back to working in linux, we need to install postgres.  ``` sudo apt-get install postgresql ```

After successfully doing that I tried to hop into the newly created database using my newly installed postgresql with: ```psql -d DB_NAME -U USER```
I got this error: ```psql: FATAL:  Peer authentication failed for user "USER"```

If this also happens to you, you may have more success loggin in via remote. Note, this is not in your ssh session, just another terminal window:
```$ psql -d DB_NAME -U USER -h DIGITAL_OCEAN_BOX_IP_ADDRESS ```
The -h is for the host, use your D.O. IP Address.  

Great, you should be set up with a postgresql database to use with your perfect project.  On to the next: Setting up folders, autocompiling, etc!

### MongoDB
Coming soon...


### Setting up the app folders, automate compiling and all that good stuff

Now that we're finished with the server setup, and all prerequisites are installed, it is the time to move forward with the app setup, its folder structure and compile automation.

The main plan is to have the app delivered to the server using `git`. If you noticed, the earlier package installation, did add `git` as an app to the server. Okay, so, `git` will deliver the raw Swift files to the server, we will need to compile the app, move it to another folder (for a clear structure) and run it from there.

To accomplish this we will use the `post-receive` git hook, which will run all the needed commands to compile. Move and restart the app. Yes, restart it, because we will also make use of the `supervisor` app to keep our Swift compiled application up and running 24/7.

First, let's setup the folder structure for our app.

*Note*: At this point you should be logged into your droplet server as your USERNAME, and be located in the home folder of this user - If you are logged as USERNAME, just `cd ~` (or `cd` for short) and hit Enter.

```
mkdir -p {running,app/.git} && cd app/.git
git init . --bare
cd hooks && rm -rf *.sample
```
This will create two folders in the USERNAME's home folder: `running` which will host your final compiled app, and `app` which will host your raw Swift files, compile by-products, and your Git repo. On the same line we're also changing location to the `app/.git` folder which will host the app's repo. `git init . --bare` will initialize a bare git repo in this folder and create a very basic repo structure. Next, we're again changing location in the hooks folder and removing all the sample files there.

Next we will need to create the `post-receive` hook that will do all the heavy lifting for us.
```
nano post-receive
```

You can copy/paste the following block as your file contents, but be sure you change the following variables with their correct values:
- `USERNAME` your current username
- `THE_APP_NAME` the actual name of the compiled app - this is the name of your app as well written in the Package.swift file
- `.build/debug/*` - if your app was compiled as release, please change it to `.build/release/*`
```
#!/bin/bash
APP_NAME="THE_APP_NAME"
USERNAME="USERNAME"
HOME="/home/$USERNAME"
APP_FOLDER="$HOME/app"
RUNNING_FOLDER="$HOME/running"
GIT_FOLDER="$APP_FOLDER/.git"
echo ""
echo "---------------------------------------"
echo "Running with the following config:"
echo "Owner: $USERNAME"
echo "Home folder: $HOME"
echo "Application name: $APP_NAME"
echo "Application folder: $APP_FOLDER"
echo "Git folder: $GIT_FOLDER"
echo "Running folder: $RUNNING_FOLDER"
echo "---------------------------------------"
echo ""
echo "Cleaning up previous commit"
sudo rm -f $HOME/.build
sudo rm -f $HOME/Packages
echo ""

# Move files from commit to the main folder
echo "Extracting files from current commit"
git --work-tree=$APP_FOLDER --git-dir=$GIT_FOLDER checkout -f

# Switch to app folder
echo "Switching to $APP_FOLDER"
cd $APP_FOLDER

# Compile the app and move it to the correct folder
echo "Starting to build Swift app"
sudo swift build
sudo chown -R $USERNAME.$USERNAME .build/ Packages
echo ""

if [ -f $APP_FOLDER/.build/debug/$APP_NAME ]; then
    echo "Copying $APP_FOLDER/.build/debug/* app to $RUNNING_FOLDER/"
    cp -f $APP_FOLDER/.build/debug/* $RUNNING_FOLDER/
fi
if [ -d "$DIRECTORY" ]; then
    echo "Copying $APP_FOLDER/webroot folder to $RUNNING_FOLDER/webroot"
    cp -r $APP_FOLDER/webroot $RUNNING_FOLDER/webroot
else
    mkdir $RUNNING_FOLDER/webroot
fi

echo ""
echo "Setting permissions"
cd $HOME
sudo chown -R $USERNAME.$USERNAME running

# Find and restart the previous process
echo ""
echo "Restarting $APP_NAME"
sudo supervisorctl restart $APP_NAME
```

Once the file was saved you will need to make it executable by executing
```
chmod +x post-receive
```

Long story short, the `post-receive` file will do the following:
- Extract the last commit files and move them to the `app` folder located in your USERNAME's home folder
- Switch working folders to the `app` folder
- Start building your app
- Copy the compiled app to the `running` folder
- Copy the `webroot` folder to the `running` folder as well
- Try to find the PID of the compiled app (in case this is not the first commit)
  - If the app was already running, the script will kill it - at this point, the `supervisor` app (which we haven't installed yet) will kick in and restart it, but it will use the new compiled version
  - If the app was not running, like the first time we will deploy it, the script will start it and `supervisor` will keep an eye on it.

At this point the app compiling automated process should be all set up and ready to receive files. Next we'll setup the `supervisor` process that will watch over our app and keep it running.

### Setting up supervisor

Before installing it, make sure your system isn't running it already. You can run
```
supervisord --version
```
to make sure if `supervisor` is installed or not. If it's not, continue with
```
sudo apt-get install supervisor -y
service supervisor restart
```
and that should be all, as far as installation is required.

Next, we'll need to set the init script for your app. All the init scripts are located in the `/etc/supervisor/conf.d` folder.

```
cd /etc/supervisor/conf.d
sudo nano THE_APP_NAME.conf
```

In order for the git hook to do its job, you will need to name the init file with the same name you have used for the `$APP_NAME` variable in the hook file. Use the below code block to fill in this file. Be sure to replace `USERNAME` and `APP_NAME` with the correct values.
```
[program:APP_NAME]
command=/home/USERNAME/running/APP_NAME
autostart=true
autorestart=true
stderr_logfile=/var/log/APP_NAME.err.log
stdout_logfile=/var/log/APP_NAME.out.log
```

Save the file and quit the editor.

Once our configuration file is created and saved, we can inform Supervisor of our new program through the `supervisorctl` command. First we tell Supervisor to look for any new or changed program configurations in the `/etc/supervisor/conf.d` directory with:
```
supervisorctl reread
```

Right now, all that is left to do is create our app on our local environment and add the server as a git remote so we deploy our code. The second you will commit your files, the server will take care of everything else, leaving you to continue coding and not worrying about deployment and other configurations.

### Setting up your local environment

For the sake of this example, I'm going to use the Perfect Template repo, made available to us by the people working on the Perfect Framework (Thank you all @Perfect for the great work!!!)

```
cd ~
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
cd PerfectTemplate
swift build
```

First we'll clone the PerfectTemplate repo, switch into the folder and build the project (just to make sure everything works). If the build process was a success you can run the app locally to test further (first time building an app will take you a bit longer because of all packages being downloaded and compiled).
```
./.build/debug/PerfectTemplate
```

You should see something like
```
[INFO] Starting HTTP server on 0.0.0.0:8181 with document root ./webroot
```
which means the app is running on your localhost on port 8181. You can open another terminal window and run `curl localhost:8181` and see the response, which in the case of the PerfectTemplate will look something like
```
<html><title>Hello, world!</title><body>Hello, world!</body></html>
```

Ok, back to the terminal window running the app. Press `Ctrl+C` to stop the execution - we need to add our server as a remote to git and deploy on our server.

In the folder hosting your Swift app (PerfectTemplate if you're following my example) you need to run the following (please adjust the variables):
```
git remote add REMOTE_NAME ssh://USERNAME@YOUR_DROPLET_IP/home/USERNAME/app/.git
```

In my case, the line looks like this:
```
git remote add live ssh://perfectapp@146.185.134.227/home/perfectapp/app/.git
```
- `REMOTE_NAME` is the name you would like to give this remote address
- `USERNAME` is the username created on the server
- `YOUR_DROPLET_IP` is the IP address of your droplet (or domain name if you've already added a DNS entry for it)

Once the new remote is added, all you need to do is push your repo to this new remote.
```
git push live master
```
A.K.A. `git push REMOTE_NAME BRANCH`

**!!! Panic ensues !!!** But, but ... it's asking me for a password - WTF!?!?!

It has time to setup SSH on your local environment to use the key you have created earlier (if you created a key that is...)

If you have chosen the easy way out, just type/paste your chosen password and press Enter. If not, let's quickly tell your local SSH how to handle the key.

We will need to edit a file located in your local `.ssh` folder, the file's name is `config`. You might have it or not, so let's make sure it's there...

```
echo >> ~/.ssh/config
nano ~/.ssh/config
```

Add the following to the end of the file (those of you that set up the key on your local environment should know the variables below):
```
Host YOUR_DROPLET_IP
IdentityFile LOCAL_KEY_PATH
```

Once the file is saved try to push to your server's git repo again... Watch the magic happen!

### Getting your app to meet the internet

Well, at this step, we have everything up and running, everything compiles successfully (at least it should), but still, the app is bound to port 8181 and on localhost. You would need root account privileges in order to bind the app on a lower port value than 1024, but we will correct that by using `nginx` and `proxy passing`.

First, lets install nginx:
```
sudo apt-get install nginx -y
```

Once installed, lets configure our virtual host
```
cd /etc/nginx/sites-available
sudo rm -rf default ../sites-enabled/default
sudo nano app.vhost
```

You can use the below code block to setup your virtual host. We'll use the following variables in the code block:
- `SERVER_NAME` The name of the domain your app will have
```
server {
    listen 80;
    server_name SERVER_NAME;
    root /home/USERNAME/running/webroot;
    access_log /var/log/nginx/SERVER_NAME.access.log;
    error_log /var/log/nginx/SERVER_NAME.error.log;

    location ^~ / {
        proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://127.0.0.1:8181;
        proxy_redirect off;
    }
}
```

Once the file is saved, you will need to tell nginx that this vhost should be enabled...
```
sudo ln -s /etc/nginx/sites-available/app.vhost /etc/nginx/sites-enabled/app.vhost
```

Next, lets change the config of nginx itself...
```
cd /etc/nginx
sudo mv nginx.conf nginx.conf.backup
sudo nano nginx.conf
```

You can use the block below to paste in the new config file. If you want you can tweak some of the settings like `worker_processes` which should be equal to the number of processors of the server.
```
user www-data;
worker_processes 1;
pid /run/nginx.pid;

events {
    worker_connections 768;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile       on;
    tcp_nopush     on;

    fastcgi_buffers 16 256k;
    fastcgi_buffer_size 256k;

    open_file_cache max=5000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;

    tcp_nodelay         on;
    types_hash_max_size 2048;
    reset_timedout_connection on;

    server_names_hash_bucket_size 64;
    server_tokens       off;

    client_body_buffer_size 10K;
    client_header_buffer_size 1k;
    client_max_body_size 20M;
    large_client_header_buffers 4 16k;

    client_body_timeout 12;
    client_header_timeout 12;
    keepalive_timeout   15;
    send_timeout        10;

    gzip                on;
    gzip_comp_level     5;
    gzip_min_length     1024;
    gzip_disable        "msie6";
    gzip_proxied        expired no-cache no-store private auth;
    gzip_vary           on;
    gzip_buffers        16 8k;
    gzip_http_version   1.1;
    gzip_types          text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript text/x-js image/svg+xml;

    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA;
    ssl_prefer_server_ciphers on;

    upstream php {
        server unix:/tmp/php5-fpm.sock;
    }

    include /etc/nginx/sites-enabled/*;
}
```

Save the file, and restart the nginx server (yes, it is running since the installation time).
```
sudo /etc/init.d/nginx restart
```

Head on over to your local environment and try
```
curl http://YOUR_DROPLET_IP
```

The output should be `<html><title>Hello, world!</title><body>Hello, world!</body></html>` provided you've used the same code as me.

##Success!

The app is now running in the background, nginx is providing a way for the app to be reached from the outside and you are done with all the complicated work - all you need to do now is change something in your app and commit the changes to the live remote.
