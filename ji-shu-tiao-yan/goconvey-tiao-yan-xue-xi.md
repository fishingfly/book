---
description: goconvey是golang的测试框架
---

# goconvey调研学习

goconvey调研及学习goconvey概述安装快速开始：浏览器中查看测试结果goconvey的使用场景1、example\_test2、bowling\_game3、 assertion\_example\_test.go总结

## goconvey调研及学习

### goconvey概述

[学习地址](https://github.com/smartystreets/goconvey) [官方例子](https://github.com/smartystreets/goconvey/wiki/Composition) GoConvey是一款针对Golang的测试框架，可以管理和运行测试用例，同时提供了丰富的断言函数，并支持很多 Web 界面特性。

* 集成go test
* 可读的，带色彩的控制台输出
* 全自动Web UI
* 大量的回归测试套件
* 测试代码生成

#### 安装

```text
go get github.com/smartystreets/goconvey
```

#### 快速开始：

```text
package package_name
​
import (
    "testing"
    . "github.com/smartystreets/goconvey/convey"
)
​
func TestSpec(t *testing.T) {
​
    // Only pass t into top-level Convey calls
    Convey("Given some integer with a starting value", t, func() {
        x := 1
​
        Convey("When the integer is incremented", func() {
            x++
​
            Convey("The value should be greater by one", func() {
                So(x, ShouldEqual, 2)
            })
        })
    })
}
```

#### 浏览器中查看测试结果

运行：

```text
$GOPATH/bin/goconvey
```

要是再windows中就去该路径下双击exe文件 浏览器访问:[http://localhost:8080/](http://localhost:8080/) 就可以看到convey界面啦

当你的测试文件在哪个地址是就修改圈出来的文件目录，然后点击右侧的刷新，他自动会跑test，然后输出结果。

### goconvey的使用场景

每个使用场景加上代码详细讲解

#### 1、example\_test

```text
package examples
​
import (
    "testing"
​
    . "github.com/smartystreets/goconvey/convey"
)
​
func TestIntegerManipulation(t *testing.T) {
    t.Parallel()
​
    Convey("Given a starting integer value", t, func() {
        x := 42
​
        Convey("When incremented", func() {
            x++
​
            Convey("The value should be greater by one", func() {
                So(x, ShouldEqual, 43)
            })
            Convey("The value should NOT be what it used to be", func() {
                So(x, ShouldNotEqual, 42)
            })
        })
        Convey("When decremented", func() {
            x--
​
            Convey("The value should be lesser by one", func() {
                So(x, ShouldEqual, 41)
            })
            Convey("The value should NOT be what it used to be", func() {
                So(x, ShouldNotEqual, 42)
            })
        })
    })
}
```

这边就一些简单的断言测试，对一个变量加减，用ShouldNotEqual和ShouldEqual来判断，当然还有更多的判断条件：

```text
ShouldEqual          = assertions.ShouldEqual
    ShouldNotEqual       = assertions.ShouldNotEqual
    ShouldAlmostEqual    = assertions.ShouldAlmostEqual
    ShouldNotAlmostEqual = assertions.ShouldNotAlmostEqual
    ShouldResemble       = assertions.ShouldResemble
    ShouldNotResemble    = assertions.ShouldNotResemble
    ShouldPointTo        = assertions.ShouldPointTo
    ShouldNotPointTo     = assertions.ShouldNotPointTo
    ShouldBeNil          = assertions.ShouldBeNil
    ShouldNotBeNil       = assertions.ShouldNotBeNil
    ShouldBeTrue         = assertions.ShouldBeTrue
    ShouldBeFalse        = assertions.ShouldBeFalse
    ShouldBeZeroValue    = assertions.ShouldBeZeroValue
    ShouldNotBeZeroValue = assertions.ShouldNotBeZeroValue
​
    ShouldBeGreaterThan          = assertions.ShouldBeGreaterThan
    ShouldBeGreaterThanOrEqualTo = assertions.ShouldBeGreaterThanOrEqualTo
    ShouldBeLessThan             = assertions.ShouldBeLessThan
    ShouldBeLessThanOrEqualTo    = assertions.ShouldBeLessThanOrEqualTo
    ShouldBeBetween              = assertions.ShouldBeBetween
    ShouldNotBeBetween           = assertions.ShouldNotBeBetween
    ShouldBeBetweenOrEqual       = assertions.ShouldBeBetweenOrEqual
    ShouldNotBeBetweenOrEqual    = assertions.ShouldNotBeBetweenOrEqual
    ....还有好多，自己去发现吧
```

#### 2、bowling\_game

bowling\_game.go

```text
package examples
​
// Game contains the state of a bowling game.
type Game struct {
    rolls   []int
    current int
}
​
// NewGame allocates and starts a new game of bowling.
func NewGame() *Game {
    game := new(Game)
    game.rolls = make([]int, maxThrowsPerGame)
    return game
}
​
// Roll rolls the ball and knocks down the number of pins specified by pins.
func (self *Game) Roll(pins int) {
    self.rolls[self.current] = pins
    self.current++
}
​
// Score calculates and returns the player's current score.
func (self *Game) Score() (sum int) {
    for throw, frame := 0, 0; frame < framesPerGame; frame++ {
        if self.isStrike(throw) {
            sum += self.strikeBonusFor(throw)
            throw += 1
        } else if self.isSpare(throw) {
            sum += self.spareBonusFor(throw)
            throw += 2
        } else {
            sum += self.framePointsAt(throw)
            throw += 2
        }
    }
    return sum
}
​
// isStrike determines if a given throw is a strike or not. A strike is knocking
// down all pins in one throw.
func (self *Game) isStrike(throw int) bool {
    return self.rolls[throw] == allPins
}
​
// strikeBonusFor calculates and returns the strike bonus for a throw.
func (self *Game) strikeBonusFor(throw int) int {
    return allPins + self.framePointsAt(throw+1)
}
​
// isSpare determines if a given frame is a spare or not. A spare is knocking
// down all pins in one frame with two throws.
func (self *Game) isSpare(throw int) bool {
    return self.framePointsAt(throw) == allPins
}
​
// spareBonusFor calculates and returns the spare bonus for a throw.
func (self *Game) spareBonusFor(throw int) int {
    return allPins + self.rolls[throw+2]
}
​
// framePointsAt computes and returns the score in a frame specified by throw.
func (self *Game) framePointsAt(throw int) int {
    return self.rolls[throw] + self.rolls[throw+1]
}
​
const (
    // allPins is the number of pins allocated per fresh throw.
    allPins          = 10
    
    // framesPerGame is the number of frames per bowling game.
    framesPerGame    = 10
    
    // maxThrowsPerGame is the maximum number of throws possible in a single game.
    maxThrowsPerGame = 21
)
```

bowling\_game\_test.go

```text
package examples
​
import (
    "testing"
​
    . "github.com/smartystreets/goconvey/convey"
)
​
func TestBowlingGameScoring(t *testing.T) {
    Convey("Given a fresh score card", t, func() {
        game := NewGame()
​
        Convey("When all gutter balls are thrown", func() {
            game.rollMany(20, 0)
​
            Convey("The score should be zero", func() {
                So(game.Score(), ShouldEqual, 0)
            })
        })
​
        Convey("When all throws knock down only one pin", func() {
            game.rollMany(20, 1)
​
            Convey("The score should be 20", func() {
                So(game.Score(), ShouldEqual, 20)
            })
        })
​
        Convey("When a spare is thrown", func() {
            game.rollSpare()
            game.Roll(3)
            game.rollMany(17, 0)
​
            Convey("The score should include a spare bonus.", func() {
                So(game.Score(), ShouldEqual, 16)
            })
        })
​
        Convey("When a strike is thrown", func() {
            game.rollStrike()
            game.Roll(3)
            game.Roll(4)
            game.rollMany(16, 0)
​
            Convey("The score should include a strike bonus.", func() {
                So(game.Score(), ShouldEqual, 24)
            })
        })
​
        Convey("When all strikes are thrown", func() {
            game.rollMany(21, 10)
​
            Convey("The score should be 300.", func() {
                So(game.Score(), ShouldEqual, 300)
            })
        })
    })
}
​
func (self *Game) rollMany(times, pins int) {
    for x := 0; x < times; x++ {
        self.Roll(pins)
    }
}
func (self *Game) rollSpare() {
    self.Roll(5)
    self.Roll(5)
}
func (self *Game) rollStrike() {
    self.Roll(10)
}
```

这边的测试涉及到了具体的函数。但是没有对其中的函数进行打桩。我不知道

#### 3、 assertion\_example\_test.go

```text
package examples
​
import (
    "bytes"
    "io"
    "testing"
    "time"
​
    . "github.com/smartystreets/goconvey/convey"
)
​
func TestAssertionsAreAvailableFromConveyPackage(t *testing.T) {
    SetDefaultFailureMode(FailureContinues)
    defer SetDefaultFailureMode(FailureHalts)
​
    Convey("Equality assertions should be accessible", t, func() {
        thing1a := thing{a: "asdf"}
        thing1b := thing{a: "asdf"}
        thing2 := thing{a: "qwer"}
​
        So(1, ShouldEqual, 1)
        So(1, ShouldNotEqual, 2)
        So(1, ShouldAlmostEqual, 1.000000000000001)
        So(1, ShouldNotAlmostEqual, 2, 0.5)
        So(thing1a, ShouldResemble, thing1b)
        So(thing1a, ShouldNotResemble, thing2)
        So(&thing1a, ShouldPointTo, &thing1a)
        So(&thing1a, ShouldNotPointTo, &thing1b)
        So(nil, ShouldBeNil)
        So(1, ShouldNotBeNil)
        So(true, ShouldBeTrue)
        So(false, ShouldBeFalse)
        So(0, ShouldBeZeroValue)
        So(1, ShouldNotBeZeroValue)
    })
​
    Convey("Numeric comparison assertions should be accessible", t, func() {
        So(1, ShouldBeGreaterThan, 0)
        So(1, ShouldBeGreaterThanOrEqualTo, 1)
        So(1, ShouldBeLessThan, 2)
        So(1, ShouldBeLessThanOrEqualTo, 1)
        So(1, ShouldBeBetween, 0, 2)
        So(1, ShouldNotBeBetween, 2, 4)
        So(1, ShouldBeBetweenOrEqual, 1, 2)
        So(1, ShouldNotBeBetweenOrEqual, 2, 4)
    })
​
    Convey("Container assertions should be accessible", t, func() {
        So([]int{1, 2, 3}, ShouldContain, 2)
        So([]int{1, 2, 3}, ShouldNotContain, 4)
        So(map[int]int{1: 1, 2: 2, 3: 3}, ShouldContainKey, 2)
        So(map[int]int{1: 1, 2: 2, 3: 3}, ShouldNotContainKey, 4)
        So(1, ShouldBeIn, []int{1, 2, 3})
        So(4, ShouldNotBeIn, []int{1, 2, 3})
        So([]int{}, ShouldBeEmpty)
        So([]int{1}, ShouldNotBeEmpty)
        So([]int{1, 2}, ShouldHaveLength, 2)
    })
​
    Convey("String assertions should be accessible", t, func() {
        So("asdf", ShouldStartWith, "a")
        So("asdf", ShouldNotStartWith, "z")
        So("asdf", ShouldEndWith, "df")
        So("asdf", ShouldNotEndWith, "as")
        So("", ShouldBeBlank)
        So("asdf", ShouldNotBeBlank)
        So("asdf", ShouldContainSubstring, "sd")
        So("asdf", ShouldNotContainSubstring, "af")
    })
​
    Convey("Panic recovery assertions should be accessible", t, func() {
        So(panics, ShouldPanic)
        So(func() {}, ShouldNotPanic)
        So(panics, ShouldPanicWith, "Goofy Gophers!")
        So(panics, ShouldNotPanicWith, "Guileless Gophers!")
    })
​
    Convey("Type-checking assertions should be accessible", t, func() {
​
        // NOTE: Values or pointers may be checked.  If a value is passed,
        // it will be cast as a pointer to the value to avoid cases where
        // the struct being tested takes pointer receivers. Go allows values
        // or pointers to be passed as receivers on methods with a value
        // receiver, but only pointers on methods with pointer receivers.
        // See:
        // http://golang.org/doc/effective_go.html#pointers_vs_values
        // http://golang.org/doc/effective_go.html#blank_implements
        // http://blog.golang.org/laws-of-reflection
​
        So(1, ShouldHaveSameTypeAs, 0)
        So(1, ShouldNotHaveSameTypeAs, "1")
​
        So(bytes.NewBufferString(""), ShouldImplement, (*io.Reader)(nil))
        So("string", ShouldNotImplement, (*io.Reader)(nil))
    })
​
    Convey("Time assertions should be accessible", t, func() {
        january1, _ := time.Parse(timeLayout, "2013-01-01 00:00")
        january2, _ := time.Parse(timeLayout, "2013-01-02 00:00")
        january3, _ := time.Parse(timeLayout, "2013-01-03 00:00")
        january4, _ := time.Parse(timeLayout, "2013-01-04 00:00")
        january5, _ := time.Parse(timeLayout, "2013-01-05 00:00")
        oneDay, _ := time.ParseDuration("24h0m0s")
​
        So(january1, ShouldHappenBefore, january4)
        So(january1, ShouldHappenOnOrBefore, january1)
        So(january2, ShouldHappenAfter, january1)
        So(january2, ShouldHappenOnOrAfter, january2)
        So(january3, ShouldHappenBetween, january2, january5)
        So(january3, ShouldHappenOnOrBetween, january3, january5)
        So(january1, ShouldNotHappenOnOrBetween, january2, january5)
        So(january2, ShouldHappenWithin, oneDay, january3)
        So(january5, ShouldNotHappenWithin, oneDay, january1)
        So([]time.Time{january1, january2}, ShouldBeChronological)
    })
}
​
type thing struct {
    a string
}
​
func panics() {
    panic("Goofy Gophers!")
}
​
const timeLayout = "2006-01-02 15:04"
```

这边包含了很多断言的测试。

### 总结

在我看来，goconvey只是用来做一些简单的断言测试的，不能用来对函数打桩。简单测试可以，复杂点的测试需要和其他测试框架一起使用才能体现出测试效果。convey测试的小技巧，摘自下面的参考文章：

* import goconvey包时，前面加点号"."，以减少冗余的代码
* 测试函数的名字必须以Test开头，而且参数类型必须为“\*testing.T”
* 每个测试用例必须使用Convey函数包裹起来，推荐使用Convey语句的嵌套，即一个函数有一个测试函数，测试函数中嵌套两级Convey语句，第一级Convey语句对应测试函数，第二级Convey语句对应测试用例
* Convey语句的第三个参数习惯以闭包的形式实现，在闭包中通过So语句完成断言
* 使用GoConvey框架的 Web 界面特性，作为命令行的补充
* 在适当的场景下使用SkipConvey函数或SkipSo函数
* 当测试中有需要时，可以定制断言函数

参考文章: \[1\][https://www.jianshu.com/p/e3b2b1194830](https://www.jianshu.com/p/e3b2b1194830) \[2\][https://www.yuque.com/fz420/golang/tcxft4](https://www.yuque.com/fz420/golang/tcxft4)

