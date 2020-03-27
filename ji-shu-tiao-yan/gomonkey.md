---
description: gomknkey是golang的测试框架，在函数打桩方面，我觉得比较好用。
---

# gomonkey

gomonkey调研文档和学习gomonkey概述examples1 为函数打桩2 函数打序列桩3、函数变量打桩5、全局变量打桩6、成员方法打桩7、成员方法打序列桩8、接口打桩总结

## gomonkey调研文档和学习

### gomonkey概述

[学习地址](https://github.com/agiledragon/gomonkey) gomonkey 是 golang 的一款打桩框架，目标是让用户在单元测试中低成本的完成打桩，从而将精力聚焦于业务功能的开发。 特性列表：

* 支持为一个函数打一个桩
* 支持为一个成员方法打一个桩
* 支持为一个接口打一个桩
* 支持为一个全局变量打一个桩
* 支持为一个函数变量打一个桩
* 支持为一个函数打一个特定的桩序列
* 支持为一个成员方法打一个特定的桩序列
* 支持为一个函数变量打一个特定的桩序列
* 支持为一个接口打一个特定的桩序列

### examples

例子中所有的代码都在这：[https://github.com/fishingfly/gomonkey\_examples](https://github.com/fishingfly/gomonkey_examples)

#### 1 为函数打桩

ApplyFunc 第一个参数是函数名，第二个参数是桩函数。测试完成后，patches 对象通过 Reset 成员方法删除所有测试桩。

```text
package test
​
import (
    "./fake"
    "encoding/json"
    . "github.com/agiledragon/gomonkey"
    . "github.com/smartystreets/goconvey/convey"
    "testing"
)
​
var (
    outputExpect = "xxx-vethName100-yyy"
)
​
func TestApplyFunc(t *testing.T) {
    Convey("TestApplyFunc", t, func() {
​
        Convey("one func for succ", func() {
            patches := ApplyFunc(fake.Exec, func(_ string, _ ...string) (string, error) {
                return outputExpect, nil
            })
            defer patches.Reset()
            output, err := fake.Exec("", "")
            So(err, ShouldEqual, nil)
            So(output, ShouldEqual, outputExpect)
        })
​
        Convey("one func for fail", func() {
            patches := ApplyFunc(fake.Exec, func(_ string, _ ...string) (string, error) {
                return "", fake.ErrActual
            })
            defer patches.Reset()
            output, err := fake.Exec("", "")
            So(err, ShouldEqual, fake.ErrActual)
            So(output, ShouldEqual, "")
        })
​
        Convey("two funcs", func() {
            patches := ApplyFunc(fake.Exec, func(_ string, _ ...string) (string, error) {
                return outputExpect, nil
            })
            defer patches.Reset()
            patches.ApplyFunc(fake.Belong, func(_ string, _ []string) bool {
                return true
            })
            output, err := fake.Exec("", "")
            So(err, ShouldEqual, nil)
            So(output, ShouldEqual, outputExpect)
            flag := fake.Belong("", nil)
            So(flag, ShouldBeTrue)
        })
​
        Convey("input and output param", func() {
            patches := ApplyFunc(json.Unmarshal, func(data []byte, v interface{}) error {
                if data == nil {
                    panic("input param is nil!")
                }
                p := v.(*map[int]int)
                *p = make(map[int]int)
                (*p)[1] = 2
                (*p)[2] = 4
                return nil
            })
            defer patches.Reset()
            var m map[int]int
            err := json.Unmarshal([]byte("123"), &m)
            So(err, ShouldEqual, nil)
            So(m[1], ShouldEqual, 2)
            So(m[2], ShouldEqual, 4)
        })
    })
}
```

这边是对fake.exec函数打桩，模拟exec函数的输出。,fake.exec函数在fake目录下，可以来下我的代码库看下。 上面这种模拟函数输出有啥应用，从我实际使用的角度出发，我们看下面例子： mytest.go

```text
package mytest
​
func AddOne(t int32) int32 {
    return t+1
}
​
func MinusOne(t int32) int32 {
    return t-1
}
​
func MultiAddOne(t int32) int32 {
    t = MinusOne(t)
    t = AddOne(t)
    t = AddOne(t)
    return t
}
```

测试类mytest\_test.go

```text
package mytest
​
import (
    . "github.com/agiledragon/gomonkey"
    . "github.com/smartystreets/goconvey/convey"
    "testing"
)
​
func TestMultiAddOne(t *testing.T) {
    Convey("TestApplyFunc", t, func() {
        Convey("input and output param", func() {
            patches := ApplyFunc(AddOne, func(t1 int32) int32 {
                return 0
            })//对函数AddOne打桩
            defer patches.Reset()
            patches.ApplyFunc(MinusOne, func(t1 int32) int32 {
                return -1
            })//对函数MinusOne打桩
            result := MultiAddOne(10) //看好了我调用的是MultiAddOne函数，而MultiAddOne函数内部调用了AddOne和MinusOne。
            So(result, ShouldEqual, 0)
        })
    })
}
​
```

我模拟了AddOne和MinusOne函数的输出，以达到MultiAddOne函数的效果。

#### 2 函数打序列桩

ApplyFuncSeq第一个参数是函数名，第二个参数是特定的桩序列参数。测试完成后，patches 对象通过 Reset 成员方法删除所有测试桩。

```text
package test
​
import (
    . "github.com/agiledragon/gomonkey"
    "github.com/agiledragon/gomonkey/test/fake"
    . "github.com/smartystreets/goconvey/convey"
    "testing"
)
​
func TestApplyFuncSeq(t *testing.T) {
    Convey("TestApplyFuncSeq", t, func() {
​
        Convey("default times is 1", func() {
            info1 := "hello cpp"
            info2 := "hello golang"
            info3 := "hello gomonkey"
            outputs := []OutputCell{
                {Values: Params{info1, nil}},// 模拟函数的第1次输出
                {Values: Params{info2, nil}},// 模拟函数的第2次输出
                {Values: Params{info3, nil}},// 模拟函数的第3次输出
            }
            patches := ApplyFuncSeq(fake.ReadLeaf, outputs)
            defer patches.Reset()
            output, err := fake.ReadLeaf("")
            So(err, ShouldEqual, nil)
            So(output, ShouldEqual, info1)
            output, err = fake.ReadLeaf("")
            So(err, ShouldEqual, nil)
            So(output, ShouldEqual, info2)
            output, err = fake.ReadLeaf("")
            So(err, ShouldEqual, nil)
            So(output, ShouldEqual, info3)
        })
​
        Convey("retry succ util the third times", func() {
            info1 := "hello cpp"
            outputs := []OutputCell{
                {Values: Params{"", fake.ErrActual}, Times: 2},// 模拟函数的第1次输出
                {Values: Params{info1, nil}},// 模拟函数的第2次输出
            }
            patches := ApplyFuncSeq(fake.ReadLeaf, outputs)
            defer patches.Reset()
            output, err := fake.ReadLeaf("")
            So(err, ShouldEqual, fake.ErrActual)
            output, err = fake.ReadLeaf("")
            So(err, ShouldEqual, fake.ErrActual)
            output, err = fake.ReadLeaf("")
            So(err, ShouldEqual, nil)
            So(output, ShouldEqual, info1)
        })
​
        Convey("batch operations failed on the third time", func() {
            info1 := "hello gomonkey"
            outputs := []OutputCell{
                {Values: Params{info1, nil}, Times: 2},
                {Values: Params{"", fake.ErrActual}},
            }
            patches := ApplyFuncSeq(fake.ReadLeaf, outputs)
            defer patches.Reset()
            output, err := fake.ReadLeaf("")
            So(err, ShouldEqual, nil)
            So(output, ShouldEqual, info1)
            output, err = fake.ReadLeaf("")
            So(err, ShouldEqual, nil)
            So(output, ShouldEqual, info1)
            output, err = fake.ReadLeaf("")
            So(err, ShouldEqual, fake.ErrActual)
        })
​
    })
}
```

#### 3、函数变量打桩

ApplyFuncVar 第一个参数是函数变量的地址，第二个参数是桩函数。测试完成后，patches 对象通过 Reset 成员方法删除所有测试桩。

```text
package test
​
import (
    . "github.com/agiledragon/gomonkey"
    "github.com/agiledragon/gomonkey/test/fake"
    . "github.com/smartystreets/goconvey/convey"
    "testing"
)
​
​
func TestApplyFuncVar(t *testing.T) {
    Convey("TestApplyFuncVar", t, func() {
​
        Convey("for succ", func() {
            str := "hello"
            patches := ApplyFuncVar(&fake.Marshal, func (_ interface{}) ([]byte, error) {
                return []byte(str), nil
            })// fake.Marshal是函数变量
            defer patches.Reset()
            bytes, err := fake.Marshal(nil)
            So(err, ShouldEqual, nil)
            So(string(bytes), ShouldEqual, str)
        })
​
        Convey("for fail", func() {
            patches := ApplyFuncVar(&fake.Marshal, func (_ interface{}) ([]byte, error) {
                return nil, fake.ErrActual
            })
            defer patches.Reset()
            _, err := fake.Marshal(nil)
            So(err, ShouldEqual, fake.ErrActual)
        })
    })
}
```

4、函数变量打序列桩 ApplyFuncVarSeq 第一个参数是函数变量地址，第二个参数是特定的桩序列参数。测试完成后，patches 对象通过 Reset 成员方法删除所有测试桩。

```text
package test
​
import (
    . "github.com/agiledragon/gomonkey"
    "github.com/agiledragon/gomonkey/test/fake"
    . "github.com/smartystreets/goconvey/convey"
    "testing"
)
​
func TestApplyFuncVarSeq(t *testing.T) {
    Convey("TestApplyFuncVarSeq", t, func() {
​
        Convey("default times is 1", func() {
            info1 := "hello cpp"
            info2 := "hello golang"
            info3 := "hello gomonkey"
            outputs := []OutputCell{
                {Values: Params{[]byte(info1), nil}},
                {Values: Params{[]byte(info2), nil}},
                {Values: Params{[]byte(info3), nil}},
            }
            patches := ApplyFuncVarSeq(&fake.Marshal, outputs)
            defer patches.Reset()
            bytes, err := fake.Marshal("")
            So(err, ShouldEqual, nil)
            So(string(bytes), ShouldEqual, info1)
            bytes, err = fake.Marshal("")
            So(err, ShouldEqual, nil)
            So(string(bytes), ShouldEqual, info2)
            bytes, err = fake.Marshal("")
            So(err, ShouldEqual, nil)
            So(string(bytes), ShouldEqual, info3)
        })
​
        Convey("retry succ util the third times", func() {
            info1 := "hello cpp"
            outputs := []OutputCell{
                {Values: Params{[]byte(""), fake.ErrActual}, Times: 2},
                {Values: Params{[]byte(info1), nil}},
            }
            patches := ApplyFuncVarSeq(&fake.Marshal, outputs)
            defer patches.Reset()
            bytes, err := fake.Marshal("")
            So(err, ShouldEqual, fake.ErrActual)
            bytes, err = fake.Marshal("")
            So(err, ShouldEqual, fake.ErrActual)
            bytes, err = fake.Marshal("")
            So(err, ShouldEqual, nil)
            So(string(bytes), ShouldEqual, info1)
        })
​
        Convey("batch operations failed on the third time", func() {
            info1 := "hello gomonkey"
            outputs := []OutputCell{
                {Values: Params{[]byte(info1), nil}, Times: 2},
                {Values: Params{[]byte(""), fake.ErrActual}},
            }
            patches := ApplyFuncVarSeq(&fake.Marshal, outputs)
            defer patches.Reset()
            bytes, err := fake.Marshal("")
            So(err, ShouldEqual, nil)
            So(string(bytes), ShouldEqual, info1)
            bytes, err = fake.Marshal("")
            So(err, ShouldEqual, nil)
            So(string(bytes), ShouldEqual, info1)
            bytes, err = fake.Marshal("")
            So(err, ShouldEqual, fake.ErrActual)
        })
​
    })
}
```

#### 5、全局变量打桩

ApplyGlobalVar 第一个参数是全局变量的地址，第二个参数是全局变量的桩。测试完成后，patches 对象通过 Reset 成员方法删除所有测试桩。

```text
package test
​
import (
    . "github.com/agiledragon/gomonkey"
    . "github.com/smartystreets/goconvey/convey"
    "testing"
)
​
var num = 10 //全局变量
​
func TestApplyGlobalVar(t *testing.T) {
    Convey("TestApplyGlobalVar", t, func() {
​
        Convey("change", func() {
            patches := ApplyGlobalVar(&num, 150)
            defer patches.Reset()
            So(num, ShouldEqual, 150)
        })
​
        Convey("recover", func() {
            So(num, ShouldEqual, 10)
        })
    })
}
```

#### 6、成员方法打桩

ApplyMethod 第一个参数是目标类的指针变量的反射类型，第二个参数是字符串形式的方法名，第三个参数是桩函数。测试完成后，patches 对象通过 Reset 成员方法删除所有测试桩。

```text
package test
​
import (
    . "github.com/agiledragon/gomonkey"
    . "github.com/smartystreets/goconvey/convey"
    "gomonkeytest/fake"
    "reflect"
    "testing"
)
​
​
func TestApplyMethod(t *testing.T) {
    slice := fake.NewSlice()
    var s *fake.Slice
    Convey("TestApplyMethod", t, func() {
​
        Convey("for succ", func() {
            err := slice.Add(1)
            So(err, ShouldEqual, nil)
            patches := ApplyMethod(reflect.TypeOf(s), "Add", func(_ *fake.Slice, _ int) error {//这边需要注意，这个成员方法开头是小写字符开头的话，是不能建立的，会报错
                return nil
            })
            defer patches.Reset()
            err = slice.Add(1)
            So(err, ShouldEqual, nil)
            err = slice.Remove(1)
            So(err, ShouldEqual, nil)
            So(len(slice), ShouldEqual, 0)
        })
​
        Convey("for already exist", func() {
            err := slice.Add(2)
            So(err, ShouldEqual, nil)
            patches := ApplyMethod(reflect.TypeOf(s), "Add", func(_ *fake.Slice, _ int) error {
                return fake.ErrElemExsit
            })
            defer patches.Reset()
            err = slice.Add(1)
            So(err, ShouldEqual, fake.ErrElemExsit)
            err = slice.Remove(2)
            So(err, ShouldEqual, nil)
            So(len(slice), ShouldEqual, 0)
        })
​
        Convey("two methods", func() {
            err := slice.Add(3)
            So(err, ShouldEqual, nil)
            defer slice.Remove(3)
            patches := ApplyMethod(reflect.TypeOf(s), "Add", func(_ *fake.Slice, _ int) error {
                return fake.ErrElemExsit
            })
            defer patches.Reset()
            patches.ApplyMethod(reflect.TypeOf(s), "Remove", func(_ *fake.Slice, _ int) error {
                return fake.ErrElemNotExsit
            })
            err = slice.Add(2)
            So(err, ShouldEqual, fake.ErrElemExsit)
            err = slice.Remove(1)
            So(err, ShouldEqual, fake.ErrElemNotExsit)
            So(len(slice), ShouldEqual, 1)
            So(slice[0], ShouldEqual, 3)
        })
​
        Convey("one func and one method", func() {
            err := slice.Add(4)
            So(err, ShouldEqual, nil)
            defer slice.Remove(4)
            patches := ApplyFunc(fake.Exec, func(_ string, _ ...string) (string, error) {
                return outputExpect, nil
            })
            defer patches.Reset()
            patches.ApplyMethod(reflect.TypeOf(s), "Remove", func(_ *fake.Slice, _ int) error {
                return fake.ErrElemNotExsit
            })
            output, err := fake.Exec("", "")
            So(err, ShouldEqual, nil)
            So(output, ShouldEqual, outputExpect)
            err = slice.Remove(1)
            So(err, ShouldEqual, fake.ErrElemNotExsit)
            So(len(slice), ShouldEqual, 1)
            So(slice[0], ShouldEqual, 4)
        })
    })
}
```

#### 7、成员方法打序列桩

ApplyMethodSeq 第一个参数是目标类的指针变量的反射类型，第二个参数是字符串形式的方法名，第三参数是特定的桩序列参数。测试完成后，patches 对象通过 Reset 成员方法删除所有测试桩。

```text
package test

import (
	. "github.com/agiledragon/gomonkey"
	"github.com/agiledragon/gomonkey/test/fake"
	. "github.com/smartystreets/goconvey/convey"
	"reflect"
	"testing"
)

func TestApplyMethodSeq(t *testing.T) {
	e := &fake.Etcd{}
	Convey("TestApplyMethodSeq", t, func() {

		Convey("default times is 1", func() {
			info1 := "hello cpp"
			info2 := "hello golang"
			info3 := "hello gomonkey"
			outputs := []OutputCell{
				{Values: Params{info1, nil}},
				{Values: Params{info2, nil}},
				{Values: Params{info3, nil}},
			}
			patches := ApplyMethodSeq(reflect.TypeOf(e), "Retrieve", outputs)
			defer patches.Reset()
			output, err := e.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
			output, err = e.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info2)
			output, err = e.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info3)
		})

		Convey("retry succ util the third times", func() {
			info1 := "hello cpp"
			outputs := []OutputCell{
				{Values: Params{"", fake.ErrActual}, Times: 2},
				{Values: Params{info1, nil}},
			}
			patches := ApplyMethodSeq(reflect.TypeOf(e), "Retrieve", outputs)
			defer patches.Reset()
			output, err := e.Retrieve("")
			So(err, ShouldEqual, fake.ErrActual)
			output, err = e.Retrieve("")
			So(err, ShouldEqual, fake.ErrActual)
			output, err = e.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
		})

		Convey("batch operations failed on the third time", func() {
			info1 := "hello gomonkey"
			outputs := []OutputCell{
				{Values: Params{info1, nil}, Times: 2},
				{Values: Params{"", fake.ErrActual}},
			}
			patches := ApplyMethodSeq(reflect.TypeOf(e), "Retrieve", outputs)
			defer patches.Reset()
			output, err := e.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
			output, err = e.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
			output, err = e.Retrieve("")
			So(err, ShouldEqual, fake.ErrActual)
		})

	})
}
```

#### 8、接口打桩

```text
package test

import (
	. "github.com/agiledragon/gomonkey"
	"github.com/agiledragon/gomonkey/test/fake"
	. "github.com/smartystreets/goconvey/convey"
	"reflect"
	"testing"
)

func TestApplyInterfaceReused(t *testing.T) {
	e := &fake.Etcd{}

	Convey("TestApplyInterfaceReused", t, func() {
		patches := ApplyFunc(fake.NewDb, func(_ string) fake.Db {
			return e
		})
		defer patches.Reset()
		db := fake.NewDb("mysql")

		Convey("TestApplyInterface", func() {
			info := "hello interface"
			patches.ApplyMethod(reflect.TypeOf(e), "Retrieve",
				func(_ *fake.Etcd, _ string) (string, error) {
					return info, nil
				})
			output, err := db.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info)
		})

		Convey("TestApplyInterfaceSeq", func() {
			info1 := "hello cpp"
			info2 := "hello golang"
			info3 := "hello gomonkey"
			outputs := []OutputCell{
				{Values: Params{info1, nil}},
				{Values: Params{info2, nil}},
				{Values: Params{info3, nil}},
			}
			patches.ApplyMethodSeq(reflect.TypeOf(e), "Retrieve", outputs)
			output, err := db.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
			output, err = db.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info2)
			output, err = db.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info3)
		})
	})
}
```

### 总结

在我看来，gomonkey是弥补了goconvey的不足，两者一起使用可以满足基本需要。goconvey负责断言，gomonkey负责为变量和函数打桩，构造各种测试条件。

