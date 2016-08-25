# 安装

- [安装](#installation)
    - [服务器要求](#server-requirements)
    - [安装 Laravel](#installing-laravel)
    - [配置](#configuration)

<a name="installation"></a>
## 安装

<a name="server-requirements"></a>
### 服务器要求

Laravel 框架对服务器环境有一定的要求。当然，[Laravel Homestead](/docs/{{version}}/homestead)虚拟机已经满足了所有的需求，强烈建议你将 Homestead 作为本地的 laravel 开发环境。

当然，如果你不想使用 Homestead，就需要确保服务器满足如下要求：

<div class="content-list" markdown="1">
- PHP >= 5.6.4
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
</div>

<a name="installing-laravel"></a>
### 安装 Laravel

Laravel 通过 [Composer](http://getcomposer.org) 管理依赖。因此，在使用 Laravel 之前，请先安装 Compoer。

#### 使用 Laravel Installer 安装器

首先，使用 Composer 下载 Laravel installer 安装器：

    composer global require "laravel/installer"

为了能够在系统中正确执行 `laravel` 命令，请确保 `~/.composer/vendor/bin` 目录（或等效目录）已经添加到系统的 $PATH 中。

安装完成后，使用 `laravel new` 命令即可在你指定的位置创建一份全新的 Laravel 副本。例如，`laravel new blog` 将会新建 `blog` 目录，里面包括一份已经满所所有依赖的全新 Laravel 副本：

    laravel new blog

#### 使用 Composer Create-Project 命令安装

此外，也可以在命令行中使用 Composer `create-project` 命令安装 Laravel：

    composer create-project --prefer-dist laravel/laravel blog

<a name="configuration"></a>
### 配置

#### Public 目录

Laravel 安装以后，应将 `public` 目录设置为 web 服务器的文档 / web 根目录。目录中的 `index.php` 会控制处理所有对应用程序的 HTTP 请求。

#### 配置文件

Laravel 框架的所有配置文件都存储在 `config` 目录。配置文件中的每个选项均有注释说明，您可随时查阅以便了解各项设置。

#### 目录权限

Laravel 安装以后，还需要做一些权限方面的配置。应确保 web 服务器对 `storage` 和 `bootstrap/cache` 目录拥有写权限，否则 Laravel 将无法运行。如果使用 [Homestead](/docs/{{version}}/homestead) 则无需进行设置，因为都已配置好了。

#### Application Key

接下来应该使用随机字符来设置 Application Key。使用 Composer 或 Laravel installer 安装时，这个 key 默认是通过 `php artisan key:generate` 命令自动设置的。

通常，Application Key 应由 32 个字符组成。可以将 key 设置在 `.env` 环境文件中。如果你的 Laravel 程序目录中还没有这个文件，请手动复制 `.env.example` 文件的副本并重命名为 `.env`。**如果不设置 application ，则无法保证 sessions 和其他加密数据的安全！**

#### 其他配置

事实上，Laravel 是开箱即用的，并不需要做额外的配置。当然，你可能希望查阅 `config/app.php` 文件及其文档，譬如文件中的 `timezone` 和 `locale` 选项可以根据需要进行设置。

你可能也想对 Laravel 的其他组件做一些配置，例如：

<div class="content-list" markdown="1">
- [Cache](/docs/{{version}}/cache#configuration)
- [Database](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)
</div>

Laravel 安装完成，你还应该 [配置你的本地环境](/docs/{{version}}/configuration#environment-configuration).
