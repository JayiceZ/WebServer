# WebServer

A C++ High Performance NetServer (version 0.5.0)

[![license](https://img.shields.io/github/license/mashape/apistatus.svg)](https://opensource.org/licenses/MIT)

## Introduction  

本项目为C++11编写的基于epoll的多线程网络服务器框架，应用层实现了简单的HTTP服务器HttpServer和一个回显服务器EchoServer，其中HTTP服务器实现了HTTP的解析和Get方法请求，目前支持静态资源访问，支持HTTP长连接；该框架不限于这两类服务器，用户可根据需要编写应用层服务。


## Build

	$ make
	$ make clean

## Run
	$ ./httpserver [port] [iothreadnum] [workerthreadnum]
	
	例：$ ./httpserver 80 4 2
	表示开启80端口，采用4个IO线程、2个工作线程的方式 
	一般情况下，业务处理简单的话，工作线程数设为0即可
    
## Tech
 * 基于epoll的IO复用机制实现Reactor模式，采用边缘触发（ET）模式，和非阻塞模式
 * 由于采用ET模式，read、write和accept的时候必须采用循环的方式，直到error==EAGAIN为止，防止漏读等清况，这样的效率会比LT模式高很多，减少了触发次数
 * Version-0.1.0基于单线程实现，Version-0.2.0利用线程池实现多IO线程，Version-0.3.0实现通用worker线程池，基于one loop per thread的IO模式，Version-0.4.0增加定时器，Version-0.5.0增加简易协程实现和异步日志实现
 * 线程模型将划分为主线程、IO线程和worker线程，主线程接收客户端连接（accept），并通过Round-Robin策略分发给IO线程，IO线程负责连接管理（即事件监听和读写操作），worker线程负责业务计算任务（即对数据进行处理，应用层处理复杂的时候可以开启）
 * 基于时间轮实现定时器功能，定时剔除不活跃连接，时间轮的插入、删除复杂度为O(1)，执行复杂度取决于每个桶上的链表长度
 * 采用智能指针管理多线程下的对象资源
 * 增加简易协程实现，目前版本基于ucontext.h（供了解学习，尚未应用到本项目中）From: [simple-coroutine](https://github.com/chenshuaihao/simple-coroutine)
 * 支持HTTP长连接
 * 支持优雅关闭连接
   * 通常情况下，由客户端主动发起FIN关闭连接
   * 客户端发送FIN关闭连接后，服务器把数据发完才close，而不是直接暴力close
   * 如果连接出错，则服务器可以直接close
  

## License
See [LICENSE](https://github.com/chenshuaihao/NetServer/blob/master/LICENSE)

## Roadmap
内存池等

