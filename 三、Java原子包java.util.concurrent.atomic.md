# 三、Java原子操作包

首先我们先看看`Atomic`包的描述：

**`Package java.util.concurrent.atomic`**

> A small toolkit of classes that support lock-free thread-safe programming on single variables.
>
> 一个小的类工具包，支持对单个变量进行无锁线程安全编程。

显然`atomic`包中的类都是为单个变量提供线程安全的操作（原子操作）。