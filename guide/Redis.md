# Redis

Redis is an open source (BSD licensed), in-memory data structure store, used as database, cache, and message broker.

More info at [http://redis.io](http://redis.io)

### Installing Redis

### macOS

```
brew install redis
```

To have launchd start redis now and restart at login: ```brew services start redis```
  
Or, if you don't want or need a background service, you can run: ```redis-server /usr/local/etc/redis.conf```

### Linux

```
sudo apt-get install redis-server
```

### Getting Started

In addition to the PerfectLib, you will need the Perfect-Redis dependency in the Package.swift file:

