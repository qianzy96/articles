# 分析 Routing（1）

今天總算來到了重頭戲－－[Routing][]，也就是負責決定什麼樣的網址要傳到指定的 controller。

Routing 的類別又比 [Session][Day10] 來得更多，而且複雜很多，不過就讓我們一天一天來看吧。

## RoutingServiceProvider

跟 Session 一樣，我們先從 service provider 開始看起。之前有提過，[Application][Day05] 一開始會載入幾個 service provider，其中一個就是 [`RoutingServiceProvider`](https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Routing/RoutingServiceProvider.php)：

```php
public function register()
{
    $this->registerRouter();
    $this->registerUrlGenerator();
    $this->registerRedirector();
    $this->registerPsrRequest();
    $this->registerPsrResponse();
    $this->registerResponseFactory();
    $this->registerControllerDispatcher();
}
```

仔細看裡面的實作，會知道註冊幾個類別如下：

    'router' => 'Illuminate\Routing\Router'
    'routes' => 'Illuminate\Routing\RouteCollection'
    'url' => 'Illuminate\Routing\UrlGenerator'
    'redirect' => 'Illuminate\Routing\Redirector'
    'Psr\Http\Message\ServerRequestInterface' => 'Zend\Diactoros\Request'
    'Psr\Http\Message\ResponseInterface' => 'Zend\Diactoros\Response'
    'Illuminate\Contracts\Routing\ResponseFactory' => 'Illuminate\Routing\ResponseFactory'
    'Illuminate\Routing\Contracts\ControllerDispatcher' => 'Illuminate\Routing\ControllerDispatcher'

除了 PSR request & response 與 ControllerDispatcher 之外，其他都是相互有關係的（為求版本精簡，把 `Illuminate\Routing\` 省略）：

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuGejod5FpKijIYp9BrB8rzLL2CelBKbrpi_9IKqkoSpFumfAJSai0hAXqiZgWNB51VbvnQbkYI3vIh02X7eki55OJRLIo4ijLyZC0peZBpcLoo4rBmNe6000)

    @startuml
    UrlGenerator o-- RouteCollection
    Redirector o-- UrlGenerator
    ResponseFactory o-- Redirector
    Router o-- RouteCollection: new instance
    @enduml

也因為這樣「食物鏈」的關係，所以會發現 `RouteCollection` 的初始化是建構 `UrlGenerator` 時處理的。

搭配上面註冊列表，可以知道這幾個實例，對應到 Laravel 開發者很熟的幾個函式或 Facade：

* `Route` Facade 對應 Router
* `action()`、`asset()`、`route()`、`url()` 函式與 `URL` Facade 對應 UrlGenerator
* `back()`、`redirect()` 函式與 `Redirect` Facade 對應 Redirector
* `response()` 函式與、`Response` Facade 對應 ResponseFactory

這些函式都定義在 Foundation 裡的 `helpers.php`，而 Facade 則是放在 `Support/Facades` 裡。

> Facade 後面會再說明細節，這裡先知道 Facade 也是對應 [Application][Day05] 註冊的某個特定實例

但從這些註冊的過程，還無法知道哪時候被拿來執行。事實上，執行的時機點，是在 Laravel 的 [`App\Providers\RouteServiceProvider`](https://github.com/laravel/laravel/blob/v5.7.0/app/Providers/RouteServiceProvider.php) 裡：

```php
protected $namespace = 'App\Http\Controllers';

public function boot()
{
    parent::boot();
}

public function map()
{
    Route::prefix('api')
         ->middleware('api')
         ->namespace($this->namespace)
         ->group(base_path('routes/api.php'));

    Route::middleware('web')
         ->namespace($this->namespace)
         ->group(base_path('routes/web.php'));
}
```

> 為說明方便，這是沒有 extract method 的內容。

這裡的程式碼，很明確寫著如何載入 routes 裡面的設定，因此 `map()` 方法是在什麼時候被呼叫就是關鍵了。實際如何使用 `map()` 方法的程式，是寫在它的父類別 [`Illuminate\Foundation\Support\Providers\RouteServiceProvider`](https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Foundation/Support/Providers/RouteServiceProvider.php)，它沒有實作 `register()`，而是實作 [`boot()`](https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Foundation/Support/Providers/RouteServiceProvider.php#L29-L43)：

```php
public function boot()
{
    // 設定 namespace 的根
    $this->setRootControllerNamespace();

    // 看 cache 有沒有 routes 設定
    if ($this->app->routesAreCached()) {
        // 如果 cache 找得到 routes 設定的話就從 cache 載入
        $this->loadCachedRoutes();
    } else {
        // 找不到就載入
        $this->loadRoutes();

        // 註冊 app boot 事件
        $this->app->booted(function () {
            // 執行 RouteCollection 的 refreshNameLookups() 與 refreshActionLookups() 方法
            $this->app['router']->getRoutes()->refreshNameLookups();
            $this->app['router']->getRoutes()->refreshActionLookups();
        });
    }
}
```

`loadRoutes()` 非常簡單，就是呼叫 `map()`。它使用 [`Application::call()`][Day04]，會把 `map()` 方法定義的依賴都注入。因為這裡已經是 `boot()` 階段了，所有註冊都已經完成，因此不會有找不到依賴的問題。

後面的 `refreshNameLookups()` 與 `refreshActionLookups()` 方法看實作是在把裡面的 name mapping 與 action mapping 重新整理，目前的資訊還無法知道為何要執行。

接著繼續看預設的 `map()` 做的事：

```php
Route::prefix('api')
     ->middleware('api')
     ->namespace($this->namespace)
     ->group(base_path('routes/api.php'));

Route::middleware('web')
     ->namespace($this->namespace)
     ->group(base_path('routes/web.php'));
```

這裡使用了 Facade 來存取 Router 實例，可以想像它其實等同於下面的程式碼：

```php
$this->app->make('router')
     ->prefix('api')
     ->middleware('api')
     ->namespace($this->namespace)
     ->group(base_path('routes/api.php'));

$this->app->make('router')
     ->middleware('web')
     ->namespace($this->namespace)
     ->group(base_path('routes/web.php'));
```

最一開始在分析 RouteServiceProvider 時就有發現，router 是對應 `Illuminate\Routing\Router` 的實例，因此這裡可以知道 Router 實例是最一開始被拿出來使用的。

今天先到此，明天會以 Router 類別為核心來分析 Routing 元件。

[Routing]: https://github.com/laravel/framework/tree/v5.7.6/src/Illuminate/Routing

[Day04]: day04.md
[Day05]: day05.md
[Day10]: day10.md
