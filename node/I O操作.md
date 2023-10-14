## 阻塞I/O
传统的阻塞式I/O编程中，与I/O请求相对应的都函数调用操作，会阻塞执行该操作的这条线程，知道操作完成为止。
每条线程为了等待系统把涉及相关连接的数据，必须闲置大量时间如果我们有大量的I/O操作；那么就会导致线程为了等待这些I/O操作的完成将会多次闲置。但是线程所占的系统资源并不少，它要使用内存，并且系统必须通过切换上下文来管理这些线程；当没有对I/O操作的处理所需的内容未完成会导致cpu资源的浪费；
## 非阻塞I/O
系统总会立即返回，不必等待相关的数据的写入和读取。在执行调用的那一刻，函数若是无法给出结果，则返回一个预先定义好的常量，表示当前没有数据能够返回。
可以在一条线程中处理许多份不同的资源，但效率不高；不断的循环会导致CPU周期的浪费；因为在大部分的情况下，迭代的资源在需要时还没准备好。这样的轮询算法会导致耗费大量的CPU时间
``` javascript
resources=[socketA,socketB,fileA];
while(!resource.isEmpty()){
	for(resource of resources){
		data = resource.read();
		// 没有可读资源
		if(data===NO_DATA_AVAILABLE){
			continue
		}
		// 资源关闭
		if(data===RESOURCE_CLOSE){
			resources.remove()
		// 存在数据，处理
		}else{
			customData(data)
		}
	}
}

```

## 事件多路分离
监测多个资源，并于涉及其中某个资源的读取操作或者写入操作执行完成后，将会返回一个新的事件。
这种同步事件多路分离器总是同步的，它会一直卡在这里，直到有新的事件需要处理为止。
``` javascript
watchedList.add(socketA,read);
watchedList.add(fileB,read);
while(events=demultiplexer.watch(watchedList)){
	for(event of events){
		data=event.read();
		if(data===RESOURCE_CLOSE){
			xxxx.unwatch(event)
		}else{
			customData(data)
		}
	}
}
```
会将资源添加到watched中，并指出需要做的操作；使用watched配置事件多路分离；watched是一个同步的方法，会一直阻塞，知道检测到某个方法可用。这个时候对多路分离做的watch调用就会返回。得到可用的心的事件，表示存在需要处理的数据。通过for循环得对应的事件，这时事件对应的数据都是可以取到的。不会发生阻塞。当时间对数据都处理完成之后，会回到while，等待下一次的待处理的事件
![[Pasted image 20230924150756.png]]

## reactor模式
将每一项I/O操作都同某个header关联。只要循环处理某个事件，header都会进行触发。
![[Pasted image 20230924150822.png]]
用户向事件多路分离器提交请求，表示存在需要完成的I/O操作，提交的时候需要制定header（操作完成后需要执行的逻辑）。向多分离器提交新的请求是非阻塞的调用，这种调用会立即把控制权还给应用程序。事件分离器执行完i/o操作后会把对应的事件推给事件队列中，事件队列会迭代事件队列中的各项事件；针对每一项事件，触发与之对应的header；当header执行完成之后会把控制权交还给事件循环；在执行中还能提出新的异步请求；事件队列中的每个条目都处理完之后，事件循环就会阻塞于事件分离器，等待下一次的事件出现；
## Libuv

libuv 是最初为 Node.js 编写的 C 库，用于抽象非阻塞 I/O
![[Pasted image 20230924152609.png]]
- 集成了事件驱动的异步I/O模型。
- 它允许同时使用CPU和其他资源，同时仍然执行I/O操作，从而实现资源和网络的高效利用。
- 它促进了事件驱动的方法，其中使用基于回调的通知来执行 I/O 和其他活动。

# 欠libuv原理和分析，子事件循环做