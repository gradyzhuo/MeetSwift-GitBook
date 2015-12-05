# 第四章 - 揭開 Optional 的神秘面紗，又愛又恨的 ? !

## Swift是一個**安全**的語言
由於在所有語言中，一個nil值的出現，可以輕易的把程式crash，雖然Objective-C可以對一個nil執行method，而不產生錯誤，但是否物件是否為nil的狀況不明，這也導致了debug時的不便。

因此，在Swift中，所有的變數在賦值時是不能 nil 的喔。

另外，不同於C與多數語言會將NULL導向一個空的記憶體位置，或使用一個數值做為NULL的代表，但 Swift 的 nil 並不存在於真實的記憶體位置，就跟**class**這個詞一樣，只是個**保留字**。


## 但變數還是有 nil 的需求，不是?

Optional其實不是新觀念，在C#與語多新興語言都看得到他的蹤跡。也有許多人在其他語言上實作了相關的概念，

節取自 Apple 在官方文件中的定義：
    
    * Optionals say either "there is a value, and it equals x" 
    or "there isn’t a value at all".
    * Optionals are "safer and more expressive than nil pointers in Objective-C"
  
說明一下：
**Optional** 是在 Swift 中，做為處理可能存在nil值變數的機制，目的是為了減少參數在傳遞過程，可能存在nil的不確定性，並可以立即明確地處理 nil 情況，並且可以在Compiler階段就判斷 nil 的處理是否合法，以減少 App Crash 的機會。

---
## 所以Optional到底是什麼?
- Optional 是一個enum，存在兩種可能：
    - Some : 有值存在
    - None : 無值存在

- Optional的定義宣告截錄：

```swift
    public enum Optional<Wrapped> : _Reflectable, NilLiteralConvertible {
        case None
        case Some(Wrapped)
        
        //Other Methods
    }
```
從 **Optional** 的宣告中，不難發現 **Optional** 使用的是就是在第一章中enum所提到的**Associated Values**的用法。

## 怎麼使用？
從前幾章所學，我們可以知道一個泛型的Associated Values的Enum如何使用，因此Optional可以透過以下的語法進行宣告。
```swift
let optionalString = Optional.Some("123") // Type: Optional<String>
```

