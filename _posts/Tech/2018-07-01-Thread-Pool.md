---
layout: post
title:  "Thread Pool in C++"
date:   2018-07-01 16:00:00
tags: [thread pool, thread, tech]
categories: Tech
---

> 线程池：一组预先创建的线程，有个管理员来调度这些线程，完成交给它的任务
> 优势：提升任务处理的效率，消除线程创建和销毁的时间，在任务处理时间比较短的时候好处显著
> [reference1](https://blog.csdn.net/jcjc918/article/details/50395528) [reference2](https://blog.csdn.net/u010090316/article/details/71159730)
> [python example](https://www.jianshu.com/p/afd9b3deb027)

### 1. 主要数据结构
1. 线程池管理器ThreadPool：创建线程池，销毁线程池，添加新任务
2. 工作线程Worker：线程池中线程，在没有任务时处于等待状态，可以循环的获取并执行任务
3. 任务接口Task/Job：规定了任务的入口，任务执行完后的收尾工作，任务的执行状态等
4. 任务队列TaskQueue：缓冲，存放待处理任务，存放任务有个数上限

![thread_pool](/res/thread_pool.png)

### 2. Python example
{% highlight Python %}
# 任务接口
class Task():
    def __init__(self, in_data):
        self.in_data = in_data
        self.cv = Condition()
        self.out_data = []

# 工作线程Worker
class ThreadWorker(Thread):
    def __init__(self, task_queue):
        Thread.__init__(self)
        self.task_queue = task_queue  # 任务队列TaskQueue
        self.daemon = True  # True means quit after Main-thread quit
    def run(self):
        # 线程开干
        while True:
            try:
                task = self.task_queue.get()  # 阻塞等活干

                out = process(task.in_data)  # 开始干活
                # write back
                task.out_data = out

                task.cv.acquire()   # 通知Main thread自己干完了，可以取结果了
                task.cv.notify()
                task.cv.release()

                self.task_queue.task_done()  # 通知queue干完了
            except BaseException as err:
                logger.error(err)

# 线程池管理器ThreadPool
class ThreadPool():
    def __init__(self, thread_num):
        self.task_queue = Queue()  # thread safe
        self.thread_num = thread_num
        for i in range(thread_num):
            thread = ThreadWorker(self.task_queue)
            thread.start()
    # 添加task的接口
    def add_job(self, task):
        self.task_queue.put(task)

# 任务发布线程
class MThread(threading.Thread):
    def __init__(self, thread_pool, in_data):
        self.thread_pool = thread_pool
        self.in_data = in_data

    def run(self):
        job = Task(self.in_data)
        self.thread_pool.add_job(job)
        # 等结果
        job.cv.acquire()
        job.cv.wait()
        # write back
        return job.out_data

if __name__ == '__main__':
    thread_pool = ThreadPool(CONCURRENT_NUM)
    threadList = []
    for idx in range(10):  # 10个任务发布线程
        threadList.append(MThread(thread_pool, data[idx]))
    for idx in range(10):
        threadList[idx].start()
{% endhighlight %}

### 3. RAII机制, [lock_guard](https://blog.csdn.net/10km/article/details/49847271)
{% highlight C++ %}
std::mutex mtx;
std::lock_guard<std::mutex> lck (mtx);
{% endhighlight %}
