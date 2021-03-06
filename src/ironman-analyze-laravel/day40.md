# 再看 tap()

`tap()` 之前有提過，是 [helpers.php][] 的方法之一：

```php
function tap($value, $callback = null)
{
    if (is_null($callback)) {
        return new HigherOrderTapProxy($value);
    }

    $callback($value);

    return $value;
}
```

先不管 HigherOrderTapProxy，來看剩下的原始碼：

```php
function tap($value, $callback)
{
    $callback($value);

    return $value;
}
```

它需要傳入一個 $value，然後它會再回傳出來，因此可以知道下面這個寫法是可行的：

```php
tap(new Collection(), $callback)->each->pay();
```

再來因為它中間有做 `$callback($value)`，因此上面這個方法的全貌可能會是長這樣的：

```php
$callback = function($collection) {
    $collection->set(new Invoice);
};

tap(new Collection(), $callback)->each->pay();
```

反過來看，如果沒有 tap() 函式的話，我們可能需要這樣寫：

```php
$collection = new Collection();
$collection->set(new Invoice);

$collection->each->pay();
```

咦，看起來似乎不用 tap() 寫起來比較乾淨。不是這樣的，是 Collection 並不適合用在這個地方。

最常看到的就是在物件初始化的時候，比方說[分析 Session][Day11] 提到了下面這段程式碼：

```php
protected function startSession(Request $request)
{
    return tap($this->getSession($request), function ($session) use ($request) {
        $session->setRequestOnHandler($request);

        $session->start();
    });
}
```

使用 `tap()` 的好處之一是剛剛有提到的，這對要使用串聯方法是有利的；另一個好處則是：有時候我們會希望對某個實例做某些事，而做這些事會需要產生一些專用的暫時變數，這時 tap() 因為可以使用 Closure，所以可以把這些暫存變數「關」在裡面，外面就不會被這些暫時變數干擾到。

## HigherOrderTapProxy

雖然是這樣，但每次要寫一堆 callable 就很煩，因此出現了另一個選擇：[HigherOrderTapProxy][]，來看看它的原始碼：

```php
// 前面只是建構的時候把 value 存到 target 而已，所以省略
public function __call($method, $parameters)
{
    $this->target->{$method}(...$parameters);

    return $this->target;
}
```

這很像 proxy pattern，唯一不同的地方在於，它固定會回傳 self。以 [分析 Log][Day21] 的例子來說，原本程式碼與改寫後的程式碼如下：

```php
tap($this->createEmergencyLogger(), function ($logger) use ($e) {
    $logger->emergency('Unable to create configured logger. Using emergency logger.', [
        'exception' => $e,
    ]);
});

tap($this->createEmergencyLogger())
    ->emergency('Unable to create configured logger. Using emergency logger.', [
        'exception' => $e,
    ]);
```

跟 [Higher Order Messages][Day39] 很像，可以省略掉一層 callback，但同時也有一樣的使用條件：以 callback 的寫法，只允許一行程式碼。

[helpers.php]: https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Support/helpers.php
[HigherOrderTapProxy]: https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Support/HigherOrderTapProxy.php

[Day11]: day11.md
[Day21]: day21.md
[Day39]: day39.md
