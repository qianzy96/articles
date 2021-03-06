# 分析 Lumen Application－－dispatch() 上篇

Lumen 在處理跟 Request 相關的程式，都放在 [RoutesRequests][] 這個 trait 裡，包括今天要看的 `dispatch()`

```php
public function dispatch($request = null)
{
    // 解析 Request，並取得 method 與 pathInfo
    list($method, $pathInfo) = $this->parseIncomingRequest($request);

    try {
        // 啟動程式，這跟 Laravel Application boot() 大同小異
        $this->boot();

        // 使用 pipeline 
        return $this->sendThroughPipeline($this->middleware, function () use ($method, $pathInfo) {
            // 取得所有 route 並看看對應的 method + pathInfo 會不會找得到
            if (isset($this->router->getRoutes()[$method.$pathInfo])) {
                // 找得到就處理它
                return $this->handleFoundRoute([true, $this->router->getRoutes()[$method.$pathInfo]['action'], []]);
            }

            // 找不到就交由 FastRoute 處理
            return $this->handleDispatcherResponse(
                $this->createDispatcher()->dispatch($method, $pathInfo)
            );
        });
    } catch (Exception $e) {
        return $this->prepareResponse($this->sendExceptionToHandler($e));
    } catch (Throwable $e) {
        return $this->prepareResponse($this->sendExceptionToHandler($e));
    }
}
```

這段程式碼，除了 FastRoute 外，其他都是之前有分析過的橋段，如 `boot()` 裡，會對所有 [ServiceProvider][Day05] 呼叫 `boot()`；或是 [Pipeline][Day07] 操作等。

這段程式碼，其中有幾個地方是我們可以再深入追下去的，如：

* `parseIncomingRequest()` 如何解析出 method 或 pathInfo，以及後面為何 `$this->router->getRoutes()` 有可能會找不到
* `handleFoundRoute()` 如何處理找到的 route
* `createDispatcher()` 如何產生 FastRoute，以及 `handleDispatcherResponse()` 如何使用
* `sendExceptionToHandler()` 如何產生
* `prepareResponse()` 如何建置 Response

## parseIncomingRequest()

```php
protected function parseIncomingRequest($request)
{
    // 如果 request 是 null，就使用 Lumen Request 依環境建置出來
    if (! $request) {
        $request = LumenRequest::capture();
    }

    // 建完立馬存 container，prepareRequest() 下面會提到
    $this->instance(Request::class, $this->prepareRequest($request));

    // 解析出 Request 後，就能輕鬆拿到 method 或 pathInfo 了
    return [$request->getMethod(), '/'.trim($request->getPathInfo(), '/')];
}

protected function prepareRequest(SymfonyRequest $request)
{
    // 如果 request 是 SymfonyRequest，則重建一個新的 Laravel Request
    if (! $request instanceof Request) {
        $request = Request::createFromBase($request);
    }

    // 上面要這樣做的原因是，這裡要設定 UserResolver 與 RouteResolver
    $request->setUserResolver(function ($guard = null) {
        return $this->make('auth')->guard($guard)->user();
    })->setRouteResolver(function () {
        return $this->currentRoute;
    });

    return $request;
}
```

從原始碼可以知道，從 Request 即可拿到很多資訊了。

## handleFoundRoute()

有了 `$routeInfo`，就能拿來做處理，它的長像如下：

```php
$routeInfo = [true, $this->router->getRoutes()[$method.$pathInfo]['action'], []]
```

而 `handleFoundRoute()` 的原始碼：

```php
protected function handleFoundRoute($routeInfo)
{
    // 注意，這裡的 currentRoute 屬性，剛好是上面 RouteResolver 的回傳
    $this->currentRoute = $routeInfo;

    // 只是搞不懂這裡為何又要再重覆一次設定 RouteResolver，呼叫的順序確認過一定會是 parseIncomingRequest() 後才 handleFoundRoute()
    $this['request']->setRouteResolver(function () {
        return $this->currentRoute;
    });

    // 這會從 Router 取得對應 Route 裡，拿到 action
    $action = $routeInfo[1];

    // Pipe through route middleware...
    if (isset($action['middleware'])) {
        // 有 middleware 的話，就集合起來
        $middleware = $this->gatherMiddlewareClassNames($action['middleware']);

        // 使用另外一組 Pipeline 處理
        return $this->prepareResponse($this->sendThroughPipeline($middleware, function () {
            return $this->callActionOnArrayBasedRoute($this['request']->route());
        }));
    }

    // 沒 middleware 就直接處理
    return $this->prepareResponse(
        $this->callActionOnArrayBasedRoute($routeInfo)
    );
}
```

這裡使用另一組 Pipeline 的方法與 Laravel 是一樣的，可以參考[解析 Middleware 的實作細節][Day20]。

而最後實際呼叫的方法寫在 `callActionOnArrayBasedRoute()` 裡。

```php
protected function callActionOnArrayBasedRoute($routeInfo)
{
    $action = $routeInfo[1];

    // 有 uses 代表是 Controller，使用 Controller 的方法呼叫
    if (isset($action['uses'])) {
        return $this->prepareResponse($this->callControllerAction($routeInfo));
    }

    // 如果有找到 Closure 就 bind 到 RoutingClosure
    foreach ($action as $value) {
        if ($value instanceof Closure) {
            $closure = $value->bindTo(new RoutingClosure);
            break;
        }
    }

    try {
        // 使用 Container::call() 呼叫 Closure
        return $this->prepareResponse($this->call($closure, $routeInfo[2]));
    } catch (HttpResponseException $e) {
        return $e->getResponse();
    }
}
```

今天先告一段落，明天繼續看剩下三個方法。

[RoutesRequests]: https://github.com/laravel/lumen-framework/blob/v5.7.6/src/Concerns/RoutesRequests.php

[Day05]: day05.md
[Day07]: day07.md
[Day20]: day20.md
