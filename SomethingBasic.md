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
一个新的结构体，这个结构体的数据指针指向原底层数组。所以对于Slice的赋值操作(包括函数传值和函数返回值)，新的切片是可以修改原切片
的数据。

[Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals)

#### 2.About range and loop with index
Range 和 普通的for循环都可以遍历数组/Slice/Map，字符串在Go中实际上是不可变的byte slice,也可以用这两种方式遍历，但是以下讨论不包括range channel.

  - Range 得到的value值是原值的拷贝，无法通过value值修改原Slice/Array/Map


