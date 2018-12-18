## Small things

#### 1. About package-level variable

  - 包级别的变量可以只声明不使用
  - 包级别的变量不能使用简短声明
  
```go
    package main
    
    import "fmt"
    
    var global1 int // 可编译可运行
    global2 := 1	// 编译出错
    
    func main() {
    	fmt.Println("this is main")
    }
```



#### 2.About array and slice
   
   - 数组在赋值的时候是值传递,传递的是原数组数据的一份拷贝,所以在函数内部无法修改外层函数中的数组.
   ```go
        func main() {
            array1 := [4]int{1, 2, 3, 4}
            array2 := array1                // array2是array1的一份拷贝
            array4 := getArray(array1)      // array4是array1的一份拷贝
        
            array1[0] = -1
            array2[1] = -2
            array4[3] = -4
            fmt.Println("array1:", array1)
            fmt.Println("array2:", array2)
            fmt.Println("array4:", array4)
        }
        
        func getArray(array3 [4]int) [4]int {
            array3[2] = -3                  // array3是array1的一份拷贝
            fmt.Println("array3:", array3)
            return array3
        }
   ```
   > array3: [1 2 -3 4]\
     array1: [-1 2 3 4]\
     array2: [1 -2 3 4]\
     array4: [1 2 -3 -4]
   
   结论：将数组变量a赋值给变量b时，b会得到a中数据的一份拷贝，所以对b中数组元素进行更改时不会影响a中的数据，
给函数传递参数和得到的函数返回值也是如此。

如果想在调用的函数内部改变原始的数组内容，可以传递原数组的指针或者原数组的一个slice


   - Slice 在赋值的时候也是值传递，但是拷贝的数据和原始数据有相同的underlying array，所以可以在函数内部修改原始slice的数据.
   
   ```go   
       func main() {
        array1 := []int{1, 2, 3, 4}
        array2 := array1			// 赋值
        array4 := getArray(array1)	// 函数返回值
       
        array1[0] = -1
        array2[1] = -2
        array4[3] = -4
        fmt.Printf("array1: %v, %p\n", array1, &array1)
        fmt.Printf("array2: %v, %p\n", array2, &array2)
        fmt.Printf("array4: %v, %p\n", array4, &array4)
       }
       
       func getArray(array3 []int) []int {
        array3[2] = -3				// 函数传参
        fmt.Printf("array3: %v, %p\n", array3, &array3)
        return array3
       }
   ```
   > array3: [1 2 -3 4], 0xc42000a0c0\
     array1: [-1 -2 -3 -4], 0xc42000a060\
     array2: [-1 -2 -3 -4], 0xc42000a080\
     array4: [-1 -2 -3 -4], 0xc42000a0a0
     
   结论：Slice实际上是一个包含数据指针，长度(len)和最大长度(cap)的结构体。数据指针指向实际的底层数组([]int{1,2,3,4})，而且切片操作会创建
一个新的结构体，这个结构体的数据指针指向原底层数组。所以对于Slice的赋值操作(包括函数传值和函数返回值)，新的切片可以修改原切片
的数据。

[Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals)

#### 2.About range and loop with index
Range和普通的for循环都可以遍历Array/Slice/Map，字符串在Go中实际上是不可变的byte slice,也可以用这两种方式遍历，以下讨论不包括range channel.

  - Range得到的value值是原值的拷贝，如果使用Slice/Array/Map保存结构体，无法通过value值修改原结构体
```go
    type Item struct {
        data int32
    }
    
    func main() {
        var array = [4]Item{{1}, {2}, {3}, {4}}
        var slice = []Item{{1}, {2}, {3}, {4}}
        testMap := map[int]Item{1: {1}, 2: {2}, 3: {3}, 4: {4}}
    
        for _, it := range array {
            it.data = 1					// 无法修改原数组中的数据
        }
        for _, it := range slice {
            it.data = 1					// 无法修改原slice中的数据
        }
        for _, it := range testMap {
            it.data = 1					// 无法修改原map中的数据
        }
    
        fmt.Println(array)
        fmt.Println(slice)
        fmt.Println(testMap)
    }
```
   > [{1} {2} {3} {4}]\
     [{1} {2} {3} {4}]\
     map[1:{1} 2:{2} 3:{3} 4:{4}]
   - 如果想修改Slice/Array中结构体数据的某个域，可以保存结构体数据的指针或者使用索引遍历。
   对于Map，只有保存指针才可以直接修改。
   
```go
    func main() {
        var array = [4]*Item{{1}, {2}, {3}, {4}}
        var slice = []Item{{1}, {2}, {3}, {4}}
        testMap1 := map[int]Item{1: {1}, 2: {2}, 3: {3}, 4: {4}}
        testMap2 := map[int]*Item{1: {1}, 2: {2}, 3: {3}, 4: {4}}
    
        for _, it := range array {
            it.data = 1 				// 可以修改原数组中的结构体数据
        }
        for i := 0; i < len(slice); i++ {
            slice[i].data = 1 			// 可以修改原slice中的结构体数据
        }
        for i := range testMap1 {
            //testMap1[i].data = 1 		// compile error， 无法直接修改
            it := testMap1[i]			// 可以先复制，然后替换掉原数据
            it.data = 1
            testMap1[i] = it
        }
        for i := range testMap2 {
            testMap2[i].data = 1 		// 如果保存结构体指针，可以直接修改
        }
    
        for _, it := range array {
            fmt.Print(it, " ")
        }
        fmt.Println()
        for _, it := range slice {
            fmt.Print(it, " ")
        }
        fmt.Println()
        for _, it := range testMap1 {
            fmt.Print(it, " ")
        }
        fmt.Println()
        for _, it := range testMap2 {
            fmt.Print(it, " ")
        }
        fmt.Println()
    }
```
   > &{1} &{1} &{1} &{1}\
     {1} {1} {1} {1}\
     {1} {1} {1} {1}\
     &{1} &{1} &{1} &{1} 

   - 使用range遍历字符串时，value会按UTF8取一个字符(Go字符串默认UTF8编码）。而使用索引遍历时，每次会步进一个字节。
   ```go
       func main() {
        var str1 = "这只是个测试"
        for i, value := range str1 {
            fmt.Printf("%c %d\n", value, i) // 输出中文，步进三个字节
        }
        fmt.Println()
        for i := 1; i < len(str1); i++ {
            fmt.Printf("%c", str1[i])       //每次步进一个字节
        }
       }
   ```
   > 这 0\
     只 3\
     是 6\
     个 9\
     测 12\
     试 15\
     ¿åªæ¯ä¸ªæµè¯

#### 2.About defer
   * 如果一个函数内包含多个defer，函数退出时先执行最后声明的defer函数，最后执行第一个声明的defer函数，执行顺序类似栈。
```go
    func main() {
        defer func() {fmt.Println("1111")}()
        defer func() {fmt.Println("2222")}()
        defer func() {fmt.Println("3333")}()
        defer func() {fmt.Println("4444")}()
        return
    }
```
   > 4444\
     3333\
     2222\
     1111
   * Defer函数的执行先于panic函数执行结束
```go
   func main() {
    defer func() {fmt.Println("1111")}()
    defer func() {fmt.Println("2222")}()
    defer func() {fmt.Println("3333")}()
    defer func() {fmt.Println("4444")}()
    panic("11111")
    return
   }
```
   > 4444\
     3333\
     2222\
     1111\
     panic: 11111
     
   * Defer函数的执行时机在给函数返回值赋值之后，函数调用返回之前
```go
    func main() {
        fmt.Println(returnTest())
    }
    
    func returnTest() (res int){
        res = 1
        defer func() {res++}()
        return 0
    }   
```
   > 1
   
   相当于
   > res = 0 // 给返回值赋值 \
     res++   // 执行defer函数\
     return  //函数返回
#### 2.About channel
   * 在值为nil的channel上的读写操作都会造成阻塞
```go
    func main() {
        var ch chan int
        go func() {
            ch <- 1 // Blocked on nil channel
            fmt.Println("Goroutine A is free")
        }()
        <-ch // Blocked on nil channel
        fmt.Println("Main Goroutine is free")
    }
```
   > fatal error: all goroutines are asleep - deadlock!
   
   * 向已经关闭的channel发消息会造成panic，监听已经关闭的channel会一直收到该channel类型的零值
```go
    func main() {
        ch := make(chan int)
        close(ch)
        ch <- 1 // panic when sending to a closed channel
    }
```
   > panic: send on closed channel
   
```go
    func main() {
        ch := make(chan int)
        close(ch)
        for {
            select {
            case <-ch:
                fmt.Println("Receive from channel")
            }
        }
    }
```
   > Receive from channel\
     Receive from channel\
     ...

## Small tips
   * 监听channel时，最好能够特殊处理channel类型的零值，避免在不知情的情况下channel关闭造成的性能问题 
```go
    func main() {
        ch := make(chan int)
        close(ch)
        for {
            select {
            case msg := <-ch:
                if msg == 0{ // 特殊处理channel关闭的情况
                    fmt.Println("Channel is closed")
                }
                fmt.Println("Receive from channel")
            }
        }
    }  
```
如果goroutine仅仅监听一个channel,不处理channel关闭的情况有可能造成协程泄露，一直都无法退出。
如果goroutine监听多个channel,不处理channel关闭的情况可能让协程一直命中监听关闭channel的case，而其他的case一直都无法执行。
   * 关闭channel的操作最好由channel的sender执行，如果有无数个sender或者无法在作为sender的协程上关闭，在receiver端关闭，并
   在sender端加上recover
```go
    func main() {
        ch := make(chan int)
        go func() {
            defer func() {
                fmt.Println("sender panic") // 接收端关闭后，发送端继续发送会panic
                recover()
            }()
            for {
                ch <- 2
                fmt.Println("sender sends")
            }
        }()
        <-ch
        close(ch) // 在接收端关闭channel
        time.Sleep(5 * time.Second)
    }   
```
   > sender sends\
     sender panic
   
   * 如果无法确定作为channel接收端的协程是否一直存在，channel发送端的协程应加上超时处理。否则容易造成发送端协程泄露
   [Timing out, moving on](https://blog.golang.org/go-concurrency-patterns-timing-out-and)
```go
    func main() {
        ch := make(chan int)
        go func() {
            select {
            case ch <- 2:
            case <-time.After(time.Second): // 1s后receiver端没有接收，sender继续
                fmt.Println("sender time out")
            }
            fmt.Println("sender continues")
            // do something
        }()
        time.Sleep(5 * time.Second)
    }
```
   > sender time out\
     sender continues\
     sender time out\
     sender continues\
     sender time out\
     sender continues\
     sender time out\
     sender continues
     
   * 如果select中有多个case有消息需要处理，执行哪个case是随机的，各个case的执行顺序也是随机的。如下代码利用在值为nil的channel
   上收发都会阻塞的特点实现了按一定顺序从各个channel中收发消息，工程代码慎用。。。
```go
    func main() {
        ch1 := make(chan int)
        ch2 := make(chan int)
        go func(ch1, ch2 chan int) {
            var cha = ch1
            var chb chan int
            for {
                select {
                case a := <-cha:
                    fmt.Println("receive from cha", a)
                    cha = nil  // 从ch1中收到数据后将cha置为nil，停止从ch1中收取消息
                    chb = ch2  // 准备从ch2中收取消息
                case b := <-chb:
                    fmt.Println("receive from chb", b)
                    chb = nil  // 从ch2中收到数据后将chb置为nil，停止从ch2中收取消息
                    cha = ch1  // 准备从ch1中收取消息
                }
            }
        }(ch1, ch2)
    
        sequence := []int{1, 2, 3}
        for i := range sequence {
            go func(x int) { ch1 <- x }(i)
        }
        for i := range sequence {
            go func(x int) { ch2 <- x }(i)
        }
        time.Sleep(5 * time.Second)
    }
```
   > receive from cha 0\
     receive from chb 0\
     receive from cha 1\
     receive from chb 1\
     receive from cha 2\
     receive from chb 2
     
   * 设计多个channel交互时，消息最好能够单向流动。
```go
    func main() {
        ch1 := make(chan int)
        ch2 := make(chan int)
        report := make(chan int)
        go func(chb, rep chan int) {        // goroutine #1
            for {
                select {
                case a := <-chb:
                    fmt.Println("receive from cha", a)
                    // do something
                    time.Sleep(time.Second)
                    //...something is wrong, report to upper goroutine
                    rep <- 1
                }
            }
        }(ch2, report)
        go func(cha, chb, rep chan int) {   // goroutine #2
            for {
                select {
                case rec := <-cha:
                    fmt.Println("send message")
                    // do something
                    time.Sleep(3 * time.Second)
                    chb <- rec
                case re := <-rep:
                    fmt.Println("ask goroutinue to exit", re)
                }
            }
        }(ch1, ch2, report)
        for {
            ch1 <- 1
        }
    }
```
   > send message\
     receive from cha 1\
     send message\
     fatal error: all goroutines are asleep - deadlock!

主协程一直给#2协程发送消息，#2进行一些处理后给#1发消息，#1在处理消息后想report给#2，可是
这时#2在执行第一个case想给#1发消息，造成死锁。解决方法包括将不带缓冲的channel改成带缓冲的
channel，#1report给#2在新的协程中执行，在#1和#2的下游增加协程处理汇总信息保证单向流动等。