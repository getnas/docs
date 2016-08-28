# 错误 & 日志

- [介绍](#introduction)
- [配置](#configuration)
    - [错误详情](#error-detail)
    - [日志存储](#log-storage)
    - [日志等级](#log-severity-levels)
    - [自定义 Monolog 配置](#custom-monolog-configuration)
- [异常处理程序](#the-exception-handler)
    - [报告方法](#report-method)
    - [渲染方法](#render-method)
- [HTTP 异常](#http-exceptions)
    - [自定义 HTTP 错误页面](#custom-http-error-pages)
- [日志](#logging)

<a name="introduction"></a>
## 介绍

Laravel 默认已完成了错误和异常处理的配置。`App\Exceptions\Handler` 类将应用引发的所有异常都记录下来，然后返回给用户。此文档将深入介绍这个类。

Laravel 利用 [Monolog](https://github.com/Seldaek/monolog) 库实现日志功能。Monolog 提供了大量的功能强大的日志处理器。Laravel 为你配置了几种处理器，你可以选择单个日志文件、ratating 日志文件或将错误信息写入系统日志。

<a name="configuration"></a>
## 配置

<a name="error-detail"></a>
### 错误详情

`config/app.php` 配置文件中的 `debug` 选项用以确定向用户显示多少错误信息。默认情况下，此项设置会调用 `.env` 文件中的 `APP_DEBUG` 环境变量的值。

对于本地开发，你应将 `APP_DEBUG` 环境变量设置为 `true`。而在生产环境中应该始终将其设置为 `false`，否则程序会因将敏感配置信息暴露给终端用户而造成安全风险。

<a name="log-storage"></a>
### 日志存储

Laravel 支持将日志信息写到 `single` 文件、`daily` 文件、`syslog` 以及  `errorlog`。你可以在 `config/app.php` 配置文件的 `log` 选项设置 Laravel 使用的存储机制。例如，你想使用 daily 日志文件：

    'log' => 'daily'

#### 最大 Daily 日志文件

使用 `daily` 日志模式时，Laravel 将默认保存日志 5 天。如果你想修改保存的时长，应该添加 `log_max_files` 选项到 `app` 配置文件中：

    'log_max_files' => 30

<a name="log-severity-levels"></a>
### 日志等级

当你使用 Monolog 时，日志信息可能有不同的等级。默认情况下，Laravel 会记录所有日志到存储。然而，在生产环境中，你可能希望调低日志等级。在 `app.php` 配置文建中设置 `log_level` 选项。

配置此选项后，Laravel 的日志将大于或等于设置的等级。例如，默认的 `log_level` 为 `error`，这将记录 **error**、**critical**、**alert** 和 **emergency**：

    'log_level' => env('APP_LOG_LEVEL', 'error'),

> {tip} Monolog 支持如下日志等级 - 程度从低到高依次为：`debug`、`info`、`notice`、`warning`、`error`、`critical`、`alert`、`emergency`。

<a name="custom-monolog-configuration"></a>
### 自定义 Monolog 配置

如果你想完全控制 Monolog，你可能需要使用应用的 `configureMonologUsing` 方法。你应将此方法在 `bootstrap/app.php` 文件的 `return $app` 变量之前调用：

    $app->configureMonologUsing(function($monolog) {
        $monolog->pushHandler(...);
    });

    return $app;

<a name="the-exception-handler"></a>
## 异常处理程序

<a name="report-method"></a>
### 报告方法

所有的异常均由 `App\Exceptions\Handler` 类处理。该类包含两个方法：`report` 和 `render`。`report` 方法用于记录异常或发送异常信息到外部服务，例如，[Bugsnag](https://bugsnag.com) 或 [Sentry](https://github.com/getsentry/sentry-laravel)。默认情况下，`report` 方法简单的将异常传送到记录异常日志的基类。当然，你可以随意对异常日志按需设置。

例如，你需要用不同的方式报告不同类别的日志，你可能要使用 PHP 的 `instanceof` 操作：

    /**
     * Report or log an exception.
     *
     * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }

        return parent::report($exception);
    }

#### 依据类型忽略异常

异常处理程序的 `$dontReport` 属性包含一个异常类型数组，数组中的记录将不会被记录。例如，与 404 错误类似的一些错误类型将不会被写入日志文件。你可以根据需要添加其他异常类型到此数组中：

    /**
     * A list of the exception types that should not be reported.
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];

<a name="render-method"></a>
### 渲染方法

`render` 方法将异常转换成 HTTP 相应回传给浏览器。默认情况下，异常会被传送到生成响应的基类。当然，你可以自由检查异常类型或返回自定义相应：

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $exception);
    }

<a name="http-exceptions"></a>
## HTTP 异常

还有一些描述服务器 HTTP 错误代码的异常。例如，"page not found" 错误（404）、"unauthorized error" (401) 或 开发者生成的 500 错误。你可以使用 `abort` 帮助器在应用的任何位置生成上述响应：

    abort(404);

`abort` 帮助器会立即引发一个异常，它们由异常处理程序渲染。另外，你也可以设置响应文本：

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### 自定义 HTTP 错误页面

Laravel 能够非常容易的为各种 HTTP 状态代码设置自定义错误页面。例如，如果你想创建自定义的 404 错误页面，则创建 `resources/views/errors/404.blade.php` 即可。应用会自动调用此视图渲染 404 错误页面。此目录中的视图文件名称应与 HTTP 状态代码相一致。`abort` 方法将作为一个 `$exception` 变量被传入 `HttpException` 实例。

<a name="logging"></a>
## 日志

Laravel 在强大的 [Monolog](http://github.com/seldaek/monolog) 类库上面构建了一个简易的抽象层。默认情况下，Laravel 的配置会在 `storage/logs` 目录创建一个日志文件。你可以使用 `Log` [facade](/docs/{{version}}/facades) 写信息到日志：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Support\Facades\Log;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

依据 [RFC 5424](http://tools.ietf.org/html/rfc5424) 定义，共有八个日志记录级别：**emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info** 和 **debug**。

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

#### Contextual 信息

contextual 数据可以以数组形式传入 log 方法。contextual 数据会被格式化并在日志消息中显示：

    Log::info('User failed to login.', ['id' => $user->id]);

#### 访问底层 Monolog 实例

Monolog 还有许多额外的处理器可以用于日志。如果需要，你可以访问 Laravel 使用的底层 Monolog 实例：

    $monolog = Log::getMonolog();
