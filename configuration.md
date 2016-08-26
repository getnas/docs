# 配置

- [介绍](#introduction)
- [访问配置的值](#accessing-configuration-values)
- [环境配置](#environment-configuration)
    - [检测当前环境](#determining-the-current-environment)
- [配置缓存](#configuration-caching)
- [维护模式](#maintenance-mode)

<a name="introduction"></a>
## 介绍

Laravel 框架的所有配置文件都存储在 `config` 目录。配置文件中的每个选项均有注释说明，您可随时查阅以便了解各项设置。

<a name="accessing-configuration-values"></a>
## 访问配置的值

通过使用全局的 `config` 帮助器方法，你可以在程序的任何位置轻松的访问到配置的值。配置值可以使用“点”语法访问，包括文件名和你需要访问的选项名。若配置项值不存在，程序会自动设置并返回默认值：

    $value = config('app.timezone');

在程序运行时设置选项的值，可以向 `config` 帮助器传入一个数组：

    config(['app.timezone' => 'America/Chicago']);

<a name="environment-configuration"></a>
## 环境配置

让应用程序在不同的环境使用不同的配置值是一件很有帮助的事情。例如，你可能希望本地的开发环境与生产服务器使用不同的缓存驱动。

Laravel 通过使用 [DotEnv](https://github.com/vlucas/phpdotenv) PHP library by Vance Lucas 来实现此功能。全新安装的 Laravel 在根目录中会有一个 `.env.example` 文件。如果是用 Composer 安装，此文件会被自动重命名为 `.env`。反之，你需要手动重命名此文件。

#### 取回环境配置

当程序接收请求时，文件中的所有变量均被载入到 PHP 的超级全局变量 `$_ENV` 里面。当然，你可以使用 `env` 帮助器去获取环境配置文件中的变量。事实上，如果你查阅过 Laravel 配置文件，你可能会注意到已有部分选项使用了这个帮助器方法：

    'debug' => env('APP_DEBUG', false),

`env` 方法的第二个参数用于设置选项的默认值，当环境变量不存在时就会返回此默认值。

由于不同的环境使用不同的环境配置文件，因此，应将 `env` 文件从版本控制系统中排除。

对于团队协作开发的情况，有必要在程序根目录保留一份 `.env.example` 文件。并在其中做一些占位值的设置，以便团队其他成员了解正确运行程序所需的配置。

<a name="determining-the-current-environment"></a>
### 检测当前环境

使用 `APP_ENV` 变量即可从 `.env` 文件中检测到当前应用的运行环境。也可使用 `App` [facade](/docs/{{version}}/facades) 的 `environment` 方法访问此值：

    $environment = App::environment();

可以通过向 `environment` 方法中传值来检查环境。方法会根据值的匹配情况做出判断：

    if (App::environment('local')) {
        // The environment is local
    }

    if (App::environment('local', 'staging')) {
        // The environment is either local OR staging...
    }

<a name="configuration-caching"></a>
## 配置缓存

想提升程序的运行速度，可以使用 Artisan 的 `config:cache` 命令将所有的配置文件整合成一个单独文件，从而加快框架载入配置文件的速度。

通常，`php artisan config:cache` 命令只在生产环境中使用。由于在本地开发时经常需要修改配置文件，因此，本地开发环境不应该使用此命令。

<a name="maintenance-mode"></a>
## 维护模式

当程序进入维护模式，任何对网站的请求都会返回一个自定义的视图。这样就能在更新或维护网站时轻松的禁用网站访问。维护模式会使用 stack middleware 对程序做检查。如果程序出于维护模式，则抛出带有 503 状态码的 `MaintenanceModeException` 错误。

执行 `down` Artisan 命令即可启用维护模式：

    php artisan down

还可以向 `down` 命令添加 `message` 和 `retry` 选项。`message` 的值用于显示消息或将消息记录到日志，`retry`的值将被设置为 HTTP 头 `Retry-After` 的值：

    php artisan down --message='Upgrading Database' --retry=60

执行 `up` 命令禁用维护模式：

    php artisan up

#### 维护模式响应模板

响应模式默认模板为 `resources/views/errors/503.blade.php`，你可以根据需要自由修改。

#### 维护模式 & 队列

出于维护模式的程序不会处理任何 [队列任务](/docs/{{version}}/queues)。当程序退出维护模式后，队列任务会重新开始处理。

#### 维护模式的替代方案

由于维护模式会导致程序在一段时间内无法访问，可以考虑类似 [Envoyer](https://envoyer.io) 的零宕机部署方案。
