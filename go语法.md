---
title: 2020-1-14 go语法 
tags: go, language
---

**channel赋值为nil**   
1、channel可以赋值为nil，赋值为nil不能对通道写入数据，如果对nil的通道写数据会core，就像C++对一个指针为空的地址写数据会core类似。   
2、close一个channel和将channel赋值为nil不同，close后就永远不能再往通道写数据了，只能读。将一个channel赋值为nil，然后重新赋值为其他make(chan interface{})的值，channel可以继续正常读写。   
3、channel可以用临时变量赋值，有一个全局变量channel，用临时channel变量赋值给全局channel变量，对临时channel变量赋值相当于对全局channel变量赋值。   
![enter description here](./images/1579052148730.png)   

**channel作为函数参数**   
说明：channel作为函数参数，一般用于其他go协程操作了通道，由本go协程来接收或者写入这个通道触发某些操作。   
示例1、2，channel作为函数参数，带end端点（方向）的参数意义更明确，提供更清晰的意图，另外编译器可以帮忙检查是否有语法错误。
示例3，没有指明end端点，说明2个端口（方向）的channel都可以作为参数。   
示例4，奇怪的用法，一般不采用。除非你想改变channel的值。这个时候通过返回channel可能会更适合。   
``` golang    
1、func serve(ch <-chan interface{}){ //do stuff }   
2、func serve(ch chan<- interface{}){ //do stuff }   
3、func serve(ch chan interface{})  { //do stuff }   
4、func server(ch *chan interface{}){ //do stuff }   
```
https://stackoverflow.com/questions/24868859/different-ways-to-pass-channels-as-arguments-in-function-in-go-golang   

**channel作为返回值**   


**select的用法**   
1、select中有异常的case不一定会捕捉到，只要有一个case满足条件就可以继续运行（例如增加默认default的case）。   
2、如果没有case满足条件，那么某个case可能会永远阻塞下去，例如一个go协程的select中只有一个接收通道的case，但是该通道在其他协程没有发送赋值，导致go协程的select无法触发下一次接收操作，可能导致该go协程永远阻塞。   
![enter description here](./images/1579056537751.png)   

**CSP Design**   


**接口的作用**   
1、结构体赋值给接口   
2、   

**参考**  
  go相关博客列表： https://github.com/golang/go/wiki/Blogs     
《go编程语言》  ： https://book.douban.com/subject/26337545/   
官方effective go  ： https://golang.org/doc/effective_go.html   
go精华文章列表  ： https://github.com/golang/go/wiki/Articles    
go talk ：                 https://github.com/golang/go/wiki/GoTalks   
github上的go资源：https://github.com/avelino/awesome-go   


