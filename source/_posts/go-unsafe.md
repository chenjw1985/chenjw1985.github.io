---
title: go unsafe
date: 2020-01-07 13:58:14
tags:
---

最近花时间阅读了下 pilosa/roaring 的源码，里面为用到了很多go unsafe这个包。正好利用放假时间学习了相关知识，这里做一些记录。
unsafe.Pointer 在 go 中是用于各种指针相互转换的桥梁，类似 C 的 void *。uintptr 是 go 的内置类型，存储指针的整型，uintptr 的底层类型是 int，它和unsafe.Pointer 可相互转换。
uintptr 和 unsafe.Pointer 的区别是：unsafe.Pointer 只是单纯的通用指针类型，用于转换不同类型指针，但是它不可以参与指针运算（所谓的指针运算就是类似`++--`）；而 uintptr 是可以用于指针运算的，但是 uintptr 无法持有对象，所以 GC 不会把 uintptr 当指针，意味着 uintptr 类型的目标会被回收。go 的 unsafe 包很强大，基本上很少会去用它，它可以像 C 一样去操作内存。

三个指针类型：
- 一种是我们常见的 `*`，用 `*` 去表示的指针；
- 一种是 unsafe.Pointer，Pointer 是 unsafe 包下的一个类型；
- 最后一种是 uintptr，uintptr 就厉害了，这玩意是可以进行运算的也就是可以 `++--`；

他们之间有这样的转换关系：
`*` <=> `unsafe.Pointer` <=> `uintptr`
> 有一点要注意的是，uintptr 并没有指针的语义，意思就是 uintptr 所指向的对象会被 gc 无情地回收。而 unsafe.Pointer 有指针语义，可以保护它所指向的对象在`“有用”`的时候不会被垃圾回收。

unsafe 操作 slice：
```go
func main() {
    s := make([]int, 10)
    s[1] = 2
    p := &s[0]
    fmt.Println(*p)
    up := uintptr(unsafe.Pointer(p))
    up += unsafe.Sizeof(int(0)) // move to next slice point
    p2 := (*int)(unsafe.Pointer(up))
    fmt.Println(*p2)
}
```

unsafe 操作 struct：
> 创建结构体会被分配一块连续的内存，结构体的地址也代表了第一个成员的地址。
```go
package main

type User struct {
    age int
    name string
}

func main() {
    user := &basic.User{}
    fmt.Println(user)
    s := (*int)(unsafe.Pointer(user))
    *s = 10
    up := uintptr(unsafe.Pointer(user)) + unsafe.Sizeof(int(0))
    namep := (*string)(unsafe.Pointer(up))
    *namep = "xxx"
    fmt.Println(user)
}
```

字符串和 byte 数组转换 inplace
```go
// 这样需要开辟额外的空间，那么如何实现不需要拷贝数据的转换呢？
s := "123"
a := []byte(s)
```

```go
// 从底层的存储角度来说，string的存储规则和[]byte是一样的，
// 也就是说，其实指针都是从某个位置开始到一段空间，中间一格一格。
// 所以利用unsafe就可以做到。
func main() {
    s := "123"
    a := []byte(s)
    print("s = " , &s, "\n")
    print("a = " , &a, "\n")
    a2 := (*[]byte)(unsafe.Pointer(&s))
    print("a2 = " , a2, "\n")
    fmt.Println(*a2)
    // 但是这个转换是有问题的，a2 的 cap 没有得到正确的初始化。
    fmt.Println("cap a =", cap(a))
    fmt.Println("cap a2 =", cap(*a2))
    // 结果是：
    // cap a = 32
    // cap a2 = 17418400
}
```
问题的原因在 src/reflect/value.go 下看
```go
type StringHeader struct {
    Data uintptr
    Len int
}

type SliceHeader struct {
    Data uintptr
    Len int
    Cap int
}
```
实际原因是 string 没有 cap 而 []byte 有，所以导致问题出现，也容易理解，string 是没有容量扩容这个说法的，所以新的 []byte 没有赋值 cap 所以使用了默认值。
所以不可以直接转换，需要构建 sliceHeader。
```go
stringHeader := (*reflect.StringHeader)(unsafe.Pointer(&s))
bh := reflect.SliceHeader{
    Data: stringHeader.Data,
    Len: stringHeader.Len,
    Cap: stringHeader.Len,
}
return *(*[]byte)(unsafe.Pointer(&bh))
```