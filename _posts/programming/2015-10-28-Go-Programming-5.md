---
layout: post
title:  "Learning Go - 5 Concurrancy"
date:   2015-10-28 15:40:00
tags: [go, programming, language]
categories: Programming
---

### 1. channel管道
默认情况下，在另一端准备好之前，发送和接收都会阻塞，使得go并发进行同步
{% highlight Go %}
func sum(a []int, c chan int) {
	sum := 0
	for _, v := range a {
		sum += v
	}
	c <- sum // 将和送入channel
}
func main() {
	a := []int{7, 2, 8, -9, 4, 0}
	c := make(chan int, 2)  // c是有类型的管道，使用前必须创建; 数字是缓冲区大小
	go sum(a[:len(a)/2], c)
	go sum(a[len(a)/2:], c)
	x, y := <-c, <-c // 从 c 中获取，箭头代表方向
	fmt.Println(x, y, x+y)  // -5 17 12，channel实现了 同步；输出顺序不重要!!!go线程调度的问题
}
{% endhighlight %}

### 2. channel的range和close
{% highlight Go %}
func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)  // 关闭channel，中断main中的range操作
}
func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {  // 不断从c拿值，直到被close。v, ok := <-ch，ch关闭后ok为false了
		fmt.Println(i)
	}
}
{% endhighlight %}

### 3. select 语句
select语句使得一个goroutine在多个通讯操作上等待。哪个ready执行哪个
{% highlight Go %}
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:  // 强制等待
			fmt.Println("quit")
			return
		}
	}
}
func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)  // 先打印完了10次c，才有quit
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}  // output: 0 1 1 2 3 5 8 13 21 34 quit
{% endhighlight %}
定时器例子：
{% highlight Go %}
func main() {
	tick := time.Tick(100 * time.Millisecond)  // 隔100ms执行一次
	boom := time.After(500 * time.Millisecond)
	for {
		select {
		case <-tick:
			fmt.Println("tick.")
		case <-boom:
			fmt.Println("BOOM!")
			return
		default:  // 当select中其他条件分支都没有准备好时，default分支执行
			fmt.Println("    .")
			time.Sleep(50 * time.Millisecond)
		}
	}
}
{% endhighlight %}

### 4. Exercise: Equivalent Binary Trees
{% highlight Go %}
type Tree struct {
    Left  *Tree
    Value int
    Right *Tree
}
// Walk 步进 tree t 将所有的值从 tree 发送到 channel ch。
func Walk(t *tree.Tree, ch chan int) {
	if(t == nil) {
		return
	}
	ch <- t.Value
	Walk(t.Left, ch)
	Walk(t.Right, ch)
}
// Same 检测树 t1 和 t2 是否含有相同的值。
func Same(t1, t2 *tree.Tree) bool {
	ch1 := make(chan int)
    ch2 := make(chan int)
    go Walk(t1, ch1)
    go Walk(t2, ch2)
    for i := 0; i < 10; i++ {
          v1 := <-ch1
          v2 := <-ch2
          if v1 != v2 {
               return false;
          }
     }
     return true
}
{% endhighlight %}

### 5. Exercise: Web Crawler
Crawl函数并行抓取URLs，并且保证不重复
{% highlight Go %}
type Fetcher interface {
	// Fetch返回URL的body，并将此页面上的URL放到slice中
	Fetch(url string) (body string, urls []string, err error)
}
// fetched tracks URLs that have been fetched.
// The lock must be held while reading from or writing to the map.
var fetched = struct {
        m map[string]bool
        sync.Mutex  // 匿名！相当于继承
}{m: make(map[string]bool)}
// Crawl使用fetcher从某URL递归爬取页面，直到最大深度。
func Crawl(url string, depth int, fetcher Fetcher) {
	if depth <= 0 {
	        fmt.Printf("Done with %v, depth 0.\n", url)
	        return
	}
	fetched.Lock()  //读写fetched，均要同步
	if _, ok := fetched.m[url]; ok {
	        fetched.Unlock()
	        fmt.Printf("Done with %v, already fetched.\n", url)
	        return
	}
	// then load it concurrently.
	body, urls, err := fetcher.Fetch(url)
	// And update the status in a synced zone.
	fetched.m[url] = true
	fetched.Unlock()

	if err != nil {
	        fmt.Printf("<- Error on %v: %v\n", url, err)
	        return
	}
	fmt.Printf("Found: %s %q\n", url, body)
	done := make(chan bool)
	for i, u := range urls {
	        fmt.Printf("-> Crawling child %v/%v of %v : %v.\n", i, len(urls), url, u)
	        go func(url string) {  // 需要套一层，直接调用go Crawl错误:done标记需要在内部!
	                Crawl(url, depth-1, fetcher)
	                done <- true
	        }(u)
	}
	for i, u := range urls {
	        fmt.Printf("<- [%v] %v/%v Waiting for child %v.\n", url, i, len(urls), u)
	        <-done
	}
	fmt.Printf("<- Done with %v\n", url)
}
func main() {
	Crawl("http://golang.org/", 4, fetcher)
}
// fakeFetcher 是返回若干结果的 Fetcher。
type fakeFetcher map[string]*fakeResult
type fakeResult struct {
	body string
	urls []string
}
func (f fakeFetcher) Fetch(url string) (string, []string, error) {
	if res, ok := f[url]; ok {
		return res.body, res.urls, nil
	}
	return "", nil, fmt.Errorf("not found: %s", url)
}
// fetcher是填充后的fakeFetcher。
var fetcher = fakeFetcher{
	"http://golang.org/": &fakeResult{
		"The Go Programming Language",
		[]string{
			"http://golang.org/pkg/",
			"http://golang.org/cmd/",
		},
	},
	"http://golang.org/pkg/": &fakeResult{
		"Packages",
		[]string{
			"http://golang.org/",
			"http://golang.org/cmd/",
			"http://golang.org/pkg/fmt/",
			"http://golang.org/pkg/os/",
		},
	},
	"http://golang.org/pkg/fmt/": &fakeResult{
		"Package fmt",
		[]string{
			"http://golang.org/",
			"http://golang.org/pkg/",
		},
	},
	"http://golang.org/pkg/os/": &fakeResult{
		"Package os",
		[]string{
			"http://golang.org/",
			"http://golang.org/pkg/",
		},
	},
}
Output:
Found: http://golang.org/ "The Go Programming Language"
-> Crawling child 0/2 of http://golang.org/ : http://golang.org/pkg/.
-> Crawling child 1/2 of http://golang.org/ : http://golang.org/cmd/.
<- [http://golang.org/] 0/2 Waiting for child http://golang.org/pkg/.
<- Error on http://golang.org/cmd/: not found: http://golang.org/cmd/
<- [http://golang.org/] 1/2 Waiting for child http://golang.org/cmd/.
Found: http://golang.org/pkg/ "Packages"
-> Crawling child 0/4 of http://golang.org/pkg/ : http://golang.org/.
-> Crawling child 1/4 of http://golang.org/pkg/ : http://golang.org/cmd/.
-> Crawling child 2/4 of http://golang.org/pkg/ : http://golang.org/pkg/fmt/.
-> Crawling child 3/4 of http://golang.org/pkg/ : http://golang.org/pkg/os/.
<- [http://golang.org/pkg/] 0/4 Waiting for child http://golang.org/.
Found: http://golang.org/pkg/os/ "Package os"
-> Crawling child 0/2 of http://golang.org/pkg/os/ : http://golang.org/.
-> Crawling child 1/2 of http://golang.org/pkg/os/ : http://golang.org/pkg/.
<- [http://golang.org/pkg/os/] 0/2 Waiting for child http://golang.org/.
Done with http://golang.org/pkg/, already fetched.
<- [http://golang.org/pkg/os/] 1/2 Waiting for child http://golang.org/pkg/.
Done with http://golang.org/, already fetched.
<- [http://golang.org/pkg/] 1/4 Waiting for child http://golang.org/cmd/.
Done with http://golang.org/cmd/, already fetched.
<- [http://golang.org/pkg/] 2/4 Waiting for child http://golang.org/pkg/fmt/.
Found: http://golang.org/pkg/fmt/ "Package fmt"
-> Crawling child 0/2 of http://golang.org/pkg/fmt/ : http://golang.org/.
-> Crawling child 1/2 of http://golang.org/pkg/fmt/ : http://golang.org/pkg/.
<- [http://golang.org/pkg/fmt/] 0/2 Waiting for child http://golang.org/.
Done with http://golang.org/pkg/, already fetched.
<- [http://golang.org/pkg/fmt/] 1/2 Waiting for child http://golang.org/pkg/.
Done with http://golang.org/, already fetched.
<- Done with http://golang.org/pkg/os/
<- [http://golang.org/pkg/] 3/4 Waiting for child http://golang.org/pkg/os/.
Done with http://golang.org/, already fetched.
<- Done with http://golang.org/pkg/fmt/
<- Done with http://golang.org/pkg/
<- Done with http://golang.org/
{% endhighlight %}