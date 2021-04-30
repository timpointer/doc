# Why Data Structure Matter
## 如何对比性能？
不对比物理时间，只比对算法的步骤，因为物理时间依赖的硬件，不同的硬件相同的算法，执行的时间会不一样。
如果一个算法用1步，另一个用两步，则认为第一个比第二个算法快。

Array是一段连续的地址空间,每个地址空间里面包含一个值。
定义四个基本操作，read, search, insert, delete

read 根据地址来读取地址里面的值。
search 通过一个值来查找array中包含这个值的地址，正好和read相反
insert 往array里面插入一个值
delete 从array中删除一个值

## Array的性能
read  O(1) //因为计算机可以直接通过地址拿到里面包含的值。(参考内存是如何实现的)
search O(n) //需要比对array中的每一个值，当值不在array中，就要遍历所有值
insert O(n) // 移动n步，插入一步
delete O(n) // 删除一步，移动n步

## Set的性能
这里指定的Set是基于Array实现的。在Array的基础上加了一个限制，不能插入array中已有的值。
我们来看下为了满足的这个限制，付出的性能是什么？

read  O(1) //因为计算机可以直接通过地址拿到里面包含的值。(参考内存是如何实现的)
search O(n) //需要比对array中的每一个值，当值不在array中，就要遍历所有值
insert O(n) // 搜索n步，移动n步，插入一步
delete O(n) // 删除一步，移动n步

唯一的不同点在于insert操作，在insert之前，我们先要执行search操作来确保array中没有相同的值，所以多做了一步search操作，最终是n+n+1步，比array多了n步。

不同的抽象数据结构，有不同的限制和性能，需要根据具体情况做权衡。
