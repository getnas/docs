# 目录结构

- [介绍](#introduction)
- [程序根目录](#the-root-directory)
    - [The `app` Directory](#the-root-app-directory)
    - [The `bootstrap` Directory](#the-bootstrap-directory)
    - [The `config` Directory](#the-config-directory)
    - [The `database` Directory](#the-database-directory)
    - [The `public` Directory](#the-public-directory)
    - [The `resources` Directory](#the-resources-directory)
    - [The `routes` Directory](#the-routes-directory)
    - [The `storage` Directory](#the-storage-directory)
    - [The `tests` Directory](#the-tests-directory)
    - [The `vendor` Directory](#the-vendor-directory)
- [The App Directory](#the-app-directory)
    - [The `Console` Directory](#the-console-directory)
    - [The `Events` Directory](#the-events-directory)
    - [The `Exceptions` Directory](#the-exceptions-directory)
    - [The `Http` Directory](#the-http-directory)
    - [The `Jobs` Directory](#the-jobs-directory)
    - [The `Listeners` Directory](#the-listeners-directory)
    - [The `Mail` Directory](#the-mail-directory)
    - [The `Notifications` Directory](#the-notifications-directory)
    - [The `Policies` Directory](#the-policies-directory)
    - [The `Providers` Directory](#the-providers-directory)

<a name="introduction"></a>
## 介绍

Laravel 程序的默认架构是为给各种规模的应用提供更好的开发起点而设计的。当然，你可以按照自己的意愿自由重新组织文件结构。由于 Compoer 会自动加载类文件，因此 Laravel 从不会强制任何类文件的存放位置。

#### 模型(Model)在哪个目录？

初次使用 Laravel，很多开发者困惑找不到 `models` 目录。当然，这个目录的缺失是有意而为之的。我们发现，人们常常对 "models" 这个词汇有不同的理解。一些开发者认为应用的 "models" 代表完整的业务逻辑，而另外一些人认为 "models" 是与关系型数据库进行交互的类。

综上，我们选择将 Eloquent models 默认存放在 `app` 目录，由开发者自行决定是否将他们放在其他什么位置。

<a name="the-root-directory"></a>
## 程序根目录

<a name="the-root-app-directory"></a>
#### The App Directory

你可能已经猜到了，`app` 目录包括应用程序的所有核心代码。我们会在后面展开介绍此目录；当然，你所开发的应用的类文件也都将存放在这里。

<a name="the-bootstrap-directory"></a>
#### The Bootstrap Directory

`bootstrap` 目录包含哪些用于启动框架和自动加载配置的文件。其中还包含一个 `cache` 目录，存放由框架自动生成的提升框架性能的例如路由和服务缓存等文件。

<a name="the-config-directory"></a>
#### The Config Directory

与 `config` 目录名称字面意思相同，此目录包含所有应用的配置文件。阅读并了解所有的配置文件对你的开发工作会很有帮助。


<a name="the-database-directory"></a>
#### The Database Directory

`database` 目录包含所有数据库迁移文件和种子文件。如果需要，你也可以在此目录存储 SQLite 数据库。

<a name="the-public-directory"></a>
#### The Public Directory

`public` 目录包含的 `index.php` 是所有程序请求的入口文件。此目录同时用于存放图片、Javascript 和 CSS 等资源文件。

<a name="the-resources-directory"></a>
#### The Resources Directory

`resources` 存放视图文件以及原生、位编译的 LESS、SASS 或 Javascript 等资源文件。也同时用作存放语言文件。

<a name="the-routes-directory"></a>
#### The Routes Directory

`routes` 目录存放所有的路由文件。默认包含三个路由文件：`web.php`、 `api.php` 和 `console.php`。

`web.php` 是由 `RouteServiceProvider` 定义的路由文件，位于 `web` 中间件组当中，该中间件提供会话状态、CSRF 保护和 cookie 加密。如果你的程序不提供 RESTful API，就应该将所有路由定义在 `web.php` 文件中。

`api.php` 是由 `RouteServiceProvider` 定义的路由文件，位于 `api` 中间件组当中，该中间件提供速度限制。此文件定义无状态路由，因此入站请求会通过 tokens 认证身份且不会访问 session 状态。

你可以在 `console.php` 文件中定义所有基于命令行的闭包。每个闭包都与一个命令行实例绑定，它与命令的 IO 方法之间建立一种简易的交互通。此文件并不用于定义 HTTP 路由，它定义基于控制台的入口点（路由）到您的应用程序中。


<a name="the-storage-directory"></a>
#### The Storage Directory

`storage` 目录存放已编译的视图模板、sessions 文件、缓存文件以及其他由框架自动生成的文件。此目录包括 `app`、`framework` 和 `logs` 三个子目录。`app` 目录存储一些由程序生成的文件。`framework` 目录存储框架生成的文件和缓存。`logs` 目录存储程序的日志文件。

`storage/app/public` 目录可能用于存储用户生成的文件，例如头像，他们应该拥有公开访问权限。你应该创建一个软连接将 `public/storage` 指向此目录。你可能会使用 `php artisan storage:link` 命令创建连接。

<a name="the-tests-directory"></a>
#### The Tests Directory

`tests` 目录用于存放测试用例。[PHPUnit](https://phpunit.de/) 提供了一个开箱即用的示例。每个测试类文件都应以 `Test` 作为后缀。你可以使用 `phpunit` 或 `php vendor/bin/phpunit` 命令运行测试。


<a name="the-vendor-directory"></a>
#### The Vendor Directory

`vendor` 目录用于存放 [Composer](https://getcomposer.org) 依赖。

<a name="the-app-directory"></a>
## The App Directory

你所开发程序的大部分文件均存放在 `app` 目录。默认情况下，此目录在 `App` 命名空间下面，且由 Composer 遵照 [PSR-4 autoloading standard](http://www.php-fig.org/psr/psr-4/) 自动加载。

`app` 目录中包含大量子目录，例如 `Console`、`Http` 和 `Providers`。不难想象，`Console` 和 `Http` 目录存放哪些提供与你程序交互的 API。HTTP 协议和命令行是两种与程序交互的机制，但并不包含程序逻辑。换言之，他们只是你的应用执行命令的两种简易方式。`Console` 目录包含所有 Artisan 命令，`Http` 目录包含所有的控制器、中间件和请求文件。

`app` 目录中的其他子目录将会在你执行 `make` Artisan 生成类文件命令时自动生成。即，除非你执行 `make:job` Artisan 命令生成 job 类，否则 `app/Jobs` 目录默认并不存在。

> {tip} `app` 目录中的很多类文件可以通过 Artisan 命令自动生成。在终端运行 `php artisan list make` 命令即可查看所有可以使用的命令。

<a name="the-console-directory"></a>
#### The Console Directory

`Console` 目录包含所有自定义的 Aritsan 命令。这些命令可能是通过使用 `make:command` 命令生成的。此目录也会存放 console kernel 文件，即用于注册自定义 Artisan 命令和 [scheduled tasks](/docs/{{version}}/scheduling) 定义的文件。

<a name="the-events-directory"></a>
#### The Events Directory

此目录默认并不存在，但在执行 `event:generate` 和 `event:make` Artisan 命令后会自动被创建。`Events` 目录存放 [event classes](/docs/{{version}}/events) 文件。Events 可能会用于在给定的行为被激发时，向程序的其他部分发出通知，提供更好的灵活性以及处理解耦。

<a name="the-exceptions-directory"></a>
#### The Exceptions Directory

`Exceptions` 目录存放程序的异常处理器和异常文件。如果你想自定义异常的日志和渲染方式，则应修改目录中的 `Handler` 类文件。

<a name="the-http-directory"></a>
#### The Http Directory

`Http` 目录存放你的控制器、中间件和表单请求。所有访问你程序的请求处理逻辑都应当放在此目录中。

<a name="the-jobs-directory"></a>
#### The Jobs Directory

此目录默认不存在，在执行 `make:job` Artisan 命令后会自动被创建。`Jobs` 目录用来存放程序的 [queueable jobs](/docs/{{version}}/queues)。Jobs 可能被程序加入处理队列或在当前请求生命周期执行同步。Jobs that run synchronously during the current request are sometimes referred to as "commands" since they are an implementation of the [command pattern](https://en.wikipedia.org/wiki/Command_pattern).

<a name="the-listeners-directory"></a>
#### The Listeners Directory

此目录默认不存在，在执行 `event:generate` 或 `make:listener` Artisan 命令后会自动被创建。`Listeners` 目录用来存放处理 [events](/docs/{{version}}/events) 的类文件。Event listeners 接收一个 event 实例，并在被激发事件的响应中执行逻辑。例如，`UserRegistered` 事件可能由 `SendWelcomeEmail` 来监听。


<a name="the-mail-directory"></a>
#### The Mail Directory

此目录默认不存在，在执行 `make:mail` Artisan 命令后会自动被创建。`Mail` 目录存放所有与电子邮件发送相关的类文件。Mail 对象允许将所有构建一封电子邮件的逻辑放在一个简单的类文件中，可以使用 `Mail::send` 方法执行发送。


<a name="the-notifications-directory"></a>
#### The Notifications Directory

此目录默认不存在，在执行 `make:notification` Artisan 命令后会自动被创建。`Notifications` 目录存放所有由应用发出的 "transactional" 通知，诸如事件发生通知。Laravel 的通知功能支持多种驱动，如电子邮件、Slack、SMS 或存储在数据库。

<a name="the-policies-directory"></a>
#### The Policies Directory

此目录默认不存在，在执行 `make:policy` Artisan 命令后会自动被创建。`Policies` 目录存放应用的授权策略类文件。策略用于检测用户是否有权限执行给定的操作。请查阅 [authorization documentation](/docs/{{version}}/authorization) 了解详情。


<a name="the-providers-directory"></a>
#### The Providers Directory

`Providers` 目录存放应用的 [service providers](/docs/{{version}}/providers)。Service providers 通过在 service container 绑定服务、注册事件或对入站请求执行任何其他任务匹配从而引导你的应用。

全新的 Laravel 应用已默认包含了几个 provider 文件。你可以根据需要自行添加 provider 到此目录。