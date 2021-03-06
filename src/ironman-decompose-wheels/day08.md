# Faker（3）－－Base 類別中的基本方法

昨天有提到 `Generator` 有 `addProvider()` 方法，可以把各式各樣的 Provider 加入 Generator。而也有提到 Generator 的屬性或方法，會轉接到 Provider 提供的方法。

雖然沒有硬性規定，不過它的 Provider 都有繼承自 `Provider\Base` 的基礎類別，這個類別定義了很多產生假資料的基本方法，讓我們一起來看看。

## Random

基本隨機取樣的方法

```php
$generator = new Faker\Generator();
$base = new Base($generator);

echo $base->randomDigit() . "\n";
echo $base->randomNumber() . "\n";
echo $base->randomFloat() . "\n";
echo $base->randomLetter() . "\n";
echo $base->randomElement() . "\n";   // 預設 array 為 [a, b, c]
```

輸出效果如下：

```
2
49902
14820.4454
d
c
```

這些是最基本的隨機取樣方法，後面提到的其他 Provider 會依賴這些方法來取得隨機樣本。

## Shuffle

基本的洗亂方法：

```php
$generator = new Faker\Generator();
$base = new Base($generator);

$shuffledArray = $base->shuffleArray(['s', 't', 'r', 'i', 'n', 'g', ]);
$shuffledString = $base->shuffleString('string');

var_export($shuffledArray);
echo PHP_EOL;
var_export($shuffledString);
```

輸出效果如下：

```
array (
  0 => 't',
  1 => 'n',
  2 => 'g',
  3 => 'r',
  4 => 's',
  5 => 'i',
)
'rinstg' 
```

## 特殊符號轉化成亂數

特別的是，它有一些方法，可以把字串樣版裡的 *wildcard* 使用亂數取代，如：

```php
$generator = new Faker\Generator();
$base = new Base($generator);

$template = "取代範例：### %%% ??? ***\n";

echo $base->numerify($template);
echo $base->lexify($template);
echo $base->asciify($template);
echo $base->bothify($template);
```

輸出效果如下：

```php
取代範例：833 554 ??? ***
取代範例：### %%% rgv ***
取代範例：### %%% ??? p@u
取代範例：420 641 xju 26z
```

規則是：

* `#` 會用 `randomDigit()` 的結果取代
* `%` 會用 `randomDigitNot()` 的結果取代
* `?` 會用 `randomLetter()` 的結果取代
* `*` 會用 `randomAscii()` 的結果取代

不同的方法會取代不同的 wildcard，這是為了保留在不同場合使用適合方法的彈性。

* `numerify()` 只會取代 `#` 和 `%` 
* `lexify()` 只會取代 `?` 
* `asciify()` 只會取代 `*` 
* `bothify()` 會全取代 

## 正則亂數產生器

這個產生器很有趣，通常我們會使用正則來檢驗 Email 是否是正常的，如：

```php
preg_match('/[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}/', 'aa@bb.cc')
```

而 `regexify()` 這個方法是反過來，用正則來產生符合規則的假資料：

```php
$regex = '/[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}/';

echo $base->regexify($regex) . PHP_EOL;
echo $base->regexify($regex) . PHP_EOL;
echo $base->regexify($regex) . PHP_EOL;
```

輸出效果：

```php
d%363@m.ncqs
92@q.rjg
v2h2@d3l.pjwb
```

> 注意：官方 doc block 有提醒，這個方法會「非常地」慢。

---

今天先介紹 Provider 基本產假資料的方法，未來介紹其他 Provider 都會使用到這些方法來產生更多樣化的假資料。
