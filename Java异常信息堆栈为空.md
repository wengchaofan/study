

#### 经过

最近现场发回来的异常信息，堆栈信息为空，无法判断异常是从哪里抛出来的。经过一番研究，原因如下：

JIT编译器对异常进行了优化，当代码中的某个位置抛出同一个异常很多次后,JIT服务端编译器(C2)会将其优化成抛出一个事先编译好的、类型匹配的异常,异常堆栈信息就看不到了。

HotSpot VM有个许多人觉得“匪夷所思”的优化，叫做fast throw：有些特定的隐式异常类型（NullPointerException、ArithmeticException（ / 0）之类）如果在代码里某个特定位置被抛出过多次的话，HotSpot Server Compiler（C2）会透明的决定用fast throw来优化这个抛出异常的地方——直接抛出一个事先分配好的、类型匹配的异常对象。这个对象的message和stack trace都被清空。抛出这个异常的速度是非常快，不但不用额外分配内存，而且也不用爬栈；但反面就是可能正好是需要知道哪里出问题的时候看不到stack trace了。从Sun JDK5开始要避免C2做这个优化还得额外传个VM参数：-XX:-OmitStackTraceInFastThrow。

#### 问题重现

代码测试如下：

![execption](image\execption.png)

#### 解决

```
-XX:-OmitStackTraceInFastThrow
```

这个虚拟机参数来告诉JIT编译器禁用这种异常fastThrow的优化,当然如果你使用-Xint参数后虚拟机运行在解释器模式也不会出现这个问题，但是禁用JIT会对整体的性能有影响,因此不建议使用-Xint参数,如果想看到具体的异常堆栈,推荐使用 `-XX:-OmitStackTraceInFastThrow` 参数。






