# Docker Machine

Machine lets you create Docker hosts on your computer, on cloud providers, and inside your own data center. It automatically creates hosts, installs Docker on them, then configures the `docker` client to talk to them. A “machine” is the combination of a Docker host and a configured client.

machine帮助你在个人电脑, 云服务或者自己的数据中心上创建docker hosts, 它可视实现自动的创建docker主机,再上面安装docker,安装docker client与dcoker通信. machine由docker host和一个配置好的client组成

Once you create one or more Docker hosts, Docker Machine supplies a number of commands for managing them. Using these commands you can

我们创建好一个或者多个docker host, docker machine提供了一些命令管理这些host.

使用这些命令我们可以做到:

- 
start, inspect, stop, and restart a host

启动, 查看,停止,重启一个host

- 
upgrade the Docker client and daemon

更新docker client和daemon

- 
configure a Docker client to talk to your host

配置一个dcoker client与host通信

## Understand Docker Machine basic concepts

理解docker machine 的基本概念

Docker Machine allows you to provision Docker on virtual machines that reside either on your local system or on a cloud provider. Docker Machine creates a host on a VM and you use the Docker Engine client as needed to build images and create containers on the host.

无论是在你的本地系统上还是云服务提供商,你都可以使用docker machine在虚拟机上安装一个docker.dcoker machine在虚拟机上创建一个host, 然后使用docker engine client制作镜像创建容器.

To create a virtual machine, you supply Docker Machine with the name of the driver you want use. The driver represents the virtual environment. For example, on a local Linux, Mac, or Windows system the driver is typically Oracle Virtual Box. For cloud providers, Docker Machine supports drivers such as AWS, Microsoft Azure, Digital Ocean and many more. The Docker Machine reference includes a complete [list of the supported drivers](https://docs.docker.com/machine/drivers/).

创建一个虚拟机时, 你指定docker machine使用哪种驱动, 驱动表示者虚拟环境. 例如:在linuxnmac,windwos系统上, 典型的驱动是vritual box. 对于云服务供应商,docker machine 支持AWS, Microsoft Azure, Digital Ocean 和一些其他的云服务. 

Since Docker runs on Linux, each VM that Docker Machine provisions relies on a base operating system. For convenience, there are default base operating systems. For the Oracle Virtual Box driver, this base operating system is the `boot2docker.iso`. For drivers used to connect to cloud providers, the base operating system is Ubuntu 12.04+. You can change this default when you create a machine. The Docker Machine reference includes a complete [list of the supported operating systems](https://docs.docker.com/machine/drivers/os-base/).

运行在linux上的docker, docker规定每一个虚拟机都依赖于一个基础的操作系统,

For each machine you create, the Docker host address is the IP address of the Linux VM. This address is assigned by the `docker-machine create`subcommand. You use the `docker-machine ls` command to list the machines you have created. The `docker-machine ip <machine-name>` command returns a specific host’s IP address.

Before you can run a `docker` command on a machine, you configure your command-line to point to that machine. The `docker-machine env <machine-name>` subcommand outputs the configuration command you should use. When you run a container on the Docker host, the container’s ports map to ports on the VM.

For a complete list of the `docker-machine` subcommands, see the[Docker Machine subcommand reference](https://docs.docker.com/machine/reference/).

## Crash Reporting

Provisioning a host is a complex matter that can fail for a lot of reasons. Your workstation may have a wide variety of shell, network configuration, VPN, proxy or firewall issues. There are also reasons from the other end of the chain: your cloud provider or the network in between.

To help `docker-machine` be as stable as possible, we added a monitoring of crashes whenever you try to `create` or `upgrade` a host. This will send, over HTTPS, to Bugsnag some information about your`docker-machine` version, build, OS, ARCH, the path to your current shell and, the history of the last command as you could see it with a `--debug` option. This data is sent to help us pinpoint recurring issues with`docker-machine` and will only be transmitted in the case of a crash of`docker-machine`.

If you wish to opt out of error reporting, you can create a `no-error-report` file in your `$HOME/.docker/machine` directory, and Docker Machine will disable this behavior. e.g.:

    $ mkdir -p ~/.docker/machine && touch ~/.docker/machine/no-error-report

Leaving the file empty is fine – Docker Machine just checks for its presence.

## Getting help

Docker Machine is still in its infancy and under active development. If you need help, would like to contribute, or simply want to talk about the project with like-minded individuals, we have a number of open channels for communication.

- To report bugs or file feature requests: please use the [issue tracker on Github](https://github.com/docker/machine/issues).
- To talk about the project with people in real time: please join the`#docker-machine` channel on IRC.
- To contribute code or documentation changes: please [submit a pull request on Github](https://github.com/docker/machine/pulls).

For more information and resources, please visit [our help page](https://docs.docker.com/project/get-help/).