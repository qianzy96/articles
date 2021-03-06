# Monolog（5）－－Handler 不要亂玩 Processor 啊

昨天提到 `AbstractHandler` 會實作存在 `Processor` 的方法，但實質上 `AbstractHandler` 是不會使用 `Processor` 的。

Monolog 的設計是另外寫一個 `AbstractProcessingHandler` 來繼承 `AbstractHandler`，在裡面處理 `Processor`：

```php
protected function processRecord(array $record)
{
    if ($this->processors) {
        foreach ($this->processors as $processor) {
            $record = call_user_func($processor, $record);
        }
    }

    return $record;
}
```

實作 `AbstractProcessingHandler` 只要實作 `write` 方法即可。

其他都是因實作功能需求，把迭代 `Processor` 的任務放到 handle 的開始、中間、或是後面。

比方說 `BufferHandler` 是把所有記錄全都放到記憶體裡，直到程序結束後，再一次性的往外送。這種情況下，就會需要在實際 Handler 裡，拿到真正的 record 後，才能跑 Processor，再存 buffer。



