# Basic

  ### Index-loop
```go
  a := make([]int, 10)
  for i:=0;  i< len(a); i++{
    fmt.Println(a[i])
  }
```

### Range-loop
```go
  a := make([]int, 10)
  for i, x := range a{
	  fmt.Println(i, x)
  }
```

# Differences
1. ***两种循环都可读写数组和切片，Range还能读写Map，go的字符串本身只读。但是Range遍历的第二个迭代变量是元素的复制，一般不能在此基础上修改原始数据***
```go
	array := [...]int{1,1,1,1,1,1}

	//index-loop读写数组
	for i:=0; i < len(array); i++{
		fmt.Println(array[i])// 输出 1
		array[i] = 2// 生效
	}
	fmt.Println(array)// 输出 [2 2 2 2 2 2]

	//range-loop读写数组
	for i, x := range array{
		fmt.Println(x)// 输出 2
		array[i] += 2// 生效
		x++// 不生效
	}
	fmt.Println(array)// 输出 [4 4 4 4 4 4]
```
```go
	slic := []int{1,1,1,1,1}
	//index-loop读写切片
	for i:=0; i < len(slic); i++{
		fmt.Println(slic[i])// 输出 1
		slic[i] = 2// 生效
	}
	fmt.Println(slic)// 输出 [2 2 2 2 2 2]

	//range-loop读写切片
	for i, x := range slic{
		fmt.Println(x)// 输出 2
		slic[i] += 2// 生效
		x++// 不生效
	}
	fmt.Println(slic)// 输出 [4 4 4 4 4 4]
```
```go
	str := "aaa"
	//index-loop读写字符串
	for i:=0; i < len(str); i++{
		fmt.Printf("%c", str[i])// 输出 a
		//str[i] = 'a'// CE
	}
	fmt.Println()
	fmt.Println(str)//输出 aaa

	//range-loop读写字符串
	for i, x := range str{
		fmt.Printf("%c%c\t", str[i], x)//输出 aa
		//str[i] = 'x'// CE
		x = 'x'// 不会CE但是不生效
	}
	fmt.Println()
	fmt.Println(str)//输出 aaa
```
```go
type Person struct {
	name 	string
	age 	int
}
	array := [...]Person{{name:"aaa", age:100}}

	//index-loop读写数组
	for i:=0; i < len(array); i++{
		fmt.Println(array[i])//{aaa 100}
		array[i].age = 200//生效
	}
	fmt.Println(array)//[{aaa 200}]

	//range-loop读写数组
	for i, x := range array{
		fmt.Println(x)//{aaa 200}
		array[i].age += 100// 生效
		x.age+=200// 不生效
	}
	fmt.Println(array)//[{aaa 300}]
```
```go
	mp := map[int]int{1:1, 2:2}

	//Range-loop读写Map
	for k, v := range mp{
		fmt.Println(k, v)//1 1\n2 2
		mp[k] += 2//生效
		v += 1//不生效
	}
	fmt.Println(mp)//map[1:3 2:4]
```

2. ***如果数组/切片/Map中的元素是引用类型的话，Range遍历的第二个迭代元素“可以”修改原始数据***
```go
	array1 := [...][1]int{{1},{1},{1}}//二维数组，元素类型为[1]int
	array2 := [...][]int{{1},{1},{1}}//一维数组，元素类型为切片
	array3 := [...]map[int]int{{1:1}, {1:2}}//一维数组，元素类型为map[int]int
	fmt.Printf("%T, %T, %T\n", array1, array2, array3)//输出 [3][1]int, [3][]int, [2]map[int]int

	for i, x := range array1{
		array1[i][0] += 2//生效
		x[0] += 1//不生效
	}
	fmt.Println(array1)//[[3] [3] [3]]

	for i, x := range array2{
		array2[i][0] += 2//生效
		x[0] += 1//生效
	}
	fmt.Println(array2)//[[4] [4] [4]]

	for i, x := range array3{
		array3[i][1] += 1//生效
		x[1] += 2//生效
		for k, v := range array3[i]{
			array3[i][k] += 10//生效
			v+=20//不生效
		}
		for k, v := range x{
			x[k] += 100//生效
			v+=200//不生效
		}
	}
	fmt.Println(array3)//[map[1:114] map[1:115]]
```
        原因在于slice和map是引用类型，在迭代时复制的是那个指向底层数据的结构体，slice的引用结构体大致长这样
    type Slice struct {
        Data unsafe.Pointer
        Len  int
        Cap  int
    }
        Map的结构体要复杂很多，本质上也是一个不保存数据只指向底层数据的结构。迭代时x复制的是元素的结构体，他们指向同一份底层数据，
    所以看上去能够修改原始数据。但是它只是修改了一部分数据，引用数据没有随之更新。看下面的例子可以明白这两者的区别
```go
	array := [...][]int{make([]int, 1, 3)}

	for _, x := range array{
		x[0] = 1//生效
		x = append(x, 3)//实际上也生效
		fmt.Println(x, len(x), cap(x))//[1 3] 2 3
	}
	fmt.Println(array, len(array[0]), cap(array[0]))//[[1]] 1 3
	fmt.Println(array[0][1])//报错
```
        这个例子中元素类型是slice所以x可以改底层数组的数据，但是往其中添加元素的append操作看起来没有生效，实际上是新的元素已经追加，
    但是元素的引用结构体中的len域没有更新，所以array[0]还是{1}。
        还有一种情况是append之后连array[0]长度范围之内的数据都改不掉。根据append的处理逻辑，如果追加的元素数量使得切片长度会超过cap，
    那么就会申请一片新的底层数组。把旧数据复制过去，然后再做追加。这个时候的迭代变量x就和原始数据一点关系都没有了
```go
	array := [...][]int{make([]int, 2)}

	for _, x := range array{
		x[0] = 1//生效
		x[1] = 2//生效
		x = append(x, []int{2,2,2}...)//会重新分配底层数组
		x[0] = 100//不会修改原始数据
		x[1] = 200//不会修改原始数据
		fmt.Println(x, len(x), cap(x))//[100 200 2 2 2] 5 6
	}
	fmt.Println(array, len(array[0]), cap(array[0]))//[[1 2]] 2 2
```
* Tips:
    * **创建二维数组时元素数组的长度需要显式指定，不能交由编译器判断。[...][...]int是错误的**
    * **拿Range遍历的元素复制x去修改原始数据始终有风险，除非这个元素是指针类型**

3. ***如果数组/切片/Map中的元素是指针类型的话，Range遍历的第二个迭代元素真的可以修改原始数据***
```go
	array := [...]*int{new(int)}

	//index-loop读写数组
	for i:=0; i < len(array); i++{
		fmt.Println(*array[i])//0
		*array[i] = 2// 生效
	}
	fmt.Println(*array[0])//2

	//range-loop读写数组
	for i, x := range array{
		fmt.Println(*x)//2
		*array[i] += 2// 生效
		*x++// 生效
	}
	fmt.Println(*array[0])//5
```
```go
	slic := []*int{new(int)}

	//index-loop读写数组
	for i:=0; i < len(slic); i++{
		fmt.Println(*slic[i])//0
		*slic[i] = 2// 生效
	}
	fmt.Println(*slic[0])//2

	//range-loop读写数组
	for i, x := range slic{
		fmt.Println(*x)//2
		*slic[i] += 2// 生效
		*x++// 生效
	}
	fmt.Println(*slic[0])//5
```
```go
	type Person struct {
		name 	string
		age 	int
	}
	array := [...]*Person{{name:"aaa", age:100}}

	//index-loop读写数组
	for i:=0; i < len(array); i++{
		fmt.Println(*array[i])//{aaa 100}
		array[i].age = 200//生效
	}
	fmt.Println(*array[0])//[{aaa 200}]

	//range-loop读写数组
	for i, x := range array{
		fmt.Println(*x)//{aaa 200}
		array[i].age += 100// 生效
		x.age+=200// 生效
	}
	fmt.Println(*array[0])//[{aaa 500}]
```
```go
	type Person struct {
		name string
		age  int
	}
	mp := map[int]*Person{1: {name: "aaa", age: 2}}

	//Range-loop读写Map
	for k, v := range mp {
		fmt.Println(k, *v)    //1 {aaa 2}
		(*mp[k]).name = "bbb" //生效
		(*v).age = 10         //生效
	}
	fmt.Println(*mp[1]) //{bbb 10}
```
```go
	array := [...]*[]int{{1, 1}}

	for _, x := range array{
		(*x)[0] = 1//生效
		(*x)[1] = 2//生效
		*x = append(*x, []int{2,2,2}...)//会重新分配底层数组
		(*x)[0] = 100//生效
		(*x)[1] = 200//生效
		(*x)[2] = 300//生效
		fmt.Println(*x, len(*x), cap(*x))//[100 200 300 2 2] 5 6
	}
	fmt.Println(*(array[0]), len(*(array[0])), cap(*(array[0])))//[100 200 300 2 2] 5 6
```
4. ***Index-loop遍历字符串时，是按byte遍历，str[i]类型为byte,而byte是uint8的类型别名。
      Range-loop遍历字符串时str[i]类型为byte，而x的类型为rune，rune是int32的别名***
```go
	str := "a"
	var tmp1 int64
	var tmp2 uint8
	for i:=0; i < len(str); i++{
		fmt.Printf("%T\n", str[i])//uint8
		//tmp1 = str[i]//CE
		//tmp1 = int64(str[i])//可行
		tmp2 = str[i]//可行
	}
	fmt.Println(tmp1, tmp2)//0 97
```
```go
	str := "a"
	var tmp1 int32
	var tmp2 uint8
	for i, x := range str{
		fmt.Printf("%T %T\n", str[i], x)//uint8 int32
		//tmp1 += str[i]//CE
		tmp2 += str[i]//可行
		tmp1 += x*2//可行
		//tmp2 += x*2//CE
	}
	fmt.Println(tmp1, tmp2)//194 97
```
* ***Tip:Go是强类型语言，所以这里即使int64的范围比uint8的范围大，这两个变量类型不一致也是不能直接赋值的。但是可以直接赋给uint8，
    因为他们有相同的底层类型,rune能够直接赋值给int32不能直接赋值给uint8也是同理***
5. ***Range-loop还可以监听channel，它会一直监听直到channel被close***
```go
	ch := make(chan int)
	go func(c chan int) {
		for i:=0; i < 2; i++{
			c <- 1
		}
		close(c)
	}(ch)

	for x := range ch{
		fmt.Println(x)
	}
```
