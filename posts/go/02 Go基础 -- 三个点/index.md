---

title: Go基础--三个点
date: 2020-07-22 10:14:52
slug: go-basic-three-dot
tags:
  - Go
categories:
  - Go

---

## 三个点

### 可变长的函数参数

```go
func Sum(nums ...int) int {
	cnt := 0
	for _, n := range nums {
		cnt += n
	}
	return cnt
}
```

###  调用可变长参数列表的函数

```go

primes := []int{2, 3, 5, 7}
Sum(primes...)
```

### 两个切片合并

```go
var str_1 = []string{
	"a",
	"b",
	"c",
}

var str_2 = []string{
	"1",
	"2",
	"3",
}

str_1 = append(str_1, str_2...)
fmt.PrintLn(str_1)
```

### 自动计算数组元素个数

```go
stooges := [...]string{"Moe", "Larry", "Curly"} // len(stooges) == 3
```

