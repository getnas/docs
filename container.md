# Service Container

- [介绍](#introduction)
- [Binding](#binding)
    - [Binding 基础](#binding-basics)
    - [Binding 接口到实现](#binding-interfaces-to-implementations)
    - [Contextual Binding](#contextual-binding)
    - [打标签](#tagging)
- [执行解析](#resolving)
    - [Make 方法](#the-make-method)
    - [自动注入](#automatic-injection)
- [容器事件](#container-events)

<a name="introduction"></a>
## 介绍

Laravel 的 service container 是一个强大的工具，用于管理类依赖和执行依赖注入。Dependency injection（依赖注入）是一个花式叫法，其本质上的意思是：通过某种构造器（constructor），将类的依赖“注入”到类中，在某些情况下，这个构造器是 "setter" 方法。

看一个简单的示例：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Repositories\UserRepository;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }

在这个例子中，`UserController` 需要从数据源取回用户列表。因此，我们就“注入”一个能够取回用户列表的服务。上例中，`UserRepository` 与 [Eloquent](/docs/{{version}}/eloquent) 从数据库中取回用户信息的方式非常相似。然而，由于仓库已被注入，我们可以轻松的将其切换成其他的实现。当测试我们的应用时，也可以简单的的“模仿”或创建 `UserRepository` 的假实例。

想深入理解 Laravel service container，你需要构建一个功能强大、规模庞大的应用才行，比如构建 Laravel 的内核。

<a name="binding"></a>
## Binding

<a name="binding-basics"></a>
### Binding 基础

`service container bindings` 总是被注册在 [service providers](/docs/{{version}}/providers) 中，因此大多数的示例都会在容器的上下文中演示。

> {tip} 如果类并不依赖任何接口，则无需将类绑定到容器中。由于反射（reflection）可以自动解析这些对象，因此容器并不需要那些构建对象的指令。

#### 简单的 Bindings

在 `service provider` 中，通过 `$this->app` 属性访问容器。使用 `bind` 方法注册 binding，传入希望注册的类或接口名，以及一个返回类实例的 `闭包`：

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

注意，我们接受容器本身作为解释器的参数。然后就可以使用容器去解析所要构建对象的子依赖。

#### Binding A Singleton

使用 `singleton` 方法绑定到容器的类或接口仅允许被解析一次。一旦 singleton 绑定被解析，到容器的后续请求将返回同一个对象实例：

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

#### Binding 实例

使用 `instance` 方法可以将已存在的对象实例绑定到容器。到容器的后续请求将总是返回给定的实例：

    $api = new HelpSpot\API(new HttpClient);

    $this->app->instance('HelpSpot\Api', $api);

#### Binding 原始值

有时，你可能有一个接受注入类的类，但也可能需要注入像 integer 这样的原始值。你可以结合上下文轻松的绑定任何类需要注入的值：

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

<a name="binding-interfaces-to-implementations"></a>
### Binding Interfaces To Implementations

`service container` 的一项非常强大的功能是能够绑定 interface（接口）到 implementation（实现）。例如，假设有一个 `EventPusher` 接口和一个 `RedisEventPusher` 实现。当我们编写了此接口的 `RedisEventPusher` 实现后，即可在服务容器中进行注册：

    $this->app->bind(
        'App\Contracts\EventPusher',
        'App\Services\RedisEventPusher'
    );

上述声明会告诉容器，当类需要 `EventPusher` 实现时注入 `RedisEventPusher`。现在我们能在构造器或任何其他需要依赖注入的地方以类型提示的方式注入 `EventPusher` 接口：

    use App\Contracts\EventPusher;

    /**
     * Create a new class instance.
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### Contextual Binding

有时，你可能有两个类都利用相同的接口，却希望向各自的类注入不同的实现。例如，两个控制器都依赖 `Illuminate\Contracts\Filesystem\Filesystem` [contract](/docs/{{version}}/contracts) 的不同实现。Laravel 提供了一种简单平滑的接口来定义这种行为：

    use Illuminate\Support\Facades\Storage;
    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;

    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });

    $this->app->when(VideoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });

<a name="tagging"></a>
### 打标签

偶尔，你可能需要对绑定的一个确定的分类进行解析。例如，你正在构建一个报告聚合器，它接受一个包含不同报 `Report` 接口实现的数组。在注册了 `Report` 实现后，就可以用 `tag` 方法为他们分配一个标签：

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

一旦服务被打上标签，就可以轻松的使用 `tagged` 方法对他们进行解析：

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="resolving"></a>
## 执行解析

<a name="the-make-method"></a>
#### `make` 方法

你可以在容器外使用 `make` 方法对类实例进行解析。`make` 接受一个参数，可传入希望解析的类名或接口名：

    $api = $this->app->make('HelpSpot\API');

如果代码所在位置无法访问 `$app` 变量，可以使用 `resolve` 全局帮助器：

    $api = resolve('HelpSpot\API');

<a name="automatic-injection"></a>
#### 自动注入

另外，也是更重要的一点，可以简单的在类的构造方法中对依赖进行"类型提示"从而使容器能够对其解析，这包括 [controllers](/docs/{{version}}/controllers)、[event listeners](/docs/{{version}}/events)、[queue jobs](/docs/{{version}}/queues)、[middleware](/docs/{{version}}/middleware)等。在实践中，这也是容器解析对象的最常见方式。

例如，你可能会对控制器的构造方法定义一个 repository 类型提示。这个 repository 将会自动被解析并注入到类中：

    <?php

    namespace App\Http\Controllers;

    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the user with the given ID.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="container-events"></a>
## 容器事件

`service container` 每次解析一个对象就会激发一个事件。可使用 `resolving` 方法监听此事件：

    $this->app->resolving(function ($object, $app) {
        // Called when container resolves object of any type...
    });

    $this->app->resolving(HelpSpot\API::class, function ($api, $app) {
        // Called when container resolves objects of type "HelpSpot\API"...
    });

如你所见，对象被解析时会被传送到回调函数，在对象被传送给使用者之前允许你设置任何附加属性。
