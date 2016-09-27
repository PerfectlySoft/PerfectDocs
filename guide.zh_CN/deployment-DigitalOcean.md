# Digital Ocean Deployment

<<<<<<< HEAD
本文档将协助您设置一个Digital Ocean Droplet（数字海洋云服务虚拟机）用于运行您自己编译运行的Swift应用程序。以下步骤与任何运行Ubuntu 15和16的Linux服务器都是完全一样的。下面我们开始：

首先创建您的droplet虚拟机，选择Ubuntu 16 (如果15版本无法获取），然后安装。一旦完成就可以使用root账号通过ssh加密终端登陆该虚拟服务器。
```
ssh root@YOUR_DROPLET_IP -v
```
这一步需要使用您在创建虚拟机的时候创建的密钥，如果有这个密码钥匙，下次从您本地计算机登陆时就不需要密码了。

### 设置服务器 - 第一部分

下一步是创建一个用户，用于运行所有命令并启动最终目标应用程序。显然，如果所有工作都用root账号完成是对系统安全有潜在威胁的。有了普通账号之后，还可以随时通过 `sudo` 命令完成更高权限的操作。请根据您的具体情况调整以下命令行的用户名 `USERNAME`和密码`USER_PASSWORD`。
=======
This guide will walk you through setting up a Digital Ocean Droplet that will run your compiled Swift application. You can actually do all of these steps on any other Linux server running Ubuntu 15 or 16. Let's start...

First create your droplet, select Ubuntu 16 (if 15 is not available), and let it finish. Once the droplet is created you can ssh into it as root.
```
ssh root@YOUR_DROPLET_IP -v
```
This should use the private key you have created earlier and it should not ask you for a password or anything.

### Setting up the server - Part 1

The next step is creating a user to run all commands through and also run the final app since it's not that nice to do all as root. Since we're still connected as root we're going to skip the `sudo` commands since there's no point in using it. Please adjust the `USERNAME` and `USER_PASSWORD` variables in the command below.
>>>>>>> PerfectlySoft/master
```
useradd -d /home/USERNAME -m -s /bin/bash USERNAME
echo "USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

<<<<<<< HEAD
以上命令能以 `USERNAME` 创建用户并允许该用户使用 `sudo`命令。通配符 `ALL` 赋予了最高权限，因此请根据您系统管理的需要进行适当调整。

用户创建之后，可以有两个选择：1、为用户创建一个连接到服务器的密码，或者2、创建一个SSH密码钥匙用于连接服务器。这两种选择我们都会讨论（个人建议用SSH）。

### 选项1 - 创建一个密码

由于目前还在使用root账号，因此我们可以直接为用户设置密码：
=======
The last command will add your new `USERNAME` to the list of accounts allowed to run commands using `sudo`. The spectrum used here, `ALL`, is a bit unrestrictive and it is advised to limit the commands to just a few.

Once the user is created, you have two options: 1. Generate a password for this user and use the password when you want to connect to the server, or 2. Generate a SSH key and use that key to connect to the server - we're going to take a look at both options below (personally I prefer the SSH key over the password).

### Option 1 - Generate a password

Since we're still connected as root we're going to set the password for our user using
>>>>>>> PerfectlySoft/master
```
passwd USERNAME
```

<<<<<<< HEAD
设置完成后就可以关闭root连接并以 USERNAME 重新连接。

### 选项2 - 创建一个SSH密码钥匙

这个选项稍微复杂一些，但是完成之后再连接服务器会非常方便 —— 不用每次都输入用户密码，也不用担心密码安全性不够高，等等。

同样有两种选择：1、使用与root一样的密码钥匙；2、为USERNAME创建一个新的密码钥匙。

#### 子选项 1

如果要使用与虚拟机创建时相同的密码钥匙，请输入：
=======
Once that is done we can drop the connection as root (Ctrl-D) and reconnect as USERNAME.

### Option 2 - Generate a SSH key

This option is a bit more complicated to set up, but in the long run it will be easier to connect since we don't have to remember the password we've set or risk that the password isn't secure enough, etc...

There are again, two options here: 1. Use the same key as root, or 2. Generate a new key for USERNAME.

#### Sub-Option 1

To use the same key you've set up when the droplet was created you can just run
>>>>>>> PerfectlySoft/master
```
mkdir /home/USERNAME/.ssh
cp /root/.ssh/authorized_keys /home/USERNAME/.ssh
chown -R USERNAME.USERNAME /home/USERNAME/.ssh
```

<<<<<<< HEAD
#### 子选项 2

首先需要在本地环境下创建一把新的SSH钥匙 —— 如果您在使用Windows，请在网上搜索一下类似的操作。请注意，这一步是在本地计算机上完成的，而不是在您的虚拟机服务器上操作。
```
ssh-keygen -t rsa
```
运行后，密钥生成器将询问钥匙存放的地点。比如，我会把钥匙存到 `/Users/rb/.ssh/id_rsa_perfectapp` —— 从现在开始，我们称之为本地钥匙目录 `LOCAL_KEY_PATH`。

一旦完成密码生成，请把公钥部分提取出来，用于存放到服务器虚拟机上：
```
cat /Users/rb/.ssh/id_rsa_perfectapp.pub
```
其内容看起来类似于
=======
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
>>>>>>> PerfectlySoft/master
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3Mzd0naYx59XfiObuSEphNTupfNqHGFSRExAl+WS0yiA8j8Kja964qTg0Bs2DUD9OoazaAIUTrt+GoQFUSlP37a4XqUxu4Pm8LyDrvx+AGEj4ylr5PdfG847x7n73ucoBSHuo3Ws6NLgfBhJ4U6R/mpFsc7GtYbaIISSZb/DUKchLA5ihpdST0Nv8G5LIS+HqgWfy9DQS1/w1RfxjXL6UvydDNyOklMCyWnpswyl19pZmW/JhvrTj1x3RM5Qb3p3EV3Xp4ZoJgY0X675ovr+uq1f7dumH1a5gqz+NeL+cThWc4B7SZTXUu+/0XIOL0485FgE+BwBg6CETDyt960L9 rb@Roberts-MacBook-Pro.local
```

<<<<<<< HEAD
下一步，请回到虚拟机服务器，我们需要设置刚刚创建的公钥并存到服务器上用于识别客户端身份：
=======
Next, moving back to the droplet server, we need to set up the public key we've just created and set it up so the server can identify us with it.
>>>>>>> PerfectlySoft/master
```
su - USERNAME
mkdir .ssh
chmod 700 .ssh
echo "THE_PUBLIC_KEY_STRING" >> .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
exit
```
<<<<<<< HEAD
简言之，切换回新用户的主目录并将内含之前创建的公钥`authorized_keys`文件保存到`.ssh`目录下。最后两行回到上一级目录并将文件夹`.ssh`的权限及其内容均赋予`USERNAME`用户。

为了测试上述操作是否有效，包括钥匙是否被服务器认可，请在本地计算机上使用命令
```
ssh -i LOCAL_KEY_PATH USERNAME@YOUR_DROPLET_IP -v
```
现在可以用新的账号USERNAME登陆虚拟机了。


### 设置服务器 - 第二部分

从以下部分开始我们可以不再用超级用户 `root` 账号登陆，而是新建的普通用户进行设置服务器，直到准备好所有即将编译部署的Swift代码。

#### 在DigitalOcean虚拟机上更新所有的软件包
=======
Long story short, we've switched to the new user's home folder, we've created the folder `.ssh` and inside of that folder we've created a file called `authorized_keys` in which we've pasted the content of the public key created earlier. The last two lines are going back up a level in the folder structure and change the permissions of the `.ssh` folder and its contents to the `USERNAME` user.

To test that everything works, and the key was added successfully, on your local environment run the following
```
ssh -i LOCAL_KEY_PATH USERNAME@YOUR_DROPLET_IP -v
```
You should be logged into your droplet now, under the USERNAME account.


### Setting up the server - Part 2

Now that we are not longer connected to the droplet as `root` we can continue setting it up and preparing it for running/compiling Swift code and the arrival of you app.

#### Updating packages installed by DigitalOcean
>>>>>>> PerfectlySoft/master
```
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
sudo dpkg-reconfigure locales
sudo apt-get update
sudo apt-get upgrade -y
```
<<<<<<< HEAD
以上操作用于配置服务器的语言环境，并下载最新的软件包并安装更新（-y表示同意所有选项）。始终保持软件更新是一个维护系统稳定性的好习惯。

下一步，安装用于Swift编译和运行的必要软件：
=======
This will configure the locales installed on the server, update the packages list to the latest available and upgrade (-y without asking you anything) the installed packages - it's always a good practice to have the latest stable versions up and running.

Next, lets installed some of the packages needed by Swift to compile, and a few other packages needed in the process. You can do this by running
>>>>>>> PerfectlySoft/master
```
sudo apt-get install make clang libicu-dev pkg-config libssl-dev libsasl2-dev libcurl4-openssl-dev uuid-dev git curl wget unzip -y
```

<<<<<<< HEAD
随后，安装Swift。迄今为止Swift最新的可执行版本是`Swift 3.0 GM Candidate`。如果无法确定哪个版本是最新的，请查看[Swift下载](https://swift.org/download/) 。因为要编译基于Perfect2.0的应用程序，因此我们需要使用 Swift3.
=======
Next, we will install Swift so the whole system has access to the binaries. At the moment, the latest available version of Swift binaries/libraries is `Swift 3.0 GM Candidate` - If you are not sure what's the latest version, head on to https://swift.org/download/ and check there. Because we're trying to make Perfect2.0 compile on this server we need to use Swift3.
>>>>>>> PerfectlySoft/master

```
cd /usr/src
sudo wget https://swift.org/builds/swift-3.0-GM-CANDIDATE/ubuntu1510/swift-3.0-GM-CANDIDATE/swift-3.0-GM-CANDIDATE-ubuntu15.10.tar.gz
sudo gunzip < swift-3.0-GM-CANDIDATE-ubuntu15.10.tar.gz | sudo tar -C / -xv --strip-components 1
sudo rm -f swift-3.0-GM-CANDIDATE-ubuntu15.10.tar.gz
```

<<<<<<< HEAD
为了确定Swift是否安装成功，或者说版本是否正确，请在终端命令行内输入：
```
swift --version
```
如果安装成功，那么无论命令行处于服务器的哪一个目录下，都会有类似与如下的版本反馈：
=======
You can test if swift was copied/installed correctly by running
```
swift --version
```
in any location on the server. You should get back something like
>>>>>>> PerfectlySoft/master
```
Swift version 3.0 (swift-3.0-GM-CANDIDATE)
Target: x86_64-unknown-linux-gnu
```

<<<<<<< HEAD
下一步要进行另一个选择 —— 如果要使用数据库，那么应该安装哪一种数据库？

### MySQL

如果您正在使用如亚马逊的RDS之类的数据库提供商，可以运行以下命令：
```
sudo apt-get install libmysqlclient-dev -y
```
但是如果希望将数据库安装在到目标Perfect2.0应用的同一个服务器上，请使用命令：
=======
At this point we have another fork in the road - What database system (if needed) will we use?

### MySQL

If you are going to use a database provider as a service (RDS from Amazon, etc) you can run
```
sudo apt-get install libmysqlclient-dev -y
```
but if you also want to host the database on the same server, you can run
>>>>>>> PerfectlySoft/master
```
sudo apt-get install mysql-client libmysqlclient-dev -y
```

<<<<<<< HEAD
下一步，请编辑位于目录`/usr/lib/x86_64-linux-gnu/pkgconfig`下的文件`mysqlclient.pc`。如果该目录下没找到这个文件，请尝试使用`find / -name mysqlclient.pc`。找到之后请打开编辑并删除文件内“-fno-omit-frame-pointer”和“-fabi-version=2”的内容：
=======
Next, you need to edit the file `mysqlclient.pc` located in the folder `/usr/lib/x86_64-linux-gnu/pkgconfig`. If you can't find the file there, please run a `find / -name mysqlclient.pc`. Once you find the file and edit it, you will need to remove any occurence of "-fno-omit-frame-pointer" and "-fabi-version=2".
>>>>>>> PerfectlySoft/master
```
sudo nano /usr/lib/x86_64-linux-gnu/pkgconfig/mysqlclient.pc
```

<<<<<<< HEAD
比如，我自己的文件内容如下：
=======
For example, my file looks like this:
>>>>>>> PerfectlySoft/master
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

<<<<<<< HEAD
编辑之后，应该是：
=======
and after the edit, the file looks like this:
>>>>>>> PerfectlySoft/master
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

<<<<<<< HEAD
如果数据库和Swift应用位于统一服务器，则需要采取下列操作对MySQL服务器进行配置：
=======
If you chose to host your database on the same server as the Swift app, you will have to do a few more steps in order to configure the MySQL server.
>>>>>>> PerfectlySoft/master

```
sudo mysql_secure_installation
```
<<<<<<< HEAD
此时会提示您输入第一步中root超级用户的密码。可以输入回车默认所有的问题，除非您希望改变超级用户的密码。

随后可以初始化MySQL数据目录，即数据库存储的文件夹。具体的操作依赖于MySQL的具体版本。操作之前可以用下列命令查看版本：
=======
This will prompt you for the root password you created in step one. You can press ENTER to accept the defaults for all the subsequent questions, with the exception of the one that asks if you'd like to change the root password.

Next, we'll initialize the MySQL data directory, which is where MySQL stores its data. How you do this depends on which version of MySQL you're running. You can check your version of MySQL with the following command.
>>>>>>> PerfectlySoft/master

```
mysql --version
```
<<<<<<< HEAD
执行后可以看到类似与下面的内容反馈：
```
mysql  Ver 14.14 Distrib 5.7.11, for Linux (x86_64) using EditLine wrapper
```
如果您当前的MySQL低于5.7.6，则需要使用`mysql_install_db`初始化数据库目录：
=======
You'll see some output like this:
```
mysql  Ver 14.14 Distrib 5.7.11, for Linux (x86_64) using EditLine wrapper
```
If you're using a version of MySQL earlier than 5.7.6, you should initialize the data directory by running `mysql_install_db`.
>>>>>>> PerfectlySoft/master
```
sudo mysql_install_db
```

<<<<<<< HEAD
请注意在MySQL 5.6系统上，可能会遭遇一个致命错误（FATAL ERROR）`Could not find my-default.cnf`。如果真是这样，请将配置文件`/usr/share/my.cnf`拷贝到`mysql_install_db`需要的目录下，然后运行。
=======
Please note that in MySQL 5.6, you might get an error that says FATAL ERROR: Could not find my-default.cnf. If you do, copy the `/usr/share/my.cnf` configuration file into the location that `mysql_install_db` expects, then rerun it.
>>>>>>> PerfectlySoft/master
```
sudo cp /etc/mysql/my.cnf /usr/share/mysql/my-default.cnf
sudo mysql_install_db
```
<<<<<<< HEAD
导致这个错误的原因是因为在MySQL 5.6版本的APT打包时有一个小错误。

MySQL 5.7.6之后的版本已经淘汰了mysql_install_db命令。如果您正在使用5.7.6或更高版本，请使用`mysqld --initialize`代替mysql_install_db。

但是，如果您是从Debian发行的Linux安装5.7的话，数据目录会自动初始化，所以不需要您进行任何手工操作；相反，如果按照上述命令进行操作，则会看到一个错误：
=======
This is due to some changes made in MySQL 5.6 and a minor error in the APT package.

The mysql_install_db command is deprecated as of MySQL 5.7.6. If you're using version 5.7.6 or later, you should use `mysqld --initialize` instead.

However, if you installed version 5.7 from the Debian distribution, the data directory was initialized automatically, so you don't have to do anything. If you try running the command anyway, you'll see the following error:
>>>>>>> PerfectlySoft/master
```
2016-03-07T20:11:15.998193Z 0 [ERROR] --initialize specified but the data directory has files in it. Aborting.
```

### Sequel3
<<<<<<< HEAD
（本节内容尚未完成，敬请关注）

### PostgreSQL
（本节内容尚未完成，敬请关注）

### MongoDB
（本节内容尚未完成，敬请关注）


### 设置应用程序文件夹并自动编译和部署

现在我们完成了服务器设置和所有必要操作，可以开始进行最终应用程序内容设置、目录打包和编译自动化。

简单来说，实现服务器应用程序部署的关键方法是`git`。细心的读者会注意到在之前的安装过程中，我们把`git` 作为应用程序之一安装到了服务器上，也就是说， `git` 可以将Swift源代码传递到服务器上，我们之后需要编译该程序并将可执行的应用程序移动到一个干净的目录结构用于运行。

为了达到这个目的，我们使用git的钩子文件`post-receive`（即收到最新版本文件后自动执行的脚本），使得所有编译、更新和程序重启操作都可以按需自动完成。没错儿，我们会使用一个监控程序 `supervisor`来管理您开发的Swift应用，实现24x7不间断运行。

首先，将目录结构设置如下：

*注意*：此时您需要用普通用户身份登陆虚拟机，然后用`cd`或`cd ~`命令回到用户的默认目录。
=======
Coming soon...

### PostgreSQL
Coming soon...

### MongoDB
Coming soon...


### Setting up the app folders, automate compiling and all that good stuff

Now that we're finished with the server setup, and all prerequisites are installed, it is the time to move forward with the app setup, its folder structure and compile automation.

The main plan is to have the app delivered to the server using `git`. If you noticed, the earlier package installation, did add `git` as an app to the server. Okay, so, `git` will deliver the raw Swift files to the server, we will need to compile the app, move it to another folder (for a clear structure) and run it from there.

To acomplish this we will use the `post-receive` git hook, which will run all the needed commands to compile. move and restart the app. Yes, restart it, because we will also make use of the `supervisor` app to keep our Swift compiled application up and running 24/7.

First, let's setup the folder structure for our app.

*Note: At this point you should be logged into your droplet server as your USERNAME, and be located in the home folder of this user - If you are logged as USERNAME, just `cd ~` (or `cd` for short) and hit Enter.
>>>>>>> PerfectlySoft/master

```
mkdir -p {running,app/.git} && cd app/.git
git init . --bare
cd hooks && rm -rf *.sample
```
<<<<<<< HEAD
运行上述命令后会在用户的默认目录下建立两个文件夹，一个是`running`，用于放置可执行程序；另一个是`app`用于放置按产品分类的Swift源代码、git代码资源库等等。同一行命令中还将命令的执行位置转移到了`app/.git`文件夹下面，这个文件夹适用于管理应用程序资源的。`git init . --bare` 命令用于初始化一个空的资源库并包含标准的基本目录结构。下一步，我们将再次改变路径，设置为钩子文件夹并删除所有的样例文件。

随后就可以编辑`post-receive`“更新后自动处理脚本”，形成部署自动化操作。
=======
This will create two folders in the USERNAME's home folder: `running` which will host your final compiled app, and `app` which will host your raw Swift files, compile by-products, and your Git repo. On the same line we're also changing location to the `app/.git` folder which will host the app's repo. `git init . --bare` will initialize a bare git repo in this folder and create a very basic repo structure. Next, we're again changing location in the hooks folder and removing all the sample files there.

Next we will need to create the `post-receive` hook that will do all the heavy lifting for us.
>>>>>>> PerfectlySoft/master
```
nano post-receive
```

<<<<<<< HEAD
您可以选择将下列文件内容拷贝粘贴到您自己的脚本中去，但是请注意各个变量的值是与您的系统保持一致：
- `USERNAME` 您当前的用户名
- `THE_APP_NAME` 应用程序名称——请与Package.swift文件中的名称保持一致
=======
You can copy/paste the following block as your file contents, but be sure you change the following variables with their correct values:
- `USERNAME` your current username
- `THE_APP_NAME` the actual name of the compiled app - this is the name of your app as well written in the Package.swift file
>>>>>>> PerfectlySoft/master
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
<<<<<<< HEAD
echo "按照下列配置执行："
echo "用户：$USERNAME"
echo "宿主目录：$HOME"
echo "应用程序名称：$APP_NAME"
echo "应用程序文件夹：$APP_FOLDER"
echo "Git目录：$GIT_FOLDER"
echo "目标文件执行目录：$RUNNING_FOLDER"
echo "---------------------------------------"
echo ""
echo "清除过期版本"
=======
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
>>>>>>> PerfectlySoft/master
sudo rm -f $HOME/.build
sudo rm -f $HOME/Packages
echo ""

<<<<<<< HEAD
# 将文件从新提交版本移动到主文件夹下
echo "从新版本中提取文件"
git --work-tree=$APP_FOLDER --git-dir=$GIT_FOLDER checkout -f

# 切换到应用程序目录
echo "切换到 $APP_FOLDER"
cd $APP_FOLDER

# 编译程序并移动到正确的目录
echo "开始编译应用程序"
=======
# Move files from commit to the main folder
echo "Extracting files from current commit"
git --work-tree=$APP_FOLDER --git-dir=$GIT_FOLDER checkout -f

# Switch to app folder
echo "Switching to $APP_FOLDER"
cd $APP_FOLDER

# Compile the app and move it to the correct folder
echo "Starting to build Swift app"
>>>>>>> PerfectlySoft/master
sudo swift build
sudo chown -R $USERNAME.$USERNAME .build/ Packages
echo ""

if [ -f $APP_FOLDER/.build/debug/$APP_NAME ]; then
<<<<<<< HEAD
    echo "将 $APP_FOLDER/.build/debug/* 程序覆盖到 $RUNNING_FOLDER/"
    cp -f $APP_FOLDER/.build/debug/* $RUNNING_FOLDER/
fi
if [ -d "$DIRECTORY" ]; then
    echo "将 $APP_FOLDER/webroot 目录覆盖到 $RUNNING_FOLDER/webroot"
=======
    echo "Copying $APP_FOLDER/.build/debug/* app to $RUNNING_FOLDER/"
    cp -f $APP_FOLDER/.build/debug/* $RUNNING_FOLDER/
fi
if [ -d "$DIRECTORY" ]; then
    echo "Copying $APP_FOLDER/webroot folder to $RUNNING_FOLDER/webroot"
>>>>>>> PerfectlySoft/master
    cp -r $APP_FOLDER/webroot $RUNNING_FOLDER/webroot
else
    mkdir $RUNNING_FOLDER/webroot
fi

echo ""
<<<<<<< HEAD
echo "设置权限"
cd $HOME
sudo chown -R $USERNAME.$USERNAME running

# 锁定并重启应用程序
echo ""
echo "重启 $APP_NAME"
sudo supervisorctl restart $APP_NAME
```

一旦文件保存完毕，请将该脚本权限设置为可执行
=======
echo "Setting permissions"
cd $HOME
sudo chown -R $USERNAME.$USERNAME running

# Find and restart the previous process
echo ""
echo "Restarting $APP_NAME"
sudo supervisorctl restart $APP_NAME
```

Once the file was saved you will need to make it executable by executing
>>>>>>> PerfectlySoft/master
```
chmod +x post-receive
```

<<<<<<< HEAD
简言之，`post-receive`自动更新脚本的操作内容为：
- 从最新版本文件中提取应用程序并覆盖到当前用户宿主文件夹下的`app`目录
- 将工作目录就切换到`app`应用程序文件夹
- 编译程序
- 将编译好的程序覆盖到`running`运行目录
- 将`webroot`目录也拷贝到 `running`运行目录下
- 尝试找到正在运行的旧版本应用程序实例并确定其PID进程标识符（如果不是第一次运行，那么应该有一个旧版本正在后台运行）
  - 如果应用程序已经运行，脚本将会终止这个实例 —— 这个工作通过监控程序`supervisor`执行完成（注意我们还没安装这个监控程序），即终止旧实例并以覆盖的新版本重启新实例
  - 如果应用程序还没有运行，则进行首次部署并用监控程序`supervisor`调度启动。

此时应用程序的同步和自动编译脚本应该已经设置完成并且开始准备接收新文件。下一步我们会设置 `supervisor` 监控程序用于跟踪和管理目标应用程序。

### 设置监控程序

在安装之前，请确定当前系统并没有运行该程序的实例，您可以用下面的命令检查：
```
supervisord --version
```
以确定监控程序`supervisor`是否安装。如果没有安装，请用以下命令进行安装：
=======
Long story short, the `post-receive` file will do the following:
- Extract the last commit files and move them to the `app` folder located in your USERNAME's home folder
- Switch working folders to the `app` folder
- Start building your app
- Copy the compiled app to the `running` folder
- Copy the `webroot` folder to the `running` folder as well
- Try to find the PID of the compiled app (in case this is not the first commit)
  - If the app was already running, the script will kill it - at this point, the `supervisor` app (which we haven't installed yet) will kick in and restart it, but it will use the new compiled version
  - If the app was not running, like the first time we will deploy it, the script will start it and `supervisor` will keep an eye on it.

At this point the app compiling automated process should be all set up and ready to receive files. Next we'll setup the `supervisor` process that will watch over our app and keep it runing.

### Setting up supervisor

Before installing it, make sure your system isn't running it already. You can run
```
supervisord --version
```
to make sure if `supervisor` is installed or not. If it's not, continue with
>>>>>>> PerfectlySoft/master
```
sudo apt-get install supervisor -y
service supervisor restart
```
<<<<<<< HEAD
这样安装就应该能够确保完成。

下一步，需要为应用程序初始化脚本。所有的初始化脚本都位于 `/etc/supervisor/conf.d` 文件夹下。
=======
and that should be all, as far as installation is required.

Next, we'll need to set the init script for your app. All the init scripts are located in the `/etc/supervisor/conf.d` folder.
>>>>>>> PerfectlySoft/master

```
cd /etc/supervisor/conf.d
sudo nano THE_APP_NAME.conf
```

<<<<<<< HEAD
为了确定git钩子功能能够正常发挥作用，您需要确定文件名为与钩子文件内的`$APP_NAME`变量值保持一致。请用下列代码修改文件，注意务必将用户名`USERNAME`和应用名`APP_NAME`替换为正确的值。
=======
In order for the git hook to do its job, you will need to name the init file with the same name you have used for the `$APP_NAME` variable in the hook file. Use the below code block to fill in this file. Be sure to replace `USERNAME` and `APP_NAME` with the correct values.
>>>>>>> PerfectlySoft/master
```
[program:APP_NAME]
command=/home/USERNAME/running/APP_NAME
autostart=true
autorestart=true
stderr_logfile=/var/log/APP_NAME.err.log
stdout_logfile=/var/log/APP_NAME.out.log
```

<<<<<<< HEAD
随后请保存文件并关闭文本编辑器。

一旦配置文件创建并完成，我们将用命令`supervisorctl`通知Supervisor监控程序配置完成可以使用。首先我们用下列命令通知Supervisor重新加载`/etc/supervisor/conf.d`目录配置：
=======
Save the file and quit the editor.

Once our configuration file is created and saved, we can inform Supervisor of our new program through the `supervisorctl` command. First we tell Supervisor to look for any new or changed program configurations in the `/etc/supervisor/conf.d` directory with:
>>>>>>> PerfectlySoft/master
```
supervisorctl reread
```

<<<<<<< HEAD
至此，剩下的工作就是创建本地文件，然后通过远程git方法部署新编代码。一旦提交部署，服务器就会自动完成剩下的所有工作，您只需关心源代码即可，无需在考虑部署和配置的额外工作。

### 设置本地环境

为了方便演示，我们使用Perfect的服务器模板文件（感谢 @Perfect 所有人的杰出贡献！！！）
=======
Right now, all that is left to do is create our app on our local environment and add the server as a git remote so we deploy our code. The second you will commit your files, the server will take care of everything else, leaving you to continue coding and not worrying about deployment and other configurations.

### Setting up your local environment

For the sake of this example, I'm going to use the Perfect Template repo, made avaialble to us by the people working on the Perfect Framework (Thank you all @Perfect for the great work!!!)
>>>>>>> PerfectlySoft/master

```
cd ~
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
cd PerfectTemplate
swift build
```

<<<<<<< HEAD
第一步是从github克隆PerfectTemplate服务器模板，切换到目标文件夹并进行编译（这个编译是为了验证一下程序本身没有问题）。如果编译成功，则可以本地进行服务器的简单测试（首次编译时间会长一些，因为中间会下载各个软件包并逐个编译）。
=======
First we'll clone the PerfectTemplate repo, switch into the folder and build the project (just to make sure everything works). If the build process was a success you can run the app locally to test further (first time building an app will take you a bit longer because of all packages being downloaded and compiled).
>>>>>>> PerfectlySoft/master
```
./.build/debug/PerfectTemplate
```

<<<<<<< HEAD
执行后应该能看到以下内容：
```
[INFO] Starting HTTP server on 0.0.0.0:8181 with document root ./webroot
```
这意味着应用服务器已经在本地8181端口运行。此时如果打开另外一个终端命令行并输入`curl localhost:8181`，应该可以看到以下反馈：
=======
You should see something like
```
[INFO] Starting HTTP server on 0.0.0.0:8181 with document root ./webroot
```
which means the app is running on your localhost on port 8181. You can open another terminal window and run `curl localhost:8181` and see the response, which in the case of the PerfectTemplate will look something like
>>>>>>> PerfectlySoft/master
```
<html><title>Hello, world!</title><body>Hello, world!</body></html>
```

<<<<<<< HEAD
现在，回到程序运行的窗口按下组合键`Ctrl+C`终止执行 —— 我们的目的是在真正的服务器上实现部署运行。

在您的Swift应用所在目录（如果您在使用我们的样例，那么应该是在PerfectTemplate文件夹下），您需要运行以下命令（请自行调整变量取值）：
=======
Ok, back to the terminal window running the app. Press `Ctrl+C` to stop the execution - we need to add our server as a remote to git and deploy on our server.

In the folder hosting your Swift app (PerfectTemplate if you're following my example) you need to run the following (please adjust the variables):
>>>>>>> PerfectlySoft/master
```
git remote add REMOTE_NAME ssh://USERNAME@YOUR_DROPLET_IP/home/USERNAME/app/.git
```

<<<<<<< HEAD
我自己安装的服务器这一部分看起来像这样：
```
git remote add live ssh://perfectapp@146.185.134.227/home/perfectapp/app/.git
```
- `REMOTE_NAME` 即远程服务器的名称
- `USERNAME` 即服务器用户名
- `YOUR_DROPLET_IP` 即您的虚拟机IP地址（如果您已经注册了DNS，则可以直接使用域名）

一旦增加了上述远程服务器配置，您就可以将新版本推送到远程服务器上。
```
git push live master
```
或者 `git push REMOTE_NAME BRANCH`

**⚠️注意⚠️** 但是，这里可能会提示我要密码？！？！什么鬼东东？！？！

如果您已经为自动登录创建了一个密码钥匙，那么是时候来使用这个密码钥匙设置本地SSH了。

如果您选择了那个不用钥匙但是使用用户名密码的方法，请将密码贴到这里来。如果您用的是钥匙，我们快速展示以下本地SSH如何使用密钥。

我们首先需要在您的本地`.ssh`文件夹下编辑一个文件，文件名为`config`。您可能已经有了这个文件，所以我们来确定一下：
=======
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
>>>>>>> PerfectlySoft/master

```
echo >> ~/.ssh/config
nano ~/.ssh/config
```

<<<<<<< HEAD
将下列内容增加到文件末尾（注意修改具体变量内容与您的系统保持一致）：
=======
Add the following to the end of the file (those of you that set up the key on your local environment should know the variables below):
>>>>>>> PerfectlySoft/master
```
Host YOUR_DROPLET_IP
IdentityFile LOCAL_KEY_PATH
```

<<<<<<< HEAD
一旦文件保存成功，请保存文件并再次推送到您服务器的git资源库……各位观众，敬请等待奇迹发生！

### 将您的应用程序推送到互联网上

至此，所有内容应该都编译成功，但仍然是在本地8181端口运行的。为了服务器在真正的互联网进行工作，您需要超级用户管理员账号将应用程序服务器端口调整到1024以下。为了实现这个目的，可以使用 `nginx` 服务器实现 `proxy passing`代理转发服务

首先安装nginx：
=======
Once the file is saved try to push to your server's git repo again... Watch the magic happen!

### Getting your app to meet the internet

Well, at this step, we have everything up and running, everything compiles successfully (at least it should), but still, the app is bound to port 8181 and on localhost. You would need root account priviledges in order to bind the app on a lower port value than 1024, but we will correct that by using `nginx` and `proxy passing`.

First, lets install nginx:
>>>>>>> PerfectlySoft/master
```
sudo apt-get install nginx -y
```

<<<<<<< HEAD
安装完毕后，配置虚拟主机：
=======
Once installed, lets configure our virtual host
>>>>>>> PerfectlySoft/master
```
cd /etc/nginx/sites-available
sudo rm -rf default ../sites-enabled/default
sudo nano app.vhost
```

<<<<<<< HEAD
您可以将下列内容复制到您的虚拟主机配置中去。我们会使用以下代码的变量内容：
- `SERVER_NAME` 服务器域名
=======
You can use the below code block to setup your virtual host. We'll use the following variables in the code block:
- `SERVER_NAME` The name of the domain your app will have
>>>>>>> PerfectlySoft/master
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

<<<<<<< HEAD
一旦文件保存之后，我们需要通知nginx虚拟主机已经准备好：
=======
Once the file is saved, you will need to tell nginx that this vhost should be enabled...
>>>>>>> PerfectlySoft/master
```
sudo ln -s /etc/nginx/sites-available/app.vhost /etc/nginx/sites-enabled/app.vhost
```

<<<<<<< HEAD
下一步，请改变nginx自身的配置
=======
Next, lets change the config of nginx itself...
>>>>>>> PerfectlySoft/master
```
cd /etc/nginx
sudo mv nginx.conf nginx.conf.backup
sudo nano nginx.conf
```

<<<<<<< HEAD
您可以用以下内容贴入您自己的配置文件。如果您希望实现优化配置，可以尝试改变`worker_processes`，使得与服务器的具体处理器数量匹配。
=======
You can use the block below to paste in the new config file. If you want you can tweak some of the settings like `worker_processes` which should be equal to the number of processors of the server.
>>>>>>> PerfectlySoft/master
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

<<<<<<< HEAD
保存文件，并重启nginx服务器（安装后这个程序会自动运行）：
=======
Save the file, and restart the nginx server (yes, it is running since the installation time).
>>>>>>> PerfectlySoft/master
```
sudo /etc/init.d/nginx restart
```

<<<<<<< HEAD
现在您可以回到本地环境尝试访问：
=======
Head on over to your local environment and try
>>>>>>> PerfectlySoft/master
```
curl http://YOUR_DROPLET_IP
```

<<<<<<< HEAD
不出意外的话，如果您的操作和我的例子完全一致，那么程序输出应该是`<html><title>Hello, world!</title><body>Hello, world!</body></html>`。

## 成功！！

现在服务器已经在背景正常运行服务器了。nginx提供了为您复杂的应用程序从外网访问的功能 —— 所有您需要做到事情是在您自己的应用程序上不断提交版本更新 —— 实现远程管理自动化。
=======
The output should be `<html><title>Hello, world!</title><body>Hello, world!</body></html>` provided you've used the same code as me.

##Successs!

The app is now running in the background, nginx is providing a way for the app to be reached from the outside and you are done with all the complicated work - all you need to do now is change something in your app and commit the changes to the live remote.
>>>>>>> PerfectlySoft/master
