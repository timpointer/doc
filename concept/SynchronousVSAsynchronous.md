# 同步与异步

同步指多个任务不能同时执行，异步是多个任务能同时执行。
同时指的是并发，而不关心是否并行。
同步指前后两个任务有依赖，必须先执行前面的任务才能开始执行后面的任务。

# 堵塞与非堵塞

同步是从任务的角度来看，堵塞是从CPU的角度来看。
从任务的角度来看thread的执行的是同步的，但如果底层实现用的是轮训，则此时从CPU的角度来看，thread一直在被调度中，所有并没有堵塞（即放弃被调度）。




- <https://stackoverflow.com/questions/748175/asynchronous-vs-synchronous-execution-what-does-it-really-mean>
- <https://developer.ibm.com/technologies/linux/articles/l-async/>
- <https://stackoverflow.com/questions/2625493/asynchronous-vs-non-blocking>