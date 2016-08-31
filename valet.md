# Laravel Valet

- [介绍](#introduction)
    - [Valet 或 Homestead](#valet-or-homestead)
- [安装](#installation)
- [版本说明](#release-notes)
- [托管网站](#serving-sites)
    - ["Park（停靠模式）" 命令](#the-park-command)
    - ["Link（链接模式）" 命令](#the-link-command)
    - [网站安全（TLS）](#securing-sites)
- [发布网站](#sharing-sites)
- [查看日志](#viewing-logs)
- [自定义 Valet 驱动](#custom-valet-drivers)
- [其他 Valet 命令](#other-valet-commands)

<a name="introduction"></a>
## 介绍

Valet 是一个 Mac 系统下极简的 Laravel 开发环境。无需 Vagrant、Apache、Nginx、`/etc/hosts` 文件。你甚至可以直接用本地主机作为网站的服务器开放互联网访问。_是的，我们也都喜欢这样。_

Laravel Valet 会设置 [Caddy](https://caddyserver.com) 始终随系统启动并在后台运行。然后，使用 [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq)，Valet 将所有发送到 `*.dev` 域名的请求都转发到本地主机上安装的站点。

换句话说，这是一个只需占用大概 7 MB 内存、巨快的的 Laravel 开发环境。Valet 并不是 Vagrant 或 Homestead 的替代品，但对那些希望更灵活快捷或内存吃紧的环境下，它是一个绝好的选择。

Valet 开箱即用的程序包括但不限于：

<div class="content-list" markdown="1">
- [Laravel](https://laravel.com)
- [Lumen](https://lumen.laravel.com)
- [Symfony](https://symfony.com)
- [Zend](http://framework.zend.com)
- [CakePHP 3](http://cakephp.org)
- [WordPress](https://wordpress.org)
- [Bedrock](https://roots.io/bedrock)
- [Craft](https://craftcms.com)
- [Statamic](https://statamic.com)
- [Jigsaw](http://jigsaw.tighten.co)
- Static HTML
</div>

当然，你可以根据个人需要扩展自己 Valet 的 [自定义驱动](#custom-valet-drivers).

<a name="valet-or-homestead"></a>
### Valet 或 Homestead

如你所知，Laravel 提供了 [Homestead](/docs/{{version}}/homestead)，另一种 Laravel 本地开发环境。Homestead 和 Valet 面向不同的受众群体。Homestead 提供完整的 Ubuntu 虚拟机且包含自动化的 Nginx 配置。如果你想要一个完整的 Linux 虚拟化开发环境或主机为 Linux / Windows 系统，Homestead 必然是更好的选择。

Valet 仅支持 Mac 系统，且需要你自己在本地安装 PHP 和 数据库，你可以通过 [Homebrew](http://brew.sh/) 进行安装，例如 `brew install php70` 和 `brew install mariadb`。Valet 能够在最小系统开销下快速提供开发环境，因此对于那些不需要完整虚拟化开发环境、只需要 PHP / MySQL 的情况下，Valet 是不错的选择。

Valet 和 Homestead 都是构建 Laravel 开发环境的良好选择。你可以依据个人需求或团队需求进行选择。

<a name="installation"></a>
## 安装

**Valet 依赖 macOS 和 [Homebrew](http://brew.sh/)。安装前，请确保没有其他的 Apache 或 Nginx 等程序占用本地 80 端口。**

<div class="content-list" markdown="1">
- 使用 `brew update` 命令安装或更新 [Homebrew](http://brew.sh/) 到最新版本。
- 使用 `brew install homebrew/php/php70` 安装 PHP 7.0。
- 使用 `composer global require laravel/valet` Valet。确保 `~/.composer/vendor/bin` 在系统的 $PATH 中。
- 运行 `valet install` 命令将自动安装并配置 Valet 和 DnsMasq，同时会设置 Valet 守护进程开机启动。
</div>

Valet 安装完毕，请尝试在命令行中 ping 任意 `*.dev` 域名，例如，`ping foobar.dev`。如果 Valet 安装正确，就会看到域名在 `127.0.0.1` 的响应信息。

Valet 会随系统启动。Valet 初次安装后就无需再运行 `valet start` 或 `valet install` 命令。

#### 使用其他域名

默认情况下，Valet 使用 `.dev` 等域名伺服项目。如果想使用其他域名，则使用 `valet domain tld-name` 命令。

例如，用 `.app` 替代 `.dev`，运行 `valet domain app` 命令，Valet 会自动将域名修改为 `*.app`。

#### 数据库

如果需要数据库，在命令行执行 `brew install mariadb` 命令即可安装 MariaDB。使用 `127.0.0.1` 主机地址，用户名为 `root`，密码为空。

<a name="release-notes"></a>
## 版本说明

### Version 1.1.5

Valet 1.1.5 版本带来了多种内部提升。

#### 升级方法

首先在终端执行 `composer global update` 命令更新 Valet，然后执行 `valet install` 命令使更新生效。

### Version 1.1.0

The 1.1.0 release of Valet brings a variety of great improvements. The built-in PHP server has been replaced with [Caddy](https://caddyserver.com/) for serving incoming HTTP requests. Introducing Caddy allows for a variety of future improvements and allows Valet sites to make HTTP requests to other Valet sites without blocking the built-in PHP server.

#### Upgrade Instructions

After updating your Valet installation using `composer global update`, you should run the `valet install` command in your terminal to create the new Caddy daemon file on your system.

<a name="serving-sites"></a>
## 托管网站

Valet 安装完毕，即可开始托管网站。Valet 提供了两个命令实现 Laravel 站点托管：`park` 和 `link`。

<a name="the-park-command"></a>
**`park`（停靠模式） 命令**

<div class="content-list" markdown="1">
- 使用 `mkdir ~/Sites` 命令在 Mac 中创建新目录。接下来，执行 `cd ~/Sites` 进入目录并运行 `valet park`。此命令会将当前目录作为 Valet 站点路径。
- 紧接着，在当前目录创建新的 Laravel 站点：`laravel new blog`。
- 在浏览器中访问 `http://blog.dev`。
</div>

**以上就是全部要做的。** 现在，当前目录中创建的所有 Laravel 项目将被自动托管，使用 `http://folder-name.dev` 访问。

<a name="the-link-command"></a>
**`link`（链接模式） 命令**

也可使用 `link` 命令托管 Laravel 站点。此命令通常用于直接托管某个项目目录，而不是将整个目录作为所有项目的托管路径。

<div class="content-list" markdown="1">
- 使用此命令，首先切换到某个项目目录中，然后运行 `valet link app-name` 命令。Valet 将在 `~/.valet/Sites` 中创建一个指向当前工作目录的符号链接。
- `link` 命令运行后，在浏览器中使用 `http://app-name.dev` 域名访问项目。
</div>

运行 `valet links` 命令查看所有 链接的目录。使用 `valet unlink app-name` 销毁符号链接。

<a name="securing-sites"></a>
**网站安全（TLS）**

默认情况下，Valet 托管的网站使用 HTTP 协议。当然，如果想使用 HTTP/2(TLS 加密链接)，可使用 `secure` 命令。例如，Valet 当前托管的项目域名为 `laravel.dev`，你可以运行如下命令增强其安全性：

    valet secure laravel

使用 `unsecure` 命令可以恢复使用 HTTP 协议：

    valet unsecure laravel

<a name="sharing-sites"></a>
## 发布网站

Valet 甚至包含了一个将网站开放给互联网访问的命令，且无需安装其他软件。

想开放网站给互联网访问，只需切换到站点目录并运行 `valet share` 命令。可以被互联网访问的 URL 会被复制到剪贴板，将其粘贴到浏览器即可访问。

使用组合键 `Control + C` 即可停止网站公共访问。

<a name="viewing-logs"></a>
## 查看日志

如果你希望在终端中查看站点的所有日志，可使用 `valet logs` 命令。当有新日志时，会显示在终端中。这种方式会将新发生的日志显示在终端的最顶端，而无需保持终端不被占用。

<a name="custom-valet-drivers"></a>
## 自定义 Valet 驱动

你可编写自己的 Valet 驱动来托管哪些并未被 Valet 原生支持的 PHP 框架或 CMS。Valet 安装完毕后，在 `~/.valet/Drivers` 文件夹中能够找到 `SampleValetDriver.php` 文件。此文件包含一个编写自定义驱动的样例。编写驱动只需实现三个方法：`serves`、`isStaticFile` 和 `frontControllerPath`。

三种方法均接受 `$sitePath`、`$siteName` 和 `$uri` 参数值。`$sitePath` 为托管站点的完整路径，例如，`/Users/Lisa/Sites/my-project`。`$siteName` 是域名(`my-project`)中 "host" / "site name" 的部分。`$uri` 为入站请求的 URI (`/foo/bar`)。

自定义 Valet 驱动编写完成后，采用 `FrameworkValetDriver.php` 命名规则将驱动放到 `~/.valet/Drivers` 目录。例如，如果你为 WordPress 编写了一个驱动，文件名应该是 `WordPressValetDriver.php`。


#### `serves` 方法

如果驱动能够处理入站请求，`serves` 方法应返回 `true`，否则应返回 `false`。因此，此方法的目标在于检测给定的 `$sitePath` 值是否是你要托管的项目类型。

例如，假设我们在编写 `WordPressValetDriver`。serve 方法可能看起来像下面这样：

    /**
     * Determine if the driver serves the request.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return void
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }

#### `isStaticFile` 方法

`isStaticFile` 方法应该用于检测入站的请求是不是静态的，例如请求一个图片或样式表。如果是静态文件，则应返回静态资源在磁盘上的完整路径。如果请求的不是静态文件，则返回 `false`：

    /**
     * Determine if the incoming request is for a static file.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string|false
     */
    public function isStaticFile($sitePath, $siteName, $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }

        return false;
    }

> {note} `isStaticFile` 方法仅在 `serves` 方法返回 `true` 且请求的 URI 不为 `/` 时被调用。

#### `frontControllerPath` 方法

`frontControllerPath` 应返回程序入口文件的完整路径，通常为 "index.php"：

    /**
     * Get the fully resolved path to the application's front controller.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public/index.php';
    }

<a name="other-valet-commands"></a>
## 其他 Valet 命令

命令  | 说明
------------- | -------------
`valet forget` | 在一个“停靠模式”的目录中执行此命令，将目录从停靠模式列表中移除。
`valet paths` | 查看所有“停靠模式”的路径。
`valet restart` | 重启 Valet 守护进程。
`valet start` | 启动 Valet 守护进程。
`valet stop` | 停止 Valet 守护进程。
`valet uninstall` | 彻底卸载 Valet 守护进程。
