# 错误返回非空的值

Why is my nil error value not equal to nil

算是面试题吧，对于理解golang的Interface有帮助，interface怎样才算空？var err error = nil,这样的申明，err就是空的了吗？答案是：当然是空啦，上代码，看结果会比较清楚

```text
package main
import (
    "fmt"
    "github.com/go-openapi/errors"
)
​
func test1() error {
    var err error = nil
    if err != nil {
        return errors.New(200, "danger") //不会进
    }
    return err
}
func main() {
    t := test1()
    if t != nil {
        fmt.Println("true")
    } else {
        fmt.Println("false")
    }
}
```

结果为false。也就是返回是空的。

那我们看下面代码：

```text
package main
​
import (
    "fmt"
    "github.com/go-openapi/errors"
)
​
type myError struct {
}
​
func (m myError)Error() string {//myError实现了error接口的Error方法
    return "test"
}
​
func test1() error {
    var err *myError = nil
    if err != nil {
        return errors.New(200, "danger") //不进
    }
    return err
}
​
func main() {
    t := test1()
    if t != nil {
        fmt.Println("true")
    } else {
        fmt.Println("false")
    }
}
```

结果是：true.第二个输出结果显示返回不为空。就是err的类型不一样，导致最后判空结果不一样，但是第二个例子中的err明明是Nil,不然会进test1\(\)中的那个判空啊。原因是啥呢？

因为接口存储分为类型T和值V，V是具体值，例如我们将类型为int的3存储进接口，接口表现为T=INT,V= 3。接口只有当V为nil且T为nil时才会判为空。所以例2中test1\(\)返回中，从\*myError转换为error。所以error是要存储原数据类型的，返回结果为T=\*\*myError,V= nil。最终显示该值不为空。

参看文章：https://golang.org/doc/faq\#nil\_error  


