## Ubuntu、Debian 系列安装 Docker

### 系统要求

Docker 支持以下版本的 [Ubuntu](https://www.ubuntu.com/server) 和 [Debian](https://www.debian.org/intro/about) 操作系统：

* Ubuntu Xenial 16.04 (LTS)
* Ubuntu Trusty 14.04 (LTS)
* Ubuntu Precise 12.04 (LTS)
* Debian testing stretch (64-bit)
* Debian 8 Jessie (64-bit)
* Debian 7 Wheezy (64-bit)（*必须启用 backports*)

Ubuntu 发行版中，LTS（Long-Term-Support）长期支持版本，会获得 5 年的升级维护支持，这样的版本会更稳定，因此在生产环境中推荐使用 LTS 版本。

Docker 目前支持的 Ubuntu 版本最低为 12.04 LTS，但从稳定性上考虑，推荐使用 14.04 LTS 或更高的版本。

Docker 需要安装在 64 位的 x86 平台或 ARM 平台上（如[树莓派](https://www.raspberrypi.org/)），并且要求内核版本不低于 3.10。但实际上内核越新越好，过低的内核版本可能会出现部分功能无法使用，或者不稳定。

用户可以通过如下命令检查自己的内核版本详细信息：

```bash
$ uname -a
Linux device 4.4.0-45-generic #66~14.04.1-Ubuntu SMP Wed Oct 19 15:05:38 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```

#### 升级内核

如果内核版本过低，可以用下面的命令升级系统内核。

##### Ubuntu 12.04 LTS

```bash
sudo apt-get install -y --install-recommends linux-generic-lts-trusty
```

##### Ubuntu 14.04 LTS

```bash
sudo apt-get install -y --install-recommends linux-generic-lts-xenial
```

##### Debian 7 Wheezy

Debian 7 的内核默认为 3.2，为了满足 Docker 的需求，应该安装 `backports` 的内核。

执行下面的命令添加 `backports` 源：

```bash
$ echo "deb http://http.debian.net/debian wheezy-backports main" | sudo tee /etc/apt/sources.list.d/backports.list
```

升级到 `backports` 内核：

```bash
$ sudo apt-get update
$ sudo apt-get -t wheezy-backports install linux-image-amd64
```

##### Debian 8 Jessie

Debian 8 的内核默认为 3.16，满足基本的 Docker 运行条件。但是如果打算使用 `overlay2` 存储层驱动，或某些功能不够稳定希望升级到较新版本的内核，可以添加 `backports` 源，升级到新版本的内核。

执行下面的命令添加 `backports` 源：

```bash
$ echo "deb http://http.debian.net/debian jessie-backports main" | sudo tee /etc/apt/sources.list.d/backports.list
```

升级到 `backports` 内核：

```bash
$ sudo apt-get update
$ sudo apt-get -t jessie-backports install linux-image-amd64
```

需要注意的是，升级到 `backports` 的内核之后，会因为 `AUFS` 内核模块不可用，而使用默认的 `devicemapper` 驱动，并且配置为 `loop-lvm`，这是不推荐的。因此，不要忘记安装 Docker 后，配置 `overlay2` 存储层驱动。

##### 配置 GRUB 引导参数

在 Docker 使用期间，或者在 `docker info` 信息中，可能会看到下面的警告信息：

```bash
WARNING: Your kernel does not support cgroup swap limit. WARNING: Your
kernel does not support swap limit capabilities. Limitation discarded.
```

或者

```bash
WARNING: No memory limit support
WARNING: No swap limit support
WARNING: No oom kill disable support
```

如果需要这些功能，就需要修改 GRUB 的配置文件 ` /etc/default/grub`，在 `GRUB_CMDLINE_LINUX` 中添加内核引导参数 `cgroup_enable=memory swapaccount=1`。

然后不要忘记了更新 GRUB：

```bash
$ sudo update-grub
$ sudo reboot
```

### 使用脚本自动安装

Docker 官方为了简化安装流程，提供了一套安装脚本，Ubuntu 和 Debian 系统可以使用这套脚本安装：

```bash
curl -sSL https://get.docker.com/ | sh
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker 安装在系统中。

不过，由于伟大的墙的原因，在国内使用这个脚本可能会出现某些下载出现错误的情况。国内的一些云服务商提供了这个脚本的修改版本，使其使用国内的 Docker 软件源镜像安装，这样就避免了墙的干扰。

#### 阿里云的安装脚本

```bash
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
```

#### DaoCloud 的安装脚本

```bash
curl -sSL https://get.daocloud.io/docker | sh
```

### 手动安装

#### 安装所需的软件包

##### 可选内核模块

从 Ubuntu 14.04 开始，一部分内核模块移到了可选内核模块包(`linux-image-extra-*`)，以减少内核软件包的体积。正常安装的系统应该会包含可选内核模块包，而一些裁剪后的系统可能会将其精简掉。`AUFS` 内核驱动属于可选内核模块的一部分，作为推荐的 Docker 存储层驱动，一般建议安装可选内核模块包以使用 `AUFS`。

如果系统没有安装可选内核模块的话，可以执行下面的命令来安装可选内核模块包：

```bash
$ sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual
```

##### 12.04 LTS 图形界面

在 Ubuntu 12.04 桌面环境下，需要一些额外的软件包，可以用下面的命令安装。

```bash
$ sudo apt-get install xserver-xorg-lts-trusty libgl1-mesa-glx-lts-trusty
```

#### 添加 APT 镜像源

虽然 Ubuntu 系统软件源中有 Docker，名为 `docker.io`，但是不应该使用系统源中的这个版本，它的版本太旧。我们需要使用 Docker 官方提供的软件源，因此，我们需要添加 APT 软件源。

由于官方源使用 HTTPS 以确保软件下载过程中不被篡改。因此，我们首先需要添加使用 HTTPS 传输的软件包以及 CA 证书。

*国内的一些软件源镜像（比如[阿里云](http://mirrors.aliyun.com/docker-engine/)）不是太在意系统安全上的细节，可能依旧使用不安全的 HTTP，对于这些源可以不执行这一步。*

```bash
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates
```

为了确认所下载软件包的合法性，需要添加 Docker 官方软件源的 GPG 密钥。

```bash
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```

然后，我们需要向 `source.list` 中添加 Docker 软件源，下表列出了不同的 Ubuntu 和 Debian 版本对应的 APT 源。

|     操作系统版本      |                            REPO                              |
|---------------------|--------------------------------------------------------------|
| Precise 12.04 (LTS) | `deb https://apt.dockerproject.org/repo ubuntu-precise main` |
| Trusty 14.04 (LTS)  | `deb https://apt.dockerproject.org/repo ubuntu-trusty main`  |
| Xenial 16.04 (LTS)  | `deb https://apt.dockerproject.org/repo ubuntu-xenial main`  |
|   Debian 7 Wheezy   | `deb https://apt.dockerproject.org/repo debian-wheezy main`  |
|   Debian 8 Jessie   | `deb https://apt.dockerproject.org/repo debian-jessie main`  |
| Debian Stretch/Sid  | `deb https://apt.dockerproject.org/repo debian-stretch main` |

用下面的命令将 APT 源添加到 `source.list`（将其中的 `<REPO>` 替换为上表的值）：

```bash
$ echo "<REPO>" | sudo tee /etc/apt/sources.list.d/docker.list
```

添加成功后，更新 apt 软件包缓存。

```bash
$ sudo apt-get update
```

#### 安装 Docker

在一切准备就绪后，就可以安装最新版本的 Docker 了，软件包名称为 `docker-engine`。

```bash
$ sudo apt-get install docker-engine
```

如果系统中存在旧版本的 Docker （`lxc-docker`, `docker.io`），会提示是否先删除，选择是即可。

#### 启动 Docker 引擎

##### Ubuntu 12.04/14.04、Debian 7 Wheezy

```bash
$ sudo service docker start
```

##### Ubuntu 16.04、Debian 8 Jessie/Stretch

```bash
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

#### 建立 docker 用户组

默认情况下，`docker` 命令会使用 [Unix socket](https://en.wikipedia.org/wiki/Unix_domain_socket) 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

建立 `docker` 组：

```bash
$ sudo groupadd docker
```

将当前用户加入 `docker` 组：

```bash
$ sudo usermod -aG docker $USER
```

### 参考文档

* [Docker 官方 Ubuntu 安装文档](https://docs.docker.com/engine/installation/linux/ubuntulinux/)
* [Docker 官方 Debian 安装文档](https://docs.docker.com/engine/installation/linux/debian/)
