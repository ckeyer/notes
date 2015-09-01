<!--
author: ckeyer
head: http://blog.ckeyer.com/blog/img/logo_l.jpg
date: 2015-09-01
title: Go语言反射规则-The Laws of Reflection
tags: 编程语言，Golang, reflect, 
category: 好文分享, Golang, 翻译
status: publish
summary: 由官方博客翻译而来，关于golang中反射的实现及原理。
-->

原文地址：[http://blog.golang.org/laws-of-reflection](http://blog.golang.org/laws-of-reflection)

### 介绍

反射在计算机的概念里是指一段程序审查自身结构的能力，主要通过类型进行审查。它是元编程的一种形式，同样也是引起混乱的重大来源。

在这篇文章里我们试图阐明Go语言中的反射是如何工作的。每种语言的反射模型是不同的(许多语言不支持反射），然而本文只与Go有关，所以我们接下来所提到的“反射”都是指Go语言中的反射。

### 类型与接口

由于反射是建立在类型系统(type system)上的，所以我们先来复习一下Go语言中的类型。

Go是一门静态类型的语言。每个变量都有一个静态类型，类型在编译的时后被知晓并确定了下来。

```
type MyInt int

var i int
var j MyInt
```
变量i的类型是int，变量j的类型是MyInt。虽然它们有着相同的基本类型，但静态类型却不一样，在没有类型转换的情况下，它们之间无法互相赋值。

接口是一个重要的类型，它意味着一个确定的的方法集合。一个接口变量可以存储任何实现了接口的方法的具体值(除了接口本身)。一个著名的例子就是io.Reader和io.Writer：

```
// Reader is the interface that wraps the basic Read method.
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Writer is the interface that wraps the basic Write method.
type Writer interface {
    Write(p []byte) (n int, err error)
}
```
如果一个类型声明实现了Reader（或Writer）方法，那么它便实现了io.Reader（或io.Writer）。这意味着一个io.Reader的变量可以持有任何一个实现了Read方法的的类型的值。

```
var r io.Reader
r = os.Stdin
r = bufio.NewReader(r)
r = new(bytes.Buffer)
// and so on
```
必须要弄清楚的一点是，不管变量r中的具体值是什么，r的类型永远是io.Reader：Go是静态类型的，r的静态类型就是io.Reader。

在接口类型中有一个极为重要的例子——空接口：

	interface{}
它表示了一个空的方法集，一切值都可以满足它，因为它们都有零值或方法。

有人说Go的接口是动态类型，这是错误的。它们都是静态类型：虽然在运行时中，接口变量存储的值也许会变，但接口变量的类型是永不会变的。我们必须精确地了解这些，因为反射与接口是密切相关的。

### 深入接口

Russ Cox在博客里写了一篇详细的文章，讲述了Go中的接口变量的意义。我们不需要列出全文，只需在这里给出一点点总结。

一个接口类型的变量里有两样东西：变量的的具体值和这个值的类型描述。更准确地来讲，这个实现了接口的值是一个基础的具体数据项，而类型描述了数据项里的所有类型。

如下所示：

```
var r io.Reader
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
if err != nil {
    return nil, err
}
r = tty
```
在此之后，r包含了(value, type)组合，(tty, *os.File)。值得注意的是，*os.File实现了Read以外的方法；虽然接口值只提供了Read方法，但它内置了所有的类型信息，这就是为什么我们可以么做：

```
var w io.Writer
w = r.(io.Writer)
```
上面的所展示表达式是一个类型断言，它断言了r中所包含的数据项实现了io.Writer，所以我们可以用它对w赋值。在此之后，w将与r一样，包含(tty, *os.File)组合。接口的静态类型决定了接口变量的哪些方法会被调用，即便也许它所含的具体值有一个更大的方法集。

接下来，我们可以这么做：

```
var empty interface{}
empty = w
```
我们的空接口变量将会在此包含同样的“组合”：(tty, *os.File)。这非常方便：一个空接口可以包含任何值和它的类型信息，我们可以在任何需要的时候了解它。

（在这里我们无需类型断言是因为w已经满足了空接口。在前面的例子中我们将一个值从一个Reader传到了Writer，因为Writer不是Reader的子集，所以我们需要使用类型断言。）

这里有一个重要细节：接口里“组合”的格式永远是（值，实体类型），而不是（值，接口类型）。接口不会包含接口值。

好了，现在让我们进入反射部分。

### 反射规则（一） - 从接口到反射对象

在基础上，反射是一个审查在接口变量中的(type, value)组合的机制。现在，我们需要了解reflect包中的两个类型：Type和Value，可以让我们访问接口变量的内容。reflect.TypeOf函数和reflect.ValueOf函数返回的reflect.Type和reflect.Value可以拼凑出一个接口值。（当然，从reflect.Value可以很轻易地得到reflect.Type，但现在还是让我们把Value和Type的概念分开来看。）

我们从TypeOf开始：

```
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x float64 = 3.4
    fmt.Println("type:", reflect.TypeOf(x))
}
```
这个程序打印了：

```
type: float64
```
看了这段代码你也许会想“接口在哪？”，这段程序里只有float64的变量x，并没有接口变量传进reflect.TypeOf。其实它是在这儿：在godoc reports的reflect.TypeOf的声明中包含了一个空接口：

```
TypeOf returns the reflection Type of the value in the interface{}.
func TypeOf(i interface{}) Type
```
当我们调用reflect.TypeOf(x)时，作为参数传入的x在此之前已被存进了一个空接口。而reflect.TypeOf解包了空接口，恢复了它所含的类型信息。

相对的，reflect.ValueOf函数则是恢复了值（从这里开始我们将修改例子并且只关注于可执行代码）：

```
var x float64 = 3.4
fmt.Println("value:", reflect.ValueOf(x))
```
打印：

	value: <float64 Value>
reflect.Type和reflect.Value拥有许多方法让我们可以审查和操作接口变量。一个重要的例子就是Value有一个Type方法返回reflect.Value的Type。另一个例子就是，Type和Value都有Kind方法，它返回一个常量，这个常量表示了被存储的元素的排列顺序：Uint, Float64, Slice等等。并且，Value的一系列方法（如Int或Float），能让我们获取被存储的值（如int64或float64）:

```
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())
fmt.Println("kind is float64:", v.Kind() == reflect.Float64)
fmt.Println("value:", v.Float())
```
打印：

```
type: float64
kind is float64: true
value: 3.4
```
有一些方法如SetInt和SetFloat涉及到了“可设置”(settability)的概念，这是反射规则的第三条，我们将在后面讨论。

反射库有两个特性是需要指出的。其一，为了保持API的简洁，Value的Getter和Setter方法是用最大的类型去操作数据：例如让所有的整型都使用int64表示。所以，Value的Int方法返回一个int64的值，SetInt需要传入int64参数；将数值转换成它的实际类型在某些时候是有必要的：

```
var x uint8 = 'x'
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())                            // uint8.
fmt.Println("kind is uint8: ", v.Kind() == reflect.Uint8) // true.
x = uint8(v.Uint())                                       // v.Uint returns a uint64.
```
其二，反射对象的Kind方法描述的是基础类型，而不是静态类型。如果一个反射对象包含了用户定义类型的值，如下：

```
type MyInt int
var x MyInt = 7
v := reflect.ValueOf(x)
```
虽然x的静态类型是MyInt而非int，但v的Kind依然是reflect.Int。虽然Type可以区分开int和MyInt，但Kind无法做到。

### 反射规则（二） - 从反射对象到接口

如同物理学中的反射一样，Go语言的反射也是可逆的。

通过一个reflect.Value我们可以使用Interface方法恢复一个接口；这个方法将类型和值信息打包成一个接口并将其返回：

```
// Interface returns v's value as an interface{}.
func (v Value) Interface() interface{}
```
于是我们得到一个结果：

```
y := v.Interface().(float64) // y will have type float64.
fmt.Println(y)
```
以上代码会打印由反射对象v表现出的float64值。

然而，我们还可以做得更好。fmt.Println和fmt.Printf的参数都是通过interface{}传入的，传入之后由fmt的私有方法解包（就像我们前面的例子所做的一样）。正是因为fmt把Interface方法的返回结果传递给了格式化打印事务（formatted print routine），所以程序才能正确打印出reflect.Value的内容：

```
fmt.Println(v.Interface())
```
（为什么不是fmt.Println(v)？因为v是一个reflect.Value，而我们想要的是它的具体值） 由于值的类型是float64，我们可以用浮点格式化打印它：

```
fmt.Printf("value is %7.1e\n", v.Interface())
```
并得出结果：

```
3.4e+00
```
在这里我们无需对v.Interface()做类型断言，这个空接口值包含了具体的值的类型信息，Printf会恢复它。

简而言之，Interface方法就是ValueOf函数的逆，除非ValueOf所得结果的类型是interface{}

重申一遍：反射从接口中来，经过反射对象，又回到了接口中去。 (Reflection goes from interface values to reflection objects and back again.)

### 反射规则（三） - 若要修改反射对象，值必须可设置

第三条规则是最微妙同时也是最混乱的，但如果我们从它的基本原理开始，那么一切都不在话下。

以下的代码虽然无法运行，但值得学习：

```
var x float64 = 3.4
v := reflect.ValueOf(x)
v.SetFloat(7.1) // Error: will panic.
```
如果你运行这些代码，它会panic这些神秘信息：

```
panic: reflect.Value.SetFloat using unaddressable value
```
问题在于7.1是不可寻址的，这意味着v就会变得不可设置。“可设置”(settability)是reflect.Value的特性之一，但并非所有的Value都是可设置的。

Value的CanSet方法返回一个布尔值，表示这个Value是否可设置：

```
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("settability of v:", v.CanSet())
```
打印：

```
settability of v: false
```
对一个不可设置的Value调用的Set方法是错误的。那么，什么是“可设置”？

“可设置”和“可寻址”(addressable)有些类似，但更严格。一个反射对象可以对创建它的实际内容进行修改，这就是“可设置”。反射对象的“可设置性”由它是否拥有原项目(orginal item)所决定。

当我们这样做的时候：

```
var x float64 = 3.4
v := reflect.ValueOf(x)
```
我们传递了一份x的拷贝到reflect.ValueOf中，所以传到reflect.ValueOf的接口值不是由x，而是由x的拷贝创建的。因此，如果下列语句

```
v.SetFloat(7.1)
```
被允许执行成功，它将不会更新x，即使看上去v是由x创建的。相反，它更新的是存于反射值中的x拷贝，x本身将不会受到影响。这是混乱且毫无用处的，所以这么做是非法的。“可设置”作为反射的特性之一就是为了预防这样的情况。

这虽然看起来怪异，但事实恰好相反。它实际上是一个我们很熟悉的情形，只是披上了一件不寻常的外衣。思考一下x是如何传递到一个函数里的：

	f(x)
我们不会指望f能够修改x因为我们传递的是一个x的拷贝，而非x。如果我们想让f直接修改x我们必须给我们的函数传入x的地址（即是x的指针）：

	f(&x)
这是直接且熟悉的，反射的工作方式也与此相同。如果我们想用反射修改x，我们必须把值的指针传给反射库。

开始吧。首先我们像刚才一样初始化x，然后创建一个指向它的反射值，命名为p：

```
var x float64 = 3.4
p := reflect.ValueOf(&x) // Note: take the address of x.
fmt.Println("type of p:", p.Type())
fmt.Println("settability of p:", p.CanSet())
```
目前的输出是：

```
type of p: *float64
settability of p: false
```
反射对象p不是可设置的，但我们想要设置的不是它，而是*p。 为了知道p指向了哪，我们调用Value的Elem方法，它通过指针定向并把结果保存在了一个Value中，命名为v：

```
v := p.Elem()
fmt.Println("settability of v:", v.CanSet())
```
现在的v是一个可设置的反射对象，如下所示：

	settability of v: true
因为它表示x，我们终于可以用v.SetFloat来修改x的值了：

```
v.SetFloat(7.1)
fmt.Println(v.Interface())
fmt.Println(x)
```
正如意料中的一样：

```
7.1
7.1
```
反射可能很难理解，但它所做的事正是编程语言所做的，尽管通过反射类型和值可以掩饰正在发生的事。 记住，用反射修改数据的时候需要传入它的指针哦。

### 结构体

在前面的例子中，v并不是指针本身，它只是来源于此。 我们一般在使用反射去修改结构体字段的时候会用到。只要我们有结构体的指针，我们就可以修改它的字段。

这里有一个解析结构体变量t的例子。我们用结构体的地址创建了反射变量，待会儿我们要修改它。然后我们对它的类型设置了typeOfT，并用调用简单的方法迭代字段（详情请见reflect包）。 注意，我们从结构体的类型中提取了字段的名字，但每个字段本身是正常的reflect.Value对象。

```
type T struct {
    A int
    B string
}
t := T{23, "skidoo"}
s := reflect.ValueOf(&t).Elem()
typeOfT := s.Type()
for i := 0; i < s.NumField(); i++ {
    f := s.Field(i)
    fmt.Printf("%d: %s %s = %v\n", i,
        typeOfT.Field(i).Name, f.Type(), f.Interface())
}
```
程序输出：

```
0: A int = 23
1: B string = skidoo
```
关于可设置性还有一点需要介绍：T的字段名是大写（字段可导出/公共字段）的原因在于，结构体中只有可导出的的字段是“可设置”的。

因为s包含了一个可设置的反射对象，我们可以修改结构体字段：

```
s.Field(0).SetInt(77)
s.Field(1).SetString("Sunset Strip")
fmt.Println("t is now", t)
```
结果：

	t is now {77 Sunset Strip}
如果我们修改了程序让s由t（而不是&t）创建，程序就会在调用SetInt和SetString的地方失败，因为t的字段是不可设置的。

### 结论

再次列出反射法则：

- 反射从接口值到反射对象中(Reflection goes from interface value to reflection object.)
- 反射从反射对象到接口值中(Reflection goes from reflection object to interface value.)
- 要修改反射对象，值必须是“可设置”的(To modify a reflection object, the value must be settable.)

一旦你了解反射法则，Go就会变得更加得心应手（虽然它仍旧微妙）。这是一个强大的工具，除非在绝对必要的时候，我们应该谨慎并避免使用它。


