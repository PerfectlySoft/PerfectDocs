# Redis

Redis是BSD许可的开源、内存驻留存储的数据结构仓库，经常用于数据库、高速缓冲和消息代理。

详见[http://redis.io](http://redis.io/)

### 安装Redis

### macOS

```
brew install redis
```

在控制台下启动redis的方法：`brew services start redis`

或者如果需要在后台以服务方式自动启动，可以这样使用：`redis-server /usr/local/etc/redis.conf`

### Linux

```
sudo apt-get install redis-server
```

### 快速上手

除了项目内引用PerfectLib之外，您还需要在Package.swift文件内设置Perfect-Redis依存关系：
