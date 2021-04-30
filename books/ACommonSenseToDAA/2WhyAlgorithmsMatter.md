# Why Algorithms Matter

算法就是完成一件事的步骤，比如做一个早餐
1. 拿出一个碗
2. 加入牛奶
3. 加入麦片
4. 用勺子搅拌一下

做一件事，可以有不同的算法，不同的算法的效率是不一样的，一些步骤多一些步骤少。

## Orderd Array
有序数组，在数组前加了一个定语，也就是限制，组数内部必须要保持有序。

insert O(n)// 先search对比，然后移动，在插入  n+2步 

对比Array，insert时多了对比的步骤。

Search - 线性搜索算法 O(n) 

对比Array, search时步骤更少，对比到比值大的值，就可以终止查询。

OrderedArray对比Array的最大优势在于，可以采用其他的搜索算法，比如binary search。

## Binary Search
// TODO

记住OrderedArray不是在每个方面都更快，insert比较慢，但search比较快。
看场景，应用是写比较频繁，还是查询比较频繁，看那种算法更合适。

值得注意的是，我们已经把binary search加入了我们的工具包。因为insert依赖search，所以我们可以把插入时的search替换为binary search。但就算这样，insert还是比经典的array要慢，因为总是多了search步骤。

# 总结
完成可以任务，你可以选择不同的算法。为了更好的分析算法，我们需要一个通用语言来表达数据结构和算法的复杂度。








