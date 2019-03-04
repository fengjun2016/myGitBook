# 特别要注意与其他语言的不同
* case 后面用加break
* 可以写default 也可以不写default的情况
* switch后面可以没有表达式
> 例如:
```golang
func grade(score int) string {
	switch {
	case score < 60:
		return "F"
	case score < 80:
		return "C"
	case score < 90:
		return "B"
	default:
		return "A"
	}
}