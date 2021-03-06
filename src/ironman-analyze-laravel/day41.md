# Lumen 簡介

與 Laravel 類似地，Lumen 也是被拆成 [Lumen][] 與 [Lumen Framework][] 兩部分。而 Lumen Framework 則是使用[第一天][Day01]提到的 [Illuminate][] 套件實作出來的。

最近幾天會來研究一下 Lumen 怎麼使用 Laravel 抽離出來的套件，會使用 [v5.7.6](https://github.com/laravel/lumen-framework/tree/v5.7.6) 版，從 src 目錄可以知道 Lumen Framework 客製化的套件庫樣貌：

```php
Auth
Bus
Console
Exceptions
Http
Providers
Routing
Testing
```

> `Concerns` 與 `Providers` 筆者認為不大像是客製化的一部分，而比較像 helper

了解 Lumen 的做法後，就能知道如何使用 Laravel 提供的 Contract 與輪子，組合出一個自幹框架。

事實上，重頭戲還是會在 Routing，因為 Lumen 使用了 [`nikic/fast-route`](https://github.com/nikic/FastRoute) 作為解析路由器。如何取代原有 Routing 正是 Lumen 程式碼最值得看的地方。

> 今天只起個頭，休息一下，明天再繼續努力了。

[Lumen]: https://github.com/laravel/lumen
[Lumen Framework]: https://github.com/laravel/lumen-framework
[Illuminate]: https://github.com/illuminate

[Day01]: day01.md
