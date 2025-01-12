## 接口

### 接口定义

使用interface关键字定义接口,跟go一样

默认是模块级别,使用pub变为公共级别

接口命名跟结构体一样,要求首字母大写,建议以er风格结尾,非强制

```v
pub interface Speaker {
		speak() string
}
```

### 接口字段

接口在其他编程语言中大部分都只能包含方法签名

V语言中接口除了可以包含方法签名,也可以包含字段,虽然包含字段也可以转为方法签名,

但是直接约束字段,代码看起来更舒服.

编译时会检查结构体及其组合结构体(子类)是否实现接口字段,

只有字段名,字段类型,是否可变,都跟接口中的字段一致,才算实现了这个接口字段.

```v
module main

interface PointInterface {
mut:
  x int //定义接口字段
  y int
}

struct Point {
mut:
	x int //接口字段实现
	y int
}

fn add(p1 PointInterface,p2 PointInterface) PointInterface {
	mut p:=Point {}
	p.x=p1.x+p2.x
	p.y=p1.y+p2.y
	return p
}

fn main() {	
	p1:=Point{
		x:1
		y:2
	}
	p2:=Point {
		x:3
		y:4
	}
	p3:=add(p1,p2)
	println('x:$p3.x,y:$p3.y')
}
```

```v
interface Animal {
mut:
	name string
}

struct Cat {
	name string //mut不匹配,编译不通过
}

fn main() {
	mut animals := []Animal{}
	animals << Cat{}
}
```

### 接口方法

接口也可以像结构体那样有自己的方法,只要结构体实现了接口,就可以调用接口的方法

```v
struct Cat {
	breed string
}

interface Animal {
	breed string
}

//接口自己实现的方法,接口方法的默认实现
fn (a Animal) info() string {
	return "I'm a ${a.breed} ${typeof(a).name}"
}

fn new_animal(breed string) Animal {
	return &Cat{ breed }
}

fn main() {
	mut a := new_animal('persian')
	println(a.info())
}	
```

### 接口组合

接口也像结构体那样支持组合

```v
module main

pub interface Reader{
	read(mut buf []byte) ?int
}

pub interface Writer {
	write(bug []byte) ?int
}

// 接口组合
pub interface ReaderWriter {
		Reader
  	Writer
}

//接口组合
interface WalkerTalker {
	Walker 
	Talker
}

interface Talker {
	nspeeches int
	talk(msg string)
}

interface Walker {
	nsteps int
	walk(newx int, newy int)
}

struct Abc {
mut:
	x         int
	y         int
	phrases   []string
	nsteps    int = 1000 //实现接口中要求的字段
	nspeeches int = 1000 //实现接口中要求的字段
}

//实现接口要求的方法
fn (mut s Abc) talk(msg string) {
	s.phrases << msg
	s.nspeeches++
}

//实现接口要求的方法
fn (mut s Abc) walk(x int, y int) {
	s.x = x
	s.y = y
	s.nsteps++
}

fn main() {
	mut wt := WalkerTalker(Abc{ 
		x: 1
		y: 1
		phrases: ['hi']
	})
	wt.talk('my name is Wally')
	wt.walk(100, 100)
	if wt is Abc {
		println(wt)
	}
}
```

### 泛型接口

使用泛型接口可以定义更为通用/泛化的接口.

参考章节:[泛型](generic.md)

### 接口实现

没有接口实现的关键字,类型无需显示声明所要实现的接口,

鸭子类型:只要结构体实现了接口定义的方法签名,就满足该接口的使用.

```v
module main

struct Dog {}

struct Cat {}

fn (d Dog) speak() string {
	return 'woof'
}

fn (c Cat) speak() string {
	return 'meow'
}

interface Speaker {
	speak() string  //普通的接口方法
}

interface Iterator<T> {
	next() ?T //带错误处理的接口方法,问号也是接口方法签名的一部分
}

fn perform(s Speaker) {
	println(s.speak())
}

fn main() {
	dog := Dog{}
	cat := Cat{}
	perform(dog) // "woof"
	perform(cat) // "meow"
}
```

接口可以作为结构体字段类型使用:

```v
struct Foo {
	speaker Speaker //接口类型字段
	speakers []Speaker //接口类型数组字段
}	
```

### 获取接口变量的具体类型

接口类型使用内置方法type_name()来返回接口变量的具体类型

```v
module main

pub interface Animal {
	speak() string
}

pub struct Cat {}
pub fn (c Cat) speak() string {
	return 'miao'
}

pub struct Dog {}

pub fn (d Dog) speak() string {
	return 'wang'
}

pub fn say(animal Animal) {
	println(typeof(animal).name) // 只会返回接口名字Animal,不会返回具体类型
	println(animal.type_name()) // 返回接口的具体类型
	println(animal.speak())
}

fn main() {
	cat:=Cat {}
	say(cat)

	dog:=Dog{}
	say(dog)
}
```

### 接口变量类型判断及匹配

使用is,!is对接口参数的具体类型进行判断

使用as对接口参数进行类型转换

使用match对接口类型参数进行匹配,跟match匹配联合类型一样,每一个分支都会自动造型,非常的好用

```v
module main

struct Dog {
	name1 string
}

struct Cat {
	name2 string
}

fn (d Dog) speak() string {
	return 'wang'
}

fn (c Cat) speak() string {
	return 'miao'
}

interface Speaker {
	speak() string
}

fn perform(s Speaker) {
	if s is Dog { // 通过is操作符,判断接口类型是否是某一个具体类型
		println('s is Dog')
	}
	if s !is Dog { // 通过!is操作符,判断接口类型不是某一个具体类型
		println('s is not Dog')
	}

	//match对接口参数的类型匹配,跟匹配联合类型一样,每一个分支都会自动造型,非常的好用
	match s {
		Dog { println('s is Dog struct,name is $s.name1') }
		Cat { println('s is Cat struct,name is $s.name2') }
		else { println('s is: $s.type_name()') }
	}
}

fn main() {
	dog := Dog{ name1:'dog' }
	cat := Cat{ name2:'cat' }
	perform(dog) // "wang"
	perform(cat) // "miao"
}
```

### 判断类型是否实现接口

```v
module main

struct Dog {
	name string
}

struct Foo {
}

fn (d Dog) speak() string {
	return 'wang'
}

interface Speaker {
	speak() string
}

fn my_fn<T>() {
	$if T is Speaker { //使用$if,在泛型函数中判断类型是否实现了某个接口
		println('type T implements Speaker interface')
	} $else {
		println('type T do not implements Speaker interface')
	}
}

fn main() {
	my_fn<Dog>()
	// my_fn<Foo>()

	// $if Dog is Speaker {
	// 	println('type Dog implements Speaker interface')
	// }
}

```

### 判断接口类型是否也实现了其他接口

可以像结构体那样使用is和as来判断接口是否也实现了其他接口，或者转换成其他接口：

```v
interface Widget {
}

interface ResizableWidget {
	Widget
	resize(x int, y int) int
}

// 实现Widget接口, 但没有实现ResizableWidget接口
struct WidgetA {
}

// 实现Widget和ResizableWidget接口
struct WidgetB {
}

fn (w WidgetB) resize(x int, y int) int {
	return x * y
}

fn draw(w Widget) {
	// w.resize(10, 20) // 此时调用resize方法并不正确，因为并不是所有的w参数变量都实现了ResizableWidget接口

	if w is ResizableWidget { //判断w变量是否也实现了ResizableWidget接口
		assert w is WidgetB
		rw := w as ResizableWidget	//把w变量从Widget接口变量转换为ResizableWidget接口变量
		assert rw is WidgetB
		//接口类型转换以后就可以调用新接口的方法
		println(rw.resize(10, 20))
	} else {
		assert w is WidgetA
	}
}

fn main() {
	draw(WidgetA{})
	draw(WidgetB{})
}
```

