# 第三章 - 泛型(Generic) 是什麼?  有什麼好處？

### 泛型代碼可以讓你寫出根據自我需求定義、適用於任何類型的，靈活且可重用的函數和類型。它的可以讓你避免重複的代碼，用一種清晰和抽像的方式來表達代碼的意圖。

你可能在C++中，聽到的術語是**Template**, 而Swift，Java和C#則採用**Generic**(泛型)，但概念是相近的。

在Swift中，泛型可作用於幾個地方：
1. Function
2. Classess/Structs/Enums
3. Protocol
4. Extension

---
## 1. Function

### 先談談
在以往，我們可能會有幾個function，做很像的事情：
例如，交換兩個數字，交換兩個字串


```swift
//交換兩個數字
func swapInts(inout lhs: Int, inout _ rhs: Int){
    //先把lhs存到temp
    let temp = lhs
    //把rhs賦值到lhs
    lhs = rhs
    //最後把temp賦值到rhs
    rhs = temp
}

var a = 10
var b = 11
swapInts(&a, &b)
a //11
b //10

//交換兩個字串
func swapStrings(inout lhs: String, inout _ rhs: String){
    //先把lhs存到temp
    let temp = lhs
    //把rhs賦值到lhs
    lhs = rhs
    //最後把temp賦值到rhs
    rhs = temp
}

var aString = "Hello"
var bString = "World"
swap(&aString, &bString)
aString //World
bString //Hello
```

### swapInts做的事，和swapStrings是不是很像呢？😏

所以這個時候，泛型就派上用場了
我們可以定義一個function叫swapValues，並透過泛型的Type來通吃對換這件事


#### a. 泛型Function的語法一：

``` func FName<Type>(a: Type, [Arguments...])->Void ```

代表泛型對象代表的**Type** 將透過**傳入值的Type**來動態決定

```swift
//一個通用於任何型別的swapValues的函式
func swapValues<Type>(inout lhs: Type, inout _ rhs: Type){
    //先把lhs存到temp
    let temp = lhs
    //把rhs賦值到lhs
    lhs = rhs
    //最後把temp賦值到rhs
    rhs = temp
}

//所以我們可以這樣使用就可以了：

var c = 12
var d = 13
swapValues(&c, &d)
c //13
d //12

var cString = "123"
var dString = "456"
swapValues(&cString, &dString)
cString //456
dString //123
```

#### b.泛型Function語法二：

``` func FName<Type>([Arguments...])->Type ```

代表**Type**的依據是透過**傳出值**決定

```swift
func convertTo<Type>(value: Any)->Type?{
    //這裡只先做單純的轉型
    //如果型別可以強轉成功，就回傳，沒有就會回傳一個Optional
    return value as? Type
}

let str = "123"
let outputStr:String? = convertTo(str)
let outputInt:Int? = convertTo(str) //這裡就會回傳出nil
```

#### c.泛型Function語法三：

``` func FName<Type>(a: Type, [Arguments...])->Type ```

代表**Type**的依據是透過**傳入值和回傳值**兩個相同的**Type**決定

```swift
func test<Type>(a: Type)->Type?{
    return a
}

let output:String? = test("124")
let output:Int? = test("123") //這一句會失敗，因為傳入的是字串，但輸出是 Int? 就不成立
```

##### Swift內建中的Function, swap就是泛型的Function，有機會可以自已查詢看看定義喔。

---
## 2. Class/Struct/Enum

Swift中，**Generic**也可以使用在**Class/Struct/Enum**，可以所定義之型態依據不同的需求產生不同的使用。常見的如**Array/Dictionary**就是泛型的容器用法。

語法糖衣說明：
- [Int] ==> Array<Int>
- [String:Any] ==> Dictionary<String:Any>

```swift 
let intArray:[Int] = Array<Int>(count: 10, repeatedValue: 10)
let dict:[String:Any] = Dictionary<String, Any>(dictionaryLiteral: ("name", "grady"))
```

---
### 來看看泛型實際上可以幫到你什麼？

假設我們有一個Person的Struct，先不要搞得太複雜，只有
- 身份證字號
- 名字

這個例子是描寫一個可能的情境
1. 身份證字號有可能是數字，也可能是字串
2. 名字可以是一個單純的字串，但也可以是一個複雜的Struct/Class/Enum

Ok, 要開始囉~

```swift
struct Person<T, U> : CustomStringConvertible {
    // 身份證字號，型別由外部傳入的第一個順位型別：T 決定
    var identifier:T
    // 姓名，型別由外部傳入的第二個順位型別：U 決定
    var name:U
    
    //實作CustomStringConvertible
    var description:String{
        return "\(self.identifier) : \(self.name)"
    }
}

//如果身分證字號和姓名都是字串
let personByString = Person<String, String>(identifier: "124146828", name: "Grady Zhuo")
//如果身分證字號是整數和姓名是Name的struct
let personByIntName = Person<Int, Name>(identifier: 124146828, name: Name(first: "Max", last: "Chen"))
```

### 覺得寫成Person< String, Name >…很麻煩嗎？

##### 1. 如果有建構式可以決定泛型型態，那就可以不用寫<Int, Name>這樣的東西

```swift
let personByIntName2 = Person(identifier: 124146828, name: Name(first: "John", last: "Yeh", middle: "Boss"))
```

##### 2. 另外你也可以透過 typealias 來幫你自定型別的名字
```swift
typealias StringPerson = Person<String, String>
//接下來就可以用StringPerson來建構person，而T, U就不用再進行定義囉
let person = StringPerson(identifier: "L123345678", name: "Hello world")
```
---
### 3. Protocol
Protocl的泛型，嚴格來說不算是標準的Generic寫法，但透過要求protocol實作者完成型別的定義，以達成模擬Generic在多型別定義的效果。

```swift
protocol Animal {
    //可以透過 typealias Name 要求實作者完成Name的完整定義
    typealias Type
    //後續就可以使用Type去定義自已需要的參數
    var type:Type { get }
}

struct Cat : Animal {
    
    typealias Type = AnimalType
    
    var type:Type {
        var type = AnimalType()
        type.科 = .貓科
        return type
    }
    
}

struct Dog : Animal {
    
    typealias Type = String
    
    var type:Type {
        return "狗狗"
    }
}

let cat = Cat()
cat.type

let dog = Dog()
dog.type
```

---
### 4. Extension

Extension 泛型的類型的時候，可以直接使用其泛型定義的類型

```swift
extension Person {
    //傳入的name型別是U, 因為前面Person就已經宣告好了。
    mutating func changeName(name: U){
        self.name = name
    }
}
//先產生一個grady
var grady = Person<String, String>(identifier: "123", name: "Grady")
//因為grady設定的是String，所以可以接受字串的Hello
grady.changeName("Hello")
//但下面這一句就不行了
let name = Name(first: "Hello", last: "World", middle: nil)
grady.changeName(name) //無法接受Name型別的物件

```