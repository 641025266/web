1）TCP断头最小长度是20字节，ip的首部都是20个字节   
2）简述TCP三次握手的过程  
在TCP/IP协议中，TCP协议提供可靠的连接服务，采用三次握手建立一个连接  
第一次握手：建立连接时，客户端发送syn包(syn=j)到服务器，并进入SYN_SEND状态，等待服务器确认  
第二次握手：服务器收到syn包，必须确认客户的SYN（ack=j+1），同时自己也发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入SYN_RECV状态  
第三次握手：客户端收到服务器的SYN＋ACK包，向服务器发送确认包ACK(ack=k+1)，此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手 ，完成三次握手，客户端与服务器开始传送数据  
整个流程就是：首先A向B发SYN（同步请求），然后B回复SYN+ACK（同步请求应答），最后A回复ACK确认，这样TCP的一次连接（三次握手）的过程就建立了  
3）tcp四次挥手过程  
由于TCP连接是全双工的，因此每个方向都必须单独进行关闭。这个原则是当一方完成它的数据发送任务后就能发送一个FIN来终止这个方向的连接。收到一个FIN只意味着这一方向上没有数据流动，一个TCP连接在收到一个FIN后仍能发送数据。首先进行关闭的一方将执行主动关闭，而另一方执行被动关闭  
(1)客户端A发送一个FIN，用来关闭客户A到服务器B的数据传送    
(2)服务器B收到这个FIN，它发回一个ACK，确认序号为收到的序号加1，和SYN一样，一个FIN将占用一个序号  
(3)服务器B关闭与客户端A的连接，发送一个FIN给客户端A  
(4)客户端A发回ACK报文确认，并将确认序号设置为收到序号加1  
TCP三次握手，四次分手原因  
这是因为服务端的LISTEN状态下的SOCKET当收到SYN报文的建连请求后，它可以把ACK和SYN（ACK起应答作用，而SYN起同步作用）放在一个报文里来发送。但关闭连接时，当收到对方的FIN报文通知时，它仅仅表示对方没有数据发送给你了；但未必你所有的数据都全部发送给对方了，所以你可以未必会马上会关闭SOCKET,也即你可能还需要发送一些数据给对方之后，再发送FIN报文给对方来表示你同意现在可以关闭连接了，所以它这里的ACK报文和FIN报文多数情况下都是分开发送的  
tcp三次握手详细介绍  
CLOSED: 这个没什么好说的了，表示初始状态  
LISTEN: 这个也是非常容易理解的一个状态，表示服务器端的某个SOCKET处于监听状态，可以接受连接了  
SYN_RCVD: 这个状态表示接受到了SYN报文，在正常情况下，这个状态是服务器端的SOCKET在建立TCP连接时的三次握手会话过程中的一个中间状态，很短暂，基本上用netstat你是很难看到这种状态的，除非你特意写了一个客户端测试程序，故意将三次TCP握手过程中最后一个ACK报文不予发送。因此这种状态时，当收到客户端的ACK报文后，它会进入到ESTABLISHED  
SYN_SENT: 这个状态与SYN_RCVD遥想呼应，当客户端SOCKET执行CONNECT连接时，它首先发送SYN报文，因此也随即它会进入到了SYN_SENT状 态，并等待服务端的发送三次握手中的第2个报文  
SYN_SENT状态表示客户端已发送SYN报文  
ESTABLISHED|ESTAB：这个容易理解了，表示连接已经建立了  
tcp四次分手详细介绍  
FIN_WAIT_1:   
这个状态要好好解释一下，其实FIN_WAIT_1和FIN_WAIT_2状态的真正含义都是表示等待对方的FIN报文。而这两种状态的区别是:FIN_WAIT_1状态实际上是当SOCKET在ESTABLISHED状态时，它想主动关闭连接，向对方发送了FIN报文，此时该SOCKET即 进入到FIN_WAIT_1状态。而当对方回应ACK报文后，则进入到FIN_WAIT_2状态，当然在实际的正常情况下，无论对方何种情况下，都应该马上回应ACK报文，所以FIN_WAIT_1状态一般是比较难见到的，而FIN_WAIT_2状态还有时常常可以用netstat看到  
FIN_WAIT_2：  
实际上FIN_WAIT_2状态下的SOCKET表示半连接，也即有一方要求close连接，但另外还告诉对方，我暂时还有点数据需要传送给你，稍后再关闭连接  
TIME_WAIT:   
表示收到了对方的FIN报文，并发送出了ACK报文。 就等2MSL后即可回到CLOSED可用状态了。MSL是指Max Segment Lifetime，即数据包在网络中的最大生存时间  
CLOSING:   
这种状态比较特殊，实际情况中应该是很少见，属于一种比较罕见的例外状态。正常情况下，当你发送FIN报文后，按理来说是应该先收到（或同时收到）对方的 ACK报文，再收到对方的FIN报文。但是CLOSING状态表示你发送FIN报文后，并没有收到对方的ACK报文，反而却也收到了对方的FIN报文。什么情况下会出现此种情况呢？其实细想一下，也不难得出结论：那就是如果双方几乎在同时close一个SOCKET的话，那么就出现了双方同时发送FIN报文的情况，也即会出现CLOSING状态，表示双方都正在关闭SOCKET连接  
CLOSE_WAIT:   
这种状态的含义其实是表示在等待关闭。当对方close一个SOCKET后发送FIN报文给自己，你系统毫无疑问地会回应一个ACK报文给对方，此时则进入到CLOSE_WAIT状态。接下来你真正需要考虑的事情是察看你是否还有数据发送给对方，如果没有的话，那么你也就可以 close这个SOCKET，发送FIN报文给对方，也即关闭连接。所以你在CLOSE_WAIT状态下，需要完成的事情是等待你去关闭连接  
LAST_ACK:   
这个状态还是比较容易好理解的，它是被动关闭一方在发送FIN报文后，最后等待对方的ACK报文。当收到ACK报文后，也即可以进入到CLOSED可用状态了  
总结：  
主动关闭方出现的状态有FIN_WAIT_1---FIN_WAIT_2(可能会出现)---TIME_WAIT(必须出现)----CLOSING--CLOSED
被动关闭方出现的状态有CLOSE_WAIT----LAST_ACK---CLOSED  
time_wait存在的原因有两点   
1.可靠的终止TCP连接，若处于time_wait的client发送给server确认报文段丢失的话，server将又一次发送FIN报文段，那么client必须处于一个可接收的状态就是time_wait而不是close状态   
2.保证迟来的TCP报文段有足够的时间被识别并丢弃，linux 中一个TCPport不能打开两次或两次以上。当client处于time_wait状态时我们将无法使用此port建立新连接，假设不存在time_wait状态，新连接可能会收到旧连接的数据。time_wait持续的时间是2MSL，保证旧的数据能够丢弃。由于网络中的数据最大存在时间是MSL(maxinum segment lifetime)  

 ![](pictures/tcp.png)


案例：
Web服务器Apache目前一共有三种稳定的MPM（Multi-Processing Module，多进程处理模块）模式。
   使用httpd -V 命令查看
   在configure配置编译参数的时候，可以使用 --with-mpm=prefork|worker|event 来指定编译为那一种MPM，当然也可以用编译为三种都支持：--enable-mpms-shared=all，
   这样在编译的时候会在modules目录下自动编译出三个MPM文件的so，然后通过修改httpd.conf配置文件更改MPM。应该是/etc/syconfig/httpd文件吧
1、Prefork MPM
Prefork MPM实现了一个非线程的、预派生的web服务器。它在Apache启动之初，就先预派生一些子进程，然后等待连接；可以减少频繁创建和销毁进程的开销，每个子进程只有一个线程，
在一个时间点内，只能处理一个请求。这是一个成熟稳定，可以兼容新老模块，也不需要担心线程安全问题，但是一个进程相对占用资源，消耗大量内存，不擅长处理高并发的场景。
2、Worker MPM
和prefork模式相比，worker使用了多进程和多线程的混合模式，worker模式也同样会先预派生一些子进程，然后每个子进程创建一些线程，同时包括一个监听线程，
每个请求过来会被分配到一个线程来服务。线程比起进程会更轻量，因为线程是通过共享父进程的内存空间，因此，内存的占用会减少一些，
在高并发的场景下会比prefork有更多可用的线程，表现会更优秀一些；
另外，如果一个线程出现了问题也会导致同一进程下的线程出现问题，如果是多个线程出现问题，也只是影响Apache的一部分，而不是全部。由于用到多进程多线程，
需要考虑到线程的安全了，在使用keepalive长连接的时候，某个线程会一直被占用，即使中间没有请求，需要等待到超时才会被释放（该问题在prefork模式下也存在）。
3、Event MPM
这是Apache最新的工作模式，它和worker模式很像，不同的是在于它解决了keepalive长连接的时候占用线程资源被浪费的问题，在event工作模式中，
会有一些专门的线程用来管理这些keepalive类型的线程，当有真实请求过来的时候，将请求传递给服务器的线程，执行完毕后，又允许它释放。这增强了在高并发场景下的请求处理。
问题：专门来管理keepalive类型的线程是什么线程？

案例：
apache与nginx的比较
nginx：
   1）nginx轻量级，同样是web服务，比apache占用更少的内存及资源，高并发能力大约是apache的10倍以上，nginx处理请求是异步非阻塞的，
      而apache则是阻塞型的，在高并发下nginx能保持低资源低消耗高性能
   2）nginx的静态处理性能力比apache强,nginx处理动态不行，一般动态请求要apache去做，nginx只适合静态和反向，
   3）核心区别，apache是同步多进程，一个连接对应一个进程，nginx是异步的，很多很多个连接可以对应一个进程
apache：
      rewrite比nginx的rewrite强大，模块多，
      基本常用模块都可以找到且bug少，nginx的bug相对较多。
      一般来说，需要性能的web服务选nginx，如果不需要性能求稳定就选apache


6）协议是HTTP/1.0和HTTP/1.1的区别：
   6.1）HTTP/1.0不支持持续链接，HTTP/1.0支持。
   持续链接分为流水线方式和非流水线方式。
   非流水线方式规定客户发送浏览请求得到响应后才能发送下一个。
   流水线方式客户不用等到响应就可以发送下一个请求，服务器收到请求后就可以连续响应，不用等待，节省了时间。
   6.2）HTTP 1.1还提供了与身份认证、状态管理和Cache缓存等机制相关的请求头和响应头

7) CGI是外部应用程序（CGI程序）与WEB服务器之间的接口标准，是在CGI程序和Web服务器之间传递信息的过程。Common Gateway Interface
   也就是允许客户端通过网页浏览器向远处服务器上的程序传输数据。


8）压力测试命令
   ab -c 10 -n 100 http://server/path
   -c，concurrency表示并发连接，也就是表示多少个ip同时连接
   -n，请求总数，同一个ip建立多少个请求通道

案例：
   写出下列命令的含义 
   Options FollowSymLinks   #跟随符号链接，允许访问符号链接所指向的原文件。为了安全，不应开启

案例：
TCP负责发现传输的问题，一有问题就发出信号，要求重新传输，直到所有数据安全正确地传输到目的地
虚拟终端协议(TELNET，TELecommunications NETwork)


OSI中的层                          功能                                        TCP/IP协议族
应用层                   文件传输，电子邮件，文件服务，虚拟终端      TFTP，HTTP，SNMP，FTP，SMTP，DNS，Telnet 等等
表示层                   数据格式化，代码转换，数据加密              没有协议
会话层                   解除或建立与别的接点的联系                  没有协议
传输层                   提供端对端的接口                            TCP，UDP
网络层                   为数据包选择路由                            IP，ICMP，OSPF，EIGRP，IGMP
数据链路层               传输有地址的帧以及错误检测功能              SLIP，CSLIP，PPP，MTU
物理层                   以二进制数据形式在物理媒体上传输数据        ISO2110，IEEE802，IEEE802.2


ethernet以太网络使用CSMA/CD（Carrier Sense Multiple Access with Collision Detection）带有冲突检测的载波监听多路访问技术
ethernet规定了多台电脑共享一个通道的方法

TCP/IP                            OSI
应用层                            应用层
                                  表示层
                                  会话层
主机到主机层（TCP）（又称传输层） 传输层
网络层（IP）(又称互联层)          网络层

网络接口层（又称链路层）          数据链路层
                                  物理层


常见的网络接口层协议有：--重要，详细参考鸟哥网络篇38页
    LAN:Ethernet 802.3（上层就是IP层）/Token Ring 802.5/ARP
   WLAN:ATM,Modem
案例：
php优化参数有哪些，fastcgi设置是多少，动态还是静态
答：
vim /etc/php.ini
open_basedir = /data/www:/tmp
display_errors = On
error_log = /usr/local/php/logs/php_errors.log
error_reporting = E_ALL & ~E_DEPRECATED
disable_functions= exec,system,passthru,error_log,ini_alter,dl,openlog,syslog,readlink, symlink,link,leak,fsockopen,proc_open,popepassthru,chroot,scandir, 
                   chgrp,chown,escapeshellcmd,escapeshellarg,shell_exec,proc_get_status,popen

A. disable_functions表示禁掉危险的函数
B. display_errors 打开错误日志
C. open_basedir 定义白名单目录

D. Php-fpm.conf中配置慢执行日志
E. Php-fpm.conf定义max_children，问题中的fastcgi设置指的就是最大进程数（max_children)，是一个动态值（dynamic）
案例：
网站出现500,502,400,403,404都是什么意思，怎么排查和解决
答案：
500：服务器内部错误，因为服务器上的程序写的有问题，需要打开错误日志，查看日志，分析错误信息。
502：网关错误，服务器作为网关或代理，从后端服务器收到无效响应。
     Nginx出现最多，出现502要么是nginx配置的不对，要么是php-fpm资源不够，可以分析php-fpm的慢执行日志，优化php-fpm的执行速度。
400：错误请求，服务器不理解请求的语法。这可能是用户发起的请求不合理，需要检查客户端的请求。
403：服务器拒绝请求。检查服务器配置，是不是对客户端做了限制。
404：未找到请求的资源。检查服务器上是否存在请求的资源，看是否是配置问题。

案例：
进程和线程的区别

1.定义
进程是系统进行资源分配和调度的一个独立单位.
线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位。
线程自己基本上不拥有系统资源,只拥有在运行中必不可少的资源(如程序计数器）,但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源.

2.关系
一个线程可以创建和撤销另一个线程;同一个进程中的多个线程之间可以并发执行.
线程是一个更加接近于执行体的概念，它可以与同进程中的其他线程共享数据，但拥有自己的栈空间，拥有独立的执行序列。

3.区别
进程和线程的主要差别在于它们是不同的操作系统资源管理方式。
进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。
线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些。
1) 简而言之,一个程序至少有一个进程,一个进程至少有一个线程.
2) 线程的划分尺度小于进程，使得多线程程序的并发性高。
3) 进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。
4) 从逻辑角度来看，多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。
   但操作系统并没有将多个线程看做多个独立的应用，来实现进程的调度和管理以及资源分配。这就是进程和线程的重要区别。
4.优缺点
线程和进程在使用上各有优缺点：线程执行开销小，但不利于资源的管理和保护；而进程正相反。同时，线程适合于在SMP机器上运行，而进程则可以跨机器迁移。
案例：
最大值计算公式
apache_max_process_with_good_perfermance < (total_hardware_memory / apache_memory_per_process ) * 2
apache_max_process = apache_max_process_with_good_perfermance * 1.5
web服务：
	C/S：
		Client: User Agent
		Server: httpd, nginx, lighttpd
	端口：
		0-1023：众所周知的端口；只有管理才有权限使用，永久的分配给某应用使用；
		1024-41951：注册端口，只有一部分被注册，分配原则不是特别严格；
		41952+：动态端口，私有端口；
		/proc/sys/net/ipv4/ip_local_port_range：定义了两个数字，表示可以做为临时端口的起始数字和结束数字；

	传输层协议：
		TCP, UDP, SCTP, DCCP
	
	套接字类型：
		tcp socket
		udp socket
		raw socket
	
	TCP协议的功能：
		连接的建立和拆除
		拥塞控制
			慢启动
			拥塞避免算法
		流量控制
			缓冲区
			滑动窗口
		确认、重传、超时
		排序
			序列号
		将数据打包成段
			校验和
	
	socket: IPC的一种实现，用于同一主机的不同进程通信；
	socket通信在domain:
		识别一个socket的方法 （socket的地址格式）
	
		domain:
			Unix Domain：基于socket机制实现同一主机上不同进程间通信的一种方式，AF_UNIX, AF_LOCAL, 地址是一个路径名（文件）
			IPv4 Domain：AF_INET，借助于ipv4协议实现不同主机（也可以是同一主机）上进程间通信的一种机制；地址格式是ipv4+16位的端口号
			IPv6 Domain: AF_INET6, 地址格式是ipv6+16位的端口号；
	
		socket的类型：
			TCP：流式socket, SOCK_STREAM; 可靠的、双向的、面向字节流的通信；
			UDP：数据报式socket，SOCK_DGRAM
	
		相关的系统调用：
			socket()：创建一新的socket
			bind()：绑定于一个套接字地址
			listen()：监听套接字
			accept(): 接收客户端请求
			connect(): 发起连接请求
			read()：接收数据；
			write()：发送数据；
			recv(), send(), recvfrom(), sendto()
			close(): 关闭连接
		tcp协议通过tcp状态来标记当前处于通信过程的哪个阶段：
			CLOSED, LISTEN, SYN_SENT, SYN_RECV, ESTABLISHED, 
	                    FIN_WAIT1, CLOSE_WAIT, FIN_WAIT2, LAST_ACK, TIME_WAIT, CLOSED


	http: hyper text transfer protocol
	html: hyper text mark language
			<html>
				<head>
					<title> TITLE </title>
				</head>
				<body>
					<h1>HEAD1</h1>
						<p>  </p>
					<h2>HEAD2</h2>
						<p> <a href="http://www.magedu.com/logo.jpg> To MageEdu</a>" </p>
				</body>
			</html>
	
			css: Cascading Style Sheet
	
		MIME: Multipurpose Internet Mail Extensions多用途互联网邮件扩展类型
			mp3 --> base64 --> base64 --> mp3
	            MIME类型是一种文本标记，表示一种主要的对象类型和一种特定的子类型：major/minor
				text/html
				text/plain
				image/jpeg
				image/gif
	
		web资源：任何一个通过web服务能单独获取的内容，就称为web资源；
	
		web资源的生成方式：
			静态资源：html, txt, jpeg, mp3
			动态资源：.php, .jsp
	
		web事务：
			client 发送请求 -->
			Server 发送响应 -->
			request --> response
	
		http协议版本：
			http/0.9：
			http/1.0: 
			http/1.1
		URI: Uniform Resource Identifier,包含URL和URN
			URL: Uniform Resource Locator
				方案, Internet地址，资源路径
				http://www.magedu.com:80/images/logo.jpg
			URN: Uniform Resource Name
				urn:ietf:rfc:1251


	一次Web请求响应的交互过程
		1、建立连接：接收客户端连接请求，涉及到tcp或者udp连接
		2、接收请求：从来自于网络的请求报文请求一次特定的资源请求；
			连接的输入/输出处理结构：
				单进程web服务器：启动一个进程接收请求，而且一次只处理一个请求；当处理结束后再接收并处理后续的请求；---最原始模型
				多进程web服务器：启动多个进程，每个进程处理一个请求；预生成模型，事先生成多个空闲子进程；进程池(线程池)；--prefork
				复用I/O的多线程web服务器：主进程启动n个子进程，每个子进程再启动m个线程，所以同时能处理的请求数：n*m---worker
	                            复用I/O的web服务器：一个进程响应多个请求；基于事件驱动模式实现；---evnet
	
		3、处理请求：对请求报文进程解析，获知请求资源等信息；
			根据请求文的首部来判定用户请求的资源；
				有许多详细信息：HEADER
					host: www.magedu.com
					url: /images/logo.jpg
					method: get
	
		4、访问资源：获取报文中指定请求的资源；
			web服务器即web资源服务器，负责发送预先创建好的或动态生成的内容；此些的放置位置称为docroot；
				/var/www/html/a.html
				/var/www/html/imags/jpgs/a.jpg
				http://www.magedu.com/a.html, http://www.magedu.com/images/jgps/a.jpg
	
			web服务器支持多种资源映射方式：
				(1) docroot
				(2) virtualhost docroot
				(3) user home docroot
				(4) alias
	
			注意：访问控制机制有可能会影响资源的访问权限；
	
		5、构建响应报文：
		6、发送响应报文：
			长连接：keepalive
			短连接：closed
	               主要是查看响应报文的connection首部的值，短连接就是closed，长连接就是keepalive
		7、记录日志


	http服务器种类：
		httpd
		nginx
		lighttpd
		IIS, tomcat, jetty, jboss, reisn
		websphere, weblogic, oc4j---这是web框架
		www.netcraft.com
	
	安装httpd: 
		 httpd的特性：
		 	高度模块化：core + modules, 
		 	DSO: Dynamic Shared Object，DSO指的就是httpd -M查看到的模块
		 	MPM：Multipath Processing Module 多进程处理模块，MPM只是DSO里面种的一类，因为要加载时都使用的是load命令，除此之外还有很多其他模块
		 		prefork: 多进程模型，每个进程响应一个请求；稳定性好，但并发能力有限；预先生成多个空闲进程；select()系统调用，1024个；
		 		worker： 多进程模型，每个进程可生成多个线程，每个线程响应一个请求；预先生成多个空闲线程；
		 		event：  一个进程直接响应n个请求；可同时启动多个进程；
		 			httpd-2.2: 测试使用；
		 			httpd-2.4: 可生产使用；
]# httpd -V  大V选项可以显示出httpd编译时的参数，也就可以显示出当前mpm模块
Server version: Apache/2.4.6 (CentOS)
Server built:   Oct 19 2017 20:39:16
Server's Module Magic Number: 20120211:24
Server loaded:  APR 1.4.8, APR-UTIL 1.5.2
Compiled using: APR 1.4.8, APR-UTIL 1.5.2
Architecture:   64-bit
Server MPM:     prefork
  threaded:     no
    forked:     yes (variable process count)
Server compiled with....
 -D APR_HAS_SENDFILE
 -D APR_HAS_MMAP
 -D APR_HAVE_IPV6 (IPv4-mapped addresses enabled)
 -D APR_USE_SYSVSEM_SERIALIZE
 -D APR_USE_PTHREAD_SERIALIZE
 -D SINGLE_LISTEN_UNSERIALIZED_ACCEPT
 -D APR_HAS_OTHER_CHILD
 -D AP_HAVE_RELIABLE_PIPED_LOGS
 -D DYNAMIC_MODULE_LIMIT=256
 -D HTTPD_ROOT="/etc/httpd"
 -D SUEXEC_BIN="/usr/sbin/suexec"
 -D DEFAULT_PIDLOG="/run/httpd/httpd.pid"
 -D DEFAULT_SCOREBOARD="logs/apache_runtime_status"
 -D DEFAULT_ERRORLOG="logs/error_log"
 -D AP_TYPES_CONFIG_FILE="conf/mime.types"
 -D SERVER_CONFIG_FILE="conf/httpd.conf"

		 httpd的功能特性：
		 	虚拟主机：
		 		基于HOST、IP、PORT实现虚拟主机
		 		CGI：Common Gateway Interface
		 		反向代理
		 		负载均衡
		 		路径别名
		 		丰富的用户认证
		 			basic
		 			digest
		 				file
		 				mysql
		 				ldap
		 		支持第三方模块
	
		 安装方式：rpm包，源码定制
		 	rpm格式的安装其工作目录：/etc/httpd
		 	配置文件：
		 		/etc/httpd/conf/httpd.conf
		 		/etc/httpd/conf.d/*.conf
		 	服务脚本：
		 		/etc/rc.d/init.d/httpd
		 		脚本的配置文件：/etc/sysconfig/httpd
		 	模块目录：
		 		/etc/httpd/modules: 符号链接文件
		 		/usr/lib64/httpd/modules
		 	主程序：
		 		/usr/sbin/httpd: 默认的MPM为prefork
		 		/usr/sbin/httpd.work
		 		/usr/sbin/httpd.event
		 	日志文件目录：
		 		/var/log/httpd/access_log：访问日志
		 		/var/log/httpd/error_log: 错误日志
		 	站点文档目录：
		 		/var/www/html
	
		配置文件：
			# grep "Section" httpd.conf
			### Section 1: Global Environment
			### Section 2: 'Main' server configuration
			### Section 3: Virtual Hosts
			注意：Main Server和Virtual Hosts不同时使用；默认启用的是Main Server；
	
			配置文件的语法 
				指令值
					指令：不区分字符大小写
					值：有可能区分字符大小写
	
			(1) 关闭欢迎页面
				/etc/httpd/conf.d/welcome.conf：重命名（不以.conf结尾）或删除
			(2) 指定监听的地址和端口
				Listen [IP:]PORT
				注意：Listen可以出现多次；


回顾：http协议，一次http事务的执行过程，httpd
	http: C/S
	URL：scheme, host:port, path to resource
	一次事务过程：建立连接、接受请求、处理请求、访问资源、构建响应、发送响应、记录日志
	并发响应：
		单进程模型：串行模式
		多进程模式：
			一个进程响应一个一请求；
			一个进程生成多个线程，一个线程一个请求；
		复用I/O模型：
			一个进程（或线程）响应多个请求；
		多进程、复用I/O模型：
			多个进程，每个进程响应多个请求；

	httpd MPM：
		prefork: 多进程模型；
		worker：多线程模型；
		event：复用I/O模型；
		http://httpd.apache.org

/etc/httpd/conf/httpd.conf配置项说明：
    UserDir public_html       用户的公用目录路径
    DefaultType text/plain     默认网页格式为文本模式
    AddLanguage en .en         加入英文字体
    DocumentRoot "/var/www/html"  网站根目录
    AddType application/x-httpd-php.php .php  增加对.php文件的支持

httpd的基础配置(2)
	http: stateless，无状态
	(3) 持久连接
		KeepAlive Off|On
		MaxKeepAliveRequests 100  最大长连接请求数。
		KeepAliveTimeout 15

	(4) MPM
		httpd -l: 显示编译进核心的模块
		httpd -M: 显示DSO模块
# prefork MPM
# StartServers: number of server processes to start 启动时的进程数量
# MinSpareServers: minimum number of server processes which are kept spare
# MaxSpareServers: maximum number of server processes which are kept spare
# ServerLimit: maximum value for MaxClients for the lifetime of the server 服务器生命周期内的MaxClients的最大值
# MaxClients: maximum number of server processes allowed to start 服务器进程允许开启的最大客户端数量，可以理解为最大并发连接数
# MaxRequestsPerChild: maximum number of requests a server process serves 
  子进程能够处理的最大请求数量，也就是如果一个进程连续的处理完4000个请求后，必须要销毁该进程，然后主进程重新启动一个子进程
补充说明：
  1）内存溢出就是内存耗尽，说的直接点就是程序占着内存不释放
  2）httpd出现内存溢出的原因和解决方法
     进程处理完一个用户的请求后，有些资源不想释放，因为有些资源是可以对下个用户重用的，这样就留在了里面，所以内存就开始攀升，
     因此针对这个问题，显示地降低MaxRequestsPerchild（每个子进程所处理的最大请求数）参数，实现在处理完指定的请求后，kill掉进程，再重新创建进程，保持内存不会溢出。

		<IfModule prefork.c>
		StartServers       8
		MinSpareServers    5
		MaxSpareServers   20
		ServerLimit      256
		MaxClients       256
		MaxRequestsPerChild  4000
		</IfModule>
# worker MPM
# StartServers: initial number of server processes to start
# MaxClients: maximum number of simultaneous client connections
# MinSpareThreads: minimum number of worker threads which are kept spare
# MaxSpareThreads: maximum number of worker threads which are kept spare
# ThreadsPerChild: constant number of worker threads in each server process 每个子进程生成的工作线程
# MaxRequestsPerChild: maximum number of requests a server process serves 子进程能够处理的最大请求数量，0表示没有限制，能处理多少是多少

		<IfModule worker.c>
		StartServers         4
		MaxClients         300
		MinSpareThreads     25
		MaxSpareThreads     75
		ThreadsPerChild     25
		MaxRequestsPerChild  0
		</IfModule>
	
	(5) DSO
		 LoadModule foo_module modules/mod_foo.so
		 相对于ServerRoot参数所指定的路径；ServerRoot /etc/httpd/
		 注意：修改了装载的模块后，reload即可生效；
		 httpd -t: 检查配置文件语法
	
	(6) 指定Main Server的docroot
		DocumentRoot "/var/www/html"
		例如：/www/htdocs
			文件系统路径：/www/htdocs/bbs/upload/a.rar
			URL路径：http://Server_IP/bbs/upload/a.rar
	
	(7) 站点路径访问控制
		基于本地文件系统路径
			<Directory "/path/to/some_directory">
	
			</Directory>
		基于URL
			<Location "/path/to/some_url">
	
			</Location>
	
	(8) Directory目录的访问权限控制设置--非常重要，详细参考鸟哥网络篇666页说明
		(a) Options此设置值表示在这个目录内能够让apache进行的操作，也就是针对apache程序的权限设置
			Indexes: 当访问的路径下无默认的主页面时，将所有资源以列表形式呈现给用户；危险，慎用；
			FollowSysLinks：字面意思就是让连接文件生效，一般来说apache的程序目录只能在/var/www/html内，
	                                    你在/var/www/html下面的连接文件只要链接到非此目录的其他地方，
	                                    则该链接文件默认是失效的。但是使用此设置可让链接文件有效的离开此目录
			None: 一个也没有；
			All: 所有
	                    ExecCGI：让此目录具有执行CGI程序的权限
		(b) AllowOverride 表示是否允许额外配置文件.htaccess的某些参数覆盖/etc/httpd/conf/httpd.conf文件里面的参数,常见参数有None，ALL等
		(c) 基于IP的访问控制
	                httpd-2.2版本
			Order allow,deny
			Allow from all
				from后面能接受的地址格式：
					IP, Network Address
					网络地址格式：
						172.16
						172.16.0.0
						172.16.0.0/16
						172.16.0.0/255.255.0.0
	
			Order allow,deny
			Deny from 172.16.100.77
			Allow from 172.16
	               httpd-2.4版本
	                    require all granted	
	
	(9) 定义默认的主页面
		DirectoryIndex index.html index.html.var
	
	(10) 配置日志功能
		ErrorLog logs/error_log：定义错误日志文件路径；
		LogLevel warn
		LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
			%h Remote host
			%l Remote logname (from identd, if supplied)
			%u Remote user (from auth; may be bogus if return status (%s) is 401)
			%t Time the request was received (standard english format)
			%r First line of request,即method  url  version
			%s Status. For requests that got internally redirected, this is the status of the *original* request --- %>s for the last.
			%b Size of response in bytes, excluding HTTP headers. In CLF format, i.e. a '-' rather than a 0 when no bytes are sent.
		%{Foobar}i The contents of Foobar: header line(s) in the request sent to the server.
				%{referer}i: 跳转至当前页面之前上一次所在的页面；
				%{User-Agent}i：用户代理；
			详情请参考：http://httpd.apache.org/docs/2.2/mod/mod_log_config.html#formats
	
	            访问日志语句： CustomLog logs/access_log  combine，即分别是指令，日志文件，日志格式名称
补充说明：
        referer：引用，参考，推荐
        简而言之，HTTP Referer是header的一部分，当浏览器向web服务器发送请求的时候，一般会带上Referer，告诉服务器我是从哪个页面链接过来的，
        服务器藉此可以获得一些信息用于处理。比如从我主页上链接到一个朋友那里，他的服务器就能够从HTTP Referer中统计出每天有多少用户点击我主页上的链接访问他的网站
		
[root@node4 httpd]# tail /var/log/httpd/access_log 
192.168.139.183 - - [03/Sep/2017:22:36:02 +0800] "GET /index.html HTTP/1.0" 404 289 "-" "ApacheBench/2.3"   合计9个字段
192.168.139.183 - - [03/Sep/2017:22:36:02 +0800] "GET /index.html HTTP/1.0" 404 289 "-" "ApacheBench/2.3"

​	

	(12) 路径别名
		docroot路径映射：
			url: docroot/
			docroot: /www/htdocs
				http://172.16.100.8/bbs/images/1.html
					/www/htdocs/bbs/images/1.html
	
			路径别名：
				/bbs  /web/forum/
					http://172.16.100.8/bbs/images/1.html
						/web/forum/images/1.html
	
		定义方法：
			Alias /URL/ "/path/to/some_directory/" 
	            路径别名的具体使用案例：
	                    vim /etc/httpd/conf/httpd.conf 在Section 2的任意位置增加如下语句
	                        Alias /bbs  /web/bbs
	                    mkdir -pv /web/bbs
	                    vim /web/bbs/index.html
	                        this is from /web/bbs/index.html
	                    service httpd start 
	                    curl http://server/bbs/ 注意：最后的/要加上去否则访问不成功
	
	(13) 设定默认字符集
		AddDefaultCharset UTF-8
	            常用字符集：UTF-8, GBK, GB2312, GB18030
	(14) 基于用户的访问控制
		质询：
			WWW-Authenticate：服务器用401状态拒绝客户端请求，说明需要用户提供用户名密码；
		认证：
			Authorization: 客户端填入账号和密码后重新发出请求；包含认证算法、用户名和密码；
		
		安全域：security realm
			docroot: /www/htdocs/
							admin/
							finance/
	
		认证方式：
			基本认证：basic
				  Base64
			摘要认证：digest
	
		配置httpd使用basic认证：
			(a) 定义security realm
				<Directory "/www/htdocs/admin">
				    Options None
				    AllowOverride None
				    AuthType Basic
				    AuthName "Admin Area"
				    AuthUserFile /etc/httpd/users/.htpasswd
				    Require user tom
				</Directory>
	
				Require valid-user: 允许账号文件中的所有用户访问；
				Require user USER1, USER2, ...：仅允许指定用户登录；			
	
			(b) 提供用于认证的文件
				htpasswd [options] password_file username
					-c: 添加第一个用户时自动创建账号文件；
					-m: 以md5格式加密密码；
					-s: 以sha格式加密密码;
					-D: 删除指定用户
	
			(c) 组认证
				<Directory "/www/htdocs/admin">
				    Options None
				    AllowOverride None
				    AuthType Basic
				    AuthName "Admin Area"
				    AuthUserFile /etc/httpd/users/.htpasswd
				    AuthGroupFile /etc/httpd/users/.htgroup
				    Require group GRP1, 
				</Directory>				
				组文件：
					每行定义一个组，格式
					GROUP: user1 user2 user3
	         基于basic认证的具体操作步骤：
	                htpasswd -c  /var/ww/html/.htpasswd chenhao      首次创建才加-c选项，接下来输入密码即可
	                vim /etc/httpd/conf/httpd.conf 
	                    Alias /bbs  /web/bbs
	                    <Directory "/web/bbs">
				    Options None
				    AllowOverride None
				    AuthType Basic
				    AuthName "chenhao"
				    AuthUserFile /var/www/html/.htpasswd
				    Require user chenhao
		       </Directory>
	                service httpd restart 
	                在浏览器中输入http://server/bbs/，输入账号和密码即可
	          补充说明:如果访问不了，查看/var/log/httpd/error.log， .htpasswd文件最好放置在/var/www/html/目录内


	(15) 虚拟主机
		一个物理服务器服务于多个站点：每个站点通过一个虚拟主机来实现；
		httpd支持三种类型的虚拟主机：
			基于IP
			基于Port
			基于Host
	
		注意：取消Main Server; 注释DocumentRoot指令即可；--重要
		定义虚拟主机：
			<VirtualHost "IP:PORT">
				ServerName
				ServerAlias
				DocumentRoot 
				<Directory >
	                                 代码段.................
				</Directory>
				ErrorLog
				CustomLog 
			</VirtualHost>
	
		示例1：基于IP
	                 vim /etc/httpd/conf.d/virtualhost
			<VirtualHost 172.16.100.8:80>
			    ServerName www.chenhao.com
			    DocumentRoot "/web/www"
			</VirtualHost>
			<VirtualHost 172.16.100.9:80>
			    ServerName bbs.chenhao.com
			    DocumentRoot "/web/bbs"
			</VirtualHost>	
	
		示例2：基于Port
	                 vim /etc/httpd/conf/httpd.conf
			Listen 80
			Listen 8080
	                 vim /etc/httpd/conf.d/virtualhost
			<VirtualHost 172.16.100.8:80>
			    ServerName www.chenhao.com
			    DocumentRoot "/web/www"
			</VirtualHost>
			<VirtualHost 172.16.100.8:8080>
			    ServerName bbs.chenhao.com 
			    DocumentRoot "/web/bbs"
			</VirtualHost>	
	
		示例3：基于Host
	                    vim /etc/httpd/conf.d/virtualhost
		            NameVirtualHost *:80   httpd 2.2版本必须加这一句,httpd 2.4版本不需要
			    <VirtualHost *:80>     要写成*:80，不能写成具体的ip地址
			       ServerName www.chenhao.com
			       DocumentRoot "/web/www"
			    </VirtualHost>
			    <VirtualHost *:80>
			       ServerName bbs.chenhao.com 
			       DocumentRoot "/web/bbs"
			     </VirtualHost>
	                     vim /etc/hosts
	                         172.16.100.8 www.chenhao.com bbs.chenhao.com 
	                     service httpd restart 


	(16) 内置的status页面
	         mod_status模块可以让管理员查看服务器的执行状态，它通过一个HTML页面展示了当前服务器的统计数据。这些数据通常包括但不限于：
	        (1) 处于工作状态的worker进程数；
	        (2) 空闲状态的worker进程数；
	        (3) 每个worker的状态，包括此worker已经响应的请求数，及由此worker发送的内容的字节数；
	        (4) 当前服务器总共发送的字节数；
	        (5) 服务器自上次启动或重启以来至当前的时长；
	        (6) 平均每秒钟响应的请求数、平均每秒钟发送的字节数、平均每个请求所请求内容的字节数；
	
		<Location /server-status>
		    SetHandler server-status
		    Order deny,allow
		    Deny from all
		    Allow from 172.16
		</Location>	

回顾：配置httpd
	日志：
		错误日志
			ErrorLog
			LogLevel
		访问日志
			LogFormat
			CustomLog	

	虚拟主机：
		基于IP
		基于Port
		基于Host
			Host: www.magedu.com
			NameVirtualHost *:80
	
		<VirtualHost IP:PORT>
			ServerName
			ServerAlias
			DocumentRoot
			ErrorLog
			CustomLog 
		</VirtualHost>
基于用户的访问控制
		<Directory "">
			Options 
			AllowOverride
			AuthType Basic
			AuthName ""
			AuthUserFile 
			AuthGroupFIle
			Require group|user ...
		</Directory>


http协议及httpd配置
	httpd: stateless
	httpd事务：请求<-->响应
	报文格式：http request请求报文和http response响应报文

		由三部分组成：
			start line: 起始行，对于报文进行描述
			header块：由一个或多个header组成
			body: 主体部分，可选；
	
			http request格式：
				<method> <request-URL> <version>
				<headers>
				<body>
	
			GET  /index.html  HTTP/1.1
	
			http response格式：
				<version> <status> <reason-phrase>
				<headers>
				<body>---body就是我们日常的网页内容

一个完整的httpd响应报文内容如下：
				HTTP/1.1 200 OK
				Date: Sat, 13 Dec 2014 01:01:59 GMT---从此处开始就是headers
				Server: Apache/2.2.15 (CentOS)
				Last-Modified: Fri, 12 Dec 2014 10:01:00 GMT
				ETag: "2000b-14-50a01f84dffd0"
				Accept-Ranges: bytes
				Content-Length: 20
				Connection: close
				Content-Type: text/html; charset=UTF-8				

		方法：客户端希望对服务器资源执行的动作；是一个单词，例如GET, HEAD, POST；
		请求URL：所请求的特定资源路径，可以是相对URL，也是绝对URL
		版本：HTTP协议版本，HTTP/<major>.<minor>
		状态码：三位的数字，第一位数字用于描述状态类别；
		原因短语：状态的码可读版本；
		headers: 每个header都有一个名字，后跟一个冒号，后跟值；
	
	http的常用方法：
		GET：从服务器获取一个资源，用来读取数据的；无需<body>部分；
		HEAD：只获取资源的http响应报文首部，无须<body>部分；
		POST：向服务器发送需要处理的数据，用到<body>; 
		PUT：通过<body>附加内容发送至服务器保存；
		TRACE：追踪一个报文经由的服务器或代理服务器；无须<body>；
		OPTIONS：判断对服务器上资源可以使用哪些方法；
		DELETE：从服务器删除一个指定的资源；无须<body>
curl命令
		curl是基于URL语法在命令行方式下工作的文件传输工具，它支持FTP, FTPS, HTTP, HTTPS, GOPHER, TELNET, DICT, FILE及LDAP等协议。
                curl支持HTTPS认证，并且支持HTTP的POST、PUT等方法， FTP上传， kerberos认证，HTTP上传，代理服务器， cookies， 用户名/密码认证， 下载文件断点续传，
                上载文件断点续传,http代理服务器管道（ proxy tunneling),甚至它还支持IPv6,socks5代理服务器,通过http代理服务器上传文件到FTP服务器等等，功能十分强大。
		curl的常用选项：
		    -A/--user-agent <string> 设置用户代理发送给服务器
		    -basic 使用HTTP基本认证
		    --tcp-nodelay 使用TCP_NODELAY选项
		    -e/--referer <URL> 来源网址
		    --cacert <file> CA证书 (SSL)
		    --compressed 要求返回是压缩的格式
		    -H/--header <line>自定义头信息传递给服务器
		    -I/--head 只显示响应报文首部信息
		    --limit-rate <rate> 设置传输速度
		    -u/--user <user[:password]>设置服务器的用户和密码
		    -0/--http1.0 使用HTTP 1.0
                    -s 表示静默模式，也就是不输出错误信息
                    -l/--list-only	列出ftp目录下的文件名称
                    -S/--show-error	显示错误	
                   -o/--output   把输出写到该文件中
                   -T/--upload-file <file>  上传文件,如果需要上传文件，最好使用-T，不要使用-X
                   -X/--request <command>   表示支持的方法或者命令
                   -d/--data <data>   HTTP POST data (H) 

[root@localhost nginx]# curl -I   http://192.168.139.169
HTTP/1.1 200 OK
Server: nginx/1.10.2
Date: Sat, 16 Sep 2017 15:15:27 GMT
Content-Type: text/html
Content-Length: 29
Last-Modified: Sat, 16 Sep 2017 10:56:03 GMT
Connection: keep-alive
ETag: "59bd0343-1d"
Accept-Ranges: bytes

	http的响应状态码：
		100-199            100-101              提示信息
		200-299            200-206              成功
		300-399            300-305              重定向
		400-499            400-415              客户端错误
		500-599            500-505              服务端错误
		200：成功，请求的所有数据通过响应报文的body部分发送，OK
		301：请求的URL已经移除，但在响应报文首部通过Location指明了资源现在所处的位置；Moved Permanently永久移除
		302: 与301相似，但在响应报文首部通过Location资源的临时位置；Found
		304：客户发出条件式请求，如果服务器发现资源未改变，则可通过响应报文告诉客户端；Not Modified
	            400：请求错误，比如请求语法错误等
		401：需要输入账号和密码，Unauhtorized
		403：请求被禁止；Forbidden
		404：服务器无法找到所请求的URL对应的资源，Not Found
		500：服务器内部错误，Internal Server Error，比如服务器内部程序错误
		502：代理服务器从后端服务收到了一条伪响应；Bad Gateway，一般指代理服务器不能从后端real server获得正常的响应报文
	    详细的http首部解释，建varnish的1631行开始
	Http协议为stateless：
		cookie：实现持久会话；
			会话cookie：临时cookie，生命周期是浏览器进程
			持久cookie：cookie会存储在硬盘上，不会浏览器进程终止而被删除；
	
	(17) https: ssl
		ssl: secure socket layer, sslv3
		tls: transport layer security, tlsv1
		服务器认证、客户端认证、完整性、保密性、效率、普适性
	
		https, http/ssl, https:// ，443/tcp
		SSL会话简化过程：
			(1) 客户端发送可供选择的加密方式并请求证书；
			(2) 发送证书以及选定加密方式给客户端；
			(3) 生成临时会话密钥，并使用服务器的公钥加密发送给服务器商；
			(4) 双方进行安全通信 
	
		PKI: Public Key Infrastructure  CA、RA、CRL、证书存取库
	
		x509证书格式：
			证书版本号
			证书序列号
			证书签名算法ID
			证书颁发者
			有效期限
			主体名称
			主体公钥
			CA的惟一ID
			主体的惟一ID
			扩展信息
			CA签名

补充说明：
    ssl会话是基于ip地址创建，所以单ip地址主机上仅可以使用一个https虚拟主机
    http本身是文本协议，但是调用了ssl协议后，编码格式就变成了二进制格式

		配置过程
			(a) 建立私有CA
			(b) 为服务器生成证书
			(c) 配置httpd使用数字证书
	
				安装相应的模块程序包
				# yum install mod_ssl -y
	                            # rpm -ql   mod_ssl 
/etc/httpd/conf.d/ssl.conf
/usr/lib64/httpd/modules/mod_ssl.so
/var/cache/mod_ssl
/var/cache/mod_ssl/scache.dir
/var/cache/mod_ssl/scache.pag
/var/cache/mod_ssl/scache.sem

				vim /etc/httpd/conf.d/ssl.conf
	                                LoadModule ssl_module modules/mod_ssl.so
	                                Listen 443
	                                <VirtualHost _default_:443>
	                                   DocumentRoot "/web/www/"
	                                   ServerName 192.168.139.198
	                                   ErrorLog logs/ssl_error_log
	                                   TransferLog logs/ssl_access_log
	                                   LogLevel warn
	                                   SSLEngine on
	                                   SSLProtocol all -SSLv2
	                                   SSLCertificateFile /etc/pki/tls/certs/httpd.crt        证书名称
	                                   SSLCertificateKeyFile /etc/pki/tls/private/httpd.key   私钥名称
	                                 </VirtualHost>
	                           # (umask 077; openssl genrsa -out /etc/pki/tls/private/httpd.key 1024) 
	                           # openssl req -new -x509  -days 365   -key /etc/pki/tls/private/httpd.key  -out /etc/pki/tls/certs/httpd.crt
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CH
State or Province Name (full name) []:guangdong
Locality Name (eg, city) [Default City]:shengzhegn
Organization Name (eg, company) [Default Company Ltd]:bencent
Organizational Unit Name (eg, section) []:rd
Common Name (eg, your name or your server's hostname) []:
Email Address []:
                              
                                 httpd -t 
                                 service httpd start 
                                 在浏览器中输入https://server/URL进行访问
    			测试：openssl s_client -connect IP:PORT -CAfile /path/to/ca_certificat_file，没有用过
    	                注意：ssl会话只能基于IP创建，这意味着如果服务器仅有一个IP，那么仅为一个虚拟主机提供https服务；
    
    (18) 访问属性配置(总结)
    	基于本地文件系统的配置：
    		<Directory [~] "">
    			...
    		</Directory>
    
    		~: 后面的路径支持模式匹配
    
    		<File [~] "">
    			...
    		</File>
    
    	基于URL的配置：
    		<Location [~] "">
    			...
    		</Location>
    		<LocationMatch "">
    			...
    		</LocationMatch>
    
    (19) 使用mod_deflate模块压缩页面优化传输速度
    
    	SetOutputFilter DEFLATE
    	# mod_deflate configuration
    	# Restrict compression to these MIME types
    	AddOutputFilterByType DEFLATE text/plain 
    	AddOutputFilterByType DEFLATE text/html
    	AddOutputFilterByType DEFLATE application/xhtml+xml
    	AddOutputFilterByType DEFLATE text/xml
    	AddOutputFilterByType DEFLATE application/xml
    	AddOutputFilterByType DEFLATE application/x-javascript
    	AddOutputFilterByType DEFLATE text/javascript
    	AddOutputFilterByType DEFLATE text/css
     
    	# Level of compression (Highest 9 - Lowest 1)
    	DeflateCompressionLevel 9
    	 
    	# Netscape 4.x has some problems.
    	BrowserMatch ^Mozilla/4 gzip-only-text/html
    	 
    	# Netscape 4.06-4.08 have some more problems
    	BrowserMatch ^Mozilla/4\.0[678] no-gzip
    	 
    	# MSIE masquerades as Netscape, but it is fine
    	BrowserMatch \bMSI[E] !no-gzip !gzip-only-text/html
    
    httpd程序自带的工具程序：
    	httpd: apache的服务程序
    		-t: 配置文件语法测试
    		-M: 列出所有已经装载的模块
    		-l: 列出所有的静态模块
    		-S：列出所有的虚拟主机,大写S
    
    	htpasswd: basic认证基于文件实现时用到账号密码文件生成器
    	apachectl: shell脚本，httpd服务控制
    	apxs: httpd得以扩展使用第三方模块的工具接口；
    	htdigest: 为digest认证创建和更新用户认证文件
    	rotatelogs: 不关闭httpd而切换其使用到的日志文件，access_log, access_log.1, access_log.2
    	suexec: 访问某些特殊权限配置的页面资源时，临时切换至指定用户
    		User apache
    		Group apache
    	ab: apache bench  bench是工作台意思


练习：

	1、建立httpd 服务器(基于编译的方式进行)，要求：编译的与yum生成的有很大区别
	   提供两个基于名称的虚拟主机:
			(a)www1.stuX.com，页面文件目录为/web/vhosts/www1；错误日志为/var/log/httpd/www1.err，访问日志为/var/log/httpd/www1.access；
			(b)www2.stuX.com，页面文件目录为/web/vhosts/www2；错误日志为/var/log/httpd/www2.err，访问日志为/var/log/httpd/www2.access；
			(c)为两个虚拟主机建立各自的主页文件index.html，内容分别为其对应的主机名；
			(d)通过www1.stuX.com/server-status输出httpd工作状态相关信息，且只允许提供帐号密码才能访问(status:status)；
	                     cd /usr/local/apache24
	                     vim conf/httpd.conf
	                     Include conf/extra/httpd-vhosts.conf 约在484行
	                     #DocumentRoot "/usr/local/apache24/htdocs"  注销中心主机
	                     vim conf/extra/httpd-vhosts.conf
	                     <VirtualHost *:80>
	                         <Location /server-status>
	                               SetHandler server-status
	                               options None
	                               AllowOverride None
	                               AuthType Basic
	                               AuthName "chenhao"
	                               AuthUserFile /usr/local/apache24/htdocs/.htpasswd
	                               Require user status
	                               Require all granted
	                          </Location>
	                     </VirtualHost>
	                    同样的配置，yum生成的能使用账户密码访问，编译的就不行？
			
	2、为上面的第2个虚拟主机提供https服务，使得用户可以通过https安全的访问此web站点；
		(1)要求使用证书认证，证书中要求使用的国家(CN)、州(Henan)、城市(Zhengzhou)和组织(MageEdu)；
		(2)设置部门为tech，主机名为www2.stuX.com，邮件为admin@stuX.com；

回顾： http的基本原理，https
	request报文：
		<method> <Request-URL> <version>
		<HEADERS>
		<entity-body>
	response报文：
		<version> <status> <reason-phrase>
		<HEADERS>
		
		<entity-body>
		
	method: GET，HEAD,  POST, PUT, DELETE, TRACE, OPTIONS
		curl -I URL
		
	HEADERS:
		通用首部
			Date，Via
		请求首部
			Host, 
		响应首部
			Server
		实体首部:
			
		扩展首部
			X-Forward-For
			
	status：
		1xx: 
		2xx
			200
		3xx
			301, 302, 304
		4xx
			403, 404
		5xx
			502
			
	https --> SSL:
		80, 443

编译安装httpd－2.4：
	版本：
		httpd-1.3
		httpd-2.0
		httpd-2.2
		httpd-2.4
	2.4的新特性：
		1) MPM支持运行时装载  --enable-mpms-shared=all --with-mpm=prefork|worker|event
		2) 支持event MPM
		3) 异步读写
		4) 支持每模块及每目录分别使用不同的日志级别
		5) 每请求配置； <If>
		6) 增强版的表达式分析器；
		7) 支持毫秒级keepalive timeout;
		8) 基于FQDN的虚拟主机不再需要NameVirtualHost； 
		9) 支持用户使用自定义变量； 
		新增一些模块：mod_proxy_fcgi， mod_ratelimit, mod_request, mod_remoteip
		修改了一些配置机制：不再支持使用order, allow, deny来实现基于IP的访问控制，改用require
		
	httpd依赖于apr, apr-util
		apr: apache portable runtime
		httpd-2.4 依賴于1.4＋及以上版本的apr
	
	   编译安装httpd－2.4的详细步骤：
	        (1) 编译安装apr
	           # tar -zxf /home/chenhao/apr-1.5.2.tar.gz
	           # cd /home/chenhao/apr-1.5.2
	           # ./configure --prefix=/usr/local/apr15
	           # make 
	           # make install
	        (2) 编译安装apr-util
	           # tar -zxf /home/chenhao/apr-util-1.5.4.tar.gz
	           # cd /home/chenhao/apr-util-1.5.3
	           # ./configure --prefix=/usr/local/apr-util15 --with-apr=/usr/local/apr15
	           # make 
	           # make install
	        (3) 编译httpd 2.4版本
	       # yum install pcre-devel glibc -y
	           # tar -zxf /home/chenhao/httpd-2.4.25.tar.gz
	           # cd /home/chenhao/httpd-2.4.25
	       # ./configure --prefix=/usr/local/apache24 --sysconfdir=/etc/httpd24 --enable-so --enable-ssl --enable-cgi --enable-rewrite --with-zlib --with-pcre 
	               --with-apr=/usr/local/apr15 --with-apr-util=/usr/local/apr-util15 --enable-modules=most --enable-mpms-shared=all --with-mpm=prefork
	       # make 
	           # make install
	    如果编译出现错误，执行make clean 
		
	基于IP的访问控制法则：
			允许所有主机访问：Require all granted
			拒绝所有主机访问：Require all deny
			控制特定IP的主机访问：
				Require ip IPPADDR
				Require not ip IPADRR
					IPADDR:
						单个IP
						Network/Mask
						Network/Length
						Net: 172.16
		
				Require host HOSTNAME
				Require not host HOSTNAME
					HOSTNAME:
						FQDN:特定主机
						DOMAIN: 域内的所有主机
						
	切换使用的MPM：LoadModule mpm_event_module modules/mod_mpm_event.so	
	博客作业：虚拟主机、SSL、基于用户的访问控制；


	LAMP:
		动态资源：脚本程序，需要运行之后生成结果（html），web服务器将由app server生成的结果当作响应主体(body)回应给客户端
		httpd自身无法解析动态资源，因此需要对某种解释器或运行环境来执行用户请求的动态资源。
		编程语言：php, 有些语言需要依赖于web开发框架(ruby, ror; python(Django, flask), perl,), jsp;
		
		LAMP
			L: linux, A: apache, M: mysql (mariadb), P: php, perl, python
			
			程序：指令＋数据
			      指令：程序代码
			      数据：文件
				
			程序：算法＋数据结构
			
			Client --> http --> httpd --> (a.php) --> cgi --> php server (执行a.php程序) --> mysql --> mysql server (加载数据)
			
			php与httpd的结合方式：
				module:  php做为httpd的模块
				cgi：    httpd通过cgi模块以及cgi协议与解释器进行通信； 
				fastcgi：解释器自己工作为一个server，监听在某套接字上； 
					
			php如何与mysql交互：
				解释器无须与mysql交互，需要使用数据服务的是相应的程序代码;
				数据服务有很多种：比如redis, mongodb, hbase等等； 程序与任何一种数据存储交互都需要用到其专用的协议和驱动； 
				
			CentOS 6.6: AMP
				PHP:以模块化方式与httpd结合工作； 
					httpd的不同的mpm模型下，其所需要的php模块格式有所不同； 
						prefork：进程式
							libphp
						worker和event：线程式
							libphp-zts
				rpm安装的php的配置文件路径：
					/etc/php.ini
					/etc/php.d/*.ini

回顾：编译安装httpd-2.4，LAMP框架
	httpd-2.4: 
		2.0，2.2， 2.4
		MPM以DSO方式的实现，毫秒级的keepalive的timeout
		Require granted|deny
		
	LAMP框架：
		httpd: 静态内容
			.jpg, .gif, .css, .html, .txt
		php(per, python)：动态内容
			.php
		mysql：数据存储、检索及管理
		rpm: php
			/etc/php.ini, /etc/php.d/*.ini
		index.php:
			<html>
				
			</html>
		http://www.magedu.com/index.php





关于PHP

		一、PHP简介
			
		PHP是通用服务器端脚本编程语言，其主要用于web开发以实现动态web页面，它也是最早实现将脚本嵌入HTML源码文档中的服务器端脚本语言之一。同时，php还提供了一个命令行接口，因此，其也可以在大多数系统上作为一个独立的shell来使用。
	
		Rasmus Lerdorf于1994年开始开发PHP，它是初是一组被Rasmus Lerdorf称作“Personal Home Page Tool” 的Perl脚本， 这些脚本可以用于显示作者的简历并记录用户对其网站的访问。后来，Rasmus Lerdorf使用C语言将这些Perl脚本重写为CGI程序，还为其增加了运行Web forms的能力以及与数据库交互的特性，并将其重命名为“Personal Home Page/Forms Interpreter”或“PHP/FI”。此时，PHP/FI已经可以用于开发简单的动态web程序了，这即是PHP 1.0。1995年6月，Rasmus Lerdorf把它的PHP发布于comp.infosystems.www.authoring.cgi Usenet讨论组，从此PHP开始走进人们的视野。1997年，其2.0版本发布。
	
		1997年，两名以色列程序员Zeev Suraski和Andi Gutmans重写的PHP的分析器(parser)成为PHP发展到3.0的基础，而且从此将PHP重命名为PHP: Hypertext Preprocessor。此后，这两名程序员开始重写整个PHP核心，并于1999年发布了Zend Engine 1.0，这也意味着PHP 4.0的诞生。2004年7月，Zend Engine 2.0发布，由此也将PHP带入了PHP 5时代。PHP5包含了许多重要的新特性，如增强的面向对象编程的支持、支持PDO(PHP Data Objects)扩展机制以及一系列对PHP性能的改进。
	
		二、PHP Zend Engine
	
		Zend Engine是开源的、PHP脚本语言的解释器，它最早是由以色列理工学院(Technion)的学生Andi Gutmans和Zeev Suraski所开发，Zend也正是此二人名字的合称。后来两人联合创立了Zend Technologies公司。
	
		Zend Engine 1.0于1999年随PHP 4发布，由C语言开发且经过高度优化，并能够做为PHP的后端模块使用。Zend Engine为PHP提供了内存和资源管理的功能以及其它的一些标准服务，其高性能、可靠性和可扩展性在促进PHP成为一种流行的语言方面发挥了重要作用。
	
		Zend Engine的出现将PHP代码的处理过程分成了两个阶段：首先是分析PHP代码并将其转换为称作Zend opcode的二进制格式(类似Java的字节码)，并将其存储于内存中；第二阶段是使用Zend Engine去执行这些转换后的Opcode。
	
		三、PHP的Opcode
	
		Opcode是一种PHP脚本编译后的中间语言，就像Java的ByteCode,或者.NET的MSL。PHP执行PHP脚本代码一般会经过如下4个步骤(确切的来说，应该是PHP的语言引擎Zend)：
		1、Scanning(Lexing) —— 将PHP代码转换为语言片段(Tokens)
		2、Parsing —— 将Tokens转换成简单而有意义的表达式
		3、Compilation —— 将表达式编译成Opocdes
		4、Execution —— 顺次执行Opcodes，每次一条，从而实现PHP脚本的功能
	
			扫描-->分析-->编译-->执行
	
		四、php的加速器
	
		基于PHP的特殊扩展机制如opcode缓存扩展也可以将opcode缓存于php的共享内存中，从而可以让同一段代码的后续重复执行时跳过编译阶段以提高性能。由此也可以看出，这些加速器并非真正提高了opcode的运行速度，而仅是通过分析opcode后并将它们重新排列以达到快速执行的目的。
	
		常见的php加速器有：
	
		1、APC (Alternative PHP Cache)
		遵循PHP License的开源框架，PHP opcode缓存加速器，目前的版本不适用于PHP 5.4。项目地址，http://pecl.php.net/package/APC。
	
		2、eAccelerator
		源于Turck MMCache，早期的版本包含了一个PHP encoder和PHP loader，目前encoder已经不在支持。项目地址， http://eaccelerator.net/。
	
		3、XCache
		快速而且稳定的PHP opcode缓存，经过严格测试且被大量用于生产环境。项目地址，http://xcache.lighttpd.net/
	
		4、Zend Optimizer和Zend Guard Loader
		Zend Optimizer并非一个opcode加速器，它是由Zend Technologies为PHP5.2及以前的版本提供的一个免费、闭源的PHP扩展，其能够运行由Zend Guard生成的加密的PHP代码或模糊代码。 而Zend Guard Loader则是专为PHP5.3提供的类似于Zend Optimizer功能的扩展。项目地址，http://www.zend.com/en/products/guard/runtime-decoders
	
		5、NuSphere PhpExpress
		NuSphere的一款开源PHP加速器，它支持装载通过NuSphere PHP Encoder编码的PHP程序文件，并能够实现对常规PHP文件的执行加速。项目地址，http://www.nusphere.com/products/phpexpress.htm
	
		五、PHP源码目录结构
	
		PHP的源码在结构上非常清晰。其代码根目录中主要包含了一些说明文件以及设计方案，并提供了如下子目录：
	
		1、build —— 顾名思义，这里主要放置一些跟源码编译相关的文件，比如开始构建之前的buildconf脚本及一些检查环境的脚本等。
		2、ext —— 官方的扩展目录，包括了绝大多数PHP的函数的定义和实现，如array系列，pdo系列，spl系列等函数的实现。 个人开发的扩展在测试时也可以放到这个目录，以方便测试等。
		3、main —— 这里存放的就是PHP最为核心的文件了，是实现PHP的基础设施，这里和Zend引擎不一样，Zend引擎主要实现语言最核心的语言运行环境。
		4、Zend —— Zend引擎的实现目录，比如脚本的词法语法解析，opcode的执行以及扩展机制的实现等等。
		5、pear —— PHP 扩展与应用仓库，包含PEAR的核心文件。
		6、sapi —— 包含了各种服务器抽象层的代码，例如apache的mod_php，cgi，fastcgi以及fpm等等接口。
		7、TSRM —— PHP的线程安全是构建在TSRM库之上的，PHP实现中常见的*G宏通常是对TSRM的封装，TSRM(Thread Safe Resource Manager)线程安全资源管理器。
		8、tests —— PHP的测试脚本集合，包含PHP各项功能的测试文件。
		9、win32 —— 这个目录主要包括Windows平台相关的一些实现，比如sokcet的实现在Windows下和*Nix平台就不太一样，同时也包括了Windows下编译PHP相关的脚本。

