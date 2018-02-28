# Configuring and running Docker on various distributions

After successfully installing Docker, the `docker` daemon runs with its default configuration.

成功安装docker之后,`docker` daemon使用它的默认配置运行.

In a production environment, system administrators typically configure the`docker` daemon to start and stop according to an organization’s requirements. In most cases, the system administrator configures a process manager such as `SysVinit`, `Upstart`, or `systemd` to manage the`docker` daemon’s start and stop.

在生产环境下,系统管理员根据需求编写配置文件启动和停止docker daemon.在大多数情况下,系统管理员配置一个管理进程例如SysVinit, Upstart, or systemd管理docker daemon的启动停止

### Running the docker daemon directly

直接运行docker daemon

The `docker` daemon can be run directly using the `docker daemon`command. By default it listens on the Unix socket`unix:///var/run/docker.sock`

我们可以通过$ docker daemon命令直接运行. 默认socket    unix:///var/run/docker.sock

    $ docker daemon
    
    INFO[0000] +job init_networkdriver()
    INFO[0000] +job serveapi(unix:///var/run/docker.sock)
    INFO[0000] Listening for HTTP on unix (/var/run/docker.sock)
    ...
    ...
    

### Configuring the docker daemon directly

直接配置daemon

If you’re running the `docker` daemon directly by running `docker daemon` instead of using a process manager, you can append the configuration options to the `docker` run command directly. Other options can be passed to the `docker` daemon to configure it.

如果通过docker daemon命令直接运行daemon而不是使用进程管理，我们可以指定运行参数进行配置

Some of the daemon’s options are:以下参数，这些参数不必死记，运行命令时，docker daemon --help可以看到所有
FlagDescription`-D`, `--debug=false`Enable or disable debug mode. By default, this is false. 是否运行debug模式，默认不使用debug模式`-H`,`--host=[]`Daemon socket(s) to connect to.指定host连接`--tls=false`Enable or disable TLS. By default, this is false.是否使用TLS（= =不懂啥玩意）默认为false
Here is a an example of running the `docker` daemon with configuration options:指定参数启动daemon

    $ docker daemon -D --tls=true --tlscert=/var/docker/server.pem --tlskey=/var/docker/serverkey.pem -H tcp://192.168.59.3:2376

These options :

- Enable `-D` (debug) mode    启动debug模式
- Set `tls` to true with the server certificate and key specified using `--tlscert` and `--tlskey` respectively   设置tls为true指定服务器证书和证书的key(= =啥玩意没搞懂)
- Listen for connections on `tcp://192.168.59.3:2376  `监听此端口

The command line reference has the [complete list of daemon flags](https://docs.docker.com/engine/reference/commandline/daemon/) with explanations.

命令行参数详情参照官网完整的daemon参数详解。

## Ubuntu

As of `14.04`, Ubuntu uses Upstart as a process manager. By default, Upstart jobs are located in `/etc/init` and the `docker` Upstart job can be found at `/etc/init/docker.conf`.

在ubuntu14.04下,使用Upstart进程管理工具.默认的,Upstart任务都在/etc/init下.

After successfully [installing Docker for Ubuntu](https://docs.docker.com/engine/installation/ubuntulinux/), you can check the running status using Upstart in this way:

可以使用一下命令检查docker是否启动

    $ sudo status docker
    
    docker start/running, process989

### Running Docker

运行docker

You can start/stop/restart the `docker` daemon using

    $ sudo start docker
    
    $ sudo stop docker
    
    $ sudo restart docker
    

### Configuring Docker

配置docker

You configure the `docker` daemon in the `/etc/default/docker` file on your system. You do this by specifying values in a `DOCKER_OPTS`variable.

修改/etc/default/docker文件来配置daemon

To configure Docker options:配置docker需要一下几点要求

1. 
Log into your host as a user with `sudo` or `root` privileges.需要sudo或者root权限

2. 
If you don’t have one, create the `/etc/default/docker` file on your host. Depending on how you installed Docker, you may already have this file.      如果没有/etc/default/docker文件,则手动创建一个, 取决于你的docker安装方式,在你的系统中可能已经存在了此文件.

3. 
Open the file with your favorite editor.    使用编辑器打开这个文件

    $ sudo vi /etc/default/docker

4. 
Add a `DOCKER_OPTS` variable with the following options. These options are appended to the `docker` daemon’s run command.   在daemon命令行中可以使用以下参数添加DOCKER_OPTS配置项进行配置.

        DOCKER_OPTS="-D--tls=true--tlscert=/var/docker/server.pem--tlskey=/var/docker/serverkey.pem-Htcp://192.168.59.3:2376"

These options :

- Enable `-D` (debug) mode   开启debug模式
- Set `tls` to true with the server certificate and key specified using `--tlscert` and `--tlskey` respectively
- Listen for connections on `tcp://192.168.59.3:2376`

The command line reference has the [complete list of daemon flags](https://docs.docker.com/engine/reference/commandline/daemon/) with explanations.

1. 
Save and close the file.  保存退出编辑文件

2. 
Restart the `docker` daemon. 重启daemon

    $ sudo restart docker
    

3. 
Verify that the `docker` daemon is running as specified with the `ps`command.

    $ ps aux | grep docker | grep -vgrep

### Logs

By default logs for Upstart jobs are located in `/var/log/upstart` and the logs for `docker` daemon can be located at`/var/log/upstart/docker.log`

通常Upstart进程的日志文件存放在/var/log/upstart目录下,dcoker daemon的日志文件存放在此路径下的docker.log中.

    $ tail -f /var/log/upstart/docker.log
    INFO[0000] Loading containers: done.
    INFO[0000] docker daemon: 1.6.04749651; execdriver: native-0.2; graphdriver: aufs
    INFO[0000] +job acceptconnections()
    INFO[0000] -job acceptconnections()= OK (0)
    INFO[0000] Daemon has completed initialization
    

## CentOS / Red Hat Enterprise Linux / Fedora

As of `7.x`, CentOS and RHEL use `systemd` as the process manager. As of `21`, Fedora uses `systemd` as its process manager.

After successfully installing Docker for [CentOS](https://docs.docker.com/engine/installation/centos/)/[Red Hat Enterprise Linux](https://docs.docker.com/engine/installation/rhel/)/[Fedora](https://docs.docker.com/engine/installation/fedora/), you can check the running status in this way:

    $ sudo systemctl status docker
    

### Running Docker

You can start/stop/restart the `docker` daemon using

    $ sudo systemctl start docker
    
    $ sudo systemctl stop docker
    
    $ sudo systemctl restart docker
    

If you want Docker to start at boot, you should also:

    $ sudo systemctl enable docker
    

### Configuring Docker

You configure the `docker` daemon in the `/etc/sysconfig/docker` file on your host. You do this by specifying values in a variable. For CentOS 7.x and RHEL 7.x, the name of the variable is `OPTIONS` and for CentOS 6.x and RHEL 6.x, the name of the variable is `other_args`. For this section, we will use CentOS 7.x as an example to configure the `docker` daemon.

By default, systemd services are located either in`/etc/systemd/service`, `/lib/systemd/system` or`/usr/lib/systemd/system`. The `docker.service` file can be found in either of these three directories depending on your host.

To configure Docker options:

1. 
Log into your host as a user with `sudo` or `root` privileges.

2. 
If you don’t have one, create the `/etc/sysconfig/docker` file on your host. Depending on how you installed Docker, you may already have this file.

3. 
Open the file with your favorite editor.

    $ sudo vi /etc/sysconfig/docker
    

4. 
Add a `OPTIONS` variable with the following options. These options are appended to the command that starts the `docker` daemon.

        OPTIONS="-D--tls=true--tlscert=/var/docker/server.pem--tlskey=/var/docker/serverkey.pem-Htcp://192.168.59.3:2376"

These options :

- Enable `-D` (debug) mode
- Set `tls` to true with the server certificate and key specified using `--tlscert` and `--tlskey` respectively
- Listen for connections on `tcp://192.168.59.3:2376`

The command line reference has the [complete list of daemon flags](https://docs.docker.com/engine/reference/commandline/daemon/) with explanations.

1. 
Save and close the file.

2. 
Restart the `docker` daemon.

    $ sudo systemctl restart docker
    

3. 
Verify that the `docker` daemon is running as specified with the `ps`command.

    $ ps aux | grep docker | grep -vgrep

### Logs

systemd has its own logging system called the journal. The logs for the`docker` daemon can be viewed using `journalctl -u docker`

    $ sudo journalctl -u docker
    May 0600:22:05 localhost.localdomain systemd[1]: Starting Docker Application Container Engine...
    May 0600:22:05 localhost.localdomain docker[2495]: time="2015-05-06T00:22:05Z" level="info" msg="+job serveapi(unix:///var/run/docker.sock)"
    May 0600:22:05 localhost.localdomain docker[2495]: time="2015-05-06T00:22:05Z" level="info" msg="Listening for HTTP on unix (/var/run/docker.sock)"
    May 0600:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="+job init_networkdriver()"
    May 0600:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="-job init_networkdriver() = OK (0)"
    May 0600:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="Loading containers: start."
    May 0600:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="Loading containers: done."
    May 0600:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="docker daemon: 1.5.0-dev fc0329b/1.5.0; execdriver: native-0.2; graphdriver: devicemapper"
    May 0600:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="+job acceptconnections()"
    May 0600:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="-job acceptconnections() = OK (0)"

*Note: Using and configuring journal is an advanced topic and is beyond the scope of this article.*

*
*

*
*

*
*

*
*

*
*