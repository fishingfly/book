# 数组类型转换



最近写代码的时候会遇到类型转换，有时候对单独的变量强制转换没有问题，但要是对复杂的变量（例如数组）进行强制转换就会出现问题。

#### 问题1：可以将\[\]T1转换为\[\]T2吗？T1和T2底层类型是一样的。

答案是不能，看一下例子：

```text
type T1 int
type T2 int
var t1 T1
var x = T2(t1) // OK
var st1 []T1
var sx = ([]T2)(st1) // NOT OK
```

在Go中，类型与方法紧密相关，因为每个命名类型都有一个（可能为空）方法集。一般规则是，您可以更改要转换的类型的名称（从而可能更改其方法集），但不能更改复合类型的元素的名称（和方法集）。 Go要求您明确说明类型转换。那golang中的复合类型指的是？复合类型包含指针、数组、切片、Map、结构体。所以数组类型进行强制转换是有问题的。但是可以遍历数组，对每个元素进行转换，来达到想要的想过即\[\]T1转换为\[\]T2

#### 问题2：可以将\[\]T强制转换为\[\]interface{}吗？

答案：不行。可以遍历数组，单个元素进行转换，但是直接对数组整体做转换会报错

语言规范不允许这样做，因为这两种类型在内存中的表示方式不同。有必要将元素分别复制到目标切片。此示例将int的一部分转换为interface {}的一部分：

```text
t := []int{1, 2, 3, 4}
s := make([]interface{}, len(t))
for i, v := range t {
    s[i] = v
}
```

csdn博客：[https://blog.csdn.net/u013276277](https://blog.csdn.net/u013276277)

参看文章：

[https://golang.org/doc/faq\#convert\_slice\_with\_same\_underlying\_type](https://golang.org/doc/faq#convert_slice_with_same_underlying_type)

