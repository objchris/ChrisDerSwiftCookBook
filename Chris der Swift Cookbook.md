# Chris der Swift Cookbook ~

以Cookbook的形式书写，带🥐标志的作为Xcode代码段方便开发使用。

13515 Words, 32185 Character, may need 50 Minutes to very fast read.

So, cut the crap. Let's go for it. ヾ(o◕∀◕)ﾉヾ

[TOC]

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

#### Discussion

String类型是值类型，因此在对其进行常量、变量赋值操作，或在函数中传递时，会进行值拷贝而不是地址传递。

使用`String.characters`可以获得字符串中每个字符。

String之间可以使用`+`连接，而String和Character之间需要对Character做转换：

```swift
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

------

### 为String?加extension

#### Problem

通常我们使用`String?`来判断String是否为`nil`，如果要判断是否为`""`，我们可以使用`isEmpty`。如何判断：如果String是`nil`或`""`，都返回`nil`呢，否则返回解析后的值？

#### Solution

为String? 加extension，在其中进行判断。对开发中逻辑的处理和代码的简洁程度都有所帮助。

> [参考:Handling empty optional strings in Swift](https://medium.com/ios-os-x-development/handling-empty-optional-strings-in-swift-ba77ef627d74)

#### Discussion

日常开发中，很可能写这样的代码：

```swift
if let title = textField.text {
    if title.isEmpty {
        // Alert: textField is empty!
    }
    // somewhere uses title
}
```

简单点可以写成，同样效果：

```swift
guard !(textField.text?.isEmpty ?? true) else {}
```

如果需要判断的`textField`很少（`String?`很少），好像没什么问题。如果有很多个呢？这样写我们就需要去处理很多个`if`、`and`、`or`、`!`。同样的代码可能需要写很多很多。

何不将此类处理放在`String?`中，如果对象是`nil`或`""`，都返回`nil`，否则返回解析后的值。运用`Swift`的`extension`🥐就可以解决：

```swift
extension Optional where Wrapped == String {
    var nilIfEmpty: String? {
        guard let strongSelf = self else {
            return nil
        }
        return strongSelf.isEmpty ? nil : strongSelf
    }
}
```

添加完`extension`之后，上面的代码可以变得简洁，且可读性也不差：

```swift
guard let title = textField.text.nilIfEmpty else {}
// somewhere uses title
```

还可以用在诸如`map`或`flatMap`等函数中~

------

### 逻辑运算符

#### Problem

逻辑运算符以及组合运算

#### Solution

逻辑运算符有`!`、 `&&`、 `||`三种，分别代表逻辑非、逻辑与、逻辑或。

#### Discussion

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

------

### 函数(Function)

#### Problem

函数在Swift中如何定义，有什么需要注意的地方

#### Solution

```swift
func funName([paramLabel] param: [inout] Type [= defaultValue] , ...) [throws] -> returnType
```

#### Discussion

定义函数的方式一目了然，分别是：

1. `func`关键字
2. 函数名称，小写开头，驼峰命名法
3. 参数部分还可细分，由以下某几部分组成
   1. 参数标签：调用者在调用的时候使用的参数标志
   2. 参数名称：在函数体内部使用的参数标志
   3. 输入输出参数标志关键字`inout`，Swift的函数参数默认是不能在函数体中被修改的，会导致编译错误。若想修改此参数，需在类型前添加`inout`关键字。**输入输出参数不能有默认值，且可变参数不能用inout标记。**
   4. 参数类型：可以是`class`、结构体`struts`、数组`[Double]`、可变参数`Double...`，一个函数最多只能有一个可变参数。
   5. 默认值：当默认值被定义后，调用这个函数时可以忽略这个参数。**一般将带默认值的参数放在最后面**
4. 根据是否抛出错误给调用者处理，决定是否添加`throws`关键字，[**错误处理**](https://github.com/objchris/ChrisDerSwiftCookBook/blob/master/Chris%20der%20Swift%20Cookbook.md#错误处理)部分将详细讲解
5. 返回类型

调用时需要注意：

1. 如果上述3.1中参数标签为`_`，则不需要写参数名称。
2. 若有3.2中的`inout`属性，只能传入变量`var`，且在参数前需要加上`&`，这一点跟C/C++有点像
3. 有上述4中的`throws`关键字，需要用`do-try-catch`代码块，[**错误处理**](https://github.com/objchris/ChrisDerSwiftCookBook/blob/master/Chris%20der%20Swift%20Cookbook.md#错误处理)部分将详细讲解
4. 函数是**第一等类型**，意味着可以是函数的参数以及返回值。类型由参数类型和返回类型决定，如：` func add(_ a:Int, _ b: Int)->Int ` 👉`(Int,Int)->Int`，二者都没有的函数的类型为：`()->Void`。
5. 因为函数可以作为参数，当一个函数的最后一个参数是另一个参数的时候，在调用的时候可以作为不写在参数括号`()`中，而是作为[**尾随闭包**](https://github.com/objchris/ChrisDerSwiftCookBook/blob/master/Chris%20der%20Swift%20Cookbook.md#闭包)用`{}`包括起来添加在函数之后。即：

```swift
// 定义
func someFunction(_ a: Int, _ b: (Int)->Int) -> Int {
    // 这里用函数b来处理a，将b的返回值作为someFunction的返回值
    return b(a)
}

// 调用，这里{}部分是someFunction的第二个参数，但是由于是最后一个参数且是函数作为参数，所以用闭包来表示
let result = someFunction(2) { (param) -> Int in
    return param + 1
}
// result = 3
```

函数可以嵌套，外围函数可以调用内部的嵌套函数或将其当做返回值：

```swift
func outsideFunc() -> ()->Void {
    func insideFunc() {
        print("I am a inside function")
    }
    return insideFunc;
}
```

------

### 枚举

#### Problem

在Swift中，枚举是怎么表示的？

#### Solution

大部分情况下，我们使用到的枚举是：

```swift
enum MyEnum {
    case caseOne
    case caseTwo
}
```

如果，我们需要将`caseOne`相关联的值存储起来，那就可以使用关联值，此时，枚举是这样的：

```swift
enum Barcode {
    case upc(Int, Int, Int, Int)
    case qrCode(String)
}
```

#### Discussion

跟大多数编程语言相同，枚举的定义还是包裹在关键字`enum`代码块中🥐。

```swift
enum <#EnumName#>: <#Type#> {
    case <#caseOne#>
    case <#caseTwo#>
    case <#caseThree#>
}
```

每个枚举定义了一个全新的类型，像Swift中其他类型一样，名字应该以大写字母开头，给枚举类型起一个单数名字而不是双数名字：

```swift
let something = EnumName.caseOne
```

若**指定了枚举成员的类型**，如`Int`，那么枚举成员可以被默认值预填充。我们称此默认值为`原始值`。在使用原始值为**整数或者字符串类型**的枚举时，不需要显式地为每一个枚举成员设置原始值，Swift 将会自动为我们赋值。

同时，由于定义了类型，枚举成员拥有了`rawValue`的属性，因此，在使用枚举类型的时候我们可以通过一个初始化方法来创建枚举，这个方法接收`rawValue`作为参数，返回`rawValue`对应的枚举成员或`rawValue`没有对应时返回`nil`。

```swift
enum CompassPoint: String {
    case north, south, east, west
}
// 定义枚举时指定了类型String，因此枚举成员拥有了原始值，rawValue即其本身
// 若没有指定类型，则不存在rawValue属性
let direction = CompassPoint.north.rawValue // north
var mayBeNil = CompassPoint(rawValue:"up") // nil
```

有关联值的枚举🥐：

```swift
enum <#EnumWithRelevance#>: <#Type#> {
    case <#caseOne#>(<#paramType#>)
    case <#caseTwo#>(<#paramType#>,<#paramType#>)
}
```

有关联值的枚举可以为枚举成员保存关联值，如果关联值的类型还是枚举本身的话，那就涉及到递归枚举了。在枚举成员前面加上`indirect`来表示该成员可递归。如：

```swift
enum ArithmeticExpression {
    case number(Int)
    indirect case addition(ArithmeticExpression, ArithmeticExpression)
    indirect case multiplication(ArithmeticExpression, ArithmeticExpression)
}
```

用递归枚举可以很方便地解决嵌套表达式问题：`(1+2)*3`

```swift
let one = ArithmeticExpression.number(1)
let two = ArithmeticExpression.number(2)
let three = ArithmeticExpression.number(3)
let sum = ArithmeticExpression.addition(one, two)
let result = ArithmeticExpression.multiplication(sum, three)

func evaluate(_ expression: ArithmeticExpression) -> Int {
    switch expression {
    case let .number(value):
        return value
    case let .addition(left, right):
        return evaluate(left) + evaluate(right)”
	case let .multiplication(left, right):
        return evaluate(left) * evaluate(right)
    }
}

print(evaluate(result))
```

## 集合类型

### 数组（Array）

#### Problem

Swift中如何表示一个数组？有什么特性？

#### Solution

Array，若是分配给变量(`var`修饰)，则可以添加、删除、更改数组中的项。若是分配给常量(`let`修饰)，则大小和内容都不能改变。

#### Discussion

数组使用`Array<Element>`或`[Element]`来创建，将它们看做是一种数据类型，所以在创建的时候都需要在类型后加上`()`，如

```swift
let arrayA = Array<Int>()
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
for i in arrayC {
    print("\(i)")
}
for (index, value) in arrayC.enumerated() {
    print("Item \(String(index + 1)): \(String(value))")
}
```

修改数组：

- 像String一样，可以使用`+`来连接两个数组，`+=`也被重载了。
- 用`append(_:)`在数组末尾添加新的数据项。
- 使用下标来改变某个已有索引值对应的数据值，也可改变一系列数据值

```swift
arrayC[0] = 2 // 此时arrayC中的项为[2,2,3,4,5]
arrayC[2...4] = [6]	// 此时arrayC中的项为[1,2,6]
```

- `insert(_:at:)`在某个**具体索引前**添加数据。
- `remove(at_)`**删除并返回**某个索引对应的数据项，后面的项往前移动。

关于Array到NSArray的桥接，未完待续。

------

### 集合（Set）

#### Problem

Swift中如何表示一个集合？有什么特性？

#### Solution

集合(Set)用来存储相同类型并且没有确定顺序的值。当集合元素顺序不重要时或者希望确保每个元素只出现一次时可以使用集合。

#### Discussion

集合是通过Hash来确定一个值当前是否存储在其中。所以为了存储在集合中，该类型必须遵循`Hashable`协议。`Hashable`是一个`Protocal`，其中只有一个计算属性`hashValue`，用于返回计算后的哈希值。要选择一个对类型中包含的属性来说较为合适的哈希算法。见  [**如何选取好的Hash算法**](https://github.com/objchris/ChrisDerSwiftCookBook/blob/master/Chris%20der%20Swift%20Cookbook.md#如何选取好的hash算法)

再者`Hashable`是符合`Equatable`协议的。所以必须重载`==`来告诉`Set`（或下面将提到的`Dictionary`），如何判断两个元素是相同的。

当你将没有遵循`Hashable`协议的类型加入到`Set`中，将会得到运行错误。

总的来说，看下面的例子：

```swift
// A point in an x-y coordinate system.
struct GridPoint {
    var x: Int
    var y: Int
}
// 若要在Set中保存，或需要作为Dictionary的Key
extension GridPoint: Hashable {
    var hashValue: Int {
        return x.hashValue ^ y.hashValue &* 16777619
    }

    static func == (lhs: GridPoint, rhs: GridPoint) -> Bool
        return lhs.x == rhs.x && lhs.y == rhs.y
    }
}
```

定义集合类型只能用`Set<Element>`，`Element`表示`Set`中存储的类型。

像`Array`一般，我们可以使用字面量来构造一个`Set`，但需要注意的是：在定义的时候需要显式指定变量或常量为`Set<Element>`，否则会被认为是数组。

```swift
var pointsSet: Set<GridPoint> = [GridPoint(x: 1,y: 1),GridPoint(x: 2,y: 2)]
```

对`Set`的操作和函数基本与`Array`相似。但是`Set`是没有确定的顺序的，可以使用`sorted()`方法对`Set`进行排序，顺序由`<`对元素进行比较的结果确定。

使用`contains(_:)`检查`Set`中是否包含一个特定的值。

```swift
if pointsSet.contains(GridPoint(x: 1,y: 1)) {
    print("Yeah! I got a point")
}
```

关于Set到NSSet的桥接，未完待续。

### 集合的组合操作

#### Problem

当我要对两个集合进行操作，取共同拥有的交集，取某一个集合相对于另一个集合的补集，或者取两个集合各自拥有的元素再组合成一个集合，应该怎么办？

#### Solution

![](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Art/setVennDiagram_2x.png)

#### Discussion

- 使用`intersection(_:)`方法根据两个集合中都包含的值创建的一个新的集合。
- 使用`symmetricDifference(_:)`方法根据在一个集合中但不在两个集合中的值创建一个新的集合。
- 使用`union(_:)`方法根据两个集合的值创建一个新的集合。
- 使用`subtracting(_:)`方法根据不在该集合中的值创建一个新的集合。

```swift
let oddDigits: Set = [1, 3, 5, 7, 9]
let evenDigits: Set = [0, 2, 4, 6, 8]
let singleDigitPrimeNumbers: Set = [2, 3, 5, 7]
 
oddDigits.union(evenDigits).sort()
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
oddDigits.intersection(evenDigits).sorted()
// []
oddDigits.subtracting(singleDigitPrimeNumbers).sorted()
// [1, 9]
oddDigits. symmetricDifference(singleDigitPrimeNumbers).sorted()
// [1, 2, 9]
```

### 集合间关系

#### Problem

如何确定两个集合是相等、包含、严格包含、部分包含？

#### Solution

![](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Art/setEulerDiagram_2x.png)

```swift
b.isSubset(of:a)	//true
a.isSuperset(of:b)	//true
b.isDisjoint(with:c)	//true
```

#### Discussion

- 使用运算符`==`来判断两个集合是否包含全部相同的值。
- 使用`isSubset(of:)`或`isSuperset(of:)`方法来判断一个集合是否是另外一个集合的子集或父集。一个集合对自己调用`isSubset(of:)`结果为`true`。
- 使用`isStrictSubset(of:)`或者`isStrictSuperset(of:)`方法来判断一个集合是否是另外一个集合的子集合或者父集合并且两个集合并不相等。一个集合对自己调用`isStrictSubset(of:)`永远是`false`。
- 使用`isDisjoint(with:)`方法来判断两个集合是否不含有相同的值(是否没有交集)。
- 上述Solution中，若要判断部分包含（即a和c的关系），可以将集合间的组合操作和集合间的关系合起来。

```swift
a.isSuperset(of:a.intersection(c))	//为true则部分包含，为false则完全不包含
```

------

### 字典

#### Problem

Swift中如何表示一个字典？有什么特性？

#### Solution

字典`Dictionary`是键值对，是一种存储多个相同类型的值的容器。可以通过键来得到存储在字典中的值。

#### Discussion

- Swift中的字典使用`Dictionary<Key, Value>`定义。
- 可以使用`[Key:Value]`的形式创建一个字典对象。
- 和`Array`一样，我们可以使用字面量来构造字典。

```swift
let dic1 = Dictionary<String, Int>()
let dic2 = [Int: String]()
var dic3 = [1:"string1",2:"string2",3:"string3"]
```

`Dictionary`的键`Key`同`Set`中的值，遵循`Hashable`协议。字典中的键是唯一的。

对`Dictionary`的操作和函数基本与`Array`相似。使用属性`count`获取字典中的数据项数量，使用属性`isEmpty`检查是否数据项是否为0。

不同于`Array`使用Int下标，`Dictionary`通过`Key`来获取或修改对应的值。除了用`[]`来修改值之外，若想在修改的同时获取修改前的旧值，可以使用`updateVale(_:forKey:)`，该函数会返回`String?`，若有旧值则返回，没有返回nil，`updateVale(_:forKey:)`在字典中添加该数据项。

```swift
dic3[1] = "string that change"	// 上文定义dic3的时候使用var，所以dic3才可以添加删除修改元素
let stringThatChange = dic3.updateValue("string that change", forKey:1)		//stringThatChange 是 String?
if let result = dic3[1]	{	// 要注意： result 是 String?
    ...
}
```

删除元素可以使用`removeValue(forKey:)`，该方法返回被移除的值或不存在值的时候返回`nil`。

像`Array`中的每一项可以以`(Index, Value)`元祖的形式返回，`Dictionary`中的每一项都可以以`(Key, Value)`元祖的形式返回。

若只需使用字典的键或值，可以使用属性`keys`或`values`来单独获取。 

------

### 如何选取好的Hash算法

#### Problem

#### Solution

#### Discussion

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

#### Discussion

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

`!` 不仅可以用来解析` ?` 声明的可选类型，还可以用来声明隐式解析可选类型。     

隐式解析可选类型在第一次赋值确定之后一定有值的时候，使用其不需要再在变量或常量后面加上` !` ，但是需要特别注意的是，一定要确保有值，不然会得到运行时错误。

在可选类型后加上空白运算符`??`将对可选类型进行空判断，若该值为`nil`，则返回`??`后跟上的默认值：

```swift
let value = mayBeNil ?? defaultValue
// 等同于
let value = (mayBeNil != nil) ? mayBeNil! : defaultValue
```

------

### Switch

#### Problem

Swift中的`switch`与其他语言不同在哪？

#### Solution

不需要在`switch`中的每一个`case`后添加`break`，且`case`分支的模式可以是值的区间。

#### Discussion

当我们在C/C++、Java等语言中使用`switch`的时候，在每一个`case`中都需要添加`break`来什么分支结束，否则就会继续往下一个分支执行。

在Swfit中，我们不再需要为每一个`case`添加`break`了，当匹配的`case`分支中的代码执行完毕后，程序会终止`switch`语句，不存在隐式的贯穿。

当然，不排除有些时候我们需要贯穿，如

```swift
switch condition {
    case A : 
        代码段A
    case B : 
        代码段A
        代码段B
}
```

这样写固然可以，但是可以使用`fallthrough`合起来写

```swift
switch condition {
    case A : 
        代码段A 
        fallthrough
    case B : 
        代码段B
}
```

`case`分支的模式可以是一个区间，区间可以简写为`1...5`表示1到5的所有数字，`1..<5`表示1到4的所有数字，因此

```swift
case 1...5:
case 1..<5:
```

`case`分支的模式可以是元祖

```swift
case (0, 0):	// 匹配二元元祖的两个元素
case (_, 0):	// 匹配二元元祖的第二个元素
case (0...5, 0..<5):	// 在元祖中使用区间
```

上述`case(_, 0)`，我们可以认为在`case`分支中的代码不需要元祖的第一个元素，如果我们要使用第一个元素，我们可以使用值绑定

```swift
case (let x, 0): 
	// some code that use x
case let (x, y): // 等价于 case (let x, let y): ，这个分支可以匹配所有的情况，所以为了保证上面的case分支可以被选中，应该将此分支放在switch最后。(相当于default)
```

可以用`where`来为case添加额外的判断条件，如：

```swift
case (let x, 2) where x == 1: // 等价于case (1,2):
case (let x, let y) where x == y:
```

------

### 如何跳出多重循环

#### Problem

开发中难免会碰到满足某个条件就终止多重循环的时候，使用Swift应该怎么做呢？

#### Solution

使用Swift的带标签的语句，即在循环前加上标签。

#### Discussion

这个特性像是借鉴于`Java`，声明一个带标签的语句是通过在该语句的关键词的同一行前面放置一个标签，作为这个语句的前导关键词，并且该标签后面跟随一个冒号。

```swift
label:while condition {
    switch something {
        case a:
            break
        case b:
            break label
	}
}
```

上述例子中，如果`something`是`a`，则会跳出`switch`，而如果`something `是`b`，那么则会跳出循环。

------

### if和guard

#### Problem

在函数中提前退出可以使用`if`和`guard`，二者有什么区别

#### Solution

见 Discussion

#### Discussion

`if`和`guard`的区别在于：

1. `guard`语句总是带有`else`从句，如果`guard`后跟的条件语句为真时，才会执行`guard`后面的代码。而`if`则没有这种要求。
2. `guard`不带从句，直接跟着`else`。
3. 在`guard`后跟的条件语句中使用`let`解析的`optional`值可以在`guard`代码段后使用，而`if`则只能在从句代码段中使用。

```swift
guard let something = mayBeNil else {
    return
}
print("\(something)")	// 合法的


if let something = mayBeNil {
    // something 只能在此代码段中使用
}
```

---

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

相当于Java中的Finally语句，在Swift中可以使用`defer`在即将离开当前代码块时执行一系列语句（用于释放资源或记录日志等）。若有多个`defer`代码块，则按照被指定的顺序的**相反顺序**执行。

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

### 闭包

#### Problem

在Swift中什么是闭包？有什么优化的地方？

#### Solution

闭包是自包含的函数代码块，可以在代码中被传递和使用。主要用在语法优化上，主要优化如下：

> - 利用上下文推断参数和返回值类型
> - 隐式返回单表达式闭包，即单表达式闭包可以省略 return 关键字
> - 参数名称缩写
> - 尾随闭包语法

#### Discussion

闭包表达式有如下一般形式：

```swift
{(paramters) -> returnType in
    statements
}
```

- 根据上下文推断参数和返回值类型

譬如，我们在说明函数的时候写过这么一个闭包

```swift
// 定义
func someFunction(_ a: Int, _ b: (Int)->Int) -> Int {
    return b(a)
}
// 调用，使用尾随闭包指定第二个参数
let result = someFunction(2, { (param: Int) -> Int in
    return param + 1
})
```

我们在定义的时候已经指定了`b`的类型是`(Int) -> Int`，那在调用的时候就可以省略闭包的参数类型和返回类型：

```swift
let result = someFunction(2, {param in
    return param + 1
})
```

- 单行表达式闭包可省略`return`关键字

什么是单行表达式闭包，就像上面的闭包那样，闭包的函数体部分只有一行代码`return param + 1`，那么此时可以省略`return `关键字：

```swift
let result = someFunction(2, {param in param + 1})
```

- 参数名称缩写

Swift自动为**内联闭包**提供参数名称缩写功能，直接通过`$0`、`$1`、`$2`来顺序调用闭包的参数。这样做可以省略参数名称（参数类型被自动推断）、`in`关键字。上述调用可以继续简化：

```swift
let result = someFunction(2, {$0 + 1})
// or another closure
let result = someFunction(2, {
    if $0 > 0 {
        return 100
    } else {
        return 0
    }
})
```

- 尾随闭包

我们在说明函数的时候写过这么一个闭包已经写过尾随闭包了，当函数的最后一个参数是另一个函数的时候，可以使用**尾随闭包**来增强程序可读性。上述调用可以写作：

```swift
let result = someFunction(2) {$0 + 1}
```

如果闭包表达式是函数或方法的唯一参数，则当你使用尾随闭包时，你甚至可以把 () 省略掉。

嵌套函数和闭包表达式会捕获在闭包中被修改的值的引用，如果闭包是类实例的属性(**闭包是引用类型**)，在使用时，应该注意不要造成**循环引用**

接下来是：_逃逸闭包_，用`@escaping`来指定这个闭包在函数返回之后才被执行。举个例子，很多启动异步操作的函数接受一个闭包参数作为` completion handler`。这类函数会在异步操作开始之后立刻返回，但是闭包直到异步操作结束后才会被调用。将一个闭包标记为 @escaping 意味着你必须在闭包中显式地引用 self。

```swift
class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}
```

还有一个是：自动闭包。通过将参数标记为 @autoclosure 来接收一个自动闭包。可以将该函数当作接受 String 类型参数（而非闭包）的函数来调用。

```swift
var customersInLine = ["a", "b", "c"]
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: customersInLine.remove(at: 0))
// 若上面的定义没有@autoclosure，则调用的时候需要传入闭包，像这样：
serve(customer: {customersInLine.remove(at: 0)})
```

自动闭包很典型的例子是：`assert(condition:message:file:line:) `， `condition`就是一个自动闭包。

## 结构类型

### 类和结构体的异同及如何选择

#### Problem

在Swift中，类和结构体有什么异同，我们在开发过程中应该选择什么结构来构造我们的实例。

#### Solution

类和结构体有很多共同点，在Swift中，二者的关系要比在其他语言中更加的密切。通过了解二者的特性，我们可以在开发中快速地知道我们需要的是什么结构。

#### Discussion

类和结构体相同在于：

- 定义属性用于存储值
- 定义方法用于提供功能
- 定义下标操作使得可以通过下标语法来访问实例所包含的值
- 定义构造器用于生成初始化值
- 通过扩展以增加默认实现的功能
- 实现协议以提供某种标准功能

而类，有更多的功能，比如：继承、多态、抽象等。

定义类和结构体使用不同的关键字，但是格式基本一致：

```swift
class SomeClass {
    let someProperty
}
struct SomeStruct {
    let someProperty
}
```

不同点在于：

- 类是引用类型，在赋值或传递到一个函数时，其值不会被拷贝。用`let`声明的类实例可以修改其中的属性，因为常量的值保存的是对实例的引用。相对的，结构体是值类型，总是通过被复制的方式在代码里传递，不使用引用计数。要修改结构体实例中的值，该结构体实例在定义的时候必须使用`var`来声明。在Swift中，**所有的结构体和枚举都是值类型**。
- 所有结构体都有一个自动生成的构造函数，用于初始化新结构体实例中成员的属性，而类实例则没有。

如果在一个类中，有结构为结构体的子属性，Swift允许直接设置结构体属性的子属性，这一点在OC中是不允许的，OC中只能重新为整个结构体属性赋新值。

使用**等价于**`===`或`!==`检测两个常量或者变量是否引用同一个实例：

```swift
let firstInstance = SomeClass()
let secondInstance = firstInstance
if firstInstance === secondInstance {
    // the same instance
}
```

按照通用的准则，当符合一条或多条以下条件时，使用结构体：

- 该数据结构的主要目的是**用来封装少量相关简单数据值。**(最常见)
- 有理由预计该数据结构的实例在被赋值或传递时，封装的数据将会被拷贝而不是被引用。
- 该数据结构中储存的值类型属性，也应该被拷贝，而不是被引用。
- 该数据结构不需要去继承另一个既有类型的属性或者行为。


---

### 属性

#### Problem

想在类、结构体等结构中存储值，在Swift中应该如何做，有什么需要注意的地方？

#### Solution

使用**属性**，存储变量或常量作为实例或类的一部分，还可以添加观察器来监控属性值的变化。

#### Discussion

写过OC都知道`@property`，就是我们常说的属性。在Swift中的属性跟OC又有点不同：

属性的分类：

- 存储属性

  直接在类或结构体中定义的变量或常量就是类或结构体的一个存储属性（**枚举类型没有实例存储属性**）：

  ```swift
  class SomeClass {
      let constantProperty: Int
      var variableProperty: Int
  }
  struct SomeStruct {
      let constantProperty: Int
      var variableProperty: Int
  }
  ```

  通过`.`获取实例的属性值：

  ```swift
  let instance = someClass()
  instance.variableProperty = 1
  let i: Int = instance.variableProperty // 1
  ```

  [类和结构体的异同及如何选择](https://github.com/objchris/ChrisDerSwiftCookBook/blob/master/Chris%20der%20Swift%20Cookbook.md#类和结构体的异同及如何选择)中，我们说到：

  > 所有结构体都有一个自动生成的构造函数，用于初始化新结构体实例中成员的属性，而类实例则没有。

  所以我们必须确保类的初始化方法中，类的所有非`Optional`属性都能得到一个初始的值。例如：

  ```swift
  class Person {
      var name: String
      init(name: String) {
        	// 不初始化name属性会导致编译错误
          self.name = name
      }
  }
  ```

  当然属性也可以有默认值，这样就省去了在初始化方法中赋值的操作。

  ```swift
  class SomeClass {
      var aProperty: Int = 1
  }
  ```

  另外一种提供默认值的方式是：通过闭包或函数来设置

  ```swift
  class SomeClass {
      var aProperty: Int = {
        	// 在此闭包内，实例的其他部分还没有初始化，因此不能在此访问其他属性，不能使用self，或调用任何实例方法。(lazy属性除外)
          return 1  // 返回值的类型必须和定义的类型相同，此例是Int
      }()  // 此处大括号后接了一对空的小括号，用来告诉Swift立即执行此闭包。
  }
  ```

  当属性的值依赖于在实例的构造过程结束后才会知道影响值的外部因素时，或者当获得属性的初始值需要复杂或大量计算时，可以只在需要的时候计算属性的值，这种属性叫做**延迟储存属性**，用`lazy`修饰

  ```swift
  lazy var lazyProperty = someComplicatedClass()	// lazy属性必须使用var来定义，因为常量let需要在构造完成之前必须要有初始值
  ```

  特别注意：`lazy`属性在没有初始化的时候同时被多个线程访问，无法保证该属性只被初始化一次。（线程不安全）


- 计算属性

  计算属性不直接存储值，而是提供`getter`和一个**可选**的`setter`函数，来间接获取和设置**其他属性或变量**的值。

  ```swift
  struct Point {
      var x = 0.0, y = 0.0
  }
  struct Size {
      var width = 0.0, height = 0.0
  }
  struct Rect {
      var origin = Point()
      var size = Size()
      var center: Point { // 通过center来获取中心点的坐标值和修改原点的值
          get {
              let centerX = origin.x + (size.width / 2)
              let centerY = origin.y + (size.height / 2)
              return Point(x: centerX, y: centerY)
          }
          set(newCenter) { // 若不指定变量名为newCenter，默认为newValue
              origin.x = newCenter.x - (size.width / 2)
              origin.y = newCenter.y - (size.height / 2)
          }
      }
  }
  ```

  若只有`getter`没有`setter`，可以简化为：

  ```swift
  var center: Point {
      let centerX = origin.x + (size.width / 2)
      let centerY = origin.y + (size.height / 2)
      return Point(x: centerX, y: centerY)
  } // 此时center为只读属性
  ```

  必须使用`var`关键字来定义计算属性，因为它们的值不是固定的。


- 类型属性

  上面讲述了实例属性，每创建一个实例，该实例就拥有自己的一套属性值，相互独立，而类属性则属于类本身，无论创建了多少个实例，这些属性只有唯一一个。

  存储型类型属性是延迟初始化的，它们只有在第一次被访问的时候才会被初始化。即使它们被多个线程同时访问，系统也保证只会对其进行一次初始化。

  用`static`或`class`来指定：

  结构体`struct`和枚举`enum`，使用`static`来指定类型属性。

  类可以使用`static`指定，但子类不能重写该属性。使用`class`来指定，子类可以通过添加关键字`override`重写该属性。

  ```swift
  class Point {
      var x: Double = 1
      var y: Double = 1
      class var z: Double {	// 若此处使用static，MyPoint的重写就不被允许
          return 1
      }
  }
  class MyPoint: Point {
      override class var z: Double {
          return 2
      }
  }
  ```

---

### 属性观察器

#### Problem

在计算属性中，我们可以通过`setter`来监控值的变化，怎么在存储属性中做到这一步呢？

#### Solution

Swift提供了属性观察器，监控和响应属性值的变化，适用于：

- **除了延迟存储属性`lazy`之外**的其他存储属性
- 继承得到的属性，可通过重载的方式添加

#### Discussion

在Swift中，可以为属性添加如下的一个或全部观察器：

- `willSet`在新的值被设置之前调用
- `didSet`在新的值被设置之后调用

```swift
class StepCounter {
    var totalSepts: Int {
        willSet {
            // newValue可得到新值
            print("new value is \(newValue)")
        }
        didSet {
            // 可以获取到被覆盖的旧值，如果在此方法中继续对totalSteps赋值，那么totalSepts会被赋上新值
            print("old value is \(OldValue)")
        }
    }
}
```

注意：

- 父类的属性在子类的构造器中被赋值时，它在父类中的`willSet`和`didSet`观察器会被调用，随后才会调用子类的观察器。在父类初始化方法调用之前，子类给属性赋值时，观察器不会被调用。 （父类属性的观察器只有在父类初始化方法调用后才会被调用）
- 如果将属性通过` in-out `方式传入函数，`willSet` 和 `didSet` 也会调用。这是因为` in-out `参数采用了拷入拷出模式：即在函数内部使用的是参数的 **copy**，函数结束后，又对参数重新赋值。

---

### 初始化构造方法

#### Problem

在Swift中，如何创建数据结构（类、结构体、枚举）的实例，有什么需要注意的地方？

#### Solution

可以通过在数据结构中定义构造器来实现。

#### Discussion

构造器大致语法：

```swift
init() {
    // 在此处执行构造过程，在此方法中对类或结构体中 没有默认值或非Optional的属性 进行赋值
}
```

如果构造器不止一个，我们需要通过**参数名和类型**来区分。

[属性观察器](https://github.com/objchris/ChrisDerSwiftCookBook/blob/master/Chris%20der%20Swift%20Cookbook.md#属性观察器)一节我们说到，我们可以监视类或结构体的属性的值的变化，但是在构造方法中是不适用的，无论设置默认值还是通过构造器来为属性赋值，它们的值都是被直接设置的。

如果类或结构体的所有属性都有默认值，同时没有自定义的构造器，那么Swift会给这些类或结构体提供一个默认构造器。如果是类，那么此默认构造器继承自父类。

讲到[类和结构体的异同](https://github.com/objchris/ChrisDerSwiftCookBook/blob/master/Chris%20der%20Swift%20Cookbook.md#类和结构体的异同及如何选择)时，我们说到结构体有一个逐一成员构造器，当你为结构体定义了自定义的构造器时，就无法访问到这个逐一成员构造器了。假如你希望默认构造器、逐一成员构造器以及你自己的自定义构造器都能用来创建实例，可以将自定义的构造器写到扩展`extension`中，而不是写在值类型的原始定义中。

```Swift
struct SomeStruct {
    var i: Double
    var j: Double
}
extension SomeStruct {
    init() {
        i = 1
        j = 2
    }
}
let s: SomeStruct = SomeStruct()
let anotherS: SomeStruct = SomeStruct(i: 1, j: 2)
```

---

### 类的继承和构造过程

#### Problem

在其他语言中，子类可以继承或重写父类的构造器，在Swift中有什么需要注意？

#### Solution

像其他语言一样，Swift也有指定构造器和便利构造器，不过在Swift中对这两种构造器做了明确区分。还提供了可失败构造器，必要构造器等。而在继承方面，也有很多语法上需要注意的点。

#### Discussion

类的构造过程相对来说会复杂一些：Swift为类提供了两种构造器来确保实例中所有存储属性都能获得初始值：指定构造器和便利构造器。

指定构造器我们在上面已经说明，便利构造器是辅助型的构造器。可以定义便利构造器来调用同一个类中的其他构造器。便利构造器使用相同的写法，但是需要在`init`关键字前添加`convenience`关键字。

```Swift
convenience init {
    // 构造过程，通常是调用其他构造器
}
```

便利构造器必须调用其他构造器，且最终必须导致一个指定构造器被调用。

类有继承的特性，通常情况下，子类的指定构造器必须直接调用父类的指定构造器。子类默认不会继承父类的构造器，除非**子类中的属性都有默认值或为`Optional`，且子类没有定义构造器**。便利构造器不能调用父类的构造器，但是有一个情况除外，就是**父类的指定构造器都被子类实现（继承自父类或提供重写）**。

定义指定构造器的步骤：

1. 初始化本类的属性（可选类型`?`、`!`，有默认值`nil`，可以不用赋值），若构造方法的参数名称和属性名称相同，可用`self`来区分；
2. 调用`super.init()`，为父类的属性赋值；**[有父类必须调用，否则不需要]**
3. `self`**初始化完成**，读取或修改实例属性或调用实例方法。**[可选]**

重写父类的指定构造器时，需要添加`override`关键字，而重写便利构造器则不需要。

```swift
override init() {
    // 重写
}
```

接下来可失败构造器，在`init`后加个`?`来表示初始化过程可能会导致失败。失败则返回nil。

```Swift
init?(some: String) {
    if some.isEmpty { return nil }
    // 一些初始化操作
}
```

可失败构造器可以被代理（即在其他便利构造器中被调用），如果可失败构造器触发构造失败，那么整个过程都会被终止。

在类的构造器前添加`required`修饰符表明所有该类的子类都必须实现该构造器，这就是必要构造器。重写父类中必要的指定构造器时，不需要添加`override`关键字，但是还是需要`required`关键字。

---

### 类实例之间的强引用循环

#### Problem

我们都听说过自动引用计数`ARC`，解决的是应用中内存的分配问题。有了这一个特性，有可能使我们的实例之间保存着强引用，理清实例之间的关系，有助于我们打破实例间的强引用。

#### Solution

下述两种场景分别对应着两种不同的强引用解决方式：`weak`、`unowned`

#### Discussion

首先，有可能存在两个类，他们之间有相互引用的关系，例如一个人对应一间公寓，因为`ARC`，我们知道，如果这二者之间都保留着对对方的强引用，那么必然会有循环引用的情况出现。

`weak`是弱引用，被声明为弱引用的属性会在引用的实例被销毁后自动将其赋值为`nil`，因此我们需要其定义为**可选类型变量**，而非常量。

> 当 ARC 设置弱引用为nil时，属性观察不会被触发。

像人和公寓的问题，可以如下定义：

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
}
class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    weak var tenant: Person?
}
// 因为多对多的关系，且我们不能知道二者的生命周期，所以在Apartment中将Person定义为weak，当tenant属性对应的实例被销毁时，该Apartment实例的tenant被置为nil
```

另一个很简单的例子就是个人与信用卡的关系。信用卡依附于个人存在，当个人不存在（消失？Whatever）时，该个人对应的信用卡自然应该销毁。

解决这种循环强引用，我们用到`unowned`关键字，叫做无主引用。使用无主引用，你必须确保引用始终指向一个未销毁的实例。因此，用`unowned`修饰的属性必须用`let`声明。

```swift
class Customer {
    let name: String
    var card: CreditCard?
    init(name: String) {
        self.name = name
    }
}
class CreditCard {
    let number: UInt64
    unowned let customer: Customer
    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
    }
}
// 在CreditCard中，我们将customer修饰为unowned，因为我们可以清晰的知道，信用卡是依附于人存在的，生命周期短于或等于Customer。
```

上面两个例子基本涵盖了多数情况。想有更详尽的了解，可以参考一下[此文](http://swifter.tips/retain-cycle/)。

> 关于两者使用的选择，Apple 给我们的建议是如果能够确定在访问时不会已被释放的话，尽量使用 `unowned`，如果存在被释放的可能，那就选择用 `weak`。

---

### 闭包引起的循环引用

#### Problem

当把一个闭包赋值给某个属性时，是将这个闭包的引用赋值给了属性。而如果这个闭包里面引用了Self，那就会导致循环引用，在OC中我们在外部定义weakself，在Swift中应该怎么做呢？

#### Solution

`Swift`提供了一种优雅的方法来解决这个问题，称之为**闭包捕获列表（closure capture list）**

#### Discussion

**闭包捕获列表**是在定义闭包时同时定义捕获列表作为闭包的一部分，通过这种方式可以解决闭包和类实例之间的循环强引用。

```swift
lazy var someClosure: Void -> String = {
    [unowned self, weak delegate = self.delegate!] in
    // 这里是闭包的函数体
}
```

在闭包和捕获的实例总是互相引用并且总是同时销毁时，将闭包内的捕获定义为无主引用。相反的，在被捕获的引用可能会变为`nil`时，将闭包内的捕获定义为弱引用。弱引用总是可选类型，并且当引用的实例被销毁后，弱引用的值会自动置为`nil`。这使我们可以在闭包体内检查它们是否存在。

---

### 扩展(Extension)

#### Problem

在OC中有分类(category)的概念，在Swift中是否有同样的功能为已有的类或结构体、枚举定义更多的功能？

#### Solution

在Swift中，使用`extension`为一个已有的类、结构体、枚举类型或者协议类型添加新功能。

#### Discussion

使用关键字`extension`来什么扩展：

```Swift
extension SomeType {
    // 添加的新内容
}
```

Swift 中的扩展可以：

- 添加**计算型属性和计算型类型属性**

- 定义实例方法和类型方法：**值类型**要修改`self`或其属性，必须将该方法标注为`mutating`
- 提供新的构造器。要注意的是，类的扩展只能定义便利构造器，值类型的构造器可以使用原有的构造器。
- 定义下标
- 定义和使用新的嵌套类型
- 使一个已有类型符合某个协议，类不能在扩展中继承其他类

---

### 协议

#### Problem

Swift中协议怎么定义？有什么需要注意的地方？

#### Solution

Swift中使用协议（Protocol），协议的定义和类、结构体、枚举的定义非常相似：

```Swift
// 下面的代码中:[]表示可选
protocol SomeProtocol: FirstProtocol[, AnotherProtocol] {
    // 这里是协议的定义部分
    [static] var property: Type{set get} // 定义属性，static值类型属性(可选)，{set get}用来指明该属性是够可读写
    [static] [mutating] func someTypeMethod() // mutating表示可以在该方法中修改它所属的实例以及实例的任意属性的值
    init(someParamter: Int) // 若在遵循协议的类中实现构造器，都必须在构造器实现标上 required 修饰符，如果类已被标记为 final ，则不需要
    init?()
}
```

#### Discussion

协议的定义不复杂，本身并未实现任何功能，但是协议可以被当做一个成熟的类型来使用。协议可以像其他普通类型一样使用，使用场景如下：

- 作为函数、方法或构造器中的参数类型或返回值类型

  协议作为参数时，可以使用`&`来将两个协议合成到一个只在局部作用域有限的临时协议中。

  ```swift
  protocol Named {
      var name: String { get }
  }
  protocol Aged {
      var age: Int { get }
  }
  func wishHappyBirthday(to someone: Named & Aged) {
      // 使用someone遵循的协议中的name和age
  }
  struct Person: Named, Aged {
      var name: String
      var age: Int
  }
  wishHappyBirthday(to: Person(name:"Chris", age:23))
  ```

- 作为常量、变量或属性的类型

- 作为数组、字典或其他容器中的元素类型

我们可以在协议的继承列表中，通过添加`class`关键字来限制协议只能被类类型遵循，而结构体或枚举不能遵循该协议。`class` 关键字必须第一个出现在协议的继承列表中，在其他继承的协议之前：

```swift
protocol SomeClassOnlyProtocol: class, OtherProtocol {
    // 这里是类类型专属协议的定义部分
}
```

协议可以定义可选要求，遵循协议的类型可以选择是否实现这些要求。在协议中使用` optional` 关键字作为前缀来定义可选要求。协议和可选要求都必须带上`@objc`属性。标记 `@objc` 特性的协议只能被继承自 OC 类的类或者 `@objc` 类遵循，其他类以及结构体和枚举均不能遵循这种协议。使用可选要求时（例如，可选的方法或者属性），它们的类型会自动变成可选的。

```swift
@objc protocol CounterDataSource {
    // optional方法和属性会自动变成可选类型
    optional func incrementForCount(count: Int) -> Int
    optional var fixedIncrement: Int { get }
}
```

协议使用扩展可以为要求的属性、方法、下标提供默认实现：

```swift
extension Named {
    var name: String {
        return "[NoNameYet]"
    }
}
```

在扩展协议的时候，可以指定一些限制条件，只有遵循协议的类型满足这些限制条件时，才能获得协议扩展提供的默认实现。使用`where`来描述：

```swift
protocol Empty {
    
}
protocol Named {
    var name: String{get}
}
// where语句指：遵循Named的结构也遵循Empty时，提供name属性的默认实现。
extension Named where Self: Empty {
    var name: String {
        return "[NoNameYet]"
    }
}
struct Unknown: Named, Empty {
    
}
let u = Unknown()
u.name // [NoNameYet]
```

> 如果多个协议扩展都为同一个协议要求提供了默认实现，而遵循协议的类型又同时满足这些协议扩展的限制条件，那么将会使用限制条件最多的那个协议扩展提供的默认实现。

---

### 类型转换

#### Problem

在Swift中如何使用类型转换？怎么判断实例是够属于哪个类或是否符合某个协议？

#### Solution

类型转换在 Swift 中使用 `is` 和 `as` 操作符实现。这两个操作符提供了一种简单达意的方式去检查值的类型或者转换它的类型。

#### Discussion

`is`比较简单，是用来判断某个实例的类型是否是我们预期中的类型或检查实例是否符合某个协议。若实例属于那个类型，类型检查操作符返回` true`，否则返回 `false`。

```Swift
if object is SomeType {}
```

`as`相对来说比较复杂，其中涉及到向上转型和向下转型：

- 向上转型：子类转成父类，用`as`。

  ```Swift
  // 例子定义了父类Animal和子类Cat，由子类转成父类
  class Animal {}
  class Cat: Animal{}
  let cat = Cat()
  let animal = cat as Animal
  ```


- `as`还可以用在`switch`表达式的`case`中用来找出只知道是`Any`或`AnyObject`类型的常量或变量的具体类型。

  - `AnyObject`可以表示任何类类型。

    ```swift
    @objc protocol AnyObject {}
    ```

  - `Any` 可以表示任何类型，包括函数类型。

    ```swift
    typealias Any = protocol<>
    ```

  ```Swift
  var things = [Any]()
  things.append(0)
  things.append(0.0)
  things.append("hello")
  for thing in things {
      switch thing {
      case 0 as Int:
          print("Int 0")
      case 0 as Double:
          print("Double 0")
      case is String:
          print("is a string")
      default:
          break
      }
  }
  ```


- 向下转型：父类转成子类，或将实例向下转换成某个协议类型，用`as?`和`as!`。

  当你不确定向下转型可以成功时，用`as?`。总是返回一个可选值，并且向下转型如果是不可能的，则返回`nil`。

  ```swift
  class Dog: Animal{}
  let zoo: [Animal] = [Cat(), Dog()]
  for animal in zoo { // animal是Animal类型
      if let cat = animal as? Cat {
          print("meow~🐱")
      }
  }
  ```

  `as!`则是确定向下转型一定会成功时使用的。当你试图向下转型为一个不正确的类型时，强制形式的类型转换会触发一个运行时错误。

------

### 泛型

#### Problem

泛型代码让我们可以编写出任意类型、灵活可重用的函数及类型。在Swift中怎么使用泛型？

#### Solution

Swift的泛型可以使用于函数、类型、协议。

#### Discussion

泛型函数的定义如下：

```swift
func aFunction<T>(param: T) {
    // 函数的主体部分
}
```

类型参数指定并命名一个占位类型，并且紧随在函数名后面，使用一对尖括号括起来（例如上面的`<T>`）

在大多数情况下，类型参数具有一个描述性名字，例如 `Dictionary<Key, Value>` 中的` Key `和 `Value`，以及 `Array<Element>` 中的 `Element`，这可以告诉阅读代码的人这些类型参数和泛型函数之间的关系。

泛型类型是在类型名称后面指定占位类型：

```swift
struct SomeType<T> {
    // 使用类型T
    var item: T
    func aFunction(param: T) -> T {
        // 作为函数的参数和返回值类型
    }
}
// 创建实例
let s = SomeType<Int>()
```

当你扩展一个泛型类型的时候，你并不需要在扩展的定义中提供类型参数列表。原始类型定义中声明的类型参数列表在扩展中可以直接使用。

```Swift
extension SomeType {
    var anotherItem: T {
        
    }
}
```

可以在泛型函数和泛型类型中的类型添加一个特定的类型约束，做一些我们需要的限制。类型约束可以指定一个类型参数必须继承自指定类，或者符合一个特定的协议或协议组合。

```swift
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // 函数的函数体部分
}
```

将重用的思想用于协议，我们有时候也需要为协议定义泛型。但是上述定义方式并不能适用于协议，会导致编译错误。在协议中，我们使用关联类型，关键字`associatedtype`，来指定，代表的实际类型在协议被采纳时才会被指定。

```swift
protocol Container {
    associatedtype ItemType
    mutating func append(item: ItemType)
    var count: Int { get }
    subscript(i: Int) -> ItemType { get }
}
// 遵循协议时，实现协议要求的功能
struct IntStack: Container {
    typealias ItemType = Int // 指定关联类型为Int，实际上，如果删除这一行代码，一切仍可以正常工作，因为Swift的类型推断能力，由实现的函数可以推断出来。
    mutating func append(item: Int) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Int {
        return items[i]
    }
}
```

有了关联类型，我们的泛型函数可以通过`where`来添加强制约束：

```Swift
func allItemsMatch<C1: Container, C2: Container>
    (_ someContainer: C1, _ anotherContainer: C2) -> Bool
    where C1.ItemType == C2.ItemType, C1.ItemType: Equatable {
    // 函数主体部分
}
```

还记得我们说`extension`的时候提到，扩展可以限制满足某些条件时才提供协议的默认实现，也是通过`where`子句来声明，关联类型也可以用作`where`中的判断条件：

```swift
extension Container where ItemType: Equatable {
    // 当ItemType遵循Equatable时提供默认实现
}
```

---

