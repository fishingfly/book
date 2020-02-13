# flota64后面保留三位小数

1、需要转成字符串形式的：

fmt.Sprintf\("%.3f", 1.23456\)

strconv.ParseFloat\(fmt.Sprintf\("%.3f",v\), 3\)

2、不需要转成字符串形式的：

value = math.Trunc\(value\*_1e3 + 0.5\)\*_1e-3

加上 0.5是为了四舍五入，想保留几位小数的话把3改掉即可

封装成函数

```text
func Round(f float64, n int) float64 {
    pow10_n := math.Pow10(n)
    return math.Trunc((f+0.5/pow10_n)*pow10_n) / pow10_n
}
```

