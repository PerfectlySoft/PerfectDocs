# 为Swift 3 ＋ Perfect 2 创建 Ubuntu 15.10 基本镜像

本文将帮助您创建Ubuntu 15.10以使用Swift 3语言在PerfectlySoft公司的Perfect 2应用程序框架下开发项目。

## Running as the Root User

请首先安装准备好一个Ubuntu 15.10系统。为了方便学习，请按照以 **root** 用户顺序执行命令。您需要使用管理员账号和配套的密码。如果忽略这一步，那么在以下大部分内容内都需要使用 **sudo** 命令作为所有命令的开头

```
sudo su
```

## Running a Screen Session

如果您是用 **SSH** 连接Ubuntu的，那么您可能需要为以下步骤创建一个 **screen** 会话过程。这种方式能够保证系统在即使您连接到服务器的会话中断的情况下依然不会挂起。这一步不是必须的，可以跳过去看下一步。如果需要建立一个名为“perfect”的 **screen** 会话，请使用：

```
screen -S perfect
```

如果 **screen** 没有安装，请用下面的命令行安装：

```
apt-get install screen
```

如果需要断开 **screen** 会话连接，可以使用组合键 **Control-A, D** 。如果需要重新回到被断开的 **screen** 会话，请使用命令：

```
screen -r perfect
```

如果是不小心断开的 **screen** 会话，可以使用以下命令重新连接：

```
screen -D -R perfect
```

## 通过APT安装项目必要的依存关系库

Swift 3 ＋ Perfect 2需要一些函数库以支持其运行，可以通过APT命令行进行安装：

```
apt-get install make git clang libicu-dev libmysqlclient-dev libpq-dev sqlite3 libsqlite3-dev apache2-dev pkg-config libssl-dev libsasl2-dev libcurl4-openssl-dev uuid-dev wget
```

## 从源代码安装 MongoDB

用下面的方法可以下载 MongoDB 源代码：

```
cd /usr/src/
wget https://github.com/mongodb/mongo-c-driver/releases/download/1.3.5/mongo-c-driver-1.3.5.tar.gz
```

下载后请解压缩：

```
gunzip mongo-c-driver-1.3.5.tar.gz
```

然后展开 MongoDB 源代码：

```
tar -xvf mongo-c-driver-1.3.5.tar
```

再删除 MongoDB 源代码档案包：

```
rm mongo-c-driver-1.3.5.tar
```

在编译 MongoDB 源代码前首先执行配置命令：

```
cd mongo-c-driver-1.3.5/
./configure --enable-sasl=yes
```

编译 MongoDB:

```
make
```

安装 MongoDB:

```
make install
```

## 结束

恭喜！现在您的系统环境已经可以支持Swift 3 ＋ Perfect 2联合工作。

## 尝试别的方法

如果您希望安装一个Swift在3.0之前的“开发快照”版本，以按照以下命令下载：

```
cd /usr/src/
wget https://swift.org/builds/swift-3.0-preview-3/ubuntu1510/swift-3.0-PREVIEW-3/swift-3.0-PREVIEW-3-ubuntu15.10.tar.gz`
```

安装方法：

```
gunzip < swift-3.0-PREVIEW-3-ubuntu15.10.tar.gz | tar -C / -xv --strip-components 1
```

完成后删除压缩档：

```
rm swift-3.0-PREVIEW-3-ubuntu15.10.tar.gz
```

获取 Perfect 2 模板项目：

```
git clone https://github.com/PerfectlySoft/PerfectTemplate
```

并编译：

```
cd PerfectTemplate
git checkout d13e8dd8eb4868fea36468758604fe05a48b9aa2
swift build
```

编译完成后应用程序会出现在 `.build/debug/` 目录下。可以为方便使用拷贝到别的路径，比如 `/root/` 目录：

```
cp .build/debug/PerfectTemplate /root/
```

现在您就可以从 `/root/` 路径下执行程序。请小心，如果您还使用的是 **root** 用户，那么您的程序会获得 **root** 用户权限！

```
/root/PerfectTemplate --port 80
```

下一步即可通过 `curl` 命令测试服务器：

```
curl http://127.0.0.1
```
如果您发现任何问题，请进行[问题报告](http://jira.perfect.org:8080/servicedesk/customer/portal/1).
