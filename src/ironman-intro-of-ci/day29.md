# 有了 CI Server，然後呢？

一開始了解了概念，到後面做自動化測試與 CI Server 環境建置，都是為了要達成 [CI 的精神][Day 2]，努力至今，也完成了一些成果。

那麼，再來呢？

我們可以分成兩個層面來思考

* 有了 CI Server，那它能幫助團隊什麼嗎？
* 有了 CI Server，那還有什麼更潮的嗎？

> 以下 CI Server 簡稱 CI。

## CI 能幫助團隊什麼？

CI 厲害的地方在於，它隨時都在背後看著版控系統，只要一提交程式碼，它就會照指示去做 build，絕不偷懶！

或許有人會覺得奇怪，之前提到[開發人員好習慣][Day 5]時，有提到要 build 通過才提交，那幹嘛還要多此一舉，叫 CI 做一樣的事？

### 沒錯就沒事，有錯就很有事了

提幾個真實串接 CI 後發生的事：

1.  開發人員遵守好習慣，先本機測過才提交，但某次提交還是失敗了。

    其實只是忘了檢查 Coding Style。雖然這問題不大，但沒有人能保證下次不會忘了測重要功能。

2.  開發人員遵守好習慣，先本機測過才提交，但某次提交還是失敗了。

    因為 CI 環境某些條件與本機不一致，這代表建置 CI 環境的方法跟本機不一致。
    * 如果 CI 環境是依照上線環境所建立的，那代表本機測試通過是假的，要盡快修正
    * 如果是開發環境跟上線環境一樣，那代表之前 CI 測試通過都是假的，要盡快修正。
    * 如果不是很確定的話，別傻了，快點修正！這代表本機跟 CI 測的都是假的！

在提交程式的階段就會知道這些問題了，而不會讓問題延伸到上線，這價值非常之高呀！

### CI 絕不偷懶

提交程式前能做完整的 build，當然是最好的。但是越完整的 build，要花的時間就越長，對大部分的開發人員來說，應該都很討厭等待。而且太晚提交，等於無法常常跟其他人整合測試碼，這也是不大好。

一個常見的做法是，把驗證切分成快速 build 和完整 build。開發人員使用快速 build 驗證過一輪，再由 CI 執行完整 build，這樣就不會花太多時間在等待 build 結束才能提交程式碼了。

## 還有什麼更潮的嗎？

CI 的目標是，能讓提交程式到整合完成是快速且可以常常執行的，並在最後**交付高品質軟體**。拿到軟體之後，需要做部署才能呈現給真實的使用者，這實在是太麻煩了。這時，開發人員的驕傲就是[懶][Day 8]嘛！要是拿到軟體之後，能自動部署某個即有環境的話，那就太棒了。

這是 *Continuous Delivery* / *Continuous Deployment* 想達成的目標，簡稱 *CD*。

CD 的需求跟所要考慮的事跟 CI 大部分都很類似，但有個要考慮的情境是大不相同的：CI 的重點在整合，也就是元件組合起來是能動的，因此通常測元件會在一個**用過即丟的環境裡執行**；CD 的重點是交付，但大部分的系統都會有資料庫要記錄使用者狀態（如登入功能），因此如何在**既有系統上更新**，同時將影響使用者的範圍縮到最小，這是 CD 所關注的重點。

---

## 今日回顧

有了 CI，團隊會有機會發現更多意想不到的問題。記得，問題只要越早被發現，越早去解決掉，都是非常有價值的。

好 CI，不串嗎？

[Day 2]: day02.md
[Day 5]: day05.md
[Day 8]: day08.md
