---
title: golang 学习笔记（一）——语法篇
date: 2019-06-08T21:53:54+08:00
categories: ["Golang"]
tags : ["Golang"]
toc: true
featured_image : ""
keywords: ["Golang"]
description : "golang 学习笔记（一）——语法篇"
---

## golang 应⽤程序⼊⼝

1. 必须是 main 包：package main
2. 必须是 main ⽅法：func main()
3. ⽂件名不⼀定是 main.go

## 退出返回值

与其他主要编程语⾔的差异

- Go 中 main 函数不⽀持任何返回值
- 通过 os.Exit 来返回状态

## 获取命令⾏参数
与其他主要编程语⾔的差异

- main 函数不⽀持传⼊参数
  func main(~~arg []string~~)
- 在程序中直接通过 os.Args 获取命令⾏参数

## 变量量赋值
**与其他主要编程语⾔言的差异**

1. 赋值可以进⾏行行⾃自动类型推断
2. 在⼀一个赋值语句句中可以对多个变量量进⾏行行同时赋值



###  常量量定义

**与其他主要编程语⾔言的差异**

- 快速设置连续值
```go
const (
  Monday = iota + 1			//Monday =  0 + 1
  Tuesday				//Tuesday =  iota + 1 =  1 + 1
  Wednesday			//Wednesday =  iota + 1 =  2 + 1
  Thursday			//Thursday =  iota + 1 =  3 + 1
  Friday			//Friday =  iota + 1 =  4 + 1
  Saturday			//Saturday =  iota + 1 =  5 + 1
  Sunday			//Sunday =  iota + 1 =  6 + 1
)
const (
  Open = 1 << iota
  Close
  Pending
)
```

`iota`是`golang`语言的常量计数器,只能在常量的表达式中使用。
`iota`在`const`关键字出现时将被重置为`0`(`const`内部的第一行之前)，`const`中每新增一行常量声明将使`iota`计数一次(`iota`可理解为const语句块中的行索引)。
使用`iota`能简化定义，在定义枚举时很有用。

- 每次 const 出现时，都会让 iota 初始化为0

```go
const a = iota // a=0
const (
  b = iota     //b=0
  c            //c=1
)
```

- 可跳过的值
```go
//如果两个const的赋值语句的表达式是一样的，那么可以省略后一个赋值表达式。
type AudioOutput int

const (
    OutMute AudioOutput = iota // 0
    OutMono                    // 1
    OutStereo                  // 2
    _
    _
    OutSurround                // 5
)
```

- 定义在一行的情况

```go
const (
    Apple, Banana = iota + 1, iota + 2
    Cherimoya, Durian   // = iota + 1, iota + 2
    Elderberry, Fig     //= iota + 1, iota + 2
)
```

iota 在下一行增长，而不是立即取得它的引用。

```go
// Apple: 1
// Banana: 2
// Cherimoya: 2
// Durian: 3
// Elderberry: 3
// Fig: 4
```

## 数据类型

### 基本数据类型
```go
 bool
 string
 int  int8  int16  int32  int64
 uint uint8 uint16 uint32 uint64 uintptr
 byte // alias for uint8
 rune // alias for int32,represents a Unicode code point
 float32 float64
 complex64 complex128
```
### 类型转化

**与其他主要编程语⾔言的差异**

1. Go 语⾔言不不允许隐式类型转换
2. 别名和原有类型也不不能进⾏行行隐式类型转换

###  类型的预定义值
1. math.MaxInt64
2. math.MaxFloat64
3. math.MaxUint32

### 指针类型

**与其他主要编程语⾔言的差异**

1. 不不⽀支持指针运算
2. string 是值类型，其默认的初始化值为空字符串串，⽽而不不是 nil


## == 运算符

在 golang 中 == 运算符在比较数组时，不同于其他其他语言是比较引用，只要数组的维数和个数是相等的话，可以使用 == 运算符来进行比较，只要每一位的数值相同，则相等。

## 按位清零(&^)

go 独有的位运算符——按位清零（&^）只要右边二进制位位 1，无论左边对应二进制位是 1 还是 0，结果都是 0，如果右边二进制位为 0，那么左边二进制位是什么，结果就取什么。

1 &^ 0 -- 1
1 &^ 1 -- 0
0 &^ 1 -- 0
0 &^ 0 -- 0

0b1100 &^ 0b0110 = 0b1000

## switch 条件

1. 条件表达式不限制为常量或者整数；
2. 单个 case 中，可以出现多个结果选项, 使⽤逗号分隔；
3. 与 C 语⾔等规则相反，Go 语⾔不需要⽤ break 来明确退出⼀个 case；
4. 可以不设定 switch 之后的条件表达式，在此种情况下，整个 switch 结
   构与多个 if…else… 的逻辑作⽤等同

```go
func TestSwitch(t *testing.T){
	for i:=0; i<5; i++ {
		switch i {
		case 0, 2:
			t.Log("Even")
		case 1, 3:
			t.Log("Odd")
		default:
			t.Log("not 0-3")
		}
	}
}

func TestSwitchCondition(t *testing.T){
	for i:=0; i<5; i++ {
		switch  {
		case i % 2 == 0:
			t.Log("Even")
		case i % 2 != 0:
			t.Log("Odd")
		default:
			t.Log("unknow")
		}
	}
}
```

## 数组

- 数组的声明

```go
var a [3]int //声明并初始化为默认零值
a[0] = 1
b := [3]int{1, 2, 3} //声明同时初始化
c := [2][2]int{{1, 2}, {3, 4}} //多维数组初始化
```

- 数组元素遍历

```go
func TestTravelArray(t *testing.T) {
    a := [...]int{1, 2, 3, 4, 5} //不指定元素个数
    for idx/*索引*/, elem/*元素\*/ := range a {
    fmt.Println(idx, elem)
    }
}
```

- 数组截取
  a[开始索引(包含), 结束索引(不包含)]

```go
a := [...]int{1, 2, 3, 4, 5}
a[1:2] //2
a[1:3] //2,3
a[1:len(a)] //2,3,4,5
a[1:] //2,3,4,5
a[:3] //1,2,3

```

## 切片

### 切片的数据结构
![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/golang-slice_struct.jpg)


切片的数据结构包含三个部分：
- 第一个是一个指针，指向一片连续的存储空间，也就是指向一个数组
- 第二个是 `len` 属性，记录的是 `slice` 中元素的个数，也是我们可以访问到的元素的个数
- 第三个是 `cap` 属性，记录的是内部数组的容量

切片是可变长的，当往切片中添加的元素超过 `cap` 属性的容量时，会重新生成一个 `2 * cap` 容量的切片，并将原有切片里的值拷贝过来。

### 切⽚片声明
```go
   var s0 []int
   s0 = append(s0, 1)
	 s := []int{}
   s1 := []int{1, 2, 3}
   s2 := make([]int, 2, 4)
/*[]type, len, cap 其中len个元素会被初始化为默认零值，未初始化元素不不可以访问
*/
```

### 切片共享存储结构

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/golang-slice_share_struct.jpg)


当多个切片都指向一片相同的存储空间时，其中一个 slice 对元素做出修改后，其他的切片都会受到影响。

### 切片和数组的区别

- 数组是定长的而切片是不定长的
- 数组与数组之间可以用 `==` 来进行比较，切片只能与 `nil` 进行比较

## Map

### Map 声明

```go
m := map[string]int{"one": 1, "two": 2, "three": 3}

m1 := map[string]int{}
m1["one"] = 1

m2 := make(map[string]int, 10 /*Initial Capacity*/) //为什什么不不初始化len?
//len 所指的单元格初始化时都会被赋值默认零值，而 Map 是不能赋值默认零值的，所以 Map 不能初始化 len 属性
```

Map 元素的访问 与其他主要编程语⾔言的差异

在访问的 `Key` 不存在时，并不会抛出异常，而是仍会返回零值，所以不能通过返回 `nil` 来判断元素是否存在

```go
if v, ok := m["four"]; ok {
  t.Log("four", v)
} else {
  t.Log("Not existing")
}
```

### Map 遍历

```go
m := map[string]int{"one": 1, "two": 2, "three": 3}
for k, v := range m {
  t.Log(k, v)
}
```


## 函数：一等公⺠

**与其他主要编程语⾔言的差异**

1. 可以有多个返回值
2. 所有参数都是值传递:slice，map，channel 会有传引⽤用的错觉
3. 函数可以作为变量量的值
4. 函数可以作为参数和返回值

### 函数:可变参数及 defer

**可变参数**

```go
func sum(ops ...int) int {
  s := 0
  for _, op := range ops {
			s += op
	}
	return s
}
```

**defer 函数**

```go
	func TestDefer(t *testing.T) {
		defer func() {
				t.Log("Clear resources")
	}()
	t.Log("Started")
	panic("Fatal error”) //defer仍会执⾏行行
}
```