# Python的List的实现
https://www.jianshu.com/p/J4U6rR

C实现

1. list申请内存空间的大小(allocated) 通常大于 list实际存储元素所占空间的大小(ob_size)，也可能等于，小于时才会重新申请新的内存空间。这是为了避免每次有新元素加入list时都要调用realloc进行内存分配。

2. 我们想要在list中追加一个整数:L.append(1)，假如新的长度大于已申请的内存，会调用list_resize()，list_resize()会申请多余的空间以避免调用多次list_resize()函数，list增长的模型是:0, 4, 8, 16, 25, 35, 46, 58, 72, 88, …

3. 当弹出list的最后一个元素：L.pop()时。调用listpop()，如果这时ob_size(新的list长度)小于allocated（已经申请的内存空间）的一半，list_resize在函数listpop()内部被调用。这时申请的内存空间将会缩小。

---
我们能看到 Python 设计者的苦心。在需要的时候扩容,但又不允许过度的浪费,适当的内存回收是非常必要的。
这个确定调整后的空间大小算法很有意思，调整后的空间肯定能存储 newsize 个元素。要关注的是预留空间的增长状况。

> #### 当新元素数量 (newsize) 小于已申请内存空间大小 (allocated) 的**一半**时，需要调整申请的内存空间大小
#### 调整后大小 (new_allocated) = 新元素数量 (newsize) + 预留空间 (new_allocated)
#### 预留算法:(newsize // 8) + (newsize < 9 and 3 or 6)。

当 newsize >= allocated,自然按照这个新的长度 "扩容" 内存。
而如果 newsize < allocated,且利用率低于一半呢?
allocated    newsize       new_size + new_allocated
10           4             4 + 3
20           9             9 + 7
很显然,这个新长度小于原来的已分配空间长度,自然会导致 realloc 收缩内存。(不容易啊)
引自《深入Python编程》