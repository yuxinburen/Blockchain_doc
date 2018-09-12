1.Go中goruntime之间进行通信有几种方法？

1. 共享内存方式。最常见的方式即是在主goruntime中声明一个变量，在新开的goruntime中进行变量的访问和修改等逻辑操作，这就是通过共享内存方式来实现通信。共享内存方式在使用时要注意，大多数情况下需要引入锁，即lock变量来操作上锁和释放锁的操作，分为为Lock()和UnLock()。多协程共同访问一个变量，会有协程安全问题。

2. Go语言中主要采用消息机制来进行各个Goruntine之间的通信，又称作channel。消息机制认为每个并发单元是自包含的，独立的个体，并且都有自己的变量，但在不同并发单元间这些变量不共享。每个并发单元的输入和输出只有一种，那就是消息。channel是Go语言在语言级别提供的goruntine间的通信方式，我们可以使用channel在多个goroutine之间传递消息。channel是进程内的通信方式，因此通过channel传递对象的过程和调用函数时的参数传递行为比较一致，比如也可以传递指针等。channel是类型相关的，一个channel只能传递一种类型的值，这个类型需要在声明channel时进行指定。

3. 同步锁。Go语言中的sync包提供了两种锁类型：sync.Mutex和sync.RWMutex，前者是互斥锁，后者是读写锁。使用锁的经典模式：

   ```
   var lck sync.Mutext
   func foo(){
       lck.Lock()
       defer lck.Unlock()
       //...
   }
   ```

   lck.Lock()会阻塞直到获取锁，然后利用defer语句在函数返回时自动释放锁。

2.