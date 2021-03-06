# Inspection

前面提到了非常多種測試方法。那程式碼都測過了，是要檢查什麼東西？

## 簡介

依照測試方法，可以分成**動態測試**與**靜態測試**。動態測試正如其名，是指程式在執行的時候所做的測試。靜態測試則是程式碼不執行的時候執行的測試。

或許會覺得奇怪，程式碼跑了才會知道結果，這才叫測試啊！那可以先思考以下問題：

* 假設寫好的原始碼給團隊成員看，而大部分的團隊成員都難以理解，這程式需不需要修改？
* 如果需要修改，那修改前與修改後，跑程式碼的測試結果會希望有所改變嗎？

通常大部分團隊，第一個答案通常會是「需要修改」的，第二個答案是會希望「不變」。在測試結果希望不變的前提下，又需要修改，代表它隱含著另一種檢查行為。是的，這就是 Inspection，也是一種只看原始碼的靜態測試。

## 種類

Inspection 有非常多種檢查項目，大部分的做法是分析原始碼，然後取得一些數據並產生報表。因為是單純分析原始碼，所以 Code Review 算是一種人工靜態測試。這類數據通常沒有標準，所以雖然可以人工檢查，但使用工具分析出來的結果會相對比較客觀。

### Coding Style

我想 Code Review 最常看的，應該就是 Coding Style 了。Coding Style 通常會有固定要遵守的規則，同時這也正是電腦最擅長的－－重複固定的檢查，因此這一類的檢查工具非常多，甚至有規則樣版與自動修正功能。PHP 最常見的就是 [PHP_CodeSniffer][]，檢查、規則樣版、自動修正的功能都有。

### PMD

[PMD][] 是原始碼分析工具，它支援多種語言，包括 PHP。裡面有許多有用的檢查如：

* Dead Code: 它可以檢查出未使用的變數、參數、或是方法等。這些程式通常會是開發人員忘了刪除較多，但相反的，也有可能是有要使用卻忘了用，這問題就會比較大。
* Possible bugs: 任何有可能會產生 bug 的地方，如空的 try catch / if else / switch block，極有可能是忘了寫條件處理的方法。
* Duplicate code: 檢查哪裡有重複程式碼，這些程式都是有可能可以抽出來變成一個 class 或 method 的。

還有很多其他的可以參考，但這三個就能幫助找到非常多程式設計上的問題了。

### 物件依賴檢查

物件依賴檢查如 [PHP Depend][]，它是檢查物件耦合的程度。當然一包原始碼一定會有耦合，耦合狀況正常就不會有太大問題。不正常如太多物件依賴某個物件，或是某個物件依賴太多物件，都不是很正常。如，某個物件被太多物件依賴，表示它壞了其他人會跟著壞，那大家只能祈禱作者有寫完整的測試。

### 循環複雜度

如何定義某一個 function 複不複雜？有一個方法：看程式會走的路徑是不是很多。換句話說，是看程式裡的判斷語法如 `if` 之類的語法，是不是有非常多種可能性。有多個可能性，代表需要多個測試來涵蓋它。越多的測試當然寫起來就越複雜。循環複雜度正是在描述路徑的多寡，這很適合當程式複雜度的標準，

## 今日回顧

動態測試與靜態測試是相輔相成的。比方說，當靜態測試發現某段程式複雜度高，又很多程式依賴，那表示這段程式被改壞的機率很高。我們可以先為該程式寫動態測試，再開始修改。

## 相關連結

* [PHP_CodeSniffer][] | GitHub
* [PMD][] | PMD
* [Robustness Testing](https://en.wikipedia.org/wiki/Robustness_testing) | 維基百科

[PHP_CodeSniffer]: https://github.com/squizlabs/PHP_CodeSniffer
[PMD]: https://pmd.github.io/
[PHP Depend]: https://pdepend.org/
