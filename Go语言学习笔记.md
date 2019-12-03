#  Go语言基础学习笔记

## 一、环境搭建

1. 下载地址：https://studygolang.com/dl

2. 配置环境变量

   GOROOT：指定SDK的安装路径

   PATH：添加SDK的/bin目录

   GOPATH：工作目录，将来我们的go项目的工作路径

3. go version查看版本

## 二、开发规范

1. 入口文件方法名是main，包名一定要是main。
2. 文件夹名和包名没有联系，即/src/test下的文件，package可以是other，但是同个文件夹下的多个文件的包名要一致。
3. 大括号不能换行写。
4. 单行语句结束不需要写分号，换行即可；但是多行语句书写在同一行，则需要分号分隔。（不建议单行写多个语句）
5. 同个文件夹下的文件相互调用不用导包，不同文件夹下的文件相互调用需要导包，导包的格式是：

```go
import(
	"文件夹路径"
)
```

6. 方法名以大写开头则为公有，小写为私有。

## 三、数据类型和变量声明

1. 值类型：变量直接存储值，内存通常在栈中分配。

   引用类型：变量存储的是地址，这个地址对应的空间存储的才是值，内存通常在堆上分配，当没有任何变量引用这个地址时，该地址对应的数据空间视为垃圾，由GC回收。

2. 基本数据类型

|       数据类型        | 值的传递类型 |
| :-------------------: | :----------: |
| 数值型(int, float...) |    值传递    |
|     字符型(byte)      |    值传递    |
|     布尔型(bool)      |    值传递    |
|    字符串(string)     |    值传递    |

3. 派生/复杂数据类型

|           数据类型           | 值的传递 |
| :--------------------------: | :------: |
| 数组([...]int, [5]string...) |  值传递  |
|        结构体(struct)        |  值传递  |
|           指针(*)            |  值传递  |
|        管道(channel)         | 引用传递 |
|             Map              | 引用传递 |
|   切片([]int, []string...)   | 引用传递 |
|             接口             | 特殊类型 |
|             函数             | 特殊类型 |

4. 变量声明

   ```go
   // var 变量名 数据类型
   // 变量名 := 值
   var a int = 45 // 给变量a初始化整型值45
   num := 10 // 给变量num初始化赋值10，:=会进行类型推导，不需要声明类型
   str, num1, flag := "Hello", 45, true // 批量定义
   ```

## 四、基本数据类型

1. 整型(默认值：0；默认声明类型：int)

|  类型  | 有无符号 | 占用存储空间 |     范围     |
| :----: | :------: | :----------: | :----------: |
|  int8  |    有    |    1字节     |   -128~127   |
| int16  |    有    |    2字节     | -2^15~2^15-1 |
| int32  |    有    |    4字节     | -2^31~2^31-1 |
| int64  |    有    |    8字节     | -2^63~2^63-1 |
| uint8  |    无    |    1字节     |    0~255     |
| uint16 |    无    |    2字节     |   0~2^16-1   |
| uint32 |    无    |    4字节     |   0~2^32-1   |
| uint64 |    无    |    8字节     |   0~2^64-1   |

```go
// 查看变量的字节大小和数据类型
var i uint = 100
fmt.Printf("i 的数据类型 %T,占用的字节数:%d\n",i, unsafe.Sizeof(i))
```

2. 小数类型/浮点型(默认值：0；默认类型：float64)

|     类型      | 占用存储空间 |         范围         |
| :-----------: | :----------: | :------------------: |
| 单精度float32 |    4字节     |  -3.403E38~3.403E38  |
| 双精度float32 |    8字节     | -1.798E308~1.798E308 |

```go
// 单精度和双精度位数= 符号位 + 整数位 + 小数位
var num1 float32 = 123.123456789
var num2 float64 = 123.123456789123456789
fmt.Println("浮点类型float32", num1)
fmt.Println("浮点类型float64", num2)
// 输出:
// 浮点类型float32 123.12346
// 浮点类型float64 123.12345678912345

// 浮点类型常量
var i = 0.178
// 科学计数法
var j = 3.14e2
var k = 3.14e-2
```

3. 字符类型

go中没有专门的字符类型，一般是用byte来保存单个字符。

```go
// 英文字母-1个字节 汉字-3个字节
var c1 byte = 'a'
var c2 int = '中'
var c3 int = 20013
fmt.Printf("%d对应的字符是%c\n",c3,c3) // 输出：20013对应的字符是中
// 字符类型是可以进行运算的，相当于一个整数，因为字符有对应的unicode码
var c4 = 3 + 'a' // ('a':97) 输出： 100
```

存储过程：字符->对应码值->二进制---->存储

读取：二进制->码值->字符---->读取

注：Go的编码统一为UTF-8，没有了编码乱码困扰。

4. 布尔类型(默认值：false)

布尔类型的值只有true和false，占1字节。

5. string类型(默认值："")

*  Go中的字符串是由多个字节连接起来的。
* 统一使用UTF-8编码。
* 字符串一旦赋值，就不能修改字符串里面的内容。
* 双引号会识别转义字符。
* 反引号会以字符串的原生形式输出，包括特殊字、符换行等。

## 五、基本数据类型转换

Go语言在不同类型的变量进行转换时，只能进行显式转换。可以从表示范围大的转换到小的，也可以表示范围小的转换为大的。被转换的数据本身没有变化，是值传递。

1. 数值间转换

```go
var i int32 = 100
var n1 int64 = int64(i)
var n2 float32 = float32(i)
var n3 int8 = int8(i)
```

表示范围较大的转范围较小的，有时候会出现溢出，虽然不会报错，但会做溢出处理。

```go
var num1 int64 = 130
var num2 int8 = int8(num1)
// 最后输出num2为-126
```

这里涉及到溢出处理。因为int8的范围是-127~128，显然130超出范围。

130的二进制原码(64位)：00000000  00000000  00000000  10000010

正数的原补反码相同，补码：00000000  00000000  00000000  10000010

强制转换为int8，所以截取8位，得补码：10000010

最终我们看到的是原码，所以要将补码变为原码。补码是原码符号位、自低向高位出现的第一个1和右边的0不变，然后其他位取反，即得到原码：11111110，转为十进制即为-126。

* 注：

  > 最高位为符号位。
  > 正数原反补码相同。
  >  原码->反码：负数反码为原码除符号位按位取反。
  >   原码->补码：负数补码为原码除符号位、最后出现的第一个1和右边的0不变外，其他按位取反。
  >    反码->补码：负数补码为反码加1。

2. string和数值类型转换

```go
var i1 = true
var i2 = 1.3
var i3 = 10
// *转化为string类型*
// 方法一：fmt.Sprintf()根据参数生成格式化的字符串并返回转换后的字符串
var s1 = fmt.Sprintf("%v", i1)
var s2 = fmt.Sprintf("%v", i2)
// 方法二：strconv
var s3 = strconv.FormatBool(i1)
// f表示格式；10表示小数位保留10位；64表示float64
var s4 = strconv.FormatFloat(i2,'f', 10, 64)
// 整型需要转换为int64，还需要指定对应的进制
var s5 = strconv.FormatInt(int64(i3),10)

// string转换为基本数据类型
// 使用strconv
var n1 int64
n1, _ = strconv.ParseInt(s3, 10, 64)
```

* 在将string类型转为基本数据类型的时候，要确保string类型能转成有效的数据，不然go会将其转为默认值。如将"hello"转为整数，虽然不报错，但会转为0。

## 六、指针

1. 基本数据类型，变量存的是值，是值类型；取地址符&。

   ```go
   var num int = 10
   fmt.Println("num的地址：", &num)
   ```

2. 指针类型，指针变量存的是一个地址，这个地址指向的空间才是值。获取指针类型所指向的值，用*。

   ```go
   var s string = "Hello"
   var ptr *string = &s
   fmt.Printf("ptr=%v\n", ptr)
   fmt.Printf("ptr的地址=%v\n", &ptr)
   fmt.Printf("ptr指向的值=%v\n", *ptr)
   ```

   ptr(地址：0xc00008e020；值：0xc00004c200)

   s(地址：0xc00004c200；值：Hello)

## 七、数组和切片

1. 数组创建

   ```go
   // var 数组名 [数组大小]数据类型
   var array1 = [...]int{}
   var array2 = [5]string{"2b","33c","2d","33a"}
   var array3 [5]int
   var array4 = [5]int{5:5,4:2}
   ```

2. 数组遍历

   ```go
   for i := 0; i < len(array) ; i++ {
       ...
   }
   for index, value := range array{
       // index为数组下标，value为数组值
       // 如果不使用这两个属性中的某一个，可以用_占位
       ...
   }
   ```

3. 切片的创建

   ```go
   // 方法一:切割数组，下标1到4，即容量为4，而内容只有下标1到2，即长度为2
   var arr = [...]string{"张学友","刘德华","黎明","郭富城","成龙","周星驰"}
   slice := arr[1:3:5]
   
   // 方法二:通过make创建，需要的参数type,len,cap(cap>=len)
   slice := make([]int, 3, 5)
   
   // 方法三:var 切片名 []数据类型
   slice := []string{"张学友","刘德华","黎明","郭富城"}
   ```

   slice是引用类型，它能直接拿到地址，它拿到的地址是第一个值的地址。

   ```go
   // 容量表示该切片能容纳多少内容，而长度表示切片内有值的部分的长度
   fmt.Printf("slice容量:%d,长度:%d,值:%v\n",cap(slice),len(slice),slice)
   fmt.Printf("slice地址:%p",slice)
   fmt.Printf("arr[1]地址:%v",&arr[1])
   // 输出：slice容量:4,长度:2,值:[刘德华 黎明]
   //      slice地址:0xc000084130
   //      arr[1]地址:0xc000084130
   ```

   slice的底层是一个数据结构(struct)

   ```go
   tyepe slice struct {
   	ptr *[2]int
   	len int
   	cap int
   }
   ```

4. 切片的遍历

   ```go
   for i := 0; i < len(slice) ; i++ {
       ...
   }
   for index,value := range slice {
       ...
   }
   ```

## 八、map

1. map介绍

   * map是引用类型，接收map后修改map，会影响原来的map。
   * map的key可以是多种类型，如bool、数字、string、指针、channel，还可以是包含这些类型的接口、结构体或数组。
   * map的key不能重复，重复定义或赋值会覆盖原来的值。
   * map的key-value是无序的。
   * map的容量不够时，会自动扩容。

2. map创建

   ```go
   // 声明map不会分配内存的，需要用make分配内存，分配内存后才能赋值和使用
   var m map[string]string
   // 方式一：make(Type, len, cap)
   m = make(map[int]string, 1)
   m[0] = "Hello"
   fmt.Println(m)
   
   // 方式二：
   var m map[string]string
   // make(Type)
   var m = make(map[int]interface{})
   m[1]="A"
   m[2]="B"
   fmt.Printf("m地址:%p;m的值:%v",m,m)
   
   // 方式三：
   var m = map[int]interface{}{
       1:"A",
       2:"B",
   }
   fmt.Printf("m地址:%p;m的值:%v",m,m)
   ```

3. map的使用

   * map增加和修改：map["key"] = value，key不存在时增，存在时改。

   * map删除：delete(map, "key")，若key存在，则删除；若不存在，不报错也不操作。不过map没有批量删除操作，要批量删除需要遍历map执行删除操作。

   * map查找：value := m[key]，当key存在，返回对应的value，否则返回对应的value类型的默认值，所以这时候不能判断key是否存在。要想判断key是否存在，应该判断返回的bool值：

     ```go
     _, ok := demo["a"]
     fmt.Println(ok)// key存在ok返回true，不存在返回false
     ```

   * map遍历

     ```go
     for key,value:= range m {
         fmt.Printf("key:%v ; value:%v\n",key,value)
     }
     ```


## 九、结构体

1. 有关结构体知识介绍
* go的面向对象编程特性是基于结构体实现的。
* go没有继承、方法重载、构造函数和析构函数等等，但它仍有继承、封装和多态的特性。go没有extends关键字，它是通过匿名字段来实现继承特性的。
* go是通过接口进行关联。

2. 结构体声明

   ```go
   /*
   type 结构体名称 struct {
   	field1 type
   	field2 type
   }
   */
   // 属性开头大写为公有
   type Student struct {
    Name string
       Age int
       Score float32
   }
   ```
   
3. 不同结构体变量的字段是独立的，互不影响。一个结构体的字段改变，不影响另外一个，结构体是值类型。

4. 结构体使用

   ```go
   type Person struct {
   	Name string
   	Age int
   }
   func main() {
   	var p1 Person
   	p1.Name="jerry"
   	p1.Age=55
   	fmt.Printf("p1的值:%v\n",p1)
       
   	var p2 Person= Person{Name:"susan",Age:10}
   	fmt.Printf("p2的值:%v\n",p2)
       
       // 这两个方式返回的是结构体指针
   	var p3 *Person = new(Person)
   	p3.Name="jeck"
   	p3.Age=66
   	fmt.Printf("p3的值:%v\n",p3)
       
   	var p4 *Person= &Person{Name:"john",Age:88}
   	fmt.Printf("p4的值:%v\n",p4)
   }
   ```

   * 结构体的所有字段在内存中是连续的。

   * 结构体是用户单独定义的类型，和其它类型进行转换时需要有完全相同的字段(名字、个数和类型)。

   * 结构体进行 type 重新定义(相当于取别名)，go认为是新的数据类型，但是相互间可以强转。

     ```go
     type Person struct {
     	Name string
     	Age int
     }
     type ps Person
     func main() {
     	var p1 ps
     	p1.Name="jerry"
     	p1.Age=55
     	fmt.Printf("p1的值:%v\n",p1)
     	var p2 Person= Person(p1)
     	fmt.Printf("p2的值:%v\n",p2)
     }
     ```

   * struct的每个字段上，可以写上一个tag，该tag可以通过反射机制获取，常见的使用场景就是序列化和反序列化。

     ```go
     type Person struct{
     	Name string `json:"name"` // `json:"name"`就是结构体tag
     	Age int // `json:"age"` ，没有写tag，序列化后则为大写字母，如果字段为小写，其他包又不可见
     }
     
     func main() {
     	person := Person{"佩奇", 18}
     	jsonStr, err := json.Marshal(person)
     	if err != nil {
     		fmt.Println(err)
     	} else {
     		fmt.Println(string(jsonStr)) // {"name":"佩奇","Age":18}
     	}
     }
     ```

## 十、方法和函数

1. 方法和函数的区别

   * 方法是特殊的函数，定义在某一特定的类型上，通过类型的实例来进行调用，这个实例被叫接收者(receiver)。

   * 调用方式不一样
     函数的调用方式: 函数名(实参列表)
     方法的调用方式: 变量  方法名(实参列表)

   * 对于普通函数，接收者为值类型时，不能将指针类型的数据直接传递，反之亦然。

   * 对于方法（如 struct 的方法），接收者为值类型时，可以直接用指针类型的变量调用方法，反过来同样也可以。

2. 方法的声明和调用

   ```go
   type Person struct {
   	Name string
   	Age int
   }
   func (p Person) hello() {
   	fmt.Printf("hello! my name is %v", p.Name)
   }
   // 有返回值
   func (p Person) calc() int{
   	sum := 0
   	for i:=0;i<1000;i++{
   		sum+=i
   	}
   	return sum
   }
   // 有返回值和参数
   func(p Person) getSum(a int, b int) int {
       return a + b
   }
   func main() {
   	var person Person
   	person.Name = "Jason"
   	person.Age = 22
   	person.hello()
       person.calc()
       person.getSum(3, 4)
   }
   ```

   * 不能返回`局部变量`的`地址值`;因为函数调用结束，栈帧释放，局部变量的地址，不再受系统保护，随时可能分配给其他程序。可以返回`局部变量`的`值`

## 十一、继承和重写

1. 属性继承

   * 在Student对象中可以直接使用父类Person的属性，如果父类和子类中有重复字段，则优先使用子类自身的属性。
   * 方法的继承和属性的继承是类似的，当一个类是另一个类的匿名属性时即可直接使用改类的方法。
   
   ```go
   type Person struct {
   	id int
   	name string
   	age int
   }
   type Student struct {
   	Person
   	id int
   	score int
   	className string
}
   func (p Person) say(msg string) {
       fmt.Println(p.name + " say " + msg)
   }
   func main() {
       var (
       	s Student
       )
       s = Student{
           Person{name: "LiBai", age: 23, id: 123},
           10,
           98,
           "301班",
       }
       // 当继承的父类中有相同属性，则优先使用子类属性；如此处输出id：10
       fmt.Printf("Id：%d\n姓名：%s\n年龄：%d\n班级：%s\n分数：%d", s.id, s.name, s.age, s.className, s.score)
       s.say("hello world")
   }
   ```
   
2. 方法的重写

   在语法格式上有一定的要求，即方法名，参数，返回值类型都必须一样，但绑定的对象不在时父类二是子类本身。

   ```go
   func (s Student) say(msg string) {
   	fmt.Println(s.name + " say: " + msg)
   }
   ```

## 十二、接口、多态

**interface{}** 是一个空接口，它表示可以接受任意对象；在go中空接口也是一中数据类型（有点类似java中的Object）。

1. 接口定义

   ```go
   type animal interface {
   	// 在接口中定义通用的方法（不用实现）
       eat()
   	sleep()
   	run()
   }
   ```

2. 接口的继承和实现，多态

   ```go
   type cat interface {
   	animal
       Climb()
   }
   type BritishShorthair struct {
       cat
   }
   
   type dog interface {
   	animal    
   }
   type husky struct {
       dog
   }
   
   func (bh BritishShorthair) eat() {
       fmt.Println("英短")
   }
   func (h husky) eat() {
       fmt.Println("哈士奇")
   }
   
   func test(a animal){
   	a.eat()
   }
   
   func main() {
       var a animal
       a = BritishShorthair{}
       test(a)
       
       a = husky{}
       test(a)
   }
   
   // 输出：
   // 英短
   // 哈士奇
   ```

3. 类型断言

   ```go
   func main() {
       var a animal
       a = BritishShorthair{}
       var b animal
       b = husky{}
       
       var animals [2]animal = [...]animal{a, b}
       
       for _,v := range animals {
           if data, ok := v.(husky); ok {
               data.eat()
               fmt.Println("this is husky")
           }
           if data, ok := v.(BritishShorthair); ok {
               data.eat()
               fmt.Println("this is BritishShorthair")
           }
       }
   }
   ```

## 十三、异常处理

1. 添加判断

   考虑到各种情况，做出对应的判断处理

   ```go
   import (
   	"errors"
       "fmt"
   )
   
   func test(a int, b int) (int, error) {
       var err error
       if a == 0 {
           return 0, errors.New("除数不能为0")
       }
       a = b / a
       return a, err
   }
   
   func main() {
       i,err := test(0, 1)
       if err == nil {
           fmt.Println("main方法正常结束!!!", i)
       }else {
           fmt.Println(err)
       }
   }
   ```

2. 捕捉异常

   在异常的场景很多的情况下，很难全部考虑或统计出来。我们就需要捕获异常，需要用到defer和recover。

   * defer：延时执行，在方法执行结束时执行。通常用于资源释放、打印日志、异常捕获等。如果有多个defer函数，调用顺序类似于栈，越后面的defer函数越先被执行(后进先出)。如果包含defer函数的外层函数有返回值，而defer函数中可能会修改该返回值，最终导致外层函数实际的返回值改变。

     ```go
     func main() {
         i := 0
         defer fmt.Println("a:", i)
         //闭包调用，将外部i传到闭包中进行计算，不会改变i的值，如上边的例3
         defer func(i int) {
             fmt.Println("b:", i)
         }(i)
         //闭包调用，捕获同作用域下的i进行计算
         defer func() {
             fmt.Println("c:", i)
         }()
         i++
     }
     // 输出：
     // c: 1
     // b: 0
     // a: 0
     ```

   * recover：使程序出现异常时不会中断，返回异常信息，可以根据异常来做出相应的处理。（recover必须在defer函数里才能生效）

     ```go
     import (
     	"fmt"
     )
     func test(a int, b int) int {
     	defer func() {
     		err := recover()
     		fmt.Println("err:",err)
     	}()
     	a = b / a
     	return a
     }
     func main() {
     	test(0, 1)
     	fmt.Println("main方法正常结束!!!")
     }
     // 输出：
     // err: runtime error: integer divide by zero
     // main方法正常结束!!!
     ```

3. 手动抛出异常

   * 并不是任何时候的可以用recover()，这样会导致僵尸程序或者引发更严重的后果，在程序报错的时候，让其正常运行可能会导致后面关联的业务也出现错误。

   * 为了保护后面的业务，我们需要抛出异常。用到panic，抛出并终止程序。

     ```go
     import (
     	"errors"
     	"fmt"
     )
     
     func test(a int) int {
     	i := 100 - a
     	if i < 0 {
     		panic(errors.New("账户金额不足！！！！"))
     	}
     	fmt.Println("=======账户扣款=====")
     	return i
     }
     
     func main() {
     	i := test(101)
     	fmt.Println("====main方法正常结束！！====余额：",i)
     }
     
     // 输出：
     // panic: 账户金额不足！！！！
     //
     // goroutine 1 [running]:
     // main.test(0x65, 0xc000006018)
     // ...
     ```

## 十四、文件操作

1. 介绍

   os.File封装了所有文件的相关操作，File是一个结构体：

   ```go
   type File
   	func Create(name string)(file *File, err error)
   	func Open(name string)(file *File, err error)
   	func OpenFile(name string, flag int, perm FileMode)(file *File, err error)
   	func NewFile(fd uintptr,name string) *File
   	func Pipe()(r *File, w *File, err error)
   	func (f *File)Name(name string) string
   	func (f *File)Stat() (fi FileInfo, err error)
   	func (f *File)Fd() uintptr
   	func (f *File)Chdir() error
   	func (f *File)Chmod(mode FileMode) error
   	func (f *File)Chown(uid int, gid int) error
   	func (f *File)Readdir(n int)(fi []FileInfo, err error)
   	func (f *File)Readdirnames(n int)(names []string,err error)
   	func (f *File)Truncate(size int64) error
   	func (f *File)Read(b []byte)(n int, err error)
   	func (f *File)ReadAt(b []byte, off int64)(n int, err error)
   	func (f *File)Write(b []byte)(n int, err error)
   	func (f *File)WriteString(s string)(ret int, err error)
   	func (f *File)WriteAt(b []byte, off int64)(n int, err error)
   	func (f *File)Seek(offset int64, whence int)(ret int64,err error)
   	func (f *File)Sync() error
   	func (f *File)Close() error
   ```

2. 创建文件    os.Create(file)
   
   ```go
   import (
       "fmt"
       "log"
       "os"
   )
   
   func main() {
       file, err := os.Create("D:\\hello.log")
       if err != nil {
           log.Fatalln(err)
       }
       fmt.Println(file)
   }
   ```
   
3. 判断文件是否存在

   ```go
   import (
       "fmt"
       "log"
       "os"
   )
   
   func main() {
       _, err := os.Stat("file.log")
       if e==nil || os.IsExist(e)  {
   		fmt.Printf("%v exist !!\n",filePath)
   	}
   	if os.IsNotExist(e) {
   		fmt.Printf("%v not exist !!\n",filePath)
   	}
   }
   ```

4. 创建目录

   ```go
   import (
       "log"
       "os"
   )
   
   func main() {
       // 创建单个目录
       err := os.Mkdir("tmp", 0755)
       if err != nil {
           log.Fatalln(err)
       }
   
       // 递归创建目录
       err = os.MkdirAll("tmp/tmp1/tmp2", 0755)
       if err != nil {
           log.Fatalln(err)
       }
   }
   ```

5. 删除文件

   ```go
   import (
       "os"
       "fmt"
   )
   
   func main() {
   	//默认路径是从项目的根路径开始的
   	filePath := "file/test.txt"
   	e := os.Remove(filePath)
   	if e!=nil{
   		fmt.Println("remove failed!!")
   	}
   }
   ```

6. 打开文件和关闭文件

   ```go
   func main() {
       file, e := os.Open("D:\\hello.txt")
       if e != nil {
           fmt.Println("文件打开失败")
       }
       e = file.Close()
       if e != nil {
           fmt.Println("文件关闭失败")
       }
   }
   ```

7. 读文件操作

   * bufio.NewReader(fliePath)

     ```go
     func main() {
         file, e := os.Open("D:\\hello.log")
         if e != nil {
             fmt.Println("文件打开失败")
         }
         defer func() {
             close := file.Close()
             if close != nil {
                 fmt.Println("文件关闭失败")
             }
         }()
         reader := bufio.NewReader(file)
         for {
             /*line, isPrefix, e := reader.ReadLine()
             if e == io.EOF {
                 fmt.Println("读取结束:", e)
             	break
             }else {
                 fmt.Println(isPrefix, string(line))
             }*/
             str, e := reader.ReadString('\n')
     		if len(str) == 0{
     			break
     		}
     		if e !=nil && e != io.EOF{
     			break
     		}
     		fmt.Println(str)
         }
     }
     ```

   * ioutil.ReadFile(fliePath)：适用于文件不大的情况

     ```go
     func main() {
     	//默认路径是从项目的根路径开始的
     	filePath := "file/test.txt"
     	bytes, e := ioutil.ReadFile(filePath)
     	if e!=nil{
     		fmt.Printf("读取文件%v出错了\n",filePath)
     		return
     	}
     	fmt.Println(string(bytes))
     }
     ```
     
    * 多种文件读取方式的比较

      ```go
      package main
      
      import (
          "bufio"
          "fmt"
          "io"
          "io/ioutil"
          "os"
          "time"
      )
      
      func read0(path string) string {
          f, err := ioutil.ReadFile(path)
          if err != nil {
              fmt.Printf("%s\n", err)
              panic(err)
          }
          return string(f)
      }
      
      func read1(path string) string {
          fi, err := os.Open(path)
          if err != nil {
              panic(err)
          }
          defer fi.Close()
      
          chunks := make([]byte, 1024, 1024)
          buf := make([]byte, 1024)
          for {
              n, err := fi.Read(buf)
              if err != nil && err != io.EOF {
                  panic(err)
              }
              if 0 == n {
                  break
              }
              chunks = append(chunks, buf[:n]...)
          }
          return string(chunks)
      }
      
      func read2(path string) string {
          fi, err := os.Open(path)
          if err != nil {
              panic(err)
          }
          defer fi.Close()
          r := bufio.NewReader(fi)
      
          chunks := make([]byte, 1024, 1024)
      
          buf := make([]byte, 1024)
          for {
              n, err := r.Read(buf)
              if err != nil && err != io.EOF {
                  panic(err)
              }
              if 0 == n {
                  break
              }
              chunks = append(chunks, buf[:n]...)
          }
          return string(chunks)
      }
      
      func read3(path string) string {
          fi, err := os.Open(path)
          if err != nil {
              panic(err)
          }
          defer fi.Close()
          fd, err := ioutil.ReadAll(fi)
          return string(fd)
      }
      
      func main() {
      
          file := "D:\\hello.log"
      
          start := time.Now()
      
          // ioutil.ReadFile(path)
          read0(file)
          t0 := time.Now()
          fmt.Printf("Cost time %v\n", t0.Sub(start))
      
          // Read()
          read1(file)
          t1 := time.Now()
          fmt.Printf("Cost time %v\n", t1.Sub(t0))
      
          // bufio.NewReader(file)
          read2(file)
          t2 := time.Now()
          fmt.Printf("Cost time %v\n", t2.Sub(t1))
      
          // ioutil.ReadAll(file)
          read3(file)
          t3 := time.Now()
          fmt.Printf("Cost time %v\n", t3.Sub(t2))
      }
      // 方法一较稳定
      // 输出：
      // Cost time 500.1µs
      // Cost time 500.1µs
      // Cost time 499.8µs
      // Cost time 1.0004ms
      ```

8. 写文件操作

   * os.OpenFile函数
   
     ```go
     func OpenFile(file string, mode int, perm FileMode)(file *File, err error)
     ```
   
     > 参数mode：文件打开的模式（可以组合）
     >
     > ```go
     > const (
     > 	O_RDONLY int = syscall.O_RDONLY // 只读模式
     >     O_WRONLY int = syscall.O_WRONLY // 只写模式
     >     O_RDWR int = syscall.O_RDWR // 读写模式
     >     O_APPEND int = syscall.O_APPEND // 写操作将数据附加
     >     O_CREATE int = syscall.O_CREATE // 若不存在则创建文件
     >     O_EXCL int = syscall.O_EXCL // 和O_CREATE配合使用，文件必须不存在
     >     O_SYNC int = syscall.O_SYNC // 打开文件用于同步I/O
     >     O_TRUNC int = syscall.O_TRUNC // 如果可能，打开时清空文件
     > )
     > ```
   
     >参数perm：权限控制（一般选6）
     >
     >>权限取值范围：0~7
     >>
     >>0：没有任何权限
     >>
     >>1：执行权限（）如果文件是可执行文件，是可以运行的
     >>
     >>2：写权限
     >>
     >>3：写权限和执行权限
     >>
     >>4：读权限
     >>
     >>5：读权限和执行权限
     >>
     >>6：读权限和写权限
     >>
     >>7：读权限、写权限和执行权限
   
   * bufio新建文件并写入数据
   
     ```go
     func main() {
     	filePath := "file/test.txt"
     	file, e := os.OpenFile(filePath, os.CREATE| os.O_EXCL | os.O_WRONLY, 0666)
         // 追加内容
         // file, e := os.OpenFile(filePath, os.O_WRONLY | os.O_APPEND, 0666)
         
         // 覆盖文件的内容
         // file, e := os.OpenFile(filePath, os.O_WRONLY, 0666)
         
         defer func() {
             if i := file.Close(); i != nil {
                 fmt.Println("文件关闭失败")
             }
         }()
         if e != nil {
             fmt.Println("文件创建失败")
         	return
         }
         writer := bufio.NewWriter(file)
         content, e := writer.Write([]byte("Hello"))
         if e != nil {
             fmt.Println("内容写入缓冲区失败")
             return
         }
         e = writer.Flush()
         if e != nil {
             fmt.Println("内容刷入文件失败")
             return
         }
         fmt.Printf("文件写入了%v字节数据", content)
     }
     ```
   
   * ioutil写入文件
   
     ```go
     func main() {
         filePath := "file/test.txt"
         // 文件不存在则按给出的权限创建文件，否则在写入数据之前清空文件
         e := ioutil.WriteFile(filePath, []byte("hello world"), 0666)
         if e != nil {
             fmt.Println("写文件出错了")
         }
     }
     ```

## 十五、协程goroutine

1. 协程介绍

   * Go协程（Goroutine）是与其他函数或方法同时运行的**函数或方法**。可以认为Go协程是轻量级的线程。
   * 一个线程中可开启多个协程。如何做到单线程并发？并发 = 切换任务+保存状态，只要找到一种方案，能够在两个任务之间切换执行并且保存状态，那就可以实现单线程并发。
   * 协程做到单线程下实现并发，通过上下文保存状态，然后切换协程执行，由于切换时间很短，看起来就是并发执行。
   * 协程的栈，go采用了动态扩张收缩的策略：初始化为2KB，最大可扩张到1GB。
   
2. 协程和线程
   
   * 线程是有固定的栈的，基本都是2MB，当然，不同系统可能大小不太一样，但是的确都是固定分配的。这个栈用于保存局部变量，用于在函数切换时使用；协程的栈，go采用了动态扩张收缩的策略：初始化为2KB，最大可扩张到1GB。
   
   * 每个线程都有一个id，这个在线程创建时就会返回，所以可以很方便的通过id操作某个线程。但是在goroutine内没有这个概念，防止被滥用，所以你不能在一个协程中杀死另外一个协程，编码时需要考虑到协程什么时候创建，什么时候释放。
   
3. goruntine调度：

   go调度里面有三个角色：三角形M代表内核线程，正方形P代表上下文，圆形G代表协程。一个M对应一个P，一个P下面挂多个G，但一个时候只有一个G在跑，其余都是放入等待队列，等待下一次切换时使用。

   * G0进入阻塞，那么P会转移到另外一个内核线程M1（此时还是1对1）。当syscall返回后，需要抢占一个P继续执行，如果抢占不到，G0挂入全局就绪队列runqueue，等待下次调度，理论上会被挂入到一个具体P下面的就绪队列runqueu（区别于全局runqueue）。

     ![goroutine3](http://vinllen.com/content/images/2018/goroutine/goroutine3.jpg)

   * 假如一个P0下面的所有G都跑完了，这时候会从别的P1下面就绪队列抢占G进行运行，个数为P1就绪队列的一半。

   ![goroutine4](http://vinllen.com/content/images/2018/goroutine/goroutine4.jpg)

4. Go协程优点

   * 与线程相比，Go协程的开销非常小。Go协程的堆栈大小只有几kb，它可以根据应用程序的需要而增长和缩小，而线程必须指定堆栈的大小，并且堆栈的大小是固定的。
   * Go协程被多路复用到较少的OS线程。在一个程序中数千个Go协程可能只运行在一个线程中。如果该线程中的任何一个Go协程阻塞（比如等待用户输入），那么Go会创建一个新的OS线程并将其余的Go协程移动到这个新的OS线程。所有这些操作都是 runtime 来完成的，我们只需要利用 Go 提供的简洁的 API 来处理并发就可以了。
   * Go 协程之间通过信道（channel）进行通信。信道可以防止多个协程访问共享内存时发生竟险（race condition）。信道可以想象成多个协程之间通信的管道。

5. Go 协程的特点

   * 有独立的栈空间

   * 共享程序堆空间

   * 调度由用户控制

   * 协程是轻量级的线程

6. Go协程的使用

   - 当创建一个Go协程时，创建这个Go协程的语句立即返回。与函数不同，程序流程不会等待Go协程结束再继续执行。程序流程在开启Go协程后立即返回并开始执行下一行代码，忽略Go协程的任何返回值。
   - 在主协程存在时才能运行其他协程，主协程终止则程序终止，其他协程也将终止。

   注：goroutine 中使用 recover解决协程中出现 panic，会导致程序崩溃。

   ```go
   import (  
       "fmt"
   )
   
   func hello() {  
       fmt.Println("Hello world goroutine")
   }
   func main() {  
       go hello()
       fmt.Println("main function")
   }
   // 输出：
   // main function
   ```

   * 用WaitGroup使主协程等待其他协程执行完再结束

   ```go
   import (
   	"fmt"
   	"sync"
   	"time"
   )
   
   func numbers() {
   	for i := 1; i <= 5; i++ {
   		time.Sleep(250 * time.Millisecond)
   		fmt.Printf("%d ", i)
   	}
   }
   func alphabets() {
   	for i := 'a'; i <= 'e'; i++ {
   		time.Sleep(250 * time.Millisecond)
   		fmt.Printf("%c ", i)
   	}
   }
   
   func main() {
   	var wg sync.WaitGroup
       // 告诉主协程有2个协程要执行
   	wg.Add(2)
   	go func() {
   		numbers()
           // 执行完协程数就-1
   		wg.Done()
   	}()
   	go func() {
   		alphabets()
   		wg.Done()
   	}()
       // 等待所有协程执行完毕
   	wg.Wait()
   	fmt.Println("main")
   }
   // 输出：
   // 1 a 2 b 3 c 4 d e 5 main
   ```

## 十六、channel

1. 使用channel原因，先介绍互斥锁

   ```go
   var (
   	s = 0
   	n = 0
   	lock sync.Mutex
   )
   
   func sum1(a int){
   	s+=a
       fmt.Println("a=", a)
       fmt.Println(s)
   }
   
   func sum2(a int){
   	lock.Lock()
   	defer lock.Unlock()
   	s+=a
   	n++
   	fmt.Println("a=", a)
       fmt.Println(s)
   }
   func main() {
       // 这样写的时候，s是混乱的
   	/*for i:=0;i<100;i++{
   		go sum1(i)
   	}
   	time.Sleep(time.Second*2)*/
       
       // 执行协程加了锁，需要当前协程结束，才会获得锁，下一个协程执行，
       // 但是循环一直在走，所以并不是从0一直加到99，
       // 而是当协程开始时，i计算到哪里，就加那个i。
       for i:=0;i<100;i++{
   		go sum2(i)
   	}
   	for{
   		if n == 100{
   			break
   		}
   	}
   }
   // 输出：
   // a= 1
   // 1
   // a= 28
   // 29
   // a= 38
   // 67
   // a= 39
   // 106
   // ...
   ```

   但是加锁就是要等待，就会影响效率，所以这里需要用到channel。

2. channel的基本介绍

   - channel 本质就是一个数据结构-队列。
   - 数据是先进先出【FIFO : 先进先出】。
   - 线程安全，多 goroutine 访问时，不需要加锁，就是说 channel 本身就是线程安全的。
   - channel 有类型的， string类型 的 channel 只能存放 string 类型数据。
   - channel是有大小的，数据放满之后，就不能放入了，需要取出之后才能再放入。
   - 在没有使用协程的情况下，如果 channel 数据取完了，再取，就会报 dead lock。
   - channel是引用类型。
   - channel 必须初始化才能写入数据, 即 make 后才能使用。

3. channel声明

   ```go
   // var 变量名 chan 数据类型
   var intChan chan int
   intChan = make(chan int, 3)
   
   fmt.Printf("intChan 的值=%v intChan 本身的地址=%p\n", intChan, &intChan) // intChan 的值=0xc000092000 intChan 本身的地址=0xc00008e018
   
   fmt.Printf("channel len= %v cap=%v \n", len(intChan), cap(intChan)) // channel len= 0 cap=3
   ```

4. 向channel写入和取出数据

   ```go
   var intChan chan int
   intChan = make(chan int, 3)
   
   // 向管道写数据
   intChan<- 10
   
   num := 211
   intChan <- num
   
   intChan <- 50
   
   fmt.Printf("channel len= %v cap=%v \n", len(intChan), cap(intChan)) // channel len= 3 cap=3
   
   // 从管道取数据
   num2 := <- intChan
   num3 := <- intChan
   num4 := <- intChan
   num5 := <- intChan // 报错deadlock!
   ```

## 十七、Go并发原理

1. 并发和并行

   * 并发：两个或两个以上的任务在一段时间内被执行。这些任务可能同时执行，也可能不是，我们只关心在一段时间内，哪怕是很短的时间（一秒或者两秒）是否执行解决了两个或两个以上任务。
   * 并行：两个或两个以上的任务在同一时刻被同时执行。

2. Go的CSP并发模型

   * Go实现了两种并发形式：多线程共享内存和CSP并发模型。

   * CSP并发模型：不同于传统的多线程通过共享内存来通信，CSP讲究的是“以通信的方式来共享内存”。“不要以共享内存的方式来通信，相反，要通过通信来共享内存。”
   * Go的CSP并发模型，是通过goroutine和channel来实现的。
   * 对线程来说，有三种映射（用户线程与内核线程的因素）模型：
     - 一对一模型（1:1）。一个用户线程映射到一个内核线程，用户线程在存活期都会绑定到一个内核线程，一旦退出，2个线程都会退出。优点是实现了真正的并发，多个线程同时跑在不同的CPU上；缺点是，如果用户线程起多了，内核线程肯定不够用，那么就需要切换，涉及到上下文的切换，代价比较大。
     - 多对一模型（M:1）。多个用户线程映射到一个内核线程。优点是，多个用户线程切换比较快，不需要内核线程上下文切换；缺点是，如果一个线程阻塞了，那么映射到同一个内核线程的用户线程将都无法运行。
     - **多对多模型（M:N）**。综合以上两种模型，go采用的就是这种。Go线程实现模型MPG。

## 十八、反射

1. 介绍

   * 反射可以在运行时动态获取变量的各种信息, 比如变量的类型(type)，类别(kind)；如果是结构体变量，还可以获取到结构体本身的信息(包括结构体的字段、方法)。
   * 通过反射，可以修改变量的值，可以调用关联的方法。
   * 使用反射，需要 import (“reflect”)。
   * Type： 类型用来表示一个go类型。
     在调用有分类限定的方法时，应先使用Kind方法获知类型的分类。调用该分类不支持的方法会导致运行时的panic。Kind是一个大的分类，比如定义了一个Person类，它的Kind就是struct 而Type的名称是Person。
   * Value： 为go值提供了反射接口。
     在调用有分类限定的方法时，应先使用Kind方法获知该值的分类。调用该分类不支持的方法会导致运行时的panic。Value类型的零值表示不持有某个值。零值的IsValid方法返回false，其Kind方法返回Invalid，而String方法返回""，所有其它方法都会panic。绝大多数函数和方法都永远不返回Value零值。如果某个函数/方法返回了非法的Value，它的文档必须显式的说明具体情况。如果某个go类型值可以安全的用于多线程并发操作，它的Value表示也可以安全的用于并发。
   * 变量、interface{} 和 reflect.Value 是可以相互转换的。

2. API

   ```go
   // ValueOf返回一个初始化为i接口保管的具体值的Value，ValueOf(nil)返回<invalid reflect.Value>。
   func ValueOf(i interface{}) Value
   
   // TypeOf返回接口中保存的值的类型，TypeOf(nil)会返回nil。
   func TypeOf(i interface{}) Type
   
   // 返回v持有的值的类型的Type表示。 其中Value可以获取到Type ,value操作的是指定的数据值其中包含数据的类型和具体的值，Type操作的是数据的类型结构，如：属性。
   func (v Value) Type() Type
   
   // 返回v持有的值的类型的分类类型。 其中Value可以获取到Kind，如：指针。
   func (v Value) Kind() Type
   
   // Elem返回v持有的接口保管的值的Value封装，或者v持有的指针指向的值的Value封装。如果v持有的值为nil，会返回panic: reflect: call of reflect.Value.Elem on zero Value。这个是Value的Elem方法，通常用这个方法获取数值类型的指针指向的数值。如ValueOf取到指针的值是个地址，指针地址的Elem就是地址对应的值了。
   func (v Value) Elem() Value
   
   // 返回索引序列指定的嵌套字段的类型，等价于用索引中每个值链式调用本方法，如非结构体将会panic。
   func (Type) Field(i int) StructField
   
   // 示例：
   ptrValueOfI := reflect.ValueOf(i)
   ptrTypeOfI := reflect.TypeOf(i)
   valueOfI := ptrValueOfI.Elem()
   
   valueOfI.SetInt(1000)
   
   fmt.Println(ptrValueOfI)
   fmt.Println(ptrValueOfI.Kind())
   fmt.Println(ptrTypeOfI)
   fmt.Println(ptrValueOfI.Elem())
   fmt.Println(ptrValueOfI.Type())
   // 输出：
   // 0xc00000a0b8
   // ptr
   // *int
   // 1000
   // *int
   // 1000
   ```

3. 反射的使用

   * 基本类型的值修改

     ```go
     import (
     	"fmt"
     	"reflect"
     )
     
     func testBasic(i interface{}){
     	//i实际是一个指针
     	ptrValueOfI := reflect.ValueOf(i)
     	fmt.Println(ptrValueOfI.Kind())
     	//获取指针指向的真正的值
     	valueOfI := ptrValueOfI.Elem()
     	//为其更改值
     	valueOfI.SetInt(1000)
     }
     
     func main() {
     	n:=1
     	testBasic(&n)
     	fmt.Println(n)
     }
     ```

   * struct类型的属性修改

     ```go
     import (
     	"fmt"
     	"reflect"
     )
     
     type Student struct{
     	Name string
     	Age int64
     }
     
     func test(i interface{}) {
     	valueOfI := reflect.ValueOf(i).Elem()
     	typeOfI := valueOfI.Type()
         
     	if typeOfI.Kind() != reflect.Struct {
     		fmt.Println("except struct")
     		return
     	}
         
     	fmt.Println(reflect.TypeOf(i))
     	fmt.Println(reflect.ValueOf(i).Type())
         
     	numField := typeOfI.NumField()
     	for i := 0; i < numField; i++ {
     		field := typeOfI.Field(i)
     		fileName := field.Name
     		fmt.Println(fileName)
     		if fileName == "Name" {
     			valueOfI.Field(i).SetString("reece")
     		}
     		if fileName == "Age" {
     			valueOfI.Field(i).SetInt(22)
     		}
     	}
     }
     
     func main () {
     	student := Student{Name:"Susan", Age: 20}
     	test(&student)
     	fmt.Println(student.Name)
     }
     
     // 输出：
     // *main.Student
     // *main.Student
     // Name
     // Age
     // reece
     ```
   
   * 引用类型修改
   
     ```go
     import (
     	"fmt"
     	"reflect"
     )
     
     func testSlice(i interface{}) {
     	valueOfI := reflect.ValueOf(i)
     	fmt.Println(valueOfI.Kind())
     	fmt.Println(valueOfI.Len())
     	fmt.Println(valueOfI.Cap())
     
     	indexValue := valueOfI.Index(0)
     	fmt.Println(indexValue)
     	indexValue.SetInt(300)
     }
     
     func main () {
     	slice := make([]int, 10, 20)
     
     	slice[0] = 200
     
     	testSlice(slice)
     	fmt.Println(slice[0])
     }
     
     // 输出：
     // slice
     // 10
     // 20
     // 200
     // 300
     ```
   
   * Map类型修改值
   
     ```go
     import (
     	"fmt"
     	"reflect"
     )
     
     func test(i interface{}) {
     	valueOfI := reflect.ValueOf(i)
     
     	lenOfI := valueOfI.Len()
     	fmt.Println("len = ",lenOfI)
     
     	keysNum := valueOfI.MapKeys()
     	for n := 0; n < len(keysNum); n++ {
     		key := keysNum[n]
     		value := valueOfI.MapIndex(key)
     
     		fmt.Println("key=",key," value=", value)
     
     		str := value.String()
     		valueOfI.SetMapIndex(key,reflect.ValueOf(string(str + "2")))
     	}
     }
     
     func main() {
     	m := make(map[int]string, 10)
     
     	m[0] = "zero"
     	m[1] = "one"
     	m[2] = "two"
     
     	test(m)
     	fmt.Println(m)
     }
     
     // 输出:
     // key= 0  value= zero
     // key= 1  value= one
     // key= 2  value= two
     // map[0:zero2 1:one2 2:two2]
     ```
   
   * 结构体反射执行方法
   
     ```go
     import (
     	"fmt"
     	"reflect"
     )
     
     type Student struct {
     	Name string
     	Age  int
     }
     
     func (s Student) Print() {
     	fmt.Println("name=", s.Name, ";age=", s.Age)
     }
     
     func (Student) Plus(a int, b int) int {
     	return a + b
     }
     
     func testMethod(i interface{}){
         valueOfI := reflect.ValueOf(i).Elem()
     	typeOfI := valueOfI.Type()
     
     	methodsNum := typeOfI.NumMethod()
     
     	for n := 0; n < methodsNum; n++ {
     		method := typeOfI.Method(n)
     		methodName := method.Name
     		if methodName == "Plus" {
     			m := valueOfI.MethodByName(methodName)
     			args := make([]reflect.Value, 2)
     			args[0] = reflect.ValueOf(78)
     			args[1] = reflect.ValueOf(80)
     			res := m.Call(args)
     			for j := 0; j < len(res); j++ {
     				fmt.Println(methodName," ivoke result", j+1, " : ", res[j].Int())
     			}
     		}
     	}
     }
     
     func main() {
     	stu := Student{Name: "susan", Age: 58}
     	testMethod(&stu)
     }
     
     // 输出：
     // Plus  ivoke result 1  :  158
     ```

## 十九、tcp网络编程

1. 介绍

   * Golang 的主要设计目标之一就是面向大规模后端服务程序，网络通信这块是服务端 程序必不可少
     也是至关重要的一部分。
   * TCP socket 编程，是网络编程的主流。之所以叫 Tcp socket 编程，是因为底层是基于 Tcp/ip 协
     议的. 比如: QQ 聊天。
   * b/s 结构的 http 编程，我们使用浏览器去访问服务器时，使用的就是 http 协议，而 http 底层依
     旧是用 tcp socket 实现的。比如: 京东商城。

2. 协议

   > OSI模型                      TCP/IP模型
   >
   > >应用层					应用层
   > >
   > >表示层					
   > >
   > >会话层
   > >
   > >传输层					传输层
   > >
   > >网络层					网络层
   > >
   > >数据链路层			链路层
   > >
   > >物理层

   ![å¨è¿éæå¥å¾çæè¿°](https://img-blog.csdnimg.cn/2019030215571166.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNzkxMDQ1Mw==,size_16,color_FFFFFF,t_70)

3. 利用go的tcp网络编程Api实现一个简单的http服务器案例

   ```go
   import (
   	"fmt"
   	"net"
   	"strconv"
   )
   
   //用来转化int为string
   type Int int
   
   func (i Int)toStr()string  {
   	return strconv.FormatInt(int64(i),10)
   }
   
   func testConn(conn net.Conn){
   	addr := conn.RemoteAddr()
   	fmt.Println(addr)
   	//返回的http消息体
   	var respBody  = "<h1>Hello World<h1>"
   	i := (Int)(len(respBody))
   	fmt.Println("响应的消息体长度："+i.toStr())
   	//返回的http消息头
   	var respHeader  = "HTTP/1.1 200 OK\n"+
   	"Content-Type: text/html;charset=ISO-8859-1\n"+
   	"Content-Length: "+i.toStr()
   	//拼装http返回的消息
   	resp := respHeader +"\n\r\n"+ respBody
   	fmt.Println(resp)
   	n, _ := conn.Write([]byte(resp))
   	fmt.Printf("发送的数据数量：%s\n",(Int)(n).toStr())
   }
   
   func main() {
   	listen, err := net.Listen("tcp", "0.0.0.0:8888")
   	if err !=nil{
   		fmt.Println(err)
   		return
   	}
   	defer listen.Close() //延时关闭 listen
   
   	for{
   		conn, err := listen.Accept()
   		if err !=nil{
   			fmt.Println(err)
   			continue
   		}
   		go testConn(conn)
   	}
   }
   
   // 输出：
   // 打开127.0.0.1:8080页面出现“Hello World!”字样
   /*控制台
   127.0.0.1:59358
   响应的消息体长度: 21
   HTTP/1.1 200 OK
   Content-Type: text/html;charset=ISO-8859-1
   Content-Length: 21
   
   <h1>Hello World!</h1>
   发送的数据数量:101
   127.0.0.1:59359
   响应的消息体长度: 21
   HTTP/1.1 200 OK
   Content-Type: text/html;charset=ISO-8859-1
   Content-Length: 21
   
   <h1>Hello World!</h1>
   发送的数据数量:101
   */
   ```

## 二十、单元测试

1. 测试

   * 测试文件必须以*_test.go的格式命名；
   * 测试方法以Test*命名
   * 参数类型是t *testing.T

   ```go
   // call.go
   package test
   
   //求平方
   func pow(a int) int {
   	return a*a
   }
   
   func sub(a int) int  {
   	return a-2
   }
   ```

   ```go
   // call_test.go
   package test
   
   import "testing"
   
   func TestSub(t *testing.T) {
   	i := sub(2)
   	if i != 0{
   		t.Fatal("sub(2) result is wrong")
   	}
   	t.Log("sub(2) result is success")
   }
   
   func TestPow(t *testing.T)  {
   	i := pow(2)
   	if i != 4{
   		t.Fatal("pow(2) result is wrong")
   	}
   	t.Log("pow(2) result is success")
   }
   ```

   运行：

   ```shell
   // 进入目录，需要把测试文件和被测试文件写明
   go test -v -cover call_test.go call.go 
   ```

2. 性能测试

   * 以Benchmark*的格式命名方法

   ```go
   //benchmark_test.go
   package cal
   
   import (
       "fmt"
       "testing"
   )
   
   func BenchmarkHello(b *testing.B) {
       for i := 0; i < b.N; i++ {
       	fmt.Sprintf("hello")
   	}
   }
   ```

   运行：

   ```shell
   $go test -v -bench="." benchmark_test.go
   goos: windows
   goarch: amd64
   BenchmarkHello-6        23078031                54.6 ns/op
   PASS
   ok      command-line-arguments  4.033s
   ```

   * 第 1 行 -bench="." 表示运行 benchmark_test.go 文件里面全部的测试，其实和-run一样【 -bench regexp 是可以接收一个正则，如果要运行所以的基准测试，请使用-bench. or -bench=.'.】
   * 第 2 行 goos 表示系统是 windows
   * 第 3 行 goarch 表示 操作系统构架是amd64
   * 第 4 行 BenchmarkHello-6 表示 测试名称 ， 23078031测试的次数 ， 54.6 ns/op表示表示每一个操作耗费多少时间（纳秒）

3. testing.B 提供了几种方法【testing.B拥有testing.T的全部接口】

   | 方法 | 描述 |
   | :----: | :----: |
   | StartTimer() | 启动计时 |
| StopTimer() | 停止计时 |
   | ResetTimer | 重置计时 |
   | SetBytes() | 设置处理字节数 |
   | ReportAllocs() | 报告内存信息 |
   | runN(n int) | 运行一个基准函数 |
   

## 参考：

* go中的协程与线程的区别:

  http://vinllen.com/gozhong-de-xie-cheng-goroutineyu-xian-cheng-de-qu-bie/

* golang基础教程:
https://blog.csdn.net/weixin_37910453/article/details/87276411

