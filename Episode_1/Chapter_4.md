# 第四章 - 揭開 Optional 的神秘面紗(操作篇)

### 這章會講到：
1. 為什麼要有`Optional`
2. 怎麼使用`Optional`
3. 如何操作 `?` 和 `!`
4. 如果遇到一連串`Optional`的情況，我要不斷解開才操作嗎？


## 一切都要從 `nil` 與 `crash` 開始說起
在多數的語言中，一個`nil`值的出現，可以輕易的讓程式產生錯誤，進而被系統強制關閉。一般使用者叫他閃退；開發者叫他崩潰，如何防止`nil`造成的錯誤，各式各樣的檢查早已不可或缺，但無論如何防堵，`nil`總是在程式運行階段(runtime)才會發生，無法在編譯(Compile)就檢查出`nil`發生問題的可能性，也沒有穩定的規則可以找出`nil`的出沒之處。

雖然Objective-C有一個有趣的特性，可以對一個`nil`執行method，而不產生錯誤，也不會crash。

1. 這只適用於呼叫method的時候。
2. 物件是否為`nil`的狀況不明，如果真的發生`nil`執行method的情況，開發者也無從得知。
3. 功能與預期行為不符時，會很難發現是什麼地方造成的。

因此，以**安全**做為slogan的Swift設計成所有的變數在賦值時只能有值，不能是`nil`，只要接受到`nil`，就會拋出錯誤。

## What? 但變數還是有 `nil` 的需求，不是?

Swift從語法支援上導入了`Optional`，但其實 `Optional` 其實不是Swift獨有的新特性，在C#與語多新興語言都看得到他的蹤跡。也有許多人在其他語言上實作了相關的概念，但不同於一些語言下採用Monad實作的Optional Pattern
([Monad Design Pattern in Java](http://ingramchen.io/blog/2014/11/monad-design-pattern-in-java.html))， Swift 導入的是語法上根本的機制。

```swift
let intValue:Int = 0 // 合法
let intValue2:Int = nil //不合法
let optionalIntValue:Int? = nil //合法，這是一個Int的Optional
```

## 所以`Optional`到底是什麼?

`Optional` 是在 Swift 中，做為處理可能存在`nil`值變數的機制，目的是為了減少參數在傳遞過程，可能存在nil的不確定性，可以立即明確地處理 `nil` 發生時的情況，並且可以在Compiler階段就判斷 `nil` 的處理是否合法，以減少 App Crash 的機會。

`Optional`的定義宣告(這裡只取`Optional`完整宣告的節錄)：

```swift
    public enum Optional<Wrapped> : _Reflectable, NilLiteralConvertible {
        case None
        case Some(Wrapped)
    }
```
從 `Optional` 的宣告中，不難發現 `Optional` 使用的是就是在第一章中`enum`所提到的`Associated Values`的用法，並搭配了第三章的泛型，並取名泛型型別為`Wrapped`，存在兩種可能的`case`：

1. None : 無值存在
2. Some : 有值存在

## `Optional` 就像是一個會爆炸的包裹
第一章在講`enum`的時候，就有提過**Associated Values Enum**像是一個**容器**。
我們如果用現實生活中來解釋`Optional`，最適合的莫過於**包裏**了。

一般說來，**包裏**有兩種狀態，並帶一個說明內容物的標籤(tag)：
![Package](images/ch4/0.png)

1. 裡面沒放東西
2. 裡面放了一台iPad

如果用Swift語法來說明就是以下這麼一回事了：

```swift
struct iPad {
	//iPad 的struct
	var 版本:String
	var 使用者名字:String? = nil
	
	init(版本: String){
		self.版本 = 版本
	}
	
	init(版本: String,  使用者名字:String?){
		self.版本 = 版本
		self.使用者名字 = 使用者名字
	}
	
}

let 一台iPad2 = iPad(版本: "2",  使用者名字:"Grady Zhuo")
let 禮物包裏:Optional<iPad> = Optional.Some(一台iPad)
let 沒有東西的包裏:Optional<iPad> = Optional.None

//或是也可以用以下寫法
let 禮物包裏2:Optional<iPad> = Optional(一台iPad2)
let 沒有東西的包裏2:Optional<iPad> = Optional()
```

那一般看到在**型別**後面的 `?`，如 `Int?` 又是怎麼一回事呢？

首先，Swift 的 **語法糖衣** 非常的多，語法糖衣指的是透過更簡易的語法，來達成另一個繁瑣語法的方式. 

以`Optional`來說，`?` 就是`Optional`的語法糖衣之一，由上面的例子繼續，可以代換如下寫法：

```swift
let 一台iPad2 = iPad(版本: "2",  使用者名字:"Grady Zhuo")
let 禮物包裏 : iPad? = 一台iPad2
let 沒有東西的包裏:iPad? = nil
```
由上面的例子可以發現，`iPad?` 指的就是 `Optional<iPad>`，另外還簡化的賦值的語法，可以不用再寫 `Optional.Some(…)`、`Optional(...)` 或 `Optional.None`、`Optional()`。

## 這有什麼好處呢？
如同Apple的解釋，節錄自 Apple 在官方文件中的定義：

    * Optionals say either "there is a value, and it equals x"
    or "there isn’t a value at all".
    * Optionals are "safer and more expressive than nil pointers in Objective-C"

**Optional**會**主動描述**這個物件是否存在**nil**的可能性，因此可以讓開發者在第一時間反應出現**nil**時要進行處理。這樣就不會發生看到黑影也開槍的窘境，不用每個變數都要檢查是否為nil，只有在出現Optional的情況下，才需要檢查，也可以減少判斷時所造成的效能成本。

#### 放入Optional的過程，術語又叫 Warpped，也就是被*包*起來的意思。

## 裝箱後，如何拆箱?

如果你接收到別人的禮物，那你第一件事是什麼？ **拆箱子**對吧? 

其實**拆禮物**也同樣有兩種情況：

1. 有禮物：你拆開了，很開心的拿走裡面的禮物。
2. 沒禮物：你暴走了，把對方打成不成人形(？！

是的，拆禮物本來就會有暴走的風險，`Optional`也一樣，當你試著取出`Optional`裡面的值，也是要承擔crash的風險。 雖然`Optional`已經可以減少大多數的 `nil` 判斷，但遇到`Optional`拆箱的時候到底怎麼拆？又該注意什麼呢？我們有 4 + 1 種方式，以下我們會一一說明。

```
* 使用美工小刀
* 交給朋友檢查
* 交給第三方信託拆禮物
* 把自已變成具現化系
```

#### `小知識：從Optional取值的過程，術語叫 Unwarpping，也就是 解包 的意思。`

--
### 方法一：使用美工小刀
* 優點：快速，短小精悍
* 缺點：如果沒東西會直接暴走

![Unwrapped](images/ch4/1.png)

Swift中，取出`Optional`裡面的值，具體的機制過程如下：

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

由上面的程式碼不難發現，如果**包裏**是一個`.None`的`case`，那就會執行`fatalError("么受喔！裡面沒東西，是要氣死誰？")`，也就是引發crash。

但不可能每次都要讓開發者自已撰寫這個過程，因此，Apple也貼心的提供了 `!` 這把**美工小刀**，同樣的，這也是**語法糖衣**。

只要在`Optional`的wrappedValue後面，劃上一刀(加上`!`)，就會執行類似我上面撰寫的邏輯，並拆箱完成，但如果遇到`nil`的情況下，就會觸發Crash。

```swift
//如果是有東西的包裏
let iPad呦 = 禮物包裏! //美而短小的美工小刀
//如果是沒有東西的包裏
let 沒東西呦 = 沒有東西的包裏! //crash
```

### 方法二：交給朋友檢查
* 優點：可以避免沒有禮物還硬拆的狀況，也可以處理沒有東西的事情
* 缺點：比方法一迂迴得多

如果不要讓收禮物的人暴走，那也可以找一個人幫忙檢查有沒有真的有禮物，有或沒有，就先跟收禮物的人報備一下，再決定要不要拆。什麼意思呢？ 意思就是先用if判斷一下，如果有再拆，再做事。

```swift
//如果是有東西的包裏
if 禮物包裏 != nil {
	let 有iPad = 禮物包裏!
	//拿iPad玩
}

//如果是沒有東西的包裏
if 沒有東西的包裏 != nil {
	// 不會執行這裡
}else{
	// 這裡會執行
}
```

### 方法三：交給第三方信託拆禮物
* 優點：可以從第三方拆完直接拿到禮物，不用自已拆
* 缺點：比方法一迂迴得多，但比方法二簡潔一些
 
Swift 導入了一種 `if let`的語法，可以判斷是否是`nil`以外，也可以直接賦值到另一個變數上，可以看做是上面寫法 `if` 和 `let` 的合併，另外，此變數的作用域(Scope)與方法二一樣，只在`if` 的`{ }`裡有效。
 
```swift 
//如果是有東西的包裏
if let iPad耶 = 禮物包裏 {
	//拿iPad玩
}

//如果是沒有東西的包裏
if let 沒有東西耶 = 沒有東西的包裏 {
	// 不會執行這裡
}else{
	// 這裡會執行
}
```

### 方法三番外篇：交給第三方信託拆禮物，但Scope不同
除了 `if let` 還可以使用 `guard let`，`guard`跟`if`類似，也是做為條件判斷的保留字，但guard是一種強制條件成立的語法，如果條件不成立，必須強制離開作用的scope。

另外`guard let`與 `if let` 不同的另一點是，Unwrapped Value的Scope也不太一樣，之後在談到`guard`的時候，會再討論到這部分。

```swift
//一般if let的用法
func 一般if的狀況(optionalValue optionalValue: Int?){
	if let unwrappedValue = optionalValue {
		//有值就印出
		// 解開的值會在 if let 的 scrope裡面
		print("unwrappedValue:\(unwrappedValue)")
	}else{
		//如果 optionalValue 為 nil，就直接return 
		return
	}
}

//來看一下guard let 是怎麼用的
func guard的狀況(optionalValue optionalValue: Int?){
	guard let unwrappedValue2 = optionalValue else {
		//與if let不一樣，這個區塊是條件不成立時，會被迫return
		return
	}
	// 解開的值會在 guard let 的 scrope外面
	print("unwrappedValue2:\(unwrappedValue2)")
}
```

### 方法四：把自已變成具現化系(？

* 優點：可以提供`nil`時，直接提供符合邏輯的初始值
* 缺點：還是要小心給替換的值，如果給的值不太對的話，會跟`if...else...`一樣出現邏輯漏洞

這個機制就是，如果沒東西，就自已提供另一個值給他，有點像是給他一個初始值。

```swift 
let 一台iPad:iPad
//如果是有東西的包裏
if let iPad = 禮物包裏 {
	一台iPad = iPad
}else{
	一台iPad = iPad(版本: "2")
}
//玩iPad
//上面 一台iPad 是 iPad裡的值

let 另一台iPad:iPad
//如果是沒有東西的包裏
if let 如果有東西 = 沒有東西的包裏 {
	// 不會執行這裡
	另一台iPad = 如果有東西
}else{
	// 這裡會執行
	另一台iPad = iPad(版本: "2")
}
//玩另一台iPad
//上面 一台iPad 是 新的iPad實體 ，是新產生的iPad

//所以不論情況如何，都一定會有一台iPad可以運作。
```

而在Swift 1.2，導入了 `??` 這個語法來簡化Unwrapping的過程。
他的規則是，如果 `??` 左邊有值，就取左邊原本的值，如果左邊是一個`nil`，那就改取右邊的值。

> * let **[UnwrappedValue]** = [**Optional**(**WrappedValue**)] ?? **[AssignValue]**
> 	* if **Optional** is **Some case**, **[UnwrappedValue]** is **[WrappedValue]**
> 	* else  **[UnwrappedValue]** is **[AssignValue]**

如果有注意到的話，會發現 `??` 跟 `?:` 很像，可以快速把檢查與賦值一個語法完成，簡單來說 這也是一個**語法糖衣**。

```swift 
let 任何包裏 = 禮物包裏 ?? iPad(版本: "2") // 這裡會拿到禮物包裏裡的禮物
let 任何包裏2 = 沒有東西的包裏 ?? iPad(版本: "2") // 這裡會拿到新iPad的實體
```

`??` 也是我比較推薦的方式，因為人都有惰性，每個`Optional`都要用`if let`處理，也太不合乎常理，因此`??`可以提供一個便利且快速處理`Optional`的方式，而且因為是自已知道這個地方可能會`nil`，所以才賦與一個可以確保後續邏輯的值。

這跟Objective-C的 `nil` 呼叫 method 的方式有些相似，但不太一樣的是Objective-C是不知道什麼時候會`nil`， 但 Swift 的 `??` 是因為你知道這裡會有 `nil` 的可能才賦值。

## Optional 一條龍
考量以下的狀況

```swift
//定義一下，有一個人
class 人 {
	//這個人的姓名
	var 姓名:String?
	//這個人禮物收藏盒，也可能是空的
	var 禮物收藏盒:iPad?
	
	
	init(姓名:String?){
		self.姓名 = 姓名
	}
	
	//假如有人送禮物
	func 收禮物(禮物包裏:iPad?){
		// Unwrapping
		if let 禮物 = 禮物包裏 {
			禮物.使用者名字 = self.姓名
			self.禮物收藏盒 = 禮物
		}
	}
}

//產生一個iPad2
let iPad2:iPad? = iPad(版本: "2")
//產生一個人
let grady = 人(姓名:"Grady Zhuo")
//把iPad2送給grady
grady.收禮物(iPad2)
```

`let 使用者名字:String = grady.禮物收藏盒.使用者名字`
如果在Playground貼上上面這一句會發現會報錯。
因為[禮物收藏盒] 是 Optional，[使用者名字] 也是 Optional 
那會有幾種處理方法

#### 用 `!` 來進行`Optional`的Unwrapping
```swift
//這個寫法，我是非常不建議的，因為很容易在過程中有nil發生，就引發crash。
let 使用者名字:String = grady.禮物收藏盒!.使用者名字!
```

#### 用 `if let` 來進行`Optional`的Unwrapping
```swift
if let 禮物 = grady.禮物收藏盒 {
	if let 使用者名字 = 禮物.使用者名字 {
		print("使用者名字:\(使用者名字)")
	}
}
```

#### 用 `??` 來進行`Optional`的Unwrapping
```swift
let 禮物 = grady.禮物收藏盒 ?? iPad(版本: "2")
let 使用者名字 = 禮物.使用者名字 ?? ""
//如果iPad有值，那使用者名字就會出現Grady Zhuo
//如果iPad沒有值，那使用者名字變成""
```

### 難道不能一行解決，又不會crash嗎？

這時候就要請出**Optional Chaining**，他的用法跟Objective-C的**對nil呼叫不回應有些相似**。 實際的寫法跟用 `!` 來進行`Optional`的Unwrapping很像，只是把`!`換成`?`，雖然只是換了一個符號，但意思是完全不同的喔。

**Optional Chaining**的解釋是這樣的：

**傳呼參數或function的過程中，如果遇到optional，其中optional有值的話，則假設性的unwrapping並往下繼續傳遞，直到語法完成，如果過程中有任何一個參數無值，則直接結束語法，並傳回nil。**

```swift 
let 使用者名字:String? = grady.禮物收藏盒?.使用者名字
```

有一個重點是，**Optional Chaining**所取得的值，都一定是`Optional`，所以也可以想像**Optional Chaining**是在`Optional`裡面去**拆包裏**，所以就可以避開Crash。

所以記得要再Unwrapping **Optional Chaining** 的結果，你可以分兩行寫，也可以直接寫在`if let`子句內。

```swift 
if let 使用者名字 = grady.禮物收藏盒?.使用者名字 {
	//自動判斷型別，所以使用者名字會是 String
	//這時就可以直接處理使用者名字的邏輯
}else{
	//如果 禮物收藏盒 或 使用者名字 是nil
	//就會到這個區塊處理
}
```

### 說好的一行解決呢？不能一行就拿到Unwrapping後的結果嗎？
學會東西就要活用，所以我們可以發現，除了 ! , 上面有一個語法可以幫助我們快速Unwrapping並快速賦值。

就是`??`，所以我們可以把**Optional Chaining**和`??`組合使用，就可以快速的一行完成。

```swift 
//假設使用者名字的初使值是空字串
let 使用者名字 = grady.禮物收藏盒?.使用者名字 ?? ""
//如果 禮物收藏盒 或 使用者名字 是 nil
//取得的使用者名字就會是 空字串，反之就會是Grady Zhuo
```

## 練習時間 - JSON的拆拆拆時間

由於 Swift 是強型別的語言，因此在型別轉換上必須名確，而JSON是一個棘手的東西，JSON因為無法一開始就確定內容物是什麼型別，會大量使用AnyObject，加上Dictionary的操作上會因為可能key不存在，因此取得Optional的Value，在使用上比Objective-C還不方便。
雖然有高手製作了 SwifyJSON ，不只好用，處理起來也很乾淨，但畢竟是第三方套件，在Swift語法還在變更的狀況下，可能都要依賴開發者的更新。
所以自已要透過原生處理的時候，該如何進行呢？

```javascript
//有一個JSON字串
{
	"address":{
		"country":"Taiwan",
		"city":"Taichung",
	},
	"name":"Grady Zhuo",
	"gift":{
		"name":"iPad",
		"version":2,
	}
}
```

假設這是一個Person的JSON，試試看透過上面的Optional所學與技巧來取得下面幾個資訊：

1. address 裡的 city
2. name
3. gift 裡的 version


## 補充
1. 不同於C與多數語言會將NULL導向一個空的記憶體位置，或使用一個數值做為NULL的代表，Swift 的 nil 並不存在於真實的記憶體位置，就跟**class**這個詞一樣，只是個**保留字**。 
