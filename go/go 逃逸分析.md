# 什么是逃逸分析？
本来在栈上分配的内容，但是因为需要在栈上返回给上层调用者。所以编译器会选择在堆上分配；这样带来的一个问题是gc耗时可能会比较高。

Go manages memory allocation automatically. 编译器决定变量分配在栈还是堆上.当栈上的变量被分配在堆上面后,带来性能问题.

** stack allocation is cheap and heap allocation is expensive. **

Go allocates memory in two places: a global heap for dynamic allocations and a local stack for each goroutine. Go prefers allocation on the stack — most of the allocations within a given Go program will be on the stack. It’s cheap because it only requires two CPU instructions: one to push onto the stack for allocation, and another to release from the stack.

** Stack allocation requires that the lifetime and memory footprint of a variable can be determined at compile time.
**

# 逃逸分析的用处（为了性能）
最大的好处应该是减少gc的压力，不逃逸的对象分配在栈上，当函数返回时就回收了资源，不需要gc标记清除。
因为逃逸分析完后可以确定哪些变量可以分配在栈上，栈的分配比堆快，性能好
同步消除，如果你定义的对象的方法上有同步锁，但在运行时，却只有一个线程在访问，此时逃逸分析后的机器码，会去掉同步锁运行。


# 打开逃逸分析的开关
go build --gcflags "-m -l" 
// 一般会同时打开 -m //编译优化提示选项、 -l：去掉内联优化选项


# 什么时候逃逸、什么时候不逃逸？

---
The basic rule is that if a reference to a variable is returned from the function where it is declared, it “escapes” - it can be referenced after the function returns, so it must be heap-allocated. This is complicated by:

functions calling other functions
references being assigned to struct members
slices and maps
cgo taking pointers to variables


多亏了 & 操作符，return 告诉你 u 被分享给调用者，因此，已经逃逸到堆中。记住，当你读代码的时候，指针是为了共享，& 操作符对应单词 "sharing"。这在提高可读性的时候非常有用，这（也）是你不想失去的部分。





逃逸分析相关介绍blog: (经典blog、必读)
https://www.ardanlabs.com/blog/2018/01/escape-analysis-flaws.html
https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-escape-analysis.html
https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html
https://www.ardanlabs.com/blog/2017/06/language-mechanics-on-memory-profiling.html
https://www.ardanlabs.com/blog/2018/01/escape-analysis-flaws.html
https://docs.google.com/document/d/1CxgUBPlx9iJzkz9JWkb6tIpTe5q32QDmz8l0BouG0Cw/edit#


http://www.agardner.me/golang/garbage/collection/gc/escape/analysis/2015/10/18/go-escape-analysis.html
https://www.reddit.com/r/golang/comments/badeql/golang_memory_escape_analysis_is_naive/

https://studygolang.com/articles/10026