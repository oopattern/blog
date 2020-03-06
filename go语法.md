---
title: 2020-1-14 go语法 
tags: go, language
---

# **channel**  

+ **channel赋值为nil**   
1、channel可以赋值为nil，赋值为nil不能对通道写入数据，如果对nil的通道写数据会core，就像C++对一个指针为空的地址写数据会core类似。   
2、close一个channel和将channel赋值为nil不同，close后就永远不能再往通道写数据了，只能读。将一个channel赋值为nil，然后重新赋值为其他make(chan interface{})的值，channel可以继续正常读写。   
3、channel可以用临时变量赋值，有一个全局变量channel，用临时channel变量赋值给全局channel变量，对临时channel变量赋值相当于对全局channel变量赋值。   
![enter description here](./images/1579052148730.png)   

+ **channel作为函数参数**   
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

+ **channel作为返回值**   


# **select**   
1、select中有异常的case不一定会捕捉到，只要有一个case满足条件就可以继续运行（例如增加默认default的case）。   
2、如果没有case满足条件，那么某个case可能会永远阻塞下去，例如一个go协程的select中只有一个接收通道的case，但是该通道在其他协程没有发送赋值，导致go协程的select无法触发下一次接收操作，可能导致该go协程永远阻塞。   
![enter description here](./images/1579056537751.png)   

# **CSP Design**   
     

# **标准库**   
## bufio：    
缓冲I/O库，目的是减少对实现的io.Reader和io.Writer接口的调用次数，如果io接口对应操作的是磁盘，则可以减少操作磁盘的次数。   
+ bufio.Writer，用于减少io.Writer接口调用，数据处理模型： producer --> buffer --> io.Writer，用户producer消息到bufio的buffer，当bufio的buffer满才触发一次调用io.Writer，用户实现的io.Writer接口可以是写文件（磁盘），打印终端，TCP数据传输等具体操作，对于实现写磁盘接口，这样可以减少大量零碎的磁盘写操作。  
+ bufio.Reader，用于减少io.Reader接口调用，数据处理模型：io.Reader --> buffer --> consumer，用户调用io.Reader将数据存到bufio的buffer中，然后使用者从buffer将数据取出，只要bufio的buffer有数据没取完，就不会触发下一次调用io.Reader的操作。   
示例：如果从磁盘读取10个字节，每次读取1个字节，会触发10次磁盘读操作，将bufio的buffer设置为4，只会触发磁盘3次读操作。   
+ bufio.Scanner: 待分析研究。。。   
+ bufio.Reader的WriteTo：待分析研究。。。   
+ 区别：   ReadBytes('\n')、ReadString('\n') 、ReadLine、Scanner?   
+ bufio作用：一般配合archive/zip、compress/*、encoding/*、net/http等标准库使用，用于提高数据传输性能。   
+ 大牛分析：https://medium.com/golangspec/introduction-to-bufio-package-in-golang-ad7d1877f762   

## json：
struct和json的关系，包括允许struct字段为空或者跳过struct某个字段，就是动态的选择struct的fields
 + 大牛分析：https://medium.com/random-go-tips/dynamic-json-schemas-part-1-8f7d103ace71
				      https://eager.io/blog/go-and-json/

**参考**  
  go相关博客列表： https://github.com/golang/go/wiki/Blogs     
《go编程语言》  ： https://book.douban.com/subject/26337545/   
官方effective go  ： https://golang.org/doc/effective_go.html   
go精华文章列表  ： https://github.com/golang/go/wiki/Articles    
go talk ：                 https://github.com/golang/go/wiki/GoTalks   
github上的go资源：https://github.com/avelino/awesome-go   


