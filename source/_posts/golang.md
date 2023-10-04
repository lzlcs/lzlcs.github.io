---
title: Golang 学习笔记
date: 2023-10-2 10:38:00
categories:
- Languages
tags: 
- Golang
- Languages
---

目标分布式系统, 此文只限于基本语法和并发部分

# 安装与运行环境

```bash
export GOROOT=$HOME/go
export PATH=$PATH:$GOROOT/bin
export GOPATH=$HOME/Applications/Go
sudo apt-get install bison ed gawk gcc libc6-dev make
```
在官方页面下载 `go<VERSION>.linux-amd64.tar.gz`
```bash
tar -xzf go1.21.1.linux-amd64.tar.gz
export PATH=$PATH:/home/lozical/go/bin/
```

# 基本结构和基本数据类型

## 文件名, 关键字, 标识符

1. `go` 的源文件后缀是 `.go`, 文件名中不包含空格及其他特殊字符
2. 标识符的规则同 C, 以字符开头
3. 关键字和预定义标识符
![image](https://github.com/lzlcs/image-hosting/raw/master/image.3vqjjb3pd5g0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.14x4x4ekqwf4.webp)

## Go 程序的基本结构和要素

**包**

每个 `go` 文件只属于一个包, 每个包可以包含多个 `go` 文件 \
所以按照惯例, 一个文件夹下的所有 `go` 文件都属于一个包\
在源文件非注释的第一行必须指定该文件属于哪个包, 注意所有的包名都要使用小写字母

`go` 的标准库中包含大量的包, 但也可以自己创建 \
每个包是编译的一个独立单元, 包中包含的文件必须一起编译

如果这个包中的文件被更改, 包含这个包的所有程序都要重新编译 \
每一段代码只会被重新编译一次

以下是引用包的一些方法
```go
import "fmt"

import "fmt"
import "os"

import "fmt"; import "os"

import (
    "fmt"
    "os"
)

// 设置别名
import f "fmt"
```

导入包的规则
1. 最好按照字母顺序在源代码中导入包
2. 导入了包却没有使用会报错
2. 如果包名不是以 `.` 和 `/` 开头, `go` 会在全局文件查找
3. 如果包名以 `./` 开头, `go` 会在相对路径中查找
4. 如果包名以 `/` 开头, `go` 会在绝对路径中查找

可见性规则: 如果标识符以大写字母开头, 那么这个对象就可以被外部引用这个包的程序使用(使用 `包名.标识符`), 这被称为导出(`public`), 小写字母开头只能在包内访问(`private`)

**函数**

注意大括号一定要跟函数定义在同一行, 因为 `go` 会给你加分号
```go
func functionName(parameter_list) (return_value_list) {
   …
}

func functionName(param1 type1, param2 type2, ...) ([ret1] type1, ...) {
   …
}
```

函数也可以作为值传递, `func(param1 type1, ...)([ret1] type1, ...)`

如果函数需要被外部调用, 那么首字母大写, 使用 `Pascal` 命名法 \
如果函数不需要被外部调用, 那么首字母小写, 使用驼峰命名法

程序正常退出的代码为 0, 异常为 1, 可以用来检测程序是否运行成功

**注释**

同 C 风格的注释

在包名前面的注释被默认为这个包的说明, 使用 `godoc` 查看

几乎所有被导出的对象都要有一个合理的注释 \
如果这种注释出现在函数前面, 则要以函数名作为开头 供 `godoc` 收集

**类型**

变量声明
```go
var name type = value
```
函数返回值
```
return var1, var2
```
重命名类型
```go
type STR string

type (
	IZ int
	FL float64
)
```

**一般结构**

1. `package xxx`
2. `import xxx`
3. 定义常量, 变量, `type xxx xxx`
4. `init` 函数, 每个含有该函数的包都会第一个执行该函数
5. 如果当前包是 `main` 包, 则定义 `main` 函数
6. 定义其余的函数

**类型转换**

```go
a := int(b)
```

类似 C 中的类型转换, 注意精度丢失的问题

## 常量

```go
const name [type] = value
```

`go` 会根据 `value` 的值自动推断类型 

数字常量没有大小和符号, 可以使用任意精度而不会溢出

常量允许并行赋值
```go
const (
    a, b, c = "1", 2, 3.0
    d, e, f = 1, 1, 1
)
```

**`iota`**

用于枚举
```go
const (
    zero = iota
    one // 1
    two // 2
)
```

`iota` 每次遇到 `const` 就会置为 0, 然后递增

## 变量

各种类型如下
```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // uint8 的别名

rune // int32 的别名
    // 表示一个 Unicode 码点

float32 float64

complex64 complex128
```
```go
var a, b int;
var c, d = 3, "4"

// 类型明确的时候可以使用 := (不能在函数外使用, 不能用于常量)
e := 3
// 类型推导, f 类型与 a 相同, 为 int
f := a
```
`go` 中所有变量都会被初始化为 0 值, 字符串为 `""`, 布尔为 `false`

## 流程控制语句

循环
```go
for i := 0; i < 10; i++ {

}
// 可选的 i := 0 和 i++
for i < 10 {

}
// 死循环
for {

}
```
分支
```go
// if 后面可以执行一条语句
// v 的作用域仅限于 if-else 内
if v := 10; v < lim {

} else {

}

if w < lim {

}

// 每个 case 后面隐含 break
// 可以加 fallthrough 关键字去除 break
switch os := runtime.GOOS; os {
case "linux": 
    fmt.Println("Linux")
case "windows":
    fmt.Println("Windows")
default:
    fmt.Println("Other")
}

// 没有条件的 switch, 类似一连串 if-else
switch {
case a < b:
    ...
case a == b:
    ...
case a > b:
    ...
}
```

```go
package main

import "fmt"

func main() {
    // defer 语句会立刻求值, 但是会在函数返回之后才执行
	defer fmt.Println("world")
    // 更多的 defer 会组成栈, 先入后出地执行
	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}
	fmt.Println("hello")
}

```

## 指针

基本操作同 C, `&val`, `*p`, 但是没有指针运算

## 结构体

```go
type name struct {
    a int
    b float64
    c string
}

func main() {
    v := name{1, 2.0, "3"}
    fmt.Println(v.a);

    p := &v;
    // 正常的指针用法
    fmt.Println((*p).b)
    // 隐式间接引用也可以
    fmt.Println(p.c)
}
```
```go
type Vertex struct {
	X, Y int
}
// 未说明的自动赋值为 0 值
var (
	v1 = Vertex{1, 2}  // 创建一个 Vertex 类型的结构体
	v2 = Vertex{X: 1}  // Y:0 被隐式地赋予
	v3 = Vertex{}      // X:0 Y:0
	p  = &Vertex{1, 2} // 创建一个 *Vertex 类型的结构体（指针）
)
```

## 数组

```go
var a [2]string
a[0] = "Hello"
a[1] = "World"
fmt.Println(a[0], a[1])
fmt.Println(a)

primes := [6]int{2, 3, 5, 7, 11, 13}
fmt.Println(primes)

// 切片, 左闭右开, s -> [3 5 7]
// 切片不存储任何数据, 修改切片的值, 对应数组的值也会被修改
// var s[] int = primes[a:b] a 默认值为 0, b 默认值为数组长度
var s []int = primes[1:4]
fmt.Println(s)

// 创建数组顺带切片
r := []bool{true, false, true, true, false, true}

// 切片的长度是切片中元素数量, 容量是从第一个元素到原数组结尾的元素个数
// 所以如果切片舍弃了前面的元素, 那么就不能恢复了
fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)

// 第二个参数为长度, 第三个参数为容量, 这是创建动态数组的方式
e := make([]int, 5, 5)

e = append(e, 2, 3, 4, 5)
// 当原数组的容量不够的时候, 切片就会指向一个新复制过来且扩充了容量的数组

// range 类似 py 中 enumerate
for i, v := range pow {
    fmt.Printf("2**%d = %d\n", i, v)
}
```

## 映射

映射的零值为 `nil`, 没有键也不能添加键
```go
var m map[string]int

func main() {
	m = make(map[string]int)
	m["Bell Labs"] = 3
	fmt.Println(m["Bell Labs"])
}
```

```go
var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}
```
创建, 修改, 获取, 删除映射
```go
package main

import "fmt"

func main() {
	m := make(map[string]int)

	m["Answer"] = 42
	fmt.Println("The value:", m["Answer"])

	m["Answer"] = 48
	fmt.Println("The value:", m["Answer"])

	delete(m, "Answer")
	fmt.Println("The value:", m["Answer"])

	v, ok := m["Answer"]
	fmt.Println("The value:", v, "Present?", ok)
}
```
## 方法

```go
type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}
```

使用指针的原因: 可以修改变量, 传递的时候无需复制

## 接口

接口名通常为 `*er`

可以实现多态

```go
// 矩形
type Rectangle struct {
    width, length int
}

func (r *Rectangle) Area() float64 {
    return float64(r.width * r.length)
}

// 圆形
type Circle struct {
    radius float64
}

func (r *Circle) Area() float64 {
    return math.Pi * r.radius * r.radius
}

// 统一接口
type Shaper interface {
    Area() float64
}

func main() {
    var a, b Shaper = &Rectangle{3, 4}, &Circle{4}

    fmt.Println(a.Area())
    fmt.Println(b.Area())
}
```

## 类型断言

```go
t, ok := i.(T)
```

## 类型选择

```go
switch v := i.(type) {
case int:
    fmt.Printf("Twice %v is %v\n", v, v*2)
case string:
    fmt.Printf("%q is %v bytes long\n", v, len(v))
default:
    fmt.Printf("I don't know about type %T!\n", v)
}
```

## `Stringer`

用来打印自己的接口, 类似 `py` 中 `__str__`

```go
package main

import "fmt"

type IPAddr [4]byte

// TODO: 给 IPAddr 添加一个 "String() string" 方法
func (i IPAddr) String() string {
	return fmt.Sprintf("%d.%d.%d.%d", i[0], i[1], i[2], i[3])
}
	
func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}
```

## `Error`

用来输出错误信息
```go
package main

import (
	"fmt"
	"math"
)

type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {
	return fmt.Sprintf("cannot Sqrt negative number: %f", float64(e))
}

func Sqrt(x float64) (float64, error) {
	if x < 0 {
		return 0, ErrNegativeSqrt(x)
	}

	return math.Sqrt(x), nil
}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))
}
```

# 同步

`goroutine` 是由 `go` 管理的轻量级线程

```go
// 启动 goroutine 执行 f(x, y, z)
go f(x, y, z)
```

## 信道

```go
package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // 将和送入 c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // 从 c 中接收

	fmt.Println(x, y, x+y)
}
```
信道可以执行缓冲区大小\
缓冲区满的时候发送数据会阻塞\
缓冲区空的时候接收数据会阻塞

```go

package main

import (
	"fmt"
)

func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
    // close 关闭信道, 此时不能在发送数据, 但是可以接收数据
    // 一般不需要关闭信道, 除非接收者使用 range, 为了终止循环需要使用 close
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}

```

`select` 使一个 `goroutine` 等待多个通信操作 \
如果有多个条件满足, 那就随机执行一个

`default` 分支是默认分支, 防止阻塞
```go
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```

## `Mutex`

通过在代码前后调用 `Lock` 和 `Unlock` 函数来上锁

使用 `defer` 语句保证一定会被解锁

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter 的并发使用是安全的。
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc 增加给定 key 的计数器的值。
func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	// Lock 之后同一时刻只有一个 goroutine 能访问 c.v
	c.v[key]++
	c.mux.Unlock()
}

// Value 返回给定 key 的计数器的当前值。
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	// Lock 之后同一时刻只有一个 goroutine 能访问 c.v
	defer c.mux.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```