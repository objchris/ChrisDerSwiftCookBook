# Chris der Swift Cookbook ~

从头学Swift的读书笔记

Base on Swift Version 3.1

主要记录Swift学习过程中遇到的可能一时间难以记住的语法。可能会连带补充一些《The Swift Programming Language》中没有的内容。

以Cookbook的形式书写，作为Xcode代码段方便以后开发。



## 基础部分

### 整数

#### Problem

Swift中的整数是怎样表示的？

#### Solution

Int、Uint 

#### Discusstion

跟C的命名方式类似，比如8位无符号整数类型是`UInt8`，32位有符号整数类型是 `Int32 `。

而不同于C语言的是，Int、UInt系列不再是short、int、long等伪装的typedef。而是结构体，而不同的结构体中定义了跟Int、UInt相关的变量和函数，例如：

可以通过min和max得到该类型所能达到的最小最大值

```
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

### 元祖(Tuples)

#### Problem

如何在Swift的函数中返回多个值，或者将一组有关联的值“捆绑”在一起

#### Solution

使用元祖，可把多个值组合成一个复合值。元组内的值可以是任意类型，并不要求是相同类型。

#### Discussion

最常见的是HTTP状态码 👉 404 对应 Page Not Found

`let http404Error = (404, "Not Found")`

在分解的时候可以分解成变量

```
let (statusCode, statusMessage) = http404Error
statusCode
statusMessage
```

或者，通过下标数字去读取

`http404Error.0 `

也可以在定义元祖的时候就给其中的元素命名

`let http200Status = (statusCode: 200, description: "OK")`

使用时通过名字来获取

```
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

```
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

String 和 String! 的区别：

------

