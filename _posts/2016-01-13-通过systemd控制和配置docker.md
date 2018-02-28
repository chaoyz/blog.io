# Control and configure Docker with systemd

Many Linux distributions use systemd to start the Docker daemon. This document shows a few examples of how to customise Docker’s settings.

很多linux发行版本都使用systemd启动docker daemon.这篇指导文档用几个小例子展示如何自定义docker配置

## Starting the Docker daemon

启动docker daemon

Once Docker is installed, you will need to start the Docker daemon.

docker安装好后,你需要启动docker daemon才能玩docker

    $ sudo systemctl start docker
    # oron older distributions, you may need touse
    $ sudo service docker start

If you want Docker to start at boot, you should also:

开机自启动docker命令如下

    $ sudo systemctl enable docker
    # oron older distributions, you may need touse
    $ sudo chkconfig docker on

## Custom Docker daemon options

自定义docker daemon参数

There are a number of ways to configure the daemon flags and environment variables for your Docker daemon.

有很多方式自定义docker daemon的配置.

The recommended way is to use a systemd drop-in file. These are local files in the `/etc/systemd/system/docker.service.d` directory. This could also be `/etc/systemd/system/docker.service`, which also works for overriding the defaults from`/lib/systemd/system/docker.service`.

推荐使用的是更改docker 的启动文件,这些文件在/etc/systemd/system/docker.service.d文件夹下,也可能在/etc/systemd/system/docker.service文件,这些文件都是用来覆盖默认启动配置,默认的启动配置文件在/lib/systemd/system/docker.service.

However, if you had previously used a package which had an`EnvironmentFile` (often pointing to `/etc/sysconfig/docker`) then for backwards compatibility, you drop a file in the`/etc/systemd/system/docker.service.d` directory including the following:

    [Service]
    EnvironmentFile=-/etc/sysconfig/docker
    EnvironmentFile=-/etc/sysconfig/docker-storage
    EnvironmentFile=-/etc/sysconfig/docker-network
    ExecStart=
    ExecStart=/usr/bin/docker daemon -H fd:// $OPTIONS \
              $DOCKER_STORAGE_OPTIONS \
              $DOCKER_NETWORK_OPTIONS \
              $BLOCK_REGISTRY \
              $INSECURE_REGISTRY
    

To check if the `docker.service` uses an `EnvironmentFile`:

    $ sudo systemctl show docker | grep EnvironmentFile
    EnvironmentFile=-/etc/sysconfig/docker (ignore_errors=yes)
    

Alternatively, find out where the service file is located:

    $ sudo systemctl status docker | grep Loaded
       Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled)
    $ sudo grep EnvironmentFile /usr/lib/systemd/system/docker.service
    EnvironmentFile=-/etc/sysconfig/docker
    

You can customize the Docker daemon options using override files as explained in the [HTTP Proxy example](https://docs.docker.com/engine/articles/systemd/#http-proxy) below. The files located in`/usr/lib/systemd/system` or `/lib/systemd/system` contain the default options and should not be edited.

我们可以通过文档接下来所讲的HTTP Proxy例子自定义docker daemon选项.在/usr/lib/systemd/system或者/lib/systemd/system文件夹下的有关docker的文件我们不应该去更改他们!

### Runtime directory and storage driver

运行时文件夹和存储驱动

You may want to control the disk space used for Docker images, containers and volumes by moving it to a separate partition.

我们可以操作磁盘上的文件分离镜像,容器和卷.

In this example, we’ll assume that your `docker.service` file looks something like:

在这个例子中,我们假设你的docker.service文件向下面这样:

    [Unit]
    Description=Docker Application Container Engine
    Documentation=https://docs.docker.com
    After=network.target docker.socket
    Requires=docker.socket
    
    [Service]
    Type=notify
    ExecStart=/usr/bin/docker daemon -H fd://
    LimitNOFILE=1048576
    LimitNPROC=1048576
    
    [Install]
    Also=docker.socket
    

This will allow us to add extra flags via a drop-in file (mentioned above) by placing a file containing the following in the`/etc/systemd/system/docker.service.d` directory:

我们可以向文件中插入一些扩展标记

    [Service]
    ExecStart=
    ExecStart=/usr/bin/docker daemon -H fd:// --graph /mnt/docker-data --storage-driver btrfs
    

You can also set other environment variables in this file, for example, the`HTTP_PROXY` environment variables described below.

To modify the ExecStart configuration, specify an empty configuration followed by a new configuration as follows:

    [Service]
    ExecStart=
    ExecStart=/usr/bin/docker daemon -H fd:// --bip=172.17.42.1/16

If you fail to specify an empty configuration, Docker reports an error such as:

    docker.service has more than one ExecStart= setting, which is only allowed forType=oneshot services. Refusing.
    

### HTTP proxy

This example overrides the default `docker.service` file.

这个例子重写默认的docker.service文件

If you are behind a HTTP proxy server, for example in corporate settings, you will need to add this configuration in the Docker systemd service file.

如果绑定了一个http代理服务器, 你需要把这些配置参数添加到docker系统配置文件中

First, create a systemd drop-in directory for the docker service:

首先为docker服务创建一个文件夹

    mkdir /etc/systemd/system/docker.service.d

Now create a file called`/etc/systemd/system/docker.service.d/http-proxy.conf` that adds the `HTTP_PROXY` environment variable:

然后在这里创建一个http-proxy.conf,其中添加一些关于http代理环境配置

    [Service]Environment="HTTP_PROXY=http://proxy.example.com:80/"

If you have internal Docker registries that you need to contact without proxying you can specify them via the `NO_PROXY` environment variable:

如果本地有registries,我们希望访问本地时不使用http代理, 所以指定NO_PROXY参数即可

    Environment="HTTP_PROXY=http://proxy.example.com:80/""NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"

Flush changes:

使更改生效

    $ sudo systemctl daemon-reload
    

Verify that the configuration has been loaded:

    $ sudo systemctl show docker --property Environment
    Environment=HTTP_PROXY=http://proxy.example.com:80/
    

Restart Docker:

    $ sudo systemctl restart docker
    

## Manually creating the systemd unit files

When installing the binary without a package, you may want to integrate Docker with systemd. For this, simply install the two unit files (service and socket) from [the github repository](https://github.com/docker/docker/tree/master/contrib/init/systemd) to `/etc/systemd/system`.