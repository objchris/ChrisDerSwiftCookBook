# Chris der Swift Cookbook ~

从头学Swift的读书笔记

Now base on 《The Swift Programming Language》 Version 3.1 

主要记录Swift学习过程中遇到的可能一时间难以记住的语法。可能会连带补充一些《The Swift Programming Language》中没有的内容。

以Cookbook的形式书写，带🥐标志的作为Xcode代码段方便开发使用。

---



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

---

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
4. 根据是否抛出错误给调用者处理，决定是否添加`throws`关键字，**错误处理**部分将详细讲解
5. 返回类型

调用时需要注意：

1. 如果上述3.1中参数标签为`_`，则不需要写参数名称。
2. 若有3.2中的`inout`属性，只能传入变量`var`，且在参数前需要加上`&`，这一点跟C/C++有点像
3. 有上述4中的`throws`关键字，需要用`do-try-catch`代码块，**错误处理**部分将详细讲解
4. 函数是**第一等类型**，意味着可以是函数的参数以及返回值。类型由参数类型和返回类型决定，如：` func add(_ a:Int, _ b: Int)->Int ` 👉`(Int,Int)->Int`，二者都没有的函数的类型为：`()->Void`。
5. 因为函数可以作为参数，当一个函数的最后一个参数是另一个参数的时候，在调用的时候可以作为不写在参数括号`()`中，而是作为**尾随闭包**用`{}`包括起来添加在函数之后。即：

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



## 集合类型

### 数组（Array）

#### Problem

Swift中如何表示一个数组？有什么特性？

#### Solution

Array，若是分配给变量(`var`修饰)，则可以添加、删除、更改数组中的项。若是分配给常量(`let`修饰)，则大小和内容都不能改变。

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

```Swift
arrayC[0] = 2 // 此时arrayC中的项为[2,2,3,4,5]
arrayC[2...4] = [6]	// 此时arrayC中的项为[1,2,6]
```

- `insert(_:at:)`在某个**具体索引前**添加数据。
- `remove(at_)`**删除并返回**某个索引对应的数据项，后面的项往前移动。

关于Array到NSArray的桥接，未完待续。

---

### 集合（Set）

#### Problem

Swift中如何表示一个集合？有什么特性？

#### Solution

集合(Set)用来存储相同类型并且没有确定顺序的值。当集合元素顺序不重要时或者希望确保每个元素只出现一次时可以使用集合。

#### Discussion

集合是通过Hash来确定一个值当前是否存储在其中。所以为了存储在集合中，该类型必须遵循`Hashable`协议。`Hashable`是一个`Protocal`，其中只有一个计算属性`hashValue`，用于返回计算后的哈希值。要选择一个对类型中包含的属性来说较为合适的哈希算法。见  **如何选取好的Hash算法**  

再者`Hashable`是符合`Equatable`协议的。所以必须重载`==`来告诉`Set`（或下面将提到的`Dictionary`），如何判断两个元素是相同的。

当你将没有遵循`Hashable`协议的类型加入到`Set`中，将会得到运行错误。

总的来说，看下面的例子：

```Swift
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

	static func == (lhs: GridPoint, rhs: GridPoint) -> Bool {
		return lhs.x == rhs.x && lhs.y == rhs.y
	}
}
```

定义集合类型只能用`Set<Element>`，`Element`表示`Set`中存储的类型。

像`Array`一般，我们可以使用字面量来构造一个`Set`，但需要注意的是：在定义的时候需要显式指定变量或常量为`Set<Element>`，否则会被认为是数组。

```Swift
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

```Swift
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

---

### 字典

#### Problem

Swift中如何表示一个字典？有什么特性？

#### Solution

字典`Dictionary`是键值对，是一种存储多个相同类型的值的容器。可以通过键来得到存储在字典中的值。

#### Discussion

- Swift中的字典使用`Dictionary<Key, Value>`定义。
- 可以使用`[Key:Value]`的形式创建一个字典对象。
- 和`Array`一样，我们可以使用字面量来构造字典。

```Swift
let dic1 = Dictionary<String, Int>()
let dic2 = [Int: String]()
var dic3 = [1:"string1",2:"string2",3:"string3"]
```

`Dictionary`的键`Key`同`Set`中的值，遵循`Hashable`协议。字典中的键是唯一的。

对`Dictionary`的操作和函数基本与`Array`相似。使用属性`count`获取字典中的数据项数量，使用属性`isEmpty`检查是否数据项是否为0。

不同于`Array`使用Int下标，`Dictionary`通过`Key`来获取或修改对应的值。除了用`[]`来修改值之外，若想在修改的同时获取修改前的旧值，可以使用`updateVale(_:forKey:)`，该函数会返回`String?`，若有旧值则返回，没有返回nil，`updateVale(_:forKey:)`在字典中添加该数据项。

```Swift
dic3[1] = "string that change"	// 上文定义dic3的时候使用var，所以dic3才可以添加删除修改元素
let stringThatChange = dic3.updateValue("string that change", forKey:1)		//stringThatChange 是 String?
if let result = dic3[1]	{	// 要注意： result 是 String?
	...
}
```

删除元素可以使用`removeValue(forKey:)`，该方法返回被移除的值或不存在值的时候返回`nil`。

像`Array`中的每一项可以以`(Index, Value)`元祖的形式返回，`Dictionary`中的每一项都可以以`(Key, Value)`元祖的形式返回。

若只需使用字典的键或值，可以使用属性`keys`或`values`来单独获取。 

---

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

---

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

! 不仅可以用来解析 ? 声明的可选类型，还可以用来声明隐式解析可选类型。     

隐式解析可选类型在第一次赋值确定之后一定有值的时候，使用其不需要再在变量或常量后面加上 ! ，但是需要特别注意的是，一定要确保有值，不然会得到运行时错误。

在可选类型后加上空白运算符`??`将对可选类型进行空判断，若该值为nil，则返回`??`后跟上的默认值：

```swift
let value = mayBeNil ?? defaultValue
// 等同于
let value = (mayBeNil != nil) ? mayBeNil! : defaultValue
```

---

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

```
switch condition {
	case A : 
		代码段A 
		fallthrough
	case B : 
		代码段B
}
```

`case`分支的模式可以是一个区间，区间可以简写为`1...5`表示1到5的所有数字，`1..<5`表示1到4的所有数字，因此

```Swift
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

---

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

---

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
print("\(something)")		// 合法的


if let something = mayBeNil {
	// something 只能在此代码段中使用
}
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

```Swift
{(paramters) -> returnType in
	statements
}
```

- 根据上下文推断参数和返回值类型

譬如，我们在说明函数的时候写过这么一个闭包

```Swift
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

```Swift
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

```Swift
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

```Swift
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

