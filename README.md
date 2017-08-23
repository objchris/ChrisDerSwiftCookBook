# Chris der Swift Cookbook ~

从头学Swift的读书笔记

Now base on 《The Swift Programming Language》 Version 3.1 

主要记录Swift学习过程中遇到的可能一时间难以记住的语法。可能会连带补充一些《The Swift Programming Language》中没有的内容。

以Cookbook的形式书写，带🥐标志的作为Xcode代码段方便开发使用。



## 基本数据类型

### 整数

#### Problem

Swift中的整数是怎样表示的？

#### Solution

Int、Uint 

#### Discusstion

跟C的命名方式类似，比如8位无符号整数类型是`UInt8`，32位有符号整数类型是 `Int32 `。

而不同于C语言的是，Int、UInt系列不再是short、int、long等伪装的typedef。而是结构体，而不同的结构体中定义了跟Int、UInt相关的变量和函数，例如：

可以通过min和max得到该类型所能达到的最小最大值

```swift
let minValue = UInt8.min	// 0
let maxValue = UInt8.max	// 255
```

一般地，Int、UInt是平台相关的，32位平台上为32位，64位平台上为64位

------

### 浮点数

#### Problem

Swift中的浮点数是怎样表示的？

#### Solution

使用Float和Double来表示浮点数类型，如不是明确需求，使用Double作为日常开发使用，表示的精度会更加精确些。

如果有需要使用到指数形式表示，二进制和十六进制分别使用e和p。

#### Discussion

- Float : 表示六位精度的浮点数
- Double : 表示十五位以上精度的浮点数

十进制浮点数也可以有一个可选的指数（exponent)，通过大写或者小写的 **e** 来指定；十六进制浮点数必须有一个指数，通过大写或者小写的 **p** 来指定。

如果一个十进制数的指数为 `exp`，那这个数相当于基数和10^`exp`的乘积：

`1.25e2` 表示 1.25 × 10^2，等于 `125.0`。
`1.25e-2` 表示 1.25 × 10^-2，等于 `0.0125`。
如果一个十六进制数的指数为`exp`，那这个数相当于基数和2^`exp`的乘积：

`0xFp2` 表示 15 × 2^2，等于 `60.0`。
`0xFp-2` 表示 15 × 2^-2，等于 `3.75`。

注：`1.25p2`会得到编译错误~

------

### 字符串和字符

#### Problem

Swift中的字符串和字符是怎么表示的？

#### Solution

String、Character

#### Discusstion

String类型是值类型，因此在对其进行常量、变量赋值操作，或在函数中传递时，会进行值拷贝而不是地址传递。

使用`String.characters`可以获得字符串中每个字符。

String之间可以使用`+`连接，而String和Character之间需要对Character做转换：

```Swift
let c: Character = "C"
var s: String = "string"
s.append(c)
// or
s += String(c)
```

String不仅重载了`+`，还重载了`+=`、`<`、`==`，比较两个字符串直接使用`==`，只要有同样的语义和外观，就认为是相同的。

String与NSString进行了无缝桥接，在String中调用NSString的方法不需要进行转换。

------

### 遍历字符串

#### Problem

怎么遍历字符串，怎么取出特定下标的字符？

#### Solution

使用String.index

#### Discussion

String和Character都是完全兼容Unicode标准的，而Character代表一个可扩展的字形群，可以通过组合不同的Unicode标量来获得另外一个Unicode标量，如（下面两个变量的值都是é）：

```swift
let eAcute: Character = "\u{E9}"  // é
let combinedEAcute: Character = "\u{65}\u{301}"  // e后面加上 ́
```

因此用`String.character.count`才能准确获得String中有多少个字符（上面两个变量都是1），而`String.length`实际上是调用`String.utf16Count`计算的是有多少个Unicode标量，即上面两个变量一个是1，另一个是2。因此，获取字符串的长度应该是`String.characters.count`。

综上得知，要知道Character的确定位置，就必须从String开头遍历每一个 Unicode 标量直到结尾。因此，Swift 的字符串不能用整数(integer)做索引。

三种方法获得索引：

- String对象的变量：`string.startIndex`、`string.endIndex`**注意：endIndex是String对象的最后一个字符的后面那个索引，所以千万不要s[s.endIndex]**
- 使用String对象的方法，调用`index(before:)`、`index(after:)`、`index(_:offsetBy:)`、`index(_:offsetBy:limitedBy:)`等函数。**这几个函数可在任何遵循Collection协议的类型中看到**
- 使用`string.character.indices`获取所有索引的Range。

```swift
for index in string.character.indices {
  ...
}
```

得到索引后，像其他语言的数组使用Int索引一样使用String的Index：`s[index]`。

------

### 插入和删除字符或字符串

#### Problem

如何在Swift中插入和删除字符或字符串？

#### Solution

insert(\_:at:)插入字符、insert(contentsOf:at:)插入字符串

remove(at:)删除字符、removeSubrange(\_:)删除子字符串

#### Discussion

可以在任意一个确认的并遵循 RangeReplaceableCollection 协议的类型里面使用 `insert(_:at:)`、`insert(contentsOf:at:)`、`remove(at:)` 和 `removeSubrange(_:)` 方法。

---

### 逻辑运算符

#### Problem

逻辑运算符以及组合运算

#### Solution

逻辑运算符有`!`、 `&&`、 `||`三种，分别代表逻辑非、逻辑与、逻辑或。

#### Discusstion

一元逻辑运算符`!`直接放在布尔型变量前，表示取反。

二元逻辑运算符`&&`和`||`是组合两个布尔型变量，根据这两个布尔型变量的值来决定语句是否为`true`。这两个运算符都是「**短路计算**」的，也就是说：当运算符前的值已经可以决定整个语句的是否为`true`时，将不再计算运算符后面的值。

可以将多个逻辑运算符组合起来使用，需要注意的是：

 Swift 逻辑操作符 && 和 || 是左结合的，这意味着拥有多元逻辑操作符的复合表达式优先计算最左边的子表达式。

```swift
if condition1 && condition2 || condition3 || condition4 {
    statements
}
//当condition1 && condition2 为 true，或condition3为true，或condition4为true，statements才会被执行。
```

所以，多个逻辑运算符组合起来使用，最好使用括号来明确优先级。

## 集合类型

### 数组

#### Problem

Swift中如何表示一个数组？有什么特性？

#### Solution

Array，若是分配给变量(`var`修饰)，则可以添加、删除、更改数组中的项。若是分配给常量(`let`修饰)，则大小和内容都不能改变。对应着OC中的NSMutableArray和NSArray。

#### Discussion

数组使用`Array[Element]`或`[Element]`来创建，将它们看做是一种数据类型，所以在创建的时候都需要在类型后加上`()`，如

```swift
let arrayA = Array[Int]()
let arrayB = [Int]()
```

因为Swift有类型推断机制，所以使用Array(repeating:count:)时，Swift会根据repeating的类型来决定Array中存储的数据项的类型。

像OC中提倡使用字面量`@[]`一般，Swift中同样可以使用字面量来定义数组：

```swift
let arrayC = [1,2,3,4,5]
```

访问数组：

- 相对于String来说比较简单，通过**Int型索引**可以获取到数组中对应位置的值。
- 通过只读`count`获取数组的项的数量。
- 通过`isEmpty`检查数组的项数量是否为0。
- 使用`for-in`来遍历数组中所有的项，或者利用`enumerated()`来获取索引和值组成的元祖。

```swift
for (index, value) in arrayC. enumerated() {
    print("Item \(String(index + 1)): \(String(value))")
}
```

修改数组：

- 像String一样，可以使用`+`来连接两个数组，`+=`也被重载了。
- 用`append(_:)`在数组末尾添加新的数据项。
- 使用下标来改变某个已有索引值对应的数据值，也可改变一系列数据值

```Swift
arrayC[0] = 2 // 此时arrayC中的项为[2,2,3,4,5]
arrayC[2...4] = [6]	// 此时arrayC中的项为[1,2,6]
```

- `insert(_:at:)`在某个**具体索引前**添加数据。
- `remove(at_)`**删除并返回**某个索引对应的数据项，后面的项往前移动。

关于Array到NSArray的桥接，未完待续。

## Swift特性

### 元祖(Tuples)

#### Problem

如何在Swift的函数中返回多个值，或者将一组有关联的值“捆绑”在一起

#### Solution

使用元祖，可把多个值组合成一个复合值。元组内的值可以是任意类型，并不要求是相同类型。

#### Discussion

最常见的是HTTP状态码 👉 404 对应 Page Not Found

`let http404Error = (404, "Not Found")`

在分解的时候可以分解成变量

```swift
let (statusCode, statusMessage) = http404Error
statusCode
statusMessage
```

或者，通过下标数字去读取

`http404Error.0 `

也可以在定义元祖的时候就给其中的元素命名

`let http200Status = (statusCode: 200, description: "OK")`

使用时通过名字来获取

```swift
http200Status.statusCode
http200Status.description
```

作为函数返回值，直接定义就好了

`func networkStatus() -> (Int, String) `

------

### 可选类型(Optional)

#### Problem

什么是可选类型，怎么用好可选类型？

#### Solution

可选类型用来处理值缺失的情况，在定义类型的时候在类型后加上? (表示可选类型) 或 ! (表示隐式解析可选类型)

#### Discusstion

Swift中任何类型都可以被定义成Optional，在使用中可以将该值赋为nil。     

不同于OC的是，nil在Swift中是一个确定的值，代表值缺失，因此可用于任何类型。而OC是将nil作为指向不存在对象的指针，只能用于对象。     

不是Optional的类型不能被赋nil值。    

在`if` 和`while`中可以使用let来判断一个可选类型是否包含值，并将该值存到一个临时变量中：

```swift
let possibleNumber = "123"
if let actualNumber = Int(possibleNumber) {
    print("\(actualNumber) is an integer")
} else {
    print("\'\(possibleNumber)\' could not be converted to an integer")
}
```

使用可选类型时，需要保证其不为空，除了上述的“可选绑定”方式外，若你确定其不为空，可以直接使用 ! 来获取值：`print("\(possibleNumber!) is an integer")`

! 不仅可以用来解析 ? 声明的可选类型，还可以用来声明隐式解析可选类型。     

隐式解析可选类型在第一次赋值确定之后一定有值的时候，使用其不需要再在变量或常量后面加上 ! ，但是需要特别注意的是，一定要确保有值，不然会得到运行时错误。

在可选类型后加上空白运算符`??`将对可选类型进行空判断，若该值为nil，则返回`??`后跟上的默认值：

```swift
let value = mayBeNil ?? defaultValue
// 等同于
let value = (mayBeNil != nil) ? mayBeNil! : defaultValue
```



## 较复杂部分

### 错误处理

#### Problem

在Swift中如何抛出错误，如何捕获错误？

#### Solution

```swift
do {
  try somethingMayThrowError
} catch {
  // 抛出错误时进行处理
}
```

#### Discussion

`somethingMayThrowError`是符合`Error`协议的类型。

将错误类型定义为enum是最好不过的了。Swift的enum可以支持带参数的case。这样对于相同类型的case就可以通过参数更进一步地区分。

- Code Snippets 🥐

```swift
enum <#ErrorTitle#> : Error {
    case <#TypeWithNoParam#>
    case <#TypeWithParam#>(<#ParamName#>: <#ParamType#>)
}
```

要么使用`do-catch`对函数判处的错误进行捕捉处理，要么将这些错误继续传递下去（函数名添加throws，使用try继续传递）

当构造函数（init）有可能发生错误时，应该在init函数中将错误抛出，由该构造函数的调用者去决定如何解决错误。

```swift
init(name: String) throws {
  ...
}
```

> catch子句不必将do子句中的代码所抛出的每一个可能的错误都作处理。如果所有catch子句都未处理错误，错误就会传递到周围的作用域。然而，错误还是必须要被某个周围的作用域处理的。

- Code Snippets 🥐

```swift
do {
    try <#expression#>
    <#statements#>
} catch <#pattern 1#> {
    <#statements#>
} catch <#pattern 2#> {
    <#statements#>
} catch <#pattern 3#> where <#condition#> {
    <#statements#>
} catch {
    <#statements#>
}
```

即使我们在编写代码的过程中知道`try`后面的函数只会抛出某一类Error，但是`do-catch`还是需要捕获全部错误，包括自己定义的和所有其他的错误。所以上面的Code Snippets 在最后加上了无匹配模式的`catch`。

可以为`catch`加`where`条件来决定是否捕获这个错误。

**如果函数没有`throws`标识符，且其中的`do-catch`不捕获或少捕获错误，会导致编译时不通过。**

`try? `可将函数的返回值类型变成对应可选类型，用这种方式来处理错误会让代码看起来更简洁（以下`x`和`y`等价）：

```swift
func someThrowingFunction() throws -> Int {
    // ...
}
let x = try? someThrowingFunction()
let y: Int?
do {
    y = try someThrowingFunction()
} catch {
    y = nil
}
```

`try!`用于断言运行时不会有错误抛出，因此也不需要处理错误。

相当于Java中的Finally语句，在Swift中可以使用`defer`在即将离开当前代码块时执行一系列语句（用于释放资源或记录日志等）。若有多个`defer`代码块，则按照被指定的顺序的相反顺序执行。

```swift
func processFile(filename: String) throws {
    if exists(filename) {
        let file = open(filename)
        defer {
            print("invoked after")
            close(file)
        }
        defer {
        	print("invoked first")
        }
        while let line = try file.readline() {
            // 处理文件。
        }
      	// 两个defer会在这里被调用
    }
}
```

------

