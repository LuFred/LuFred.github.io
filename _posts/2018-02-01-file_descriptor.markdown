---
layout: post
title: "Unix描述符"
subtitle: ""
date: 2018-02-01
author: LuJiangBo
tags: Linux TCP/IP
keyword: Linux,Unix,TCP/IP
finished: true
---

描述符(File descriptor)是计算机科学中的一个术语，是一个用于表述指向文件的引用的抽象化概念。

## 介绍

在UNIX/Linux系统中,不管是设备、管道、目录、插口等统统都与文件有关(一句话:一切皆文件)，既然一切皆文件，那就需要有一个标志来指向这些文件。这就是这里要说的**描述符**(很多也称为文件描述符)。  
简称**fd**,**fd**通常是一个很小的非负整数，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个fd。
  

## 结构

系统的每一个进程在其生存期内都会有一个进程表项(也称文件表项)存在，该进程表项维护的就是该进程所打开的文件数组，也就是说每一个描述符其实就是进程表项中的一个数组的小标，这个数组项指向一个打开文件表的结构，而这个结构又指向一个描述此文件的i-node或者v-node结构.
* i-node:可以理解为一个文件索引,i节点中存放了文件信息：文件所有者、文件类型、文件访问权限位、文件长度、该文件所占用的数据块(可能有多个)的指针等，据此就可以访问磁盘上存放文件内容的数据块了.
* v-node:每个打开的文件都有一个v-node结构，v-node包含了文件i-node信息，这些信息是在文件打开时从磁盘上读入内存的，因此文件的相关信息是可以快速访问的.
![unix进程结构]({{ post.url| prepend: site.url  }}/content/images/201802/file_descriptor01.png)
上图显示了一个涉及socket的描述符，其中proc{}表示一个进程对象，它的具体结构如下：
![unix proc结构]({{ post.url| prepend: site.url  }}/content/images/201802/file_descriptor02.png)
这张代码截图取自4.4BSD-Lite2源码[proc.h](https://github.com/sergev/4.4BSD-Lite2/blob/master/usr/src/sys/sys/proc.h)的部分截图。  
当进程执行了一个系统调用，如socket,内核就会访问进程表结构。在这个结构中的项**p_fd**指向进程的[**filedesc**](https://github.com/sergev/4.4BSD-Lite2/blob/master/usr/src/sys/sys/filedesc.h)结构。
![filedesc结构]({{ post.url| prepend: site.url  }}/content/images/201802/file_descriptor03.png)
在这个结构中有两项是现在值得关注的:
* 一个是**fd_ofileflags**,它是一个字符数组指针(每一个描述符有一个描述符标志)。
* 一个是**fd_ofiles**,它是一个指向文件表结构的指针数组的指针。  
**fd_ofileflags**和**fd_ofiles**是一一对应关系，即例如**fd_ofileflags[3]**和**fd_ofiles[3]**表示该进程打开的第三个文件的描述符标志和对应的文件结构体地址。这里**fd_ofiles**指向的数据结构用***file{}[]**表示，它是一个指向**file**结构的指针数组。这个数组及描述符标志数组的下标就是描述符本身:0,1,2等。
![file结构]({{ post.url| prepend: site.url  }}/content/images/201802/file_descriptor04.png)
结构[**file**](https://github.com/sergev/4.4BSD-Lite2/blob/master/usr/src/sys/sys/file.h)表示当前进程中一个打开的文件，该结构的主要成员作用如下：
* **f_type**:指示描述符的类型是**DTYPE_SOCKET**和**DTYPE_VNODE**。v-node是一个通用机制，允许内核支持不同类型的文件系统————磁盘文件系统、网络文件系统(如NFS)、CD_ROM文件系统、基于存储器的文件系统等等。
* **f_data**:指向一个socket结构或者一个vnode结构，它是根据描述符类型来决定。
* **f_ops**:指向一个有5个函数指针的向量,这些函数指针用在 read、 readv、write、 writev、 ioctl、 select和close系统调用中.这些系统调用每次被调用时都要查看**f_type**的值， 然后做出相应的跳转。
*  **f_type**:打开文件的类型
    * #define   DTYPE_VNODE     1     /*file */  表示打开的文件是一个普通的文件
    * #define   DTYPE_SOCKET    2     /*communications endpoint */  表示打开的文件是一个socket
* **f_data**:对应了该打开文件所对应的数据部分，对应vnode或者socket结构体
    >  当打开的文件类型为普通文件时(f_type值为DTYPE_VNODE)，f_data指向了一个struct vnode结构体，  
    当打开的文件类型为socket时(f_type值为DTYPE_SOCKET)，f_data指向了一个[struct socket结构体](https://github.com/sergev/4.4BSD-Lite2/blob/master/usr/src/sys/sys/socketvar.h)，  
    其中在socket结构中需要关注short so_type和caddr_t so_pcb两个成员变量，其中:so_type表示socket类型，例如SOCK_DGRAM表示UDP类型，SOCK_STREAM表示TCP类型；so_pcb指向一个协议控制块的双向链表。    
       
    > 协议控制块[so_pcb](https://github.com/sergev/4.4BSD-Lite2/blob/master/usr/src/sys/netinet/in_pcb.h)在socket结构体中，它是一个双向链表形式的数据结构，主要成员包括：前一个inpcb、后一个inpcb、inpcb双向链表的首节点、当前inpcb对应socket的本地IP、本地端口、远端IP、远端端口、当前inpcb对应的socket结构体的地址等等；  
    每个socket都有一个协议控制块inpub与之对应：可通过socket的so_pcb成员来访问它，同时协议控制块中也有个成员socket *inp_socket用于指向自己所属的socket结构体。在OS中，每种类型(如TCP/UDP)的socket有且只有一个inpcb链表与之对应。  
    例如：所有的TCP的socket的inpcb都在同一个TCP的inpcb双向链表中，所有的UDP的socket的inpcb都在同一个UDP的inpcb双向链表中。  
    在通过socket接收数据时，OS从网卡驱动中拿到数据后，先搜索inpcb的双向链表，通过比对本地ip地址、本地端口号、远端ip地址、远端端口号找到匹配的inpcb，进而找到对应socket，并将数据保存到socket的接收缓存中。

![socket调用后的内核数据结构]({{ post.url| prepend: site.url  }}/content/images/201802/file_descriptor05.png)
上图是取自《TCP/IP详解 卷2》的一张socket调用后的内核数据结构图，结合改图应该能稍微容易理解上面介绍的各个结构之间的相互调用关系。

## 其它

* 在UNIX/Linux平台上,标准输入，标准输出，标准错误输出也对应了三个文件描述符。它们分别是0,1,2。当然在实际编程中你也可以修改这3个描述符的值，虽然没有严格的要求，但是改了可能会出现意想不到的错误也说不定(大家都默认012，你非要改掉那出个bug就不好说了)

## 参考

* [作]史蒂文斯,[译]陆雪莹.TCP/IP详解 卷2：实现[M]：机械工业出版社，2004：7