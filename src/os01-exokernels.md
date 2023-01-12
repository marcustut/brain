# Exokernels

The concept was first introduced by Dawson R. Engler in his 1995 paper [Exokernel: An Operating System Architecture for Application-Level Resource Management](https://people.eecs.berkeley.edu/~kubitron/courses/cs262a-F19/handouts/papers/engler95exokernel.pdf), it was a paradigm shift in which we normally think about operating systems. He claims that abstractions in traditional operating systems are expensive and restricts domain specific optimisations.

The idea of an *exokernel* is to provide a thin layer on top of raw hardware that does two things: **secure bindings** and **multiplexes resources**. As for abstractions such as filesystems, process scheduling, etc. would be offloaded to application-level libraries called *LibOS*. The diagram below depicts an implementation of the exokernel architecture.

![Exokernel Architecture](images/exokernel-architecture.png)

<div align="center"><i>Figure 1.1.1: Exokernel Architecture</i></div>

As can be seen from the diagram, abstractions such as *POSIX*[^posix] or *TCP*[^tcp] lives in the user-space and they are basically plug-and-play by different applications. In other words, developers are free to write their own abstractionsand this opens up opportunity for building highly customised abstractions tailored to domain-specific use cases and machine capabilities.

[^posix]: [Portable Operating System Interface (POSIX)](https://en.wikipedia.org/wiki/POSIX) is a family of standards specified by the IEEE Computer Society for maintaining compatibility between operating systems.

[^tcp]: [Transmission Control Protocol (TCP)](https://en.wikipedia.org/wiki/Transmission_Control_Protocol) is one of the main protocols of the Internet protocol suite. It originated in the initial network implementation in which it complemented the Internet Protocol. Therefore, the entire suite is commonly referred to as TCP/IP.

## Secure bindings

The motivation behind secure bindings is to implement protection by guarding each resources and this is very crucial for resource multiplexing to be implemented securely. *LibOS* can bind to resources without worrying about resource competition through secure bindings provided by the *exokernel*. 

To put it in simple words, secure bindings is a protection mechanism that decouples authorization from the actual use of a resource. Application-level softwares defines how to use the resources while the kernel is responsible for protecting these resources, and because of secure bindings, the kernel can protect these resources wihtout understanding how they are being used. Because of this, the process is usally computationally cheap and does not introduce a lot of overhead. 

One of the core concepts is **ownership** where each application owns a specific resource and no processes will be able to access resources that they do not own. The job of the kernel here is simply associating ownership tags between applications and resources. This can be done in a straightforward way by introducing a software cache layer such as a software *TLB*[^tlb].

[^tlb]: [Translation Lookaside Buffer (TLB)](https://en.wikipedia.org/wiki/Translation_lookaside_buffer) is a memory cache that stores the recent translations of virtual memory to physical memory.

## Multiplexes resources

### Multiplexing physical memory

Due to the implementation of *secure bindings*, multiplexing memory isn't a difficult task. Because the kernel can leverage the idea of ownership to assign different regions of memory to different applications without worrying they overlap with each other. However, the assumption here is that there will only be one actor to this task which is the kernel, which means the page table must be read-only at application level.

In the case where an application drop its ownership for a particular region of memory, the kernel needs to break the secure binding by changing the ownership association and mark the resource as free. Subsequently, it cancels any queued DMA[^dma] requests to that freed region of memory.

[^dma]: [Direct Memory Access (DMA)](https://en.wikipedia.org/wiki/Direct_memory_access) is a feature of computer systems and allows certain hardware subsystems to access main system memory independently of the central processing unit (CPU).

### Multiplexing network

Multiplexing network on the other hand would be much more challenging since protocol-specific knowledge is required to interpret incoming messages and identify the intended recipient. One such method to do this is through **packet filtering**[^pf]. However, the logic for filtering packets can change overtime due to implementation of new protocols and one smarter way to handle them is through *Dynamic Packet Filtering (DPF)*. The specifics of DPF will not be covered here, it can be found in the paper by Engler but the gist of it is that the code for filtering logic would be downloaded into the kernel.

[^pf] [Packer filtering](https://www.techopedia.com/definition/4038/packet-filtering) is a technique used to control network access by monitoring outgoing and incoming packets and allowing them to pass or halt based on the source and destination Internet Protocol (IP) addresses, protocols and ports.

## Difference between hypervisors

*TODO*
At first glance, *exokernel* seems to resembles the purpose of a hypervisor on a high-level but ... 


## Advantages of being close to the hardware

The performance for highly optimised systems would be incredible since they are no longer bottlenecked by the software layer. This would be beneficial to applications that run in isolation such as supercomputers, or highly-customised server-side applications. 

Training machine learning models would make a good use case as it usually needs a lot of hardware resources and normally does not interact with the outside world except the initial stage where data is fed. Other than that, utilising *exokernel* for quantum computers would really make sense as developers would want to leverage the maximum capacity of the CPU and normally quantum computers are used for very specific computing such as cryptography.

In short, it would be good for anything that doesn't require general purpose computing.

## What's the catch? 

Almost everything comes with a compromise and there is no exception for *exokernel*. The biggest issue would be the lack of common standards and this makes even doing simple things such as opening a file difficult. That is why *exokernel* would be a bad idea for general-purpose operating system.

In addition, different *LibOS* would have different implementations and the result is cross-platform development would be very difficult if not impossible. To give a counterexample, because most devices runs Linux nowadays such as android phones, laptops, as such. Their filesystems abstraction is the similar and would allow for reusable application code.

## OSes that uses exokernels

Exokernel is a less popular concept and thus it has been used in less number of OSes as compared to traditional monolithic or microkernels. However, some notable research projects that have implemented exokernel-based operating systems include:

- ExOS: an exokernel operating system developed at MIT that was used as a testbed for research on exokernels and other low-level systems topics.

- ExoKernel: an exokernel operating system developed at Berkeley that aims to provide high performance and flexibility for network-based services.

- K42: an operating system developed at Harvard that uses an exokernel architecture to provide a high-performance, low-overhead environment for large-scale data centers.

- Flask: a security-oriented exokernel developed at the University of Utah that aims to provide flexible, fine-grained security mechanisms.

These systems were developed as research projects and may not be widely used in production environments, as the adoption of exokernels as a mainstream concept is less popular.

### Interesting use cases for *exokernel*

Due to the fact that components are loosely coupled, it is easier to test new OS-level features such as filesystems, scheduling techniques, etc. Because these abstractions can be implemented at the *LibOS* level, which is easily swappable and has a lower chance of crashing the kernel.

## Comparison with other type of kernels

### Microkernels

Both *exokernel* and *microkernel* relate to each other in a pretty similar way in which they both aim to make operating system more modular and secure, however they take a very different approach in doing so. 

In a *microkernel*, there are still abstractions implemented in the kernel space such as memory management, process management and inter-process communication (IPC). These are considered as *essential services* by a microkernel and other *non-essential services* such as filesystems and network protocols will be implemented at the user-space. On the other hand, an *exokernel* does not implement any abstractions at all, which means even *essential services* such as memory management lives at the user-space. Because of that, we can say *exokernel* is much more "low-level" in terms of developers' freedom in accessing hardware resources. 

In summary, both *microkernel* and *exokernel* tries to achieve the same goal but *microkernel* does it by separating the kernel from other system services and connecting them through a small set of communication interfaces whereas *exokernel* exposes direct access of resources to the users through a very thin kernel layer and allow the users to manage these resources themselves.

- A microkernel example ⮕ [RedoxOS](https://doc.redox-os.org/book/ch01-04-how-redox-compares.html#the-kernel)

### Monolithic kernels

TODO

## Related Links 🔗

- [🌐 Exokernel - OSDev Wiki](https://wiki.osdev.org/Exokernel)
- [📚 Exokernel: An Operating System Architecture for Application-Level Resource Management](https://people.eecs.berkeley.edu/~kubitron/courses/cs262a-F19/handouts/papers/engler95exokernel.pdf)
- [🎥 Video Summary of the paper: "Exokernel: An Operating System Architecture for Application-Level Resource Management"](https://youtu.be/TZB60F5kNSk)
- [🖼 Lecture Slides from MIT](https://pdos.csail.mit.edu/archive/exo/exo-slides/sld001.htm)
