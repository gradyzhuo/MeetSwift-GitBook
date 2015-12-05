# 第四章 - 揭開 Optional 的神秘面紗

## 一切都要從 nil 與 crash 開始說起
在多數的語言中，一個nil值的出現，可以輕易的讓程式產生錯誤，進而被系統強制關閉。一般使用者叫他閃退；開發者叫他崩潰，如何防止nil造成的錯誤，各式各樣的檢查早已不可或缺，但無論如何防堵，nil總是在程式運行階段(runtime)才會發生，無法在編譯(Compile)就檢查出nil發生問題的可能性，也沒有穩定的規則可以找出nil的出沒之處。

雖然Objective-C有一個有趣的特性，可以對一個nil執行method，而不產生錯誤，也不會crash。

1. 這只適用於呼叫method的時候。
2. 物件是否為nil的狀況不明，如果真的發生nil執行method的情況，開發者也無從得知。
3. 功能與預期行為不符時，會很難發現是什麼地方造成的。

因此，以**安全**做為slogan的Swift設計成所有的變數在賦值時只能有值，不能是nil，只要接受到nil，就會拋出錯誤。

## What? 但變數還是有 nil 的需求，不是?

Swift從根本導入了Optional，但其實 Optional 其實不是Swift獨有的新特性，在C#與語多新興語言都看得到他的蹤跡。也有許多人在其他語言上實作了相關的概念，但不同於一些語言下採用Monad實作的Optional Pattern
([Monad Design Pattern in Java](http://ingramchen.io/blog/2014/11/monad-design-pattern-in-java.html))， Swift 導入的是語法上根本的機制。

```swift
let intValue:Int = 0 // 合法
let intValue2:Int = nil //不合法
let optionalIntValue:Int? = nil //合法，這是一個Int的Optional
```

## 所以Optional到底是什麼?

**Optional** 是在 Swift 中，做為處理可能存在nil值變數的機制，目的是為了減少參數在傳遞過程，可能存在nil的不確定性，可以立即明確地處理 nil 發生時的情況，並且可以在Compiler階段就判斷 nil 的處理是否合法，以減少 App Crash 的機會。

**Optional**的定義宣告(這裡只取Optional完整宣告的節錄)：

```swift
    public enum Optional<Wrapped> : _Reflectable, NilLiteralConvertible {
        case None
        case Some(Wrapped)
    }
```
從 **Optional** 的宣告中，不難發現 **Optional** 使用的是就是在第一章中enum所提到的**Associated Values**的用法，並搭配了第三章的泛型，並取名泛型型別為**Wrapped**，存在兩種可能的case：

1. None : 無值存在
2. Some : 有值存在

## Optional 就像是一個會爆炸的包裹
第一章在講**Enum**的時候，就有提過**Associated Values Enum**像是一個**容器**。
我們如果用現實生活中來解釋Optional，最適合的莫過於**包裏**了。

一般說來，**包裏**有兩種狀態，並帶一個說明內容物的標籤(tag)：
![Package](images/ch4/0.png)

1. 裡面沒放東西
2. 裡面放了一台iPad

如果用Swift語法來說明就是以下這麼一回事了：

```swift
struct iPad {
	//iPad 的struct
}

let 一台iPad = iPad()
let 禮物包裏:Optional<iPad> = Optional.Some(一台iPad)
let 沒有東西的包裏:Optional<iPad> = Optional.None
```

那一般看到在**型別**後面的 **?**，如 **Int?** 又是怎麼一回事呢？

Swift 的 **語法糖衣** 非常的多，語法糖衣指的是透過更簡易的語法，來達成另一個繁瑣語法的方式. 

以Optional來說，**?** 就是Optional的語法糖衣之一，由上面的例子繼續，可以代換如下寫法：

```swift
let 一台iPad = iPad()
let 禮物包裏 : iPad? = 一台iPad
let 沒有東西的包裏:iPad? = nil
```
由上面的例子可以發現，**iPad?** 指的就是 Optional\<iPad>，另外還簡化的賦值的語法，可以不用再寫 **Optional.Some(…)** 或 **Optional.None**。

## 這有什麼好處呢？
如同Apple的解釋，節錄自 Apple 在官方文件中的定義：

    * Optionals say either "there is a value, and it equals x"
    or "there isn’t a value at all".
    * Optionals are "safer and more expressive than nil pointers in Objective-C"

**Optional**會**主動描述**這個物件是否存在**nil**的可能性，因此可以讓開發者在第一時間反應出現**nil**時要進行處理。這樣就不會發生看到黑影也開槍的窘境，不用每個變數都要檢查是否為nil，只有在出現Optional的情況下，才需要檢查，也可以減少判斷時所造成的效能成本。

#### 放入Optional的過程，術語又叫 Warpped，也就是被*包*起來的意思。


## 裝箱後，如何拆箱?

如果你接收到別人的禮物，那你第一件事是什麼？ **拆禮物**對吧? 

其實**拆禮物**也同樣有兩種情況：

1. 有禮物：你放下了，拿著iPad去使用
2. 沒禮物：你暴走了，把對方打成不成人形(？！

是的，拆禮物本來就會有暴走的風險，Optional也一樣，當你試著取出Optional裡面的值，也是要承擔crash的風險。 雖然Optional已經可以減少大多數的 nil 判斷，但遇到Optional拆箱的時候到底怎麼拆？要注意什麼呢？

#### 從Optional取值的過程，術語叫 Unwarpping，也就是*解包*的意思。

### 方法一：使用美工小刀
> 短小精悍，但會直接暴走

![Unwrapped](images/ch4/1.png)

Swift中，取出Optional裡面的值，具體的機制過程如下：

```swift
//繼續延續上面的例子
//定義一個 拆開包裏 的泛型function
func 拆開包裏<包裏的內容物>(包裏:Optional<包裏的內容物>)->包裏的內容物 {
    switch 包裏 {
    case .None:
        fatalError("么受喔！裡面沒東西，是要氣死誰？")
    case let .Some(內容物):
        return 內容物
    }
}

let iPad = 拆開包裏(禮物包裏) //拿到iPad
let 沒東西 = 拆開包裏(沒有東西的包裏) //Crash，么受喔！裡面沒東西，是要氣死誰？

```

由上面的程式碼不難發現，如果**包裏**是一個.None的case，那就會執行**fatalError("么受喔！裡面沒東西，是要氣死誰？")**，也就是引發crash。

但不可能每次都要讓開發者自已撰寫這個過程，因此，Apple也貼心的提供了 **!** 這把**美工小刀**，同樣的，這也是**語法糖衣**。

只要在Optional的wrappedValue後面，劃上一刀(加上**!**)，就會執行類似我上面撰寫的邏輯，並拆箱完成，但如果遇到nil的情況下，就會觸發Crash。

```swift
let iPad呦 = 禮物包裏! //美而短小的美工小刀
let 沒東西呦 = 沒有東西的包裏! //crash
```

### 方法二：交給朋友檢查
> 較迂迴，但可以避免沒有禮物還硬拆的狀況

如果不要讓收禮物的人暴走，那也可以找一個人幫忙檢查有沒有真的有禮物，有或沒有，就先跟收禮物的人報備一下，再決定要不要拆。什麼意思呢？ 意思就是先用if判斷一下，如果有再拆，再做事。

```swift
if 禮物包裏 != nil {
	let 有iPad = 禮物包裏!
	//拿iPad玩
}

if 沒有東西的包裏 != nil {
	// 不會執行這裡
}else{
	// 這裡會執行
}
```

### 方法三：交給第三方信託拆禮物
> 較迂迴，但可以從第三方拆完直接拿到禮物，不用自已拆
 
Swift 導入了一種 if let的語法，可以判斷是否是nil以外，也可以直接賦值到另一個變數上，可以看做是上面寫法 if 和 let 的合併，另外，此變數的作用域(Scope)與方法二一樣，只在if的{ }裡有效。
 
```swift 
if let iPad耶 = 禮物包裏 {
	//拿iPad玩
}

if let 沒有東西耶 = 沒有東西的包裏 {
	// 不會執行這裡
}else{
	// 這裡會執行
}
```

### 方法四：把自已變成具現化系(？
> 可以提供nil時，符合邏輯的初始值，但如果用不對，之後會找不太到問題點。

這個機制就是，如果沒東西，就自已產生一個假的給他，有點像是給他一個初使值。


```swift 
let 任何包裏 = 禮物包裏 ?? iPad() // 這裡會拿到禮物
let 任何包裏2 = 沒有東西的包裏 ?? iPad() // 這裡會拿到iPad()的實體
```



## 補充
1. 不同於C與多數語言會將NULL導向一個空的記憶體位置，或使用一個數值做為NULL的代表，Swift 的 nil 並不存在於真實的記憶體位置，就跟**class**這個詞一樣，只是個**保留字**。 
