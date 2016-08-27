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

All exceptions are handled by the `App\Exceptions\Handler` class. This class contains two methods: `report` and `render`. We'll examine each of these methods in detail. The `report` method is used to log exceptions or send them to an external service like [Bugsnag](https://bugsnag.com) or [Sentry](https://github.com/getsentry/sentry-laravel). By default, the `report` method simply passes the exception to the base class where the exception is logged. However, you are free to log exceptions however you wish.

For example, if you need to report different types of exceptions in different ways, you may use the PHP `instanceof` comparison operator:

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

#### Ignoring Exceptions By Type

The `$dontReport` property of the exception handler contains an array of exception types that will not be logged. For example, exceptions resulting from 404 errors, as well as several other types of errors, are not written to your log files. You may add other exception types to this array as needed:

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
### The Render Method

The `render` method is responsible for converting a given exception into an HTTP response that should be sent back to the browser. By default, the exception is passed to the base class which generates a response for you. However, you are free to check the exception type or return your own custom response:

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
## HTTP Exceptions

Some exceptions describe HTTP error codes from the server. For example, this may be a "page not found" error (404), an "unauthorized error" (401) or even a developer generated 500 error. In order to generate such a response from anywhere in your application, you may use the `abort` helper:

    abort(404);

The `abort` helper will immediately raise an exception which will be rendered by the exception handler. Optionally, you may provide the response text:

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### Custom HTTP Error Pages

Laravel makes it easy to display custom error pages for various HTTP status codes. For example, if you wish to customize the error page for 404 HTTP status codes, create a `resources/views/errors/404.blade.php`. This file will be served on all 404 errors generated by your application. The views within this directory should be named to match the HTTP status code they correspond to. The `HttpException` instance raised by the `abort` function will be passed to the view as an `$exception` variable.

<a name="logging"></a>
## Logging

Laravel provides a simple abstraction layer on top of the powerful [Monolog](http://github.com/seldaek/monolog) library. By default, Laravel is configured to create a log file for your application in the `storage/logs` directory. You may write information to the logs using the `Log` [facade](/docs/{{version}}/facades):

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

The logger provides the eight logging levels defined in [RFC 5424](http://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** and **debug**.

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

#### Contextual Information

An array of contextual data may also be passed to the log methods. This contextual data will be formatted and displayed with the log message:

    Log::info('User failed to login.', ['id' => $user->id]);

#### Accessing The Underlying Monolog Instance

Monolog has a variety of additional handlers you may use for logging. If needed, you may access the underlying Monolog instance being used by Laravel:

    $monolog = Log::getMonolog();
