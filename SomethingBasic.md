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
   
   - 数组在赋值的时候是值传递, 传递的是原数组数据的一份拷贝,所以在函数内部无法修改外层函数中的数组.
   ```go
package main

import "fmt"

func main() {
	array1 := [4]int{1, 2, 3, 4}
	array2 := array1
	array4 := getArray(array1)

	array1[0] = -1
	array2[1] = -2
	array4[3] = -4
	fmt.Println("array1:", array1)
	fmt.Println("array2:", array2)
	fmt.Println("array4:", array4)
}

func getArray(array3 [4]int) [4]int {
	array3[2] = -3
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
   package main
   
   import "fmt"
   
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
