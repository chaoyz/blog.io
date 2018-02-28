# Link via an ambassador container

## 使用&#39;使节&#39;容器链接两个服务

## Introduction

Rather than hardcoding硬编码 network links between a service consumer and provider, Docker encourages service portability, for example instead of:

docker提倡服务的可移植性,不使用硬编码链接服务的提供者和使用者

    (consumer) --> (redis)

Requiring you to restart the `consumer` to attach it to a different `redis`service, you can add ambassadors:

重启我们的服务容器链接redis服务时中间使用一个ambassador链接两者

    (consumer) -->(redis-ambassador) --> (redis)
    

Or

    (consumer) -->(redis-ambassador) ---network--->(redis-ambassador) --> (redis)
    

When you need to rewire your consumer to talk to a different Redis server, you can just restart the `redis-ambassador` container that the consumer is connected to.

当你需要更换到另一个redis服务的时候,可以通过重新启动与客户链接的redis-ambassador容器.

This pattern also allows you to transparently move the Redis server to a different docker host from the consumer.

这种模式也可以让你移动redis服务器到其他的docker主机上

Using the `svendowideit/ambassador` container, the link wiring is controlled entirely from the `docker run` parameters.

使用svendowideit/ambassador容器.

## Two host example

两个host情况下的示例

Start actual Redis server on one Docker host

在docker上开启一个redis服务

    big-server $ docker run -d --name redis crosbymichael/redis

Then add an ambassador linked to the Redis server, mapping a port to the outside world

然后使用一个ambassador容器连接到redis服务容器上,映射一个端口到外面的世界

    big-server $ docker run -d --link redis:redis --name redis_ambassador -p 6379:6379 svendowideit/ambassador

On the other host, you can set up another ambassador setting environment variables for each remote port we want to proxy to the `big-server`

在另一台机器上,可以安装另一个ambassador容器作为big-server的远程代理

    client-server $ docker run -d --name redis_ambassador --expose 6379 -e REDIS_PORT_6379_TCP=tcp://192.168.1.52:6379 svendowideit/ambassador

Then on the `client-server` host, you can use a Redis client container to talk to the remote Redis server, just by linking to the local Redis ambassador.

然后在客户机上,就可以使用本地的Redis ambassador与远程的redis服务器通信了

    client-server $ docker run -i -t --rm --link redis_ambassador:redis relateiq/redis-cli
    redis 172.17.0.160:6379> ping
    PONG
    

## How it works

这是如何工作的

The following example shows what the `svendowideit/ambassador`container does automatically (with a tiny amount of `sed`)

下面的例子将会展示ambassador容器如何工作.

On the Docker host (192.168.1.52) that Redis will run on:

    # start actual redis server启动真正的redis服务
    $ docker run -d--name redis crosbymichael/redis
    
    # get a redis-cli containerforconnection testing获取一个redis-client用于测试
    $ docker pull relateiq/redis-cli
    
    # test the redis serverby talking to it directly测试一下直接与redis服务通信
    $ docker run -t -i--rm --link redis:redis relateiq/redis-cli
    redis 172.17.0.136:6379> ping
    PONG
    ^D
    
    # add redis ambassador增加一个容器
    $ docker run -t -i--link redis:redis --name redis_ambassador -p 6379:6379 alpine:3.2 sh

In the `redis_ambassador` container, you can see the linked Redis containers `env`:在redis_ambassador容器内,可以看到链接到的redis容器环境

    / # env
    REDIS_PORT=tcp://172.17.0.136:6379REDIS_PORT_6379_TCP_ADDR=172.17.0.136REDIS_NAME=/redis_ambassador/redis
    HOSTNAME=19d7adf4705e
    SHLVL=1HOME=/root
    REDIS_PORT_6379_TCP_PORT=6379REDIS_PORT_6379_TCP_PROTO=tcp
    REDIS_PORT_6379_TCP=tcp://172.17.0.136:6379TERM=xterm
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    PWD=/
    / # exit
    

This environment is used by the ambassador `socat` script to expose Redis to the world (via the `-p 6379:6379` port mapping):

    $ docker rm redis_ambassador
    $ CMD="apk update && apk add socat && sh"
    $ docker run -t -i --link redis:redis --name redis_ambassador -p 6379:6379 alpine:3.2 sh -c "$CMD"
    [...]
    / # socat -t 100000000 TCP4-LISTEN:6379,fork,reuseaddr TCP4:172.17.0.136:6379

Now ping the Redis server via the ambassador:

Now go to a different server:

    $ CMD="apk update && apk add socat && sh"
    $ docker run -t -i --expose 6379 --name redis_ambassador alpine:3.2 sh -c "$CMD"
    [...]
    / # socat -t 100000000 TCP4-LISTEN:6379,fork,reuseaddr TCP4:192.168.1.52:6379

And get the `redis-cli` image so we can talk over the ambassador bridge.

    $ docker pull relateiq/redis-cli
    $ docker run -i -t --rm --link redis_ambassador:redis relateiq/redis-cli
    redis 172.17.0.160:6379> ping
    PONG
    

## The svendowideit/ambassador Dockerfile

The `svendowideit/ambassador` image is based on the `alpine:3.2`image with `socat` installed. When you start the container, it uses a small`sed` script to parse out the (possibly multiple) link environment variables to set up the port forwarding. On the remote host, you need to set the variable using the `-e` command line option.

    --expose 1234 -e REDIS_PORT_1234_TCP=tcp://192.168.1.52:6379
    

Will forward the local `1234` port to the remote IP and port, in this case`192.168.1.52:6379`.

    #
    # do
    #   docker build -t svendowideit/ambassador .
    # thento run it (on the host that has the real backend on it)
    #   docker run -t -i -link redis:redis -name redis_ambassador -p6379:6379 svendowideit/ambassador
    # on the remote host, you can set up another ambassador
    #    docker run -t -i -name redis_ambassador -expose 6379 -e REDIS_PORT_6379_TCP=tcp://192.168.1.52:6379 svendowideit/ambassador sh
    # you can read more about this process at https://docs.docker.com/articles/ambassador_pattern_linking/
    
    # use alpine because its a minimal image with a package manager.
    # prettymuch all that is needed is a container that has a functioning env and socat (or equivalent)
    FROM    alpine:3.2
    MAINTAINER  SvenDowideit@home.org.au
    
    RUN apk update && \
        apk add socat && \
        rm -r /var/cache/
    
    CMD env | grep _TCP= | sed &#39;s/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat -t 100000000 TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \& wait/&#39; | sh