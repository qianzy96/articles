# 分析 Facade

Laravel 的 Facade 是個很神奇的設計。使用的時候是靜態呼叫，但實質上是對某個實例呼叫。也因這個特性，所以有辦法做[測試替身（Test Double）][CI Day10]。

## 基本概念

[Facade][] 利用了 PHP 的幾個特性並搭配 Laravel 元件實作出來的，首先是 Magic Method，不一定要為類別定義方法，就能呼叫；再來是靜態方法可以被覆寫，讓靜態類別可以被繼承的；最後它跟 Container 是互相搭配的好夥伴。

剛有提到靜態方法可以覆寫，那個方法是 `getFacadeAccessor()`：

```php
protected static function getFacadeAccessor()
{
    throw new RuntimeException('Facade does not implement getFacadeAccessor method.');
}
```

這個方法預設會丟例外。意指如果子類沒有覆寫，將無法使用這個功能。

我們來看一個基本的範例 [Request][]：

```php
class Request extends Facade
{
    protected static function getFacadeAccessor()
    {
        return 'request';
    }
}
```

它繼承的實作是回傳了 `request` 字串。而當我們使用這個 Facade，如 `Request::ip()` 時，`__callStatic()` 即會被觸發：

```php
public static function __callStatic($method, $args)
{
    // 取得實例
    $instance = static::getFacadeRoot();

    if (! $instance) {
        throw new RuntimeException('A facade root has not been set.');
    }

    // 對實例呼叫方法，以 Request::ip() 為例，即會呼叫 $instance->ip()
    return $instance->$method(...$args);
}
```

再來就是看 `getFacadeRoot()` 是如何解析實例的：

```php
public static function getFacadeRoot()
{
    // 取得 accessor 並解析 Facade 實例
    return static::resolveFacadeInstance(static::getFacadeAccessor());
}

protected static function resolveFacadeInstance($name)
{
    // 如果已經是實例的話就回傳
    if (is_object($name)) {
        return $name;
    }

    // 如果已經被解析過就回傳
    if (isset(static::$resolvedInstance[$name])) {
        return static::$resolvedInstance[$name];
    }

    // 使用 static::$app 解析
    return static::$resolvedInstance[$name] = static::$app[$name];
}
```

這個 `$app` 正是剛剛提到要搭配的 Container，使用 `setFacadeApplication()` 即可設定。只要有 Container 就能建構所有實例。至於究竟什麼地方被呼叫到了？在[分析 bootstrap 流程][Day02]時，有一小段細節並沒有提到：[RegisterFacades][] 的實作：

```php
public function bootstrap(Application $app)
{
    // 清除所有已被解析過的實例，這是用在 feature 測試上
    Facade::clearResolvedInstances();

    // 就是這裡
    Facade::setFacadeApplication($app);

    // 設定 alias loader
    AliasLoader::getInstance(array_merge(
        $app->make('config')->get('app.aliases', []),
        $app->make(PackageManifest::class)->aliases()
    ))->register();
}
```

這下謎底全揭曉了，因此我們可以知道，下面這些程式碼得到的結果都是一樣的：

```php
Request::ip();
request()->ip();
app()->make('request')->ip();
```

一樣都是從 Container 取得 `request` 並呼叫對應的方法與回傳。

但需注意是，預設的 Facade 初始化的時機點在哪，像 Request 是在進 route middleware 之前才初始化的，global middleware 或在更之前，如設定 service provider 時，Container 還沒有存放實例，這時 Facade 的 magic 就會失效了。

另一個類別 AliasLoader 留到明天繼續討論。

[Facade]: https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Support/Facades/Facade.php
[RegisterFacades]: https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Foundation/Bootstrap/RegisterFacades.php
[Request]: https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Support/Facades/Request.php

[CI Day10]: /src/ironman-intro-of-cif-ci/day10.md

[Day02]: day02.md
