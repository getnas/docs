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

The `storage/app/public` directory may be used to store user-generated files, such as profile avatars, that should be publicly accessible. You should create a symbolic link at `public/storage` which points to this directory. You may create the link using the `php artisan storage:link` command.

<a name="the-tests-directory"></a>
#### The Tests Directory

The `tests` directory contains your automated tests. An example [PHPUnit](https://phpunit.de/) is provided out of the box. Each test class should be suffixed with the word `Test`. You may run your tests using the `phpunit` or `php vendor/bin/phpunit` commands.

<a name="the-vendor-directory"></a>
#### The Vendor Directory

The `vendor` directory contains your [Composer](https://getcomposer.org) dependencies.

<a name="the-app-directory"></a>
## The App Directory

The majority of your application is housed in the `app` directory. By default, this directory is namespaced under `App` and is autoloaded by Composer using the [PSR-4 autoloading standard](http://www.php-fig.org/psr/psr-4/).

The `app` directory contains a variety of additional directories such as `Console`, `Http`, and `Providers`. Think of the `Console` and `Http` directories as providing an API into the core of your application. The HTTP protocol and CLI are both mechanisms to interact with your application, but do not actually contain application logic. In other words, they are simply two ways of issuing commands to your application. The `Console` directory contains all of your Artisan commands, while the `Http` directory contains your controllers, middleware, and requests.

A variety of other directories will be generated inside the `app` directory as you use the `make` Artisan commands to generate classes. So, for example, the `app/Jobs` directory will not exist until you execute the `make:job` Artisan command to generate a job class.

> {tip} Many of the classes in the `app` directory can be generated by Artisan via commands. To review the available commands, run the `php artisan list make` command in your terminal.

<a name="the-console-directory"></a>
#### The Console Directory

The `Console` directory contains all of the custom Artisan commands for your application. These commands may be generated using the `make:command` command. This directory also houses your console kernel, which is where your custom Artisan commands are registered and your [scheduled tasks](/docs/{{version}}/scheduling) are defined.

<a name="the-events-directory"></a>
#### The Events Directory

This directory does not exist by default, but will be created for you by the `event:generate` and `event:make` Artisan commands. The `Events` directory, as you might expect, houses [event classes](/docs/{{version}}/events). Events may be used to alert other parts of your application that a given action has occurred, providing a great deal of flexibility and decoupling.

<a name="the-exceptions-directory"></a>
#### The Exceptions Directory

The `Exceptions` directory contains your application's exception handler and is also a good place to place any exceptions thrown by your application. If you would like to customize how your exceptions are logged or rendered, you should modify the `Handler` class in this directory.

<a name="the-http-directory"></a>
#### The Http Directory

The `Http` directory contains your controllers, middleware, and form requests. Almost all of the logic to handle requests entering your application will be placed in this directory.

<a name="the-jobs-directory"></a>
#### The Jobs Directory

This directory does not exist by default, but will be created for you if you execute the `make:job` Artisan command. The `Jobs` directory houses the [queueable jobs](/docs/{{version}}/queues) for your application. Jobs may be queued by your application or run synchronously within the current request lifecycle. Jobs that run synchronously during the current request are sometimes referred to as "commands" since they are an implementation of the [command pattern](https://en.wikipedia.org/wiki/Command_pattern).

<a name="the-listeners-directory"></a>
#### The Listeners Directory

This directory does not exist by default, but will be created for you if you execute the `event:generate` or `make:listener` Artisan commands. The `Listeners` directory contains the classes that handle your [events](/docs/{{version}}/events). Event listeners receive an event instance and perform logic in response to the event being fired. For example, a `UserRegistered` event might be handled by a `SendWelcomeEmail` listener.

<a name="the-mail-directory"></a>
#### The Mail Directory

This directory does not exist by default, but will be created for you if you execute the `make:mail` Artisan command. The `Mail` directory contains all of your classes that represent emails sent by your application. Mail objects allow you to encapsulate all of the logic of building an email in a single, simple class that may be sent using the `Mail::send` method.

<a name="the-notifications-directory"></a>
#### The Notifications Directory

This directory does not exist by default, but will be created for you if you execute the `make:notification` Artisan command. The `Notifications` directory contains all of the "transactional" notifications that are sent by your application, such as simple notifications about events that happen within your application. Laravel's notification features abstracts sending notifications over a variety of drivers such as email, Slack, SMS, or stored in a database.

<a name="the-policies-directory"></a>
#### The Policies Directory

This directory does not exist by default, but will be created for you if you execute the `make:policy` Artisan command. The `Policies` directory contains the authorization policy classes for your application. Policies are used to determine if a user can perform a given action against a resource. For more information, check out the [authorization documentation](/docs/{{version}}/authorization).

<a name="the-providers-directory"></a>
#### The Providers Directory

The `Providers` directory contains all of the [service providers](/docs/{{version}}/providers) for your application. Service providers bootstrap your application by binding services in the service container, registering events, or performing any other tasks to prepare your application for incoming requests.

In a fresh Laravel application, this directory will already contain several providers. You are free to add your own providers to this directory as needed.
