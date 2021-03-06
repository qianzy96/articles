# 簡單看看 TestResponse

[TestResponse][] 是一個輔助測試 response 用的物件，它內建混入（mixin）了 [Response][] 物件功能：

```php
use Macroable {
    __call as macroCall;
}

public function __call($method, $args)
{
    // 實作了 Macroable 的功能
    if (static::hasMacro($method)) {
        return $this->macroCall($method, $args);
    }

    // 混入 Response
    return $this->baseResponse->{$method}(...$args);
}

// 包括屬性也都能取得，不過只能取得 public 屬性
public function __get($key)
{
    return $this->baseResponse->{$key};
}
```

並提供了許多 assert 方法，如 `assertSuccessful()`：

```php
public function assertSuccessful()
{
    PHPUnit::assertTrue(
        $this->isSuccessful(),
        'Response status code ['.$this->getStatusCode().'] is not a successful status code.'
    );

    return $this;
}
```

這讓測試程式碼變得更加容易理解。

---

大部分的 assert 方法都是很單純的包裝 Response 的結果，少部分比較特別的如 View：

```php
// assert view 的名稱
public function assertViewIs($value)
{
    // 確定 Response 是 View
    $this->ensureResponseHasView();

    PHPUnit::assertEquals($value, $this->original->getName());

    return $this;
}

protected function ensureResponseHasView()
{
    // 透過 __get 取得 original，並確認它是 View
    if (! isset($this->original) || ! $this->original instanceof View) {
        return PHPUnit::fail('The response is not a view.');
    }

    return $this;
}
```

JSON 的 assert 設計也是非常厲害，如看是否有 JSON 片段 `assertJsonFragment()`：

```php
public function assertJsonFragment(array $data)
{
    // 將 response json 排序並 encode 回字串
    $actual = json_encode(Arr::sortRecursive(
        (array) $this->decodeResponseJson()
    ));

    // 依續把傳入的 data 做比對
    foreach (Arr::sortRecursive($data) as $key => $value) {
        // 產生預期的字串組
        $expected = $this->jsonSearchStrings($key, $value);

        // 如果有找到就是有
        PHPUnit::assertTrue(
            Str::contains($actual, $expected),
            'Unable to find JSON fragment: '.PHP_EOL.PHP_EOL.
            '['.json_encode([$key => $value]).']'.PHP_EOL.PHP_EOL.
            'within'.PHP_EOL.PHP_EOL.
            "[{$actual}]."
        );
    }

    return $this;
}

protected function jsonSearchStrings($key, $value)
{
    // 假設是 'a' 與 1，則會產生字串 `"a":1` 
    $needle = substr(json_encode([$key => $value]), 1, -1);

    return [
        $needle.']', // `"a":1]`
        $needle.'}', // `"a":1}`
        $needle.',', // `"a":1,`
    ];
}
```

[Response]: https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Http/Response.php
[TestResponse]: https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Foundation/Testing/TestResponse.php