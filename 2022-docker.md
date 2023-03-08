Docker



---

https://docs.docker.com/get-started/

---

#### Start

```
docker run -d -p 80:80 docker/getting-started
```



- `-d` - run the container in detached mode (in the background) 。。独立模式(后台运行)
- `-p 80:80` - map port 80 of the host to port 80 in the container 。。主机的80 映射到 容器的 80
- `docker/getting-started` - the image to use 。。使用的 image(预定义的容器配置)



单个字符的 flag 可以组合，来缩短命令

```
docker run -dp 80:80 docker/getting-started
```



---

#### Docker Dashboard

用来查看 运行在你的机器上的 容器。 Mac 和 Windows 有这个。

让你快速访问 容器日志，让你得到 容器中的 shell，简单地管理 容器 生命周期 (停止，移除 等)

要获得dashboard，遵循 Dockers Desktop manual。



---

#### What is a container?

简单来说，容器是 在你机器上的 沙盒的process，在主机上，和其它processor 隔离。这种隔离利用了 内核命名空间 和 cgroups，这些特性已经在linux中存在了很久。Docker致力于将这些功能变得平易近人且 易于使用。

总之，一个容器：

1. 是image的一个可运行实例。你可以 创建，启动，停止，移动，删除 一个容器
2. 能运行于 本地机器上，虚拟机，或部署到云上。
3. 是 portable (轻便的) (能在任何OS上运行)
4. 容器之间 相互隔离，运行它们自己的 软件，配置。



---

#### What is a conainter image?

当运行容器时，它使用 独立的 文件系统。这个自定义文件系统 通过 container image 来提供。

由于image 包含容器的 文件系统，所以它必须包含 运行应用 所需要的 所有的东西：所有的依赖，配置，脚本，binaries 等。image 也包含 容器的 其他配置，如环境变量，运行的默认命令，等。









---

#### Docker

https://docs.docker.com/engine/reference/commandline/docker/

| Command                                                      | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [docker attach](https://docs.docker.com/engine/reference/commandline/attach/) | Attach local standard input, output, and error streams to a running container |
| [docker build](https://docs.docker.com/engine/reference/commandline/build/) | Build an image from a Dockerfile                             |
| [docker builder](https://docs.docker.com/engine/reference/commandline/builder/) | Manage builds                                                |
| [docker checkpoint](https://docs.docker.com/engine/reference/commandline/checkpoint/) | Manage checkpoints                                           |
| [docker commit](https://docs.docker.com/engine/reference/commandline/commit/) | Create a new image from a container’s changes                |
| [docker config](https://docs.docker.com/engine/reference/commandline/config/) | Manage Docker configs                                        |
| [docker container](https://docs.docker.com/engine/reference/commandline/container/) | Manage containers                                            |
| [docker context](https://docs.docker.com/engine/reference/commandline/context/) | Manage contexts                                              |
| [docker cp](https://docs.docker.com/engine/reference/commandline/cp/) | Copy files/folders between a container and the local filesystem |
| [docker create](https://docs.docker.com/engine/reference/commandline/create/) | Create a new container                                       |
| [docker diff](https://docs.docker.com/engine/reference/commandline/diff/) | Inspect changes to files or directories on a container’s filesystem |
| [docker events](https://docs.docker.com/engine/reference/commandline/events/) | Get real time events from the server                         |
| [docker exec](https://docs.docker.com/engine/reference/commandline/exec/) | Run a command in a running container                         |
| [docker export](https://docs.docker.com/engine/reference/commandline/export/) | Export a container’s filesystem as a tar archive             |
| [docker history](https://docs.docker.com/engine/reference/commandline/history/) | Show the history of an image                                 |
| [docker image](https://docs.docker.com/engine/reference/commandline/image/) | Manage images                                                |
| [docker images](https://docs.docker.com/engine/reference/commandline/images/) | List images                                                  |
| [docker import](https://docs.docker.com/engine/reference/commandline/import/) | Import the contents from a tarball to create a filesystem image |
| [docker info](https://docs.docker.com/engine/reference/commandline/info/) | Display system-wide information                              |
| [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/) | Return low-level information on Docker objects               |
| [docker kill](https://docs.docker.com/engine/reference/commandline/kill/) | Kill one or more running containers                          |
| [docker load](https://docs.docker.com/engine/reference/commandline/load/) | Load an image from a tar archive or STDIN                    |
| [docker login](https://docs.docker.com/engine/reference/commandline/login/) | Log in to a Docker registry                                  |
| [docker logout](https://docs.docker.com/engine/reference/commandline/logout/) | Log out from a Docker registry                               |
| [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) | Fetch the logs of a container                                |
| [docker manifest](https://docs.docker.com/engine/reference/commandline/manifest/) | Manage Docker image manifests and manifest lists             |
| [docker network](https://docs.docker.com/engine/reference/commandline/network/) | Manage networks                                              |
| [docker node](https://docs.docker.com/engine/reference/commandline/node/) | Manage Swarm nodes                                           |
| [docker pause](https://docs.docker.com/engine/reference/commandline/pause/) | Pause all processes within one or more containers            |
| [docker plugin](https://docs.docker.com/engine/reference/commandline/plugin/) | Manage plugins                                               |
| [docker port](https://docs.docker.com/engine/reference/commandline/port/) | List port mappings or a specific mapping for the container   |
| [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) | List containers                                              |
| [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) | Pull an image or a repository from a registry                |
| [docker push](https://docs.docker.com/engine/reference/commandline/push/) | Push an image or a repository to a registry                  |
| [docker rename](https://docs.docker.com/engine/reference/commandline/rename/) | Rename a container                                           |
| [docker restart](https://docs.docker.com/engine/reference/commandline/restart/) | Restart one or more containers                               |
| [docker rm](https://docs.docker.com/engine/reference/commandline/rm/) | Remove one or more containers                                |
| [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/) | Remove one or more images                                    |
| [docker run](https://docs.docker.com/engine/reference/commandline/run/) | Run a command in a new container                             |
| [docker save](https://docs.docker.com/engine/reference/commandline/save/) | Save one or more images to a tar archive (streamed to STDOUT by default) |
| [docker search](https://docs.docker.com/engine/reference/commandline/search/) | Search the Docker Hub for images                             |
| [docker secret](https://docs.docker.com/engine/reference/commandline/secret/) | Manage Docker secrets                                        |
| [docker service](https://docs.docker.com/engine/reference/commandline/service/) | Manage services                                              |
| [docker stack](https://docs.docker.com/engine/reference/commandline/stack/) | Manage Docker stacks                                         |
| [docker start](https://docs.docker.com/engine/reference/commandline/start/) | Start one or more stopped containers                         |
| [docker stats](https://docs.docker.com/engine/reference/commandline/stats/) | Display a live stream of container(s) resource usage statistics |
| [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) | Stop one or more running containers                          |
| [docker swarm](https://docs.docker.com/engine/reference/commandline/swarm/) | Manage Swarm                                                 |
| [docker system](https://docs.docker.com/engine/reference/commandline/system/) | Manage Docker                                                |
| [docker tag](https://docs.docker.com/engine/reference/commandline/tag/) | Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE        |
| [docker top](https://docs.docker.com/engine/reference/commandline/top/) | Display the running processes of a container                 |
| [docker trust](https://docs.docker.com/engine/reference/commandline/trust/) | Manage trust on Docker images                                |
| [docker unpause](https://docs.docker.com/engine/reference/commandline/unpause/) | Unpause all processes within one or more containers          |
| [docker update](https://docs.docker.com/engine/reference/commandline/update/) | Update configuration of one or more containers               |
| [docker version](https://docs.docker.com/engine/reference/commandline/version/) | Show the Docker version information                          |
| [docker volume](https://docs.docker.com/engine/reference/commandline/volume/) | Manage volumes                                               |
| [docker wait](https://docs.docker.com/engine/reference/commandline/wait/) | Block until one or more containers stop, then print their exit codes |





---

https://docs.docker.com/engine/reference/commandline/version/

#### docker version

```
docker version [OPTIONS]
```

默认，会以 方便阅读的 形式 呈现 所有 version 信息。 如果指定了 format，那么会使用这个format。

Go的 text/template 包 描述了所有format的 细节。



---

Options

| Name, shorthand   | Default | Description                                       |
| ----------------- | ------- | ------------------------------------------------- |
| `--format` , `-f` |         | Format the output using the given Go template     |
| `--kubeconfig`    |         | deprecated Kubernetes<br />Kubernetes config file |



---

Examples

默认输出

```
docker version
```

获得 server version

```
docker version --format '{{.Server.Version}}'
```

显式raw JSON

```
docker version --format '{{json .}}'
```

打印当前context

```
docker version --format='{{.Client.Context}}'
```

这个输出可以用来 动态切换你的shell 来表明你当前的context。下面的例子展示了 怎么使用这个输出。

在你的 ~/.bashrc 中声明一个方法 来获得当前context，且设置这个命令作为你的 PROMPT_COMMAND

```
function docker_context_prompt() {
	PS1 = "context: %(docker version --format='{{.Client.Context}}')> "
}
PROMPT_COMMAND=docker_context_prompt
```



---

https://docs.docker.com/engine/reference/commandline/run/

。。命令都是这种的。。

#### docker run

在新的容器中 运行一条命令

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

docker run 命令首先 creates 一个可写的容器层，在指定的image之上，然后 starts 它 通过指定的命令。 即 docker run 等价于 API /containers/create 然后 /containers/(id)/start。

可以使用 docker start 来重新启动 一个已经停止的 容器，并且 之前的 修改 都存在。docker ps -a 来查看 所有容器的 列表。

docker run 命令 可以和 docker commit 结合使用 来更改容器运行的 命令

##### Options

| Name, shorthand           | Default   | Description                                                  |
| ------------------------- | --------- | ------------------------------------------------------------ |
| `--add-host`              |           | Add a custom host-to-IP mapping (host:ip)                    |
| `--attach` , `-a`         |           | Attach to STDIN, STDOUT or STDERR                            |
| `--blkio-weight`          |           | Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0) |
| `--blkio-weight-device`   |           | Block IO weight (relative device weight)                     |
| `--cap-add`               |           | Add Linux capabilities                                       |
| `--cap-drop`              |           | Drop Linux capabilities                                      |
| `--cgroup-parent`         |           | Optional parent cgroup for the container                     |
| `--cgroupns`              |           | [API 1.41+](https://docs.docker.com/engine/api/v1.41/) Cgroup namespace to use (host\|private) 'host':    Run the container in the Docker host's cgroup namespace 'private': Run the container in its own private cgroup namespace '':        Use the cgroup namespace as configured by the           default-cgroupns-mode option on the daemon (default) |
| `--cidfile`               |           | Write the container ID to the file                           |
| `--cpu-count`             |           | CPU count (Windows only)                                     |
| `--cpu-percent`           |           | CPU percent (Windows only)                                   |
| `--cpu-period`            |           | Limit CPU CFS (Completely Fair Scheduler) period             |
| `--cpu-quota`             |           | Limit CPU CFS (Completely Fair Scheduler) quota              |
| `--cpu-rt-period`         |           | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) Limit CPU real-time period in microseconds |
| `--cpu-rt-runtime`        |           | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) Limit CPU real-time runtime in microseconds |
| `--cpu-shares` , `-c`     |           | CPU shares (relative weight)                                 |
| `--cpus`                  |           | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) Number of CPUs |
| `--cpuset-cpus`           |           | CPUs in which to allow execution (0-3, 0,1)                  |
| `--cpuset-mems`           |           | MEMs in which to allow execution (0-3, 0,1)                  |
| `--detach` , `-d`         |           | Run container in background and print container ID           |
| `--detach-keys`           |           | Override the key sequence for detaching a container          |
| `--device`                |           | Add a host device to the container                           |
| `--device-cgroup-rule`    |           | Add a rule to the cgroup allowed devices list                |
| `--device-read-bps`       |           | Limit read rate (bytes per second) from a device             |
| `--device-read-iops`      |           | Limit read rate (IO per second) from a device                |
| `--device-write-bps`      |           | Limit write rate (bytes per second) to a device              |
| `--device-write-iops`     |           | Limit write rate (IO per second) to a device                 |
| `--disable-content-trust` | `true`    | Skip image verification                                      |
| `--dns`                   |           | Set custom DNS servers                                       |
| `--dns-opt`               |           | Set DNS options                                              |
| `--dns-option`            |           | Set DNS options                                              |
| `--dns-search`            |           | Set custom DNS search domains                                |
| `--domainname`            |           | Container NIS domain name                                    |
| `--entrypoint`            |           | Overwrite the default ENTRYPOINT of the image                |
| `--env` , `-e`            |           | Set environment variables                                    |
| `--env-file`              |           | Read in a file of environment variables                      |
| `--expose`                |           | Expose a port or a range of ports                            |
| `--gpus`                  |           | [API 1.40+](https://docs.docker.com/engine/api/v1.40/) GPU devices to add to the container ('all' to pass all GPUs) |
| `--group-add`             |           | Add additional groups to join                                |
| `--health-cmd`            |           | Command to run to check health                               |
| `--health-interval`       |           | Time between running the check (ms\|s\|m\|h) (default 0s)    |
| `--health-retries`        |           | Consecutive failures needed to report unhealthy              |
| `--health-start-period`   |           | [API 1.29+](https://docs.docker.com/engine/api/v1.29/) Start period for the container to initialize before starting health-retries countdown (ms\|s\|m\|h) (default 0s) |
| `--health-timeout`        |           | Maximum time to allow one check to run (ms\|s\|m\|h) (default 0s) |
| `--help`                  |           | Print usage                                                  |
| `--hostname` , `-h`       |           | Container host name                                          |
| `--init`                  |           | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) Run an init inside the container that forwards signals and reaps processes |
| `--interactive` , `-i`    |           | Keep STDIN open even if not attached                         |
| `--io-maxbandwidth`       |           | Maximum IO bandwidth limit for the system drive (Windows only) |
| `--io-maxiops`            |           | Maximum IOps limit for the system drive (Windows only)       |
| `--ip`                    |           | IPv4 address (e.g., 172.30.100.104)                          |
| `--ip6`                   |           | IPv6 address (e.g., 2001:db8::33)                            |
| `--ipc`                   |           | IPC mode to use                                              |
| `--isolation`             |           | Container isolation technology                               |
| `--kernel-memory`         |           | Kernel memory limit                                          |
| `--label` , `-l`          |           | Set meta data on a container                                 |
| `--label-file`            |           | Read in a line delimited file of labels                      |
| `--link`                  |           | Add link to another container                                |
| `--link-local-ip`         |           | Container IPv4/IPv6 link-local addresses                     |
| `--log-driver`            |           | Logging driver for the container                             |
| `--log-opt`               |           | Log driver options                                           |
| `--mac-address`           |           | Container MAC address (e.g., 92:d0:c6:0a:29:33)              |
| `--memory` , `-m`         |           | Memory limit                                                 |
| `--memory-reservation`    |           | Memory soft limit                                            |
| `--memory-swap`           |           | Swap limit equal to memory plus swap: '-1' to enable unlimited swap |
| `--memory-swappiness`     | `-1`      | Tune container memory swappiness (0 to 100)                  |
| `--mount`                 |           | Attach a filesystem mount to the container                   |
| `--name`                  |           | Assign a name to the container                               |
| `--net`                   |           | Connect a container to a network                             |
| `--net-alias`             |           | Add network-scoped alias for the container                   |
| `--network`               |           | Connect a container to a network                             |
| `--network-alias`         |           | Add network-scoped alias for the container                   |
| `--no-healthcheck`        |           | Disable any container-specified HEALTHCHECK                  |
| `--oom-kill-disable`      |           | Disable OOM Killer                                           |
| `--oom-score-adj`         |           | Tune host's OOM preferences (-1000 to 1000)                  |
| `--pid`                   |           | PID namespace to use                                         |
| `--pids-limit`            |           | Tune container pids limit (set -1 for unlimited)             |
| `--platform`              |           | [API 1.32+](https://docs.docker.com/engine/api/v1.32/) Set platform if server is multi-platform capable |
| `--privileged`            |           | Give extended privileges to this container                   |
| `--publish` , `-p`        |           | Publish a container's port(s) to the host                    |
| `--publish-all` , `-P`    |           | Publish all exposed ports to random ports                    |
| `--pull`                  | `missing` | Pull image before running ("always"\|"missing"\|"never")     |
| `--read-only`             |           | Mount the container's root filesystem as read only           |
| `--restart`               | `no`      | Restart policy to apply when a container exits               |
| `--rm`                    |           | Automatically remove the container when it exits             |
| `--runtime`               |           | Runtime to use for this container                            |
| `--security-opt`          |           | Security Options                                             |
| `--shm-size`              |           | Size of /dev/shm                                             |
| `--sig-proxy`             | `true`    | Proxy received signals to the process                        |
| `--stop-signal`           | `SIGTERM` | Signal to stop a container                                   |
| `--stop-timeout`          |           | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) Timeout (in seconds) to stop a container |
| `--storage-opt`           |           | Storage driver options for the container                     |
| `--sysctl`                |           | Sysctl options                                               |
| `--tmpfs`                 |           | Mount a tmpfs directory                                      |
| `--tty` , `-t`            |           | Allocate a pseudo-TTY                                        |
| `--ulimit`                |           | Ulimit options                                               |
| `--user` , `-u`           |           | Username or UID (format: <name\|uid>[:<group\|gid>])         |
| `--userns`                |           | User namespace to use                                        |
| `--uts`                   |           | UTS namespace to use                                         |
| `--volume` , `-v`         |           | Bind mount a volume                                          |
| `--volume-driver`         |           | Optional volume driver for the container                     |
| `--volumes-from`          |           | Mount volumes from the specified container(s)                |
| `--workdir` , `-w`        |           | Working directory inside the container                       |

##### Eamples

分配名字和 pseudo-TTY (--name, -it)

```
$ docker run --name test -it debian
root@assdf123:/# exit 13
$ echo $?
$ docker ps -a | grep test
```

运行 名字是test的，使用了debian:lastest 这个image的 容器。

`-it` 指示 Docker 分配一个 连接到 容器 标准输入的 伪TTY；在容器中创建交互式bash shell。在本例中，bash shell通过 exit 13 退出。退出代码 会传递到 docker run 的 调用者，并记录到 test 容器的 metadata中。



----

#### Capture conainer ID (--cidfile)

```
docker run --cidfile /tmp/docker_test.cid ubuntu echo "test"
```

这个会创建一个容器 且 打印test到 console。cidflag 标记 使得 docker 试图 创建一个 新文件，并且将 容器ID 写道 新文件中。如果文件已经存在，docker会返回一个错误。docker会 关闭这个文件，当docker run 退出时。



---

##### Full container capabilities (--privileged)

```
$ docker run -t -i --rm ubuntu bash
root@asdf1234:/# mount -t tmpfs none /mnt
mount: permission denied
```

这不会有作用，因为默认下，大部分潜在危险的内核功能都会被丢弃；包括 cap_sys_admin (这个是用来挂载文件系统的)，但是，--privileged 标记 将允许它执行

```
docker run -t -i --privileged ubuntu bash
```

--privileged 标记 将全部的功能 给予 容器，它还解除了 通过 `device` cgroup 控制器 实施的 所有限制。换句话说，容器几乎可以做 任何主机可以做的事情。这个标记 是为了允许  特殊的用例，例如 在 docker 中运行 docker。



---

##### Set working directory (-w)

```
docker  run -w /path/to/dir/ -i -t  ubuntu pwd
```

-w 使得 命令 在给定的目录中 执行，这里是 /path/to/dir 。如果路径不存在，会被创建。



---

##### Set storage driver options per container

```
docker run -it --storage-opt size=120G fedora /bin/bash
```

size 会允许 设置容器的 rootfs (。。root file system) 大小为 120g，在容器创建时。这个选项 仅适用于 devicemapper, btrfs, overlay2, windowsfilter, zfs 图形驱动器。对于 devicemapper, btrfs, windowsfilter, zfs 图形驱动器，用户不能 设置一个 小于 默认的 BaseFS 大小的值。 对于 overlay2 storage driver，size选项 只有当 底层fs 是 xfs 且 通过 pquota 挂载选项挂载时 才可用。满足这些条件后，用户可以传递 小于 底层fs 大小的 size。



---

##### Mount tmpfs (--tmpfs)

```
docker run -d --tmpfs /run:rw,noexec,nosuid,size=65536k my_image
```

--tmpfs 挂载 一个空的 tmpfs 到容器，并且有着 rw, noexex, nosuid, size=65535k 的选项。



---

Mount volumn (-v, --read-only)

```
docker  run  -v `pwd`:`pwd` -w `pwd` -i -t  ubuntu pwd
```

-v 挂载 当前工作目录 到 容器中。
-w 让命令 在当前工作目录 中执行，方法是 将目录更改为 pwd 返回的值。
所以这个组合 使用容器 执行命令，但是在当前工作目录中。



```
docker run -v /doesnt/exist:/foo -w /foo -i -t ubuntu bash
```

当挂载的卷 的主机目录不存在时，docker 会自动 在主机上创建 该目录。上面的例子中，docker将在 启动容器之前创建 /doesnt/exist 文件夹



```
docker run --read-only -v /icanwrite busybox touch /icanwrite/here
```

卷 可以和 --read-only 联用 来控制 容器写入文件的 位置。--read-only 将容器的 root fs 挂载为 只读，禁止 写入到 容器指定卷 以外的位置。



```
docker run -t -i -v /var/run/docker.sock:/var/run/docker.sock -v /path/to/static-docker-binary:/usr/bin/docker busybox sh
```

通过 bind-mounting docker unix socket 和 静态链接的docker 二进制文件，你可以为容器提供 创建和 操作主机的 docker 守护程序的 完全访问权限。

windows下，路径必须是windows 风格

```
PS C:\> docker run -v c:\foo:c:\dest microsoft/nanoserver cmd /s /c type c:\dest\somefile.txt
Contents of file

PS C:\> docker run -v c:\foo:d: microsoft/nanoserver cmd /s /c type d:\somefile.txt
Contents of file
```



---

##### Add bind mounts or volumes using the --mount flag

--mount 标记 允许你 挂载 volumes，主机目录，tmpfs 到容器
--mount 支持 大部分 -v 或 --volumn 支持的 选项，但 使用不同的语法。更多 --mount 的信息，及 --volume 和 --mount 之间的差别，查看：https://docs.docker.com/engine/reference/commandline/service_create/#add-bind-mounts-volumes-or-memory-filesystems

尽管没有 弃用 --volume 的计划，但是还是 推荐 --mount。

```
docker run --read-only --mount type=volume,target=/icanwrite busybox touch /icanwrite/here
```

```
docker run -t -i --mount type=bind,src=/data,dst=/data busybox sh
```



---

##### Publish or expose port (-p, --expose)

```
docker run -p 127.0.0.1:80:8080/tcp ubuntu bash
```

绑定的 容器的 8080 端口 到 主机的127.0.0.1的 80 这个TCP 端口。你还可以指定 udp，sctp 端口。

https://docs.docker.com/network/links/ 中详细说明了 如何在docker中操作ports

注意，没有绑定到 host 的 端口 (即 -p 80:80 而不是 -p 127.0.0.1:80:80 ) 会被从 外部 访问。这依然有用，即使你将 UFW 配置为 阻止此 特定接口，因为 docker 管理自己的 iptables 规则。

。。。？ 由主机转发到容器 和 直接访问容器 的区别？



```
docker run --expose 80 ubuntu bash
```

这个暴露容器的 80 端口，不会将 端口发布到主机系统的接口



---

##### Set Environment variables (-e, --env, --env-file)

```
docker run -e MYVAR1 --env MYVAR2=foo --env-file ./env.list ubuntu bash
```

使用 -e, --env, --env-file 来 设置 简单( 非数组) 环境变量 到你运行的 容器中，或覆盖 你运行的 image 的 Dockerfile 中的 变量。

你可以定义 变量和它的值 当运行容器时：

```
docker run --env VAR1=value1 --env VAR2=value2 ubuntu env | grep VAR
```

你也可以使用已经 export (。。export JAVA_HOME=/xx/x ) 到你本地环境 的变量：

```
docker run --env VAR1 --env VAR2 ubuntu env | grep VAR
```

当运行命令时，docker cli 客户端 检查 你本地环境中的变量的 值，传递它到容器。如果 没有 提供 =，且 你的本地环境中 找不到 这个变量，变量不会被设置到 容器中。

你也可以加载 环境变量 从文件中。文件应该使用 语法： `<variable>=value` (这个设置 变量为给定值)， 或 `<variable>` (这个从本地环境获得值) ， # 用来注释

```
$ cat env.list
# This is a comment
VAR1=value1
VAR2=value2
USER
$ docker run --env-file env.list ubuntu env | grep VAR
VAR1=value1
VAR2=value2
USER=denis
```



---

##### Set metadata on container (-l, --label, --label-file)

label 是一个 key=value 对，应用到 容器的 metadata。用2个label 标记容器：

```
docker run -l my-label --label com.example.foo=bar ubuntu bash
```

my-label 不指定 值，会使用默认值 空字符串 ("") ，要增加多个label，重复 label的标记 (-l , --label)

key=value 必须唯一，避免覆盖 label 值。如果你指定 多个label，具有相同的值，但 不同的 value， 后面的覆盖前面的。docker 使用 你提供的 最后一个 key=value。

使用 --label-file 来 从 文件加载多个 label。文件中 label 的分隔符是 EOL 标记。下面的例子，从当前目录的 labels 文件加载 label

```
docker run --label-file ./labels ubuntu bash
```

label文件的 格式类似于 加载环境变量文件的格式 ( 不同于环境变量，label 对于容器中的 线程 不可见)。下面是label文件 格式：

```
com.example.label1="a label"
# this is a comment
com.example.label2=another\ label
com.example.label3
```

你可以通过 多个 --label-file 来 加载多个 label文件



---

##### Connect a container to a network ( --network)

使用--network 标志 启动容器 来 连接它到网络。这增加busybox 容器 到 my-net 网络

```
docker run -itd --network=my-net busybox
```

你也可以 为 容器 选择 ip 地址，通过 --ip, --ip6 ，当你在 用户定义的网络 中启动 容器的时候。

```
docker run -itd --network=my-net --ip=10.10.9.75 busybox
```

如果你想 把 运行中的 容器增加到 网络，使用 docket network connect 命令。

你可以连接 多个容器 到 同一个网络。一旦连接，只需要 另一个容器的ip或名字 就可以 通信。对于支持多主机连接的 overlay 网络 或自定义插件，连接到 同一个多主机网络 但 从不同引擎启动的 容器也可以通过这种方式进行通信。

服务发现，在默认 bridge 网络 上 不可用。默认下，容器可以通过ip 来通信。要通过name来通信，他们必须被连接。

使用docker network disconnect 命令 来 取消 容器和 网络的 连接。



---

##### Mount volumes from container (--volumes-from)

```
docker run --volumes-from 777f7dc92da7 --volumes-from ba8c0c54f0f2:ro -i -t ubuntu pwd
```

--volumes-from 标记 从 引用的 容器 加载 所有定义的 卷。
指定多个容器，通过 重复 --volumes-from 。
容器id 可以 可选地 以 :ro , :rw 后缀结尾，来以 只读，读写模式 挂载 卷。
默认下，卷的模式 和 被引用的容器挂载卷 的模式 一致。

像SElinux 这样的 标签系统 要求在安装到 容器中的 卷内容上 放置适当的 标签。如果没有标签，安全系统可能会阻止在容器中运行的 进程 使用卷的内容。默认下，docker不会改变OS设置的标签。

要在容器上下文中 改变 label，你可以增加 :z 或 :Z 后缀。这些后缀告诉 docker 在共享的 卷上 重新label 文件对象。z选项告诉 docker： 2个容器共享 卷内容。因此，docker 使用 共享内容标签 来标记内容。共享内容标签 允许所有容器 读/写 内容。Z选项告诉docker，使用 私有非共享标签 来标记内容。只有当前容器可以使用 私有卷。



----

##### Attach to stdin/stdout/stderr (-a)

-a 告诉 docker run 去绑定 容器的 stdin,stdout,stderr。这使得在必要时 可能操作 输入输出

```
echo "test" | docker run -i -a stdin ubuntu cat -
```

这个pipe 数据到容器，并通过附加到容器的  stdin 来打印容器的id。



```
docker run -a stderr ubuntu echo test
```

这个不会打印任何东西，除非有error，因为我们只把 stderr 附加到了 容器。容器的日志 仍然会存储 应写到 stderr, stdout 中的内容。



```
cat somefile | docker run -i -a stdin mybuilder dobuild
```

这就是 如何将文件 传输到容器中 以进行构建。构建完成后，会打印容器的 id，并且 可以使用 docker logs 检索日志。如果你需要 将文件 或 其他内容 pipe 到容器中，并在容器完成运行后 检索容器的id，这将非常有用。



---

##### Add host device to container (--device)

```
docker run --device=/dev/sdc:/dev/xvdc \
             --device=/dev/sdd --device=/dev/zero:/dev/nulo \
             -i -t \
             ubuntu ls -l /dev/{xvdc,sdd,nulo}
```

经常需要直接暴露device(设备) 到容器，--device 选项允许这种行为。例如，一个特定的块存储设备 或 循环设备 或 音频设备 添加到 其他非特权容器 ( 没有 ---privilieged) 并让应用直接访问它。

默认下，容器可以 read,write,mknod 对这些设备。这可以使用 --device 的第三组 :rwm 选项来覆盖。如果容器在 privileged mode下运行，指定的权限会被忽视。

```
 docker run --device=/dev/sda:/dev/xvdc --rm -it ubuntu fdisk  /dev/xvdc
 docker run --device=/dev/sda:/dev/xvdc:r --rm -it ubuntu fdisk  /dev/xvdc
 docker run --device=/dev/sda:/dev/xvdc:rw --rm -it ubuntu fdisk  /dev/xvdc
 docker run --device=/dev/sda:/dev/xvdc:m --rm -it ubuntu fdisk  /dev/xvdc
```



--device 不能安全地 和 临时设备一起使用。不应使用 --device 将可能被删除的 block device 添加到不受信任的 容器中。

对于windows， --device选项传递的 字符串的 格式是 `--device=<IdType>/<Id>`。从windows server 2019 和 windows 10 october 2018 update 开始，windows 只支持 class 的IdType 且 id是一个 device interface class GUID。有关容器支持的设备接口类GUID的列表，查看windows 容器文档中定义的表 https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/hardware-devices-in-containers

如果为进程隔离的windows容器指定此选项，则所有实现请求的设备接口类GUID的设备 在容器中都可用。例如，下面的命令使得主机的所有 COM端口在容器中可见。

```
PS C:\> docker run --device=class/86E0D1E0-8089-11D0-9CE4-08003E301F73 mcr.microsoft.com/windows/servercore:ltsc2019
```

--device只支持 进程隔离的 windows容器，如果容器隔离是 hyperv 或 在windows上执行linux容器时 失败。



---

Access an NVIDIA GPU

--gpus 允许你访问 NVIDIA GPU 资源。首先，你需要安装 nvidia-container-runtime。查看 specify a container's resources 获得更多信息。

使用 --gpus 指定 哪些(or 全部) GPU 可用。如果没有提供值，会使用所有可用的 GPU。

下面是暴露所有可用GPU

```
docker run -it --rm --gpus all ubuntu nvidia-smi
```

使用device 选项来指定 GPU：

```
docker run -it --rm --gpus device=GPU-3a23c669-1f69-c64e-cf85-44e9b07e7a2a ubuntu nvidia-smi
```

暴露第一个，第三个GPU

```
docker run -it --rm --gpus device=0,2 nvidia-smi
```



---

##### Restart policies (--restart)

使用docker的 --restart 来 指定容器的 重启策略。重启策略 控制 docker守护进程 是否在退出后重启容器。

Docker支持以下策略：

| Policy                     | Result                                                       |
| :------------------------- | :----------------------------------------------------------- |
| `no`                       | Do not automatically restart the container when it exits. This is the default. 。。容器退出时，不会自动重启，这是默认的 |
| `on-failure[:max-retries]` | Restart only if the container exits  with a non-zero exit status. Optionally, limit the number of restart  retries the Docker daemon attempts. 。。当容器退出时的退出状态非0时重启，括号内是可选的，限制 docker守护线程 尝试重启的 次数。 |
| `unless-stopped`           | Restart the container unless it is explicitly stopped or Docker itself is stopped or restarted.。。重启容器 除非 它被显式停止，或docker它自己是停止的 或 被重启的。 |
| `always`                   | Always restart the container  regardless of the exit status. When you specify always, the Docker  daemon will try to restart the container indefinitely. The container  will also always start on daemon startup, regardless of the current  state of the container.。。总是重启容器，不管退出状态。当你指定 always，docker守护线程 会 一直尝试重启容器。守护线程启动时，容器也会被启动，不管容器的当前状态。 |

```
docker run --restart=always redis
```

这个会运行 redis 容器，并且重启策略是 always，这样，如果容器退出，docker会立刻重启它。

更多关于重启策略的信息：https://docs.docker.com/engine/reference/run/#restart-policies---restart



---

##### Add entries to container hosts file (--add-host)

你可以增加其他 host 到容器的 /etc/hosts 文件，通过 一个或多个 --add-host 标记。
下面的例子，为 docker这个host增加了一个 ip地址的 映射

```
$ docker run --add-host=docker:93.184.216.34 --rm -it alpine
/ #ping docker
```

有时，你需要连接 docker host from你的容器。为了允许这种行为，使用 --add-host 传递 docker host的 ip地址 到容器。要找到 host的地址，使用 ip addr show 命令。



你传递给ip addr show 的标记取决于 你在容器中 使用的是 ipv4 还是 ipv6 网络。对于ipv4使用下面的标记 来检索 名为 eth0 的网络设备的 ipv4地址。

```
$ HOSTIP=`ip -4 addr show scope global dev eth0 | grep inet | awk '{print $2}' | cut -d / -f 1 | sed -n 1p`
$ docker run  --add-host=docker:${HOSTIP} --rm -it debian
```

对于 ipv6 使用 -6 标记 而不是 -4。 对于其他网络设备，将 eth0替换为 正确的 设备名字。



---

##### Set ulimits in container (--ulimit)

由于设置 ulimit 配置 到容器中，需要 默认容器中不可用的 额外权限，因此你可以使用 --ulimit 标志设置这些配置。--ulimit 指定了soft ，hard limit：`<type>=<soft limit>[:<hard limit>]`

```
docker run --ulimit nofile=1024:1024 --rm debian sh -c "ulimit -n"
```

如果你不提供 hard limit，那么会使用 soft limit 作为 hard limit。如果没有设置ulimit，它们会继承守护线程 上设置的默认 ulimit 设置。as 选项现在 被禁用。换句话说，不支持下面的脚本：

```
docker run -it --ulimit as=1024 fedora /bin/bash`
```

这些值在被设置时 ，被发送到 适当的 syscall。docker不执行 任何的 字节转换。设置值时要考虑这一点。

对于 `nproc` 用法

小心 通过ulimit 来设置 nproc， 因为 nproc 是由 linux 设计的，用于设置 用户可用的 最大进程数，而不是容器。例如，使用 daemon 用户启动4个容器：

```
 docker run -d -u daemon --ulimit nproc=3 busybox top
 docker run -d -u daemon --ulimit nproc=3 busybox top
 docker run -d -u daemon --ulimit nproc=3 busybox top
 docker run -d -u daemon --ulimit nproc=3 busybox top
```

第四个容器失败，报告“[8] System error: resource temporarily unavailable”错误。这个失败是因为 调用者设置了 nproc=3，导致前3个容器用完了 为 daemon 用户设置的 3个进程配额。



---

##### Stop container with signal (--stop-signal)

--stop-signal 标记 设置 将发送给容器 以退出的 系统调用信号。这个信号可以是一个`SIG<NAME>`的信号名字，比如 SIGKILL，或者一个 匹配 kernel 的 syscall table 的无符号数字，比如 9。

默认 SIGTERM



---

##### Optional security options (--security-opt)

windows上，这个标记用来指定 credentialspec 选项。credentialspec的格式必须是 `file://spec.txt` 或 `registry://keyname`。



---

##### Stop container with timeout (--stop-timeout)

--stop-timeout 标记设置 在 放发送预定义的 (--stop-signal) system call 信号后，等待 容器 停止的 秒数。如果在timeout 时间内 没有exit，那么会被 SIGKILL 信号 强制kill。

如果 --stop-timeout 设置为 -1，不会应用 timeout，daemon会无限期等待 容器退出。

默认值 有daemon 决定，linux容器10s，windows容器30s。



---

##### Specify isolation technology for container (--isolation)

这个选项 当你在windows 上运行docker 容器时 很有用。 `--isolation <value>` 选项设置 容器的隔离技术。在linux上，只支持 使用 linux 命名空间的 default 选项，下面 2个命令 在linux 上 是等价的

```
 docker run -d busybox top
 docker run -d --isolation default busybox top
```



windows上，--isolation 可以选择 以下之一：

| Value     | Description                                                  |
| :-------- | :----------------------------------------------------------- |
| `default` | Use the value specified by the Docker daemon’s `--exec-opt` or system default (see below). |
| `process` | Shared-kernel namespace isolation (not supported on Windows client operating systems older than Windows 10 1809). |
| `hyperv`  | Hyper-V hypervisor partition-based isolation.                |

windows server OS 上默认隔离是 process， windows client OS上默认 hyperv， 尝试 通过 --isolation process  在 早于 windows 10 1809 的版本的 client OS 上 启动容器，会失败。

在windows server上，考虑默认配置后， 下面的是等价的，都是设置 process 隔离

```
PS C:\> docker run -d microsoft/nanoserver powershell echo process
PS C:\> docker run -d --isolation default microsoft/nanoserver powershell echo process
PS C:\> docker run -d --isolation process microsoft/nanoserver powershell echo process
```

如果你设置 --exec-opt isolation=hyperv 选项 到 docker daemon，或者运行基于 windows client的 daemon，下面的是等价的，都是设置 hyperv 隔离：

```
PS C:\> docker run -d microsoft/nanoserver powershell echo hyperv
PS C:\> docker run -d --isolation default microsoft/nanoserver powershell echo hyperv
PS C:\> docker run -d --isolation hyperv microsoft/nanoserver powershell echo hyperv
```



---

##### Spcify hard limits on memory available to containers (-m, --memory)

这些参数用来设置 容器可用内存的 上限。linux中，这些被设置到 cgroup，容器中的 app 可以在 /sys/fs/cgroup/memory/memory.limit_in_bytes 中查询到。

windows，这些根据 isolation 的不同，而产生不同的影响。

在process 隔离下，windows会报告 主机系统的 完整内存，而不是容器中app的限制

```
  PS C:\> docker run -it -m 2GB --isolation=process microsoft/nanoserver powershell Get-ComputerInfo *memory*

  CsTotalPhysicalMemory      : 17064509440
  CsPhyicallyInstalledMemory : 16777216
  OsTotalVisibleMemorySize   : 16664560
  OsFreePhysicalMemory       : 14646720
  OsTotalVirtualMemorySize   : 19154928
  OsFreeVirtualMemory        : 17197440
  OsInUseVirtualMemory       : 1957488
  OsMaxProcessMemorySize     : 137438953344
```



在hyperv 隔离下，windows 会创建一个 足以容纳内存限制的 utility VM，以及托管容器所需的最小OS。这个size 被报告为 "Total Physical Memory"。

```
  PS C:\> docker run -it -m 2GB --isolation=hyperv microsoft/nanoserver powershell Get-ComputerInfo *memory*

  CsTotalPhysicalMemory      : 2683355136
  CsPhyicallyInstalledMemory :
  OsTotalVisibleMemorySize   : 2620464
  OsFreePhysicalMemory       : 2306552
  OsTotalVirtualMemorySize   : 2620464
  OsFreeVirtualMemory        : 2356692
  OsInUseVirtualMemory       : 263772
  OsMaxProcessMemorySize     : 137438953344
```



---

##### Configure namespaced kernel parameters (sysctls) at runtime

--sysctl 设置容器的 namespaced kernel parameters (sysctls)。例如，要在容器 网络 命名空间中打开 IP转发，请执行下面命令

```
docker run --sysctl net.ipv4.ip_forward=1 someimage
```

不是所有的sysctl 都是 命名空间的。docker 不支持更改容器的 sysctl 也会修改主机系统。



---

##### Currently sypported sysctls

IPC 命名空间：

1. `kernel.msgmax`, `kernel.msgmnb`, `kernel.msgmni`, `kernel.sem`, `kernel.shmall`, `kernel.shmmax`, `kernel.shmmni`, `kernel.shm_rmid_forced`.
2. Sysctls beginning with `fs.mqueue.*`
3. If you use the `--ipc=host` option these sysctls are not allowed.

Network 命名空间

1. Sysctls beginning with `net.*`
2. If you use the `--network=host` option using these sysctls are not allowed.





---

docker image

https://docs.docker.com/engine/reference/commandline/image/

```
 docker image COMMAND
```

| Command                                                      | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [docker image build](https://docs.docker.com/engine/reference/commandline/image_build/) | Build an image from a Dockerfile                             |
| [docker image history](https://docs.docker.com/engine/reference/commandline/image_history/) | Show the history of an image                                 |
| [docker image import](https://docs.docker.com/engine/reference/commandline/image_import/) | Import the contents from a tarball to create a filesystem image |
| [docker image inspect](https://docs.docker.com/engine/reference/commandline/image_inspect/) | Display detailed information on one or more images           |
| [docker image load](https://docs.docker.com/engine/reference/commandline/image_load/) | Load an image from a tar archive or STDIN                    |
| [docker image ls](https://docs.docker.com/engine/reference/commandline/image_ls/) | List images                                                  |
| [docker image prune](https://docs.docker.com/engine/reference/commandline/image_prune/) | Remove unused images                                         |
| [docker image pull](https://docs.docker.com/engine/reference/commandline/image_pull/) | Pull an image or a repository from a registry                |
| [docker image push](https://docs.docker.com/engine/reference/commandline/image_push/) | Push an image or a repository to a registry                  |
| [docker image rm](https://docs.docker.com/engine/reference/commandline/image_rm/) | Remove one or more images                                    |
| [docker image save](https://docs.docker.com/engine/reference/commandline/image_save/) | Save one or more images to a tar archive (streamed to STDOUT by default) |
| [docker image tag](https://docs.docker.com/engine/reference/commandline/image_tag/) | Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE        |



---

docker images

https://docs.docker.com/engine/reference/commandline/images/

```
 docker images [OPTIONS] [REPOSITORY[:TAG]]
```

| Name, shorthand   | Default | Description                                         |
| ----------------- | ------- | --------------------------------------------------- |
| `--all` , `-a`    |         | Show all images (default hides intermediate images) |
| `--digests`       |         | Show digests                                        |
| `--filter` , `-f` |         | Filter output based on conditions provided          |
| `--format`        |         | Pretty-print images using a Go template             |
| `--no-trunc`      |         | Don't truncate output                               |
| `--quiet` , `-q`  |         | Only show image IDs                                 |

```
docker images
```

```
docker images java
```

```
docker images java:8
```

```
docker images java:0
```

```
docker images --no-trunc
```

```
docker images --digests
```

```
docker images --filter "dangling=true"
```

```
docker rmi $(docker images -f "dangling=true" -q)
```

```
docker images --filter "label=com.example.version"
```

```
docker images --filter "label=com.example.version=1.0"
```

```
docker images --filter "label=com.example.version=0.1"
```

```
docker images --filter "before=image1"
```

```
docker images --filter "since=image3"
```

```
docker images --filter=reference='busy*:*libc'
```

```
docker images --filter=reference='busy*:uclibc' --filter=reference='busy*:glibc'
```

Format the output

| Placeholder     | Description                              |
| --------------- | ---------------------------------------- |
| `.ID`           | Image ID                                 |
| `.Repository`   | Image repository                         |
| `.Tag`          | Image tag                                |
| `.Digest`       | Image digest                             |
| `.CreatedSince` | Elapsed time since the image was created |
| `.CreatedAt`    | Time when the image was created          |
| `.Size`         | Image disk size                          |

```
docker images --format "{{.ID}}: {{.Repository}}"
```

```
docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```



---

##### docker system

https://docs.docker.com/engine/reference/commandline/system/

```
docker system COMMAND
```

| Command                                                      | Description                          |
| ------------------------------------------------------------ | ------------------------------------ |
| [docker system df](https://docs.docker.com/engine/reference/commandline/system_df/) | Show docker disk usage               |
| [docker system events](https://docs.docker.com/engine/reference/commandline/system_events/) | Get real time events from the server |
| [docker system info](https://docs.docker.com/engine/reference/commandline/system_info/) | Display system-wide information      |
| [docker system prune](https://docs.docker.com/engine/reference/commandline/system_prune/) | Remove unused data                   |



---

##### docker top

```
docker top CONTAINER [ps OPTIONS]
```



---

##### docker pull

https://docs.docker.com/engine/reference/commandline/pull/

```
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

| Name, shorthand           | Default | Description                                                  |
| ------------------------- | ------- | ------------------------------------------------------------ |
| `--all-tags` , `-a`       |         | Download all tagged images in the repository                 |
| `--disable-content-trust` | `true`  | Skip image verification                                      |
| `--platform`              |         | [API 1.32+](https://docs.docker.com/engine/api/v1.32/) Set platform if server is multi-platform capable |
| `--quiet` , `-q`          |         | Suppress verbose output                                      |

```
docker pull debian
```

```
docker pull debian:jessie
```

```
docker images
```

```
docker pull ubuntu:20.04
```

```
docker pull ubuntu@sha256:82becede498899ec668628e7cb0ad87b6e1c371cb8a1e597d83a47fac21d6af3
```

```
docker pull myregistry.local:5000/testing/test-image
```

```
docker pull --all-tags fedora
```

```
docker images fedora
```

```
docker pull fedora
```



---

docker save

https://docs.docker.com/engine/reference/commandline/save/

```
docker save [OPTIONS] IMAGE [IMAGE...]
```

| Name, shorthand   | Default | Description                        |
| ----------------- | ------- | ---------------------------------- |
| `--output` , `-o` |         | Write to a file, instead of STDOUT |

```
docker save busybox > busybox.tar
```

```
docker save --output busybox.tar busybox
```

```
docker save -o fedora-all.tar fedora
```

```
docker save -o fedora-latest.tar fedora:latest
```

```
docker save myimage:latest | gzip > myimage_latest.tar.gz
```

```
docker save -o ubuntu.tar ubuntu:lucid ubuntu:saucy
```



---

docker start

https://docs.docker.com/engine/reference/commandline/start/

```
docker start [OPTIONS] CONTAINER [CONTAINER...]
```

| Name, shorthand        | Default | Description                                                  |
| ---------------------- | ------- | ------------------------------------------------------------ |
| `--attach` , `-a`      |         | Attach STDOUT/STDERR and forward signals                     |
| `--checkpoint`         |         | [experimental (daemon)](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file) Restore from this checkpoint |
| `--checkpoint-dir`     |         | [experimental (daemon)](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file) Use a custom checkpoint storage directory |
| `--detach-keys`        |         | Override the key sequence for detaching a container          |
| `--interactive` , `-i` |         | Attach container's STDIN                                     |

```
docker start my_container
```



---

##### docker build

Build an image from a Dockerfile

```
docker build [OPTIONS] PATH | URL | -
```

| Build Syntax Suffix            | Commit Used           | Build Context Used |
| ------------------------------ | --------------------- | ------------------ |
| `myrepo.git`                   | `refs/heads/master`   | `/`                |
| `myrepo.git#mytag`             | `refs/tags/mytag`     | `/`                |
| `myrepo.git#mybranch`          | `refs/heads/mybranch` | `/`                |
| `myrepo.git#pull/42/head`      | `refs/pull/42/head`   | `/`                |
| `myrepo.git#:myfolder`         | `refs/heads/master`   | `/myfolder`        |
| `myrepo.git#master:myfolder`   | `refs/heads/master`   | `/myfolder`        |
| `myrepo.git#mytag:myfolder`    | `refs/tags/mytag`     | `/myfolder`        |
| `myrepo.git#mybranch:myfolder` | `refs/heads/mybranch` | `/myfolder`        |

```
docker build http://server/context.tar.gz
```

```
docker build - < Dockerfile
```

| Name, shorthand           | Default | Description                                                  |
| ------------------------- | ------- | ------------------------------------------------------------ |
| `--add-host`              |         | Add a custom host-to-IP mapping (host:ip)                    |
| `--build-arg`             |         | Set build-time variables                                     |
| `--cache-from`            |         | Images to consider as cache sources                          |
| `--cgroup-parent`         |         | Optional parent cgroup for the container                     |
| `--compress`              |         | Compress the build context using gzip                        |
| `--cpu-period`            |         | Limit the CPU CFS (Completely Fair Scheduler) period         |
| `--cpu-quota`             |         | Limit the CPU CFS (Completely Fair Scheduler) quota          |
| `--cpu-shares` , `-c`     |         | CPU shares (relative weight)                                 |
| `--cpuset-cpus`           |         | CPUs in which to allow execution (0-3, 0,1)                  |
| `--cpuset-mems`           |         | MEMs in which to allow execution (0-3, 0,1)                  |
| `--disable-content-trust` | `true`  | Skip image verification                                      |
| `--file` , `-f`           |         | Name of the Dockerfile (Default is 'PATH/Dockerfile')        |
| `--force-rm`              |         | Always remove intermediate containers                        |
| `--iidfile`               |         | Write the image ID to the file                               |
| `--isolation`             |         | Container isolation technology                               |
| `--label`                 |         | Set metadata for an image                                    |
| `--memory` , `-m`         |         | Memory limit                                                 |
| `--memory-swap`           |         | Swap limit equal to memory plus swap: '-1' to enable unlimited swap |
| `--network`               |         | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) Set the networking mode for the RUN instructions during build |
| `--no-cache`              |         | Do not use cache when building the image                     |
| `--output` , `-o`         |         | [API 1.40+](https://docs.docker.com/engine/api/v1.40/) Output destination (format: type=local,dest=path) |
| `--platform`              |         | [API 1.38+](https://docs.docker.com/engine/api/v1.38/) Set platform if server is multi-platform capable |
| `--progress`              | `auto`  | Set type of progress output (auto, plain, tty). Use plain to show container output |
| `--pull`                  |         | Always attempt to pull a newer version of the image          |
| `--quiet` , `-q`          |         | Suppress the build output and print image ID on success      |
| `--rm`                    | `true`  | Remove intermediate containers after a successful build      |
| `--secret`                |         | [API 1.39+](https://docs.docker.com/engine/api/v1.39/) Secret file to expose to the build (only if BuildKit enabled): id=mysecret,src=/local/secret |
| `--security-opt`          |         | Security options                                             |
| `--shm-size`              |         | Size of /dev/shm                                             |
| `--squash`                |         | [experimental (daemon)](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)[API 1.25+](https://docs.docker.com/engine/api/v1.25/) Squash newly built layers into a single new layer |
| `--ssh`                   |         | [API 1.39+](https://docs.docker.com/engine/api/v1.39/) SSH agent socket or keys to expose to the build (only if BuildKit enabled)  (format: default\|<id>[=<socket>\|<key>[,<key>]]) |
| `--stream`                |         | Stream attaches to server to negotiate build context         |
| `--tag` , `-t`            |         | Name and optionally a tag in the 'name:tag' format           |
| `--target`                |         | Set the target build stage to build.                         |
| `--ulimit`                |         | Ulimit options                                               |

```
docker build github.com/creack/docker-firefox
```

```
docker build -f ctx/Dockerfile http://server/ctx.tar.gz
```

```
docker build - < Dockerfile
```

```
docker build - < context.tar.gz
```

```
docker build -t vieux/apache:2.0 .
```

```
docker build -t whenry/fedora-jboss:latest -t whenry/fedora-jboss:v2.1 .
```

```
docker build -f Dockerfile.debug .
```

```
docker build -f dockerfiles/Dockerfile.debug -t myapp_debug .
```

```
docker build -f ../../../../dockerfiles/debug /home/me/myapp
```

```
docker build -f /home/me/myapp/dockerfiles/debug /home/me/myapp
```

。。还有。。



---

docker commit

Create a new image from a container’s changes

https://docs.docker.com/engine/reference/commandline/commit/

```
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

| Name, shorthand    | Default | Description                                                |
| ------------------ | ------- | ---------------------------------------------------------- |
| `--author` , `-a`  |         | Author (e.g., "John Hannibal Smith <hannibal@a-team.com>") |
| `--change` , `-c`  |         | Apply Dockerfile instruction to the created image          |
| `--message` , `-m` |         | Commit message                                             |
| `--pause` , `-p`   | `true`  | Pause container during commit                              |

```
 docker ps
 docker commit c3f279d17e0a  svendowideit/testimage:version3
 docker images
```



---

##### docker stop

Stop one or more running containers

https://docs.docker.com/engine/reference/commandline/stop/

```
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

| Name, shorthand | Default | Description                                |
| --------------- | ------- | ------------------------------------------ |
| `--time` , `-t` | `10`    | Seconds to wait for stop before killing it |

```
docker stop my_container
```



---

docker kill

https://docs.docker.com/engine/reference/commandline/kill/

```
docker kill [OPTIONS] CONTAINER [CONTAINER...]
```

| Name, shorthand   | Default | Description                     |
| ----------------- | ------- | ------------------------------- |
| `--signal` , `-s` | `KILL`  | Signal to send to the container |

```
docker kill my_container
```

```
docker kill --signal=SIGHUP  my_container
```

```
 docker kill --signal=SIGHUP my_container
 docker kill --signal=HUP my_container
 docker kill --signal=1 my_container
```



---

##### docker exec

https://docs.docker.com/engine/reference/commandline/exec/

```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

| Name, shorthand        | Default | Description                                                  |
| ---------------------- | ------- | ------------------------------------------------------------ |
| `--detach` , `-d`      |         | Detached mode: run command in the background                 |
| `--detach-keys`        |         | Override the key sequence for detaching a container          |
| `--env` , `-e`         |         | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) Set environment variables |
| `--env-file`           |         | [API 1.25+](https://docs.docker.com/engine/api/v1.25/) Read in a file of environment variables |
| `--interactive` , `-i` |         | Keep STDIN open even if not attached                         |
| `--privileged`         |         | Give extended privileges to the command                      |
| `--tty` , `-t`         |         | Allocate a pseudo-TTY                                        |
| `--user` , `-u`        |         | Username or UID (format: <name\|uid>[:<group\|gid>])         |
| `--workdir` , `-w`     |         | [API 1.35+](https://docs.docker.com/engine/api/v1.35/) Working directory inside the container |

```
docker run --name ubuntu_bash --rm -i -t ubuntu bash
```

```
docker exec -d ubuntu_bash touch /tmp/execWorks
```

```
docker exec -it ubuntu_bash bash
```

```
docker exec -it -e VAR=1 ubuntu_bash bash
```

```
docker exec -it ubuntu_bash pwd
```

```
docker exec -it -w /root ubuntu_bash pwd
```



---

##### docker create

Create a new container

```
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```



https://docs.docker.com/engine/reference/commandline/create/

这个参数太多了。。。



---

##### docker cp

Copy files/folders between a container and the local filesystem

https://docs.docker.com/engine/reference/commandline/cp/



---

docker container

https://docs.docker.com/engine/reference/commandline/container/

```
docker container COMMAND
```

| Command                                                      | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [docker container attach](https://docs.docker.com/engine/reference/commandline/container_attach/) | Attach local standard input, output, and error streams to a running container |
| [docker container commit](https://docs.docker.com/engine/reference/commandline/container_commit/) | Create a new image from a container’s changes                |
| [docker container cp](https://docs.docker.com/engine/reference/commandline/container_cp/) | Copy files/folders between a container and the local filesystem |
| [docker container create](https://docs.docker.com/engine/reference/commandline/container_create/) | Create a new container                                       |
| [docker container diff](https://docs.docker.com/engine/reference/commandline/container_diff/) | Inspect changes to files or directories on a container’s filesystem |
| [docker container exec](https://docs.docker.com/engine/reference/commandline/container_exec/) | Run a command in a running container                         |
| [docker container export](https://docs.docker.com/engine/reference/commandline/container_export/) | Export a container’s filesystem as a tar archive             |
| [docker container inspect](https://docs.docker.com/engine/reference/commandline/container_inspect/) | Display detailed information on one or more containers       |
| [docker container kill](https://docs.docker.com/engine/reference/commandline/container_kill/) | Kill one or more running containers                          |
| [docker container logs](https://docs.docker.com/engine/reference/commandline/container_logs/) | Fetch the logs of a container                                |
| [docker container ls](https://docs.docker.com/engine/reference/commandline/container_ls/) | List containers                                              |
| [docker container pause](https://docs.docker.com/engine/reference/commandline/container_pause/) | Pause all processes within one or more containers            |
| [docker container port](https://docs.docker.com/engine/reference/commandline/container_port/) | List port mappings or a specific mapping for the container   |
| [docker container prune](https://docs.docker.com/engine/reference/commandline/container_prune/) | Remove all stopped containers                                |
| [docker container rename](https://docs.docker.com/engine/reference/commandline/container_rename/) | Rename a container                                           |
| [docker container restart](https://docs.docker.com/engine/reference/commandline/container_restart/) | Restart one or more containers                               |
| [docker container rm](https://docs.docker.com/engine/reference/commandline/container_rm/) | Remove one or more containers                                |
| [docker container run](https://docs.docker.com/engine/reference/commandline/container_run/) | Run a command in a new container                             |
| [docker container start](https://docs.docker.com/engine/reference/commandline/container_start/) | Start one or more stopped containers                         |
| [docker container stats](https://docs.docker.com/engine/reference/commandline/container_stats/) | Display a live stream of container(s) resource usage statistics |
| [docker container stop](https://docs.docker.com/engine/reference/commandline/container_stop/) | Stop one or more running containers                          |
| [docker container top](https://docs.docker.com/engine/reference/commandline/container_top/) | Display the running processes of a container                 |
| [docker container unpause](https://docs.docker.com/engine/reference/commandline/container_unpause/) | Unpause all processes within one or more containers          |
| [docker container update](https://docs.docker.com/engine/reference/commandline/container_update/) | Update configuration of one or more containers               |
| [docker container wait](https://docs.docker.com/engine/reference/commandline/container_wait/) | Block until one or more containers stop, then print their exit codes |





---



















































