# 第五章 - 匪疑所思的  Optional - 宣告篇

了解了 Optional 在 Swift 中愛恨交織的情節後，這章我們將更深入探討 `!` 這傢伙還有什麼讓人更匪疑所思的地方?


### 本章重點：
1. 使用 `?` 和 `!` 宣告，跟運算子是一樣的事情嗎？有什麼差別？
2. 型別中使用的 `??` ， `!!` 又是怎麼一回事？ 
3. 多層的 Optional 我可以怎麼處理？

## 宣告上使用 **!** 跟 **?** 有什麼不同？

有開始寫Swift的朋友應該都會在許多地方看到 ! 在宣告時使用的情況。
例如：
```swift 
var name:String! = "Grady"
```
這看起來還算可以理解，就是一個 `String!` 型別的變數，放入了一個字串。
但為什麼一定要用 `String!` 呢？ 直接寫 `String` 不也可以?

直到發現 `String!` 也可以放 `nil` ...
```swift 
var name:String! = nil
```
心中不免困惑了起來…
```
咦? 為什麼 ! 又可以放值，又可以放 nil ?
那它跟 ? 又有什麼不同? 
為什麼不直接使用 ? 就好了???
```
---
會有這樣的疑惑也是很正常的，因為在官方文件中看到 `Optional` 章節的話，官容主要都是講述 `!` 是 Unwrapping 的`語法糖衣`，可以讓 swift 從 `Optional` 中將值取出。 

但 Apple 還有一個小小的篇幅在講解 [**Implicitly Unwrapped Optional**](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID334)，這又是什麼東東呢？

因為這裡討論的的 `!` 不是操作運算子，而是另一個 `Optional` 機制，名叫 **Implicitly Unwrapped Optional** ，中文叫 **隱式被解開的 Optional** ，有趣的是，它也是一個**enum**，所以其實你完全可以把他當成我們之前討論的 `Optional` 也就豆 `?` 來看待。

## 為什麼要搞兩個 Optional ?


