---
title: "并发访问 slice 如何做到优雅和安全?"
slug: ""
date: 2023-02-22T17:35:24+08:00
lastmod: 2023-02-22T17:35:24+08:00
author: ["路非非"]
tags: # 标签
- golang
series:
- 避坑指南
description: ""
weight:
draft: false # 是否为草稿
comments: true # 本页面是否显示评论
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "" #图片路径例如：posts/tech/123/123.png
  caption: "" #图片底部描述
  alt: ""
  relative: false
---

go中，切片可以算是我们最常用的结构之一，但是由于 slice/map 是引用类型，golang函数是传值调用，所用参数副本依然是原来的slice, 在并发访问同一个
资源的情况下会导致静态条件。

# 非线程安全现象
如以下代码
```go
func TestSlice(t *testing.T) {
	testMap := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	wg := sync.WaitGroup{}

	m := make([]int, 0)

	for _, v := range testMap {
		wg.Add(1)
		go func(v int) {
			defer wg.Done()
			m = append(m, v)
		}(v)
	}
	wg.Wait()
	t.Log(m)
}
```
真实的输出结果并不如我们所期望的那样，每次输出都会不一样。

问题出在哪？
```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```
![slice](slice.png)

使用append向Slice追加元素时，如果Slice空间不足，将会触发Slice扩容。扩容实际上是重新分配一块更大的内存，然后将原Slice数据拷贝进新Slice，然后返回新Slice，扩容后再将数据追加进去。

在并发情况下，如果该slice始终空间不足，那么其是线程安全的，因为每次append实际都是新生成的内存，不存在抢占的情况。但是，当slice空间充足，也即是cap>len, 有剩余的空间时，
比如说，下一个空闲内存是a, 那么并发情况下，就会出现多个线程抢占往a中写数据的情况。

>slice扩容遵从以下原则:
> 
>如果原Slice容量小于1024，则新Slice容量将扩大为原来的2倍；<br/>
如果原Slice容量大于等于1024，则新Slice容量将扩大为原来的1.25倍；

那么如何解决这个问题呢？

# 锁
针对内存占用，最简单粗暴的办法就是给内存加锁，如下:
```go
func TestSlice(t *testing.T) {
	testMap := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	wg := sync.WaitGroup{}

	m := make([]int, 0)
	var lock sync.Mutex

	for _, v := range testMap {
		wg.Add(1)
		go func(v int) {
			defer wg.Done()
			lock.Lock()
			m = append(m, v)
			lock.Unlock()
		}(v)
	}
	wg.Wait()
	t.Log(m)
}
```
>优点是比较简单，适合对性能要求不高的场景。

通过内存加锁，确保，同一时刻，只有一个线程对该块内存进行append操作，这就从根源上避免了抢占的问题。

# 使用 channel 串行化操作
```go

type SliceData struct {
	ch   chan int // 用来同步的channel
	data []int    //存储数据的slice
}

func (s *SliceData) Schedule() {
	// 从 channel 接收数据
	for i := range s.ch {
		s.data = append(s.data, i)
	}
}

func (s *SliceData) Close() {
	// 最后关闭 channel
	close(s.ch)
}

func (s *SliceData) AddData(v int) {
	s.ch <- v
}

func NewSliceData(size int, done func()) *SliceData {
	s := &SliceData{
		ch:   make(chan int, size),
		data: make([]int, 0),
	}
	go func() {
		s.Schedule()
		done()
	}()

	return s
}

func TestChannelSlice(t *testing.T) {
	var (
		wg sync.WaitGroup
		n  = 50
	)

	c := make(chan struct{})
	s := NewSliceData(n, func() {
		c <- struct{}{}
	})

	for i := 0; i < n; i++ {
		wg.Add(1)
		go func(v int) {
			defer wg.Done()

			s.AddData(v)
		}(i)
	}

	wg.Wait()
	s.Close()
	<-c

	t.Log(s.data)
}

```
>实现相对复杂，优点是性能很好，利用了channel的优势

