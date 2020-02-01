# append函数的使用方式

内置函数append使用方式

看一下内置函数append在buildin.go中的注释就知道了

```text
// The append built-in function appends elements to the end of a slice. If
// it has sufficient capacity, the destination is resliced to accommodate the
// new elements. If it does not, a new underlying array will be allocated.
// Append returns the updated slice. It is therefore necessary to store the
// result of append, often in the variable holding the slice itself:
//  slice = append(slice, elem1, elem2)
//  slice = append(slice, anotherSlice...)
// As a special case, it is legal to append a string to a byte slice, like this:
//  slice = append([]byte("hello "), "world"...)
func append(slice []Type, elems ...Type) []Type
```

注释翻译如下：

apeend内中函数是将元素添加到slice的末尾，如果容量足够，目标元素被容纳进切片。如果容量不足，一个新的底层数组将被分配，append函数返回结果是一个更新后的切片。因此，有必要将添加的结果存储在通常包含切片本身的变量中。

append的使用方式：

slice = append\(slice, elem\)

slice = append\(slice, elem1, elem2\)

slice = append\(slice, anotherSlice...\) // 最后这三个点点点要加上，不然会报错，这是用于两个切片的合并的。

当然还有例外：

将string拼接到byte切片是合法的，例如

slice = append\(\[\]byte\("hello "\), "world"...\)

