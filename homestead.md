# Laravel Homestead

- [介绍](#introduction)
- [安装 & 设置](#installation-and-setup)
    - [第一步](#first-steps)
    - [配置 Homestead](#configuring-homestead)
    - [运行 Vagrant Box](#launching-the-vagrant-box)
    - [将 Homestead 安装到项目中](#per-project-installation)
    - [安装 MariaDB](#installing-mariadb)
- [日常使用](#daily-usage)
    - [Homestead 全局访问](#accessing-homestead-globally)
    - [SSH 连接](#connecting-via-ssh)
    - [连接到数据库](#connecting-to-databases)
    - [增加网站](#adding-additional-sites)
    - [配置计划任务](#configuring-cron-schedules)
    - [端口](#ports)
- [网络接口](#network-interfaces)

<a name="introduction"></a>
## 介绍

Laravel 致力于创造令人愉悦的 PHP 开发体验，包括你的本地开发环境。[Vagrant](http://vagrantup.com) 提供了简单优雅的虚拟机管理方式。

Laravel Homestead 是由官方预包装的 Vagrant box，它为你提供了完备的开发环境，换言之，不需要你在本地电脑安装 PHP、HHVM、Web Server 以及其他任何服务器软件。你不用再担心弄乱自己的操作系统！Vagrant boxes 完全是一次性的，如果什么地方出了状况，分分钟就可以销毁并重建 box！

Homestead 能够运行在任何 Windows、Mac 或 Linux 操作系统。它包扩了你在开发 Laravel 应用时所需的全部软件 Nginx web server、PHP 7.0、MySQL、Postgres、Redis、Memcached、Node 等。

> {note} 如果你使用 Windows 系统，你应该启用主板的硬件虚拟化支持 (VT-x)。通常在 BIOS 中设置。如果你在 UEFI 系统上使用 Hyper-V，你可能还需禁用 Hyper-V 才能访问 VT-x。

<a name="included-software"></a>
### 包含的软件

- Ubuntu 16.04
- Git
- PHP 7.0
- HHVM
- Nginx
- MySQL
- MariaDB
- Sqlite3
- Postgres
- Composer
- Node (With PM2, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd

<a name="installation-and-setup"></a>
## 安装 & 设置

<a name="first-steps"></a>
### 第一步

在运行 Homestead 环境之前，必须安装 [VirtualBox 5.x](https://www.virtualbox.org/wiki/Downloads) 或 [VMWare](http://www.vmware.com) 以及 [Vagrant](http://www.vagrantup.com/downloads.html)。这些软件为主流的操作系统均提供了可视化的安装工具，安装起来非常简单。

使用 VMware，需要购买 VMware Fusion / Workstation 和 [VMware Vagrant plug-in](http://www.vagrantup.com/vmware)。 虽然需要付费，但 VMware 能提供更快的文件夹共享性能。

#### 安装 Homestead Vagrant Box

VirtualBox / VMware 和 Vagrant 安装完成以后，在终端使用以下命令将 `laravel/homestead` box 添加到 Vagrant。它会从服务器下载 box，取决于网速，这一步可能需要一些时间：

    vagrant box add laravel/homestead

如果命令出错，请检查 Vagrant 是否是最新版。

#### 安装 Homestead

从仓库将 Homestead 克隆至本地。建议将仓库克隆至 "home" 目录的 `Homestead` 文件夹中，这样 Homestead box 能为本地所有的 Laravel 项目服务：

    cd ~

    git clone https://github.com/laravel/homestead.git Homestead

仓库克隆成功以后，在 Homestead 目录运行 `bash init.sh` 命令创建 `Homestead.yaml` 配置文件。`Homestead.yaml` 会被存放在 `~/.homestead` 隐藏的目录中：

    bash init.sh

<a name="configuring-homestead"></a>
### 配置 Homestead

#### 设置 Provider

`~/.homestead/Homestead.yaml` 文件中的 `provider` 用以指定 Vagrant 使用哪个 provider：`virtualbox`、`vmware_fusion` 或 `vmware_workstation`。请根据需要设置：

    provider: virtualbox

#### 配置共享文件夹

`Homestead.yaml` 文件中的 `folders` 属性列出了你想与 Homestead 环境共享的所有文件夹。共享文件夹中的文件发生变化时，他们将在本地环境和 Homestead 环境之间保持同步。你可以根据需要配置任意多个共享文件夹：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code

若想启用 [NFS](http://docs.vagrantup.com/v2/synced-folders/nfs.html)，只要在同步文件夹配置项中增加一个标记即可：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "nfs"

#### 配置 Nginx 网站

即使你不熟悉 Nginx 也没关系。通过 `sites` 属性可以非常容易的将一个 "域名" 映射到 Homestead 环境的一个文件夹。`Homestead.yaml` 文件中有一个示例网站配置。另外，你可以根据需要添加任意多个网站到 Homestead 环境。Homestead 能够方便的为你所有的 Laravel 项目提供虚拟化环境：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public

通过设置 `hhvm` 选项为 `true`，就可以让 Homestead 中的所有网站使用 [HHVM](http://hhvm.com)：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          hhvm: true

修改 `sites` 属性后，应该执行一次 `vagrant reload --provision` 从而使虚拟机中的 Nginx 配置生效。

#### Hosts 文件

必须将添加到 Nginx 配置中的域名同时添加到系统的 `hosts` 文件。`hosts` 会将域名访问请求重定向到 Homestead。Mac 和 Linux 系统的 `hosts` 文件位于 `/etc/hosts`。Windows 系统位于 `C:\Windows\System32\drivers\etc\hosts`。请参照如下格式添加：

    192.168.10.10  homestead.app

请使用 `~/.homestead/Homestead.yaml` 中设置的 IP 地址。在 `hosts` 添加域名并启动 Vagrant box 后，就可以在浏览器中访问网站了：

    http://homestead.app

<a name="launching-the-vagrant-box"></a>
### 启动 Vagrant Box

完成了 `Homestead.yaml` 配置文件的编辑。现在就可以切换到 Homestead 目录执行 `vagrant up` 命令，Vagrant 会启动虚拟机并自动配置共享文件夹和 Nginx 站点。

使用 `vagrant destroy --force` 命令可销毁虚拟机。

<a name="per-project-installation"></a>
### 将 Homestead 安装到项目中

如果不想让所有项目共享一个 Homestead box，则可以为每一个项目单独配置 Homestead 实例。当你要为项目关联 `Vagrantfile` 文件时单独的 Homestead 就格外有用，各项目独立使用 `vagrant up` 命令运行。

执行以下 Composer 命令，将 Homestead 直接安装到项目目录：

    composer require laravel/homestead --dev

Homestead 安装完毕，使用 `make` 命令在项目根目录生成 `Vagrantfile` 和 `Homestead.yaml`。`make` 命令会自动配置 `Homestead.yaml` 中的 `sites` 和 `folders` 指令。

Mac / Linux:

    php vendor/bin/homestead make

Windows:

	vendor\\bin\\homestead make

接下来，在终端中执行 `vagrant up` 命令并使用浏览器访问 `http://homestead.app`。请记住，将 Homestead 安装在独立的项目中仍旧需要将域名（`homestead.app` 或其他你希望使用的域名）添加到 `/etc/hosts` 文件中。

<a name="installing-mariadb"></a>
### 安装 MariaDB

如果你想用 MariaDB 替代 MySQL 数据库，在 `Homestead.yaml` 文件中添加 `mariadb` 选项即可。此选项会移除 MySQL 并安装 MariaDB。MariaDB 只是 MySQL 的一个简易的替代，因此在你程序的数据库配置中应保持使用 `mysql` 驱动：

    box: laravel/homestead
    ip: "192.168.20.20"
    memory: 2048
    cpus: 4
    provider: virtualbox
    mariadb: true

<a name="daily-usage"></a>
## 日常使用

<a name="accessing-homestead-globally"></a>
### Homestead 全局访问

有时你可能希望在文件系统的任意位置用 `vagrant up` 去控制 Homestead 虚拟机。为此，可以在你的 Bash 配置文件中添加一个简单的 Bath 函数。有了这个函数就可以在任意位置直接运行所有 Vagrant 命令且自动将命令指向 Homestead 的安装位置：

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

请根据你本地所安装 Homestead 的实际位置对以上函数中的 `~/Homestead` 路径部分进行修改。函数创建完毕，接下来你就可以在系统的任意位置执行运行 `homestead up` 或 `homestead ssh` 命令去控制 Homestead 了。

<a name="connecting-via-ssh"></a>
### SSH 连接

在 Homestead 目录执行 `vagrant ssh` 命令即可通过 SSH 协议访问虚拟机。

如果你经常需要使用 SSH 访问 Homestead 虚拟机，则可以考虑添加上一节提到的“函数”，从而解决快速使用 SSH 连接 Homestead box 的需求。

<a name="connecting-to-databases"></a>
### 连接到数据库

Homestead 虚拟机中已经在 MySQL 和 Postgres 数据库中预创建了名为 `homestead` 的数据库。为了便于使用，Laravel 的 `.env` 文件也已经对该数据库做了预配置（开箱即用）。

To connect to your MySQL or Postgres database from your host machine via Navicat or Sequel Pro, you should connect to `127.0.0.1` and port `33060` (MySQL) or `54320` (Postgres). The username and password for both databases is `homestead` / `secret`.
在主机上使用 Navicat 或 Sequel Pro 客户端访问数据库时，主机地址应使用 `127.0.0.1`，端口应使用 `33060` (MySQL) 或 `54320` (Postgres)。用户明和密码分别为 `homestead` / `secret`。

> {note} 注意，只有在你从主机连接数据库时使用非标准端口。运行在虚拟机中的 Laravel 程序访问数据库时应该使用默认的 3306 和 5432 标准端口。

<a name="adding-additional-sites"></a>
### 增加网站

Homestead 环境开始运行以后，你可能希望为 Laravel 应用添加其他的 Nginx 站点。你可以在一个 Homestead 环境中运行任意多个 Laravel 应用。增加额外的网站，只需在 `~/.homestead/Homestead.yaml` 文件中添加并从 Homestead 目录执行 `vagrant provision` 命令即可。

<a name="configuring-cron-schedules"></a>
### 配置计划任务

Laravel 提供了方便的 [调度计划任务](/docs/{{version}}/scheduling) 的方式，使用 `schedule:run` Artisan 命令即可安排每分钟执行一次检查。`schedule:run` 命令会检查 `App\Console\Kernel` 类中的任务调度设置，从而判断应执行哪个任务。

如果希望用 `schedule:run` 命令运行 Homestead 中的站点，则需要将网站配置中的 `schedule` 选项设置为 `true`：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          schedule: true

网站的计划任务在虚拟机的 `/etc/cron.d` 文件夹中定义。

<a name="ports"></a>
### 端口

默认情况下，以下主机端口会被转发到 Homestead 环境：

- **SSH:** 2222 &rarr; Forwards To 22
- **HTTP:** 8000 &rarr; Forwards To 80
- **HTTPS:** 44300 &rarr; Forwards To 443
- **MySQL:** 33060 &rarr; Forwards To 3306
- **Postgres:** 54320 &rarr; Forwards To 5432

#### 转发其他端口

如果需要，可以添加额外的端口转发到 Vagrant box：

    ports:
        - send: 93000
          to: 9300
        - send: 7777
          to: 777
          protocol: udp

<a name="network-interfaces"></a>
## 网络接口

`Homestead.yaml` 文件中的 `networks` 属性用于配置 Homestead 环境的网络接口。你可以根据需要增加任意多个接口：

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

启用 [bridged（桥接）](https://www.vagrantup.com/docs/networking/public_network.html) 接口，配置 `bridge` 设置并修改网络类型为 `public_network`：

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

启用 [DHCP](https://www.vagrantup.com/docs/networking/public_network.html)，只需从配置中移除 `ip` 选项即可：

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"
