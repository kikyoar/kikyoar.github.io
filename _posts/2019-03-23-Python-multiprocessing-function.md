---
layout:     post
title:      "Python多进程编程"
subtitle:   "multiprocessing详解"
date:       2019-03-23
author:     "kikyoar"
header-img: "img/post-bg-python-version.jp"
tags:
    - Python
--- 

[摘自博客园](https://www.cnblogs.com/kaituorensheng/p/4445418.html#_label1)  

**multiprocessing支持子进程、通信和共享数据、执行不同形式的同步，提供了Process、Queue、Pipe、Lock等组件**


### Process

- **创建进程的类**：Process([group [, target [, name [, args [, kwargs]]]]])，target表示调用对象，args表示调用对象的位置参数元组。kwargs表示调用对象的字典。name为别名。group实质上不使用  

- **方法**：is_alive()、join([timeout])、run()、start()、terminate()。其中，Process以start()启动某个进程

- **属性**：authkey、daemon（要通过start()设置）、exitcode(进程在运行时为None、如果为–N，表示被信号N结束）、name、pid.其中daemon是父进程终止后自动终止，且自己不能产生新进程，必须在start()之前设置  


**例1.1：创建函数并将其作为单个进程**  
	
	import multiprocessing
	import time
	
	
	def worker(interval):
	    n = 5
	    while n > 0:
	        # 输出当前时间
	        print("The time is {0}".format(time.ctime()))
	        time.sleep(interval)
	        n -= 1
	
	
	if __name__ == "__main__":
	
	    # 创建子进程，传参
	    p = multiprocessing.Process(target=worker, args=(3,))
	    p.start()
	    print("p.pid:" + str(p.pid))
	    print("p.name:", p.name)
	    print("p.is_alive:", p.is_alive())

输出结果：

	p.pid:30760
	p.name: Process-1
	p.is_alive: True
	The time is Sat Mar 23 09:53:58 2019
	The time is Sat Mar 23 09:54:01 2019
	The time is Sat Mar 23 09:54:04 2019
	The time is Sat Mar 23 09:54:07 2019
	The time is Sat Mar 23 09:54:10 2019  
	
**例1.2：创建函数并将其作为多个进程**  

	import multiprocessing
	import time
	
	
	def worker_1(interval):
	    print("worker_1")
	    time.sleep(interval)
	    print("end worker_1")
	
	
	def worker_2(interval):
	    print("worker_2")
	    time.sleep(interval)
	    print("end worker_2")
	
	
	def worker_3(interval):
	    print("worker_3")
	    time.sleep(interval)
	    print("end worker_3")
	
	
	if __name__ == "__main__":
	    p1 = multiprocessing.Process(target=worker_1, args=(2,))
	    p2 = multiprocessing.Process(target=worker_2, args=(3,))
	    p3 = multiprocessing.Process(target=worker_3, args=(4,))
	
	    p1.start()
	    p2.start()
	    p3.start()
	
	    print("The number of CPU is:" + str(multiprocessing.cpu_count()))
	    for p in multiprocessing.active_children():
	        time.sleep(5)
	        print("child   p.name:" + p.name + "\tp.id" + str(p.pid))
	    print("END!!!!!!!!!!!!!!!!!")  
	    
输出结果：

	The number of CPU is:4
	worker_1
	worker_2
	worker_3
	end worker_1
	end worker_2
	end worker_3
	child   p.name:Process-3	p.id30838
	child   p.name:Process-1	p.id30836
	child   p.name:Process-2	p.id30837
	END!!!!!!!!!!!!!!!!!


**例1.3：将进程定义为类**  

`注：进程p调用start()时，自动调用run()`  

	import multiprocessing
	import time
	
	
	class ClockProcess(multiprocessing.Process):
	    def __init__(self, interval):
	        multiprocessing.Process.__init__(self)
	        self.interval = interval
	
	    # 重写run方法
	    def run(self):
	        n = 5
	        while n > 0:
	            print("the time is {0}".format(time.ctime()))
	            time.sleep(self.interval)
	            n -= 1
	
	
	if __name__ == '__main__':
	    p = ClockProcess(3)
	    p.start()

输出结果：

	the time is Sat Mar 23 10:01:30 2019
	the time is Sat Mar 23 10:01:33 2019
	the time is Sat Mar 23 10:01:36 2019
	the time is Sat Mar 23 10:01:39 2019
	the time is Sat Mar 23 10:01:42 2019  
	

**例：daemon程序对比结果**  

- 不加daemon属性  

		import multiprocessing
		import time
		
		
		def worker(interval):
		    print("work start:{0}".format(time.ctime()));
		    time.sleep(interval)
		    print("work end:{0}".format(time.ctime()));
		
		
		if __name__ == "__main__":
		    p = multiprocessing.Process(target=worker, args=(3,))
		    p.start()
		    print("end!")  
		    
	输出结果：
		
		end!
		work start:Sat Mar 23 10:03:31 2019
		work end:Sat Mar 23 10:03:34 2019

	
- 加上daemon属性 

	`注：因子进程设置了daemon属性，主进程结束，它们就随着结束了`   

		import multiprocessing
		import time
		
		
		def worker(interval):
		    print("work start:{0}".format(time.ctime()));
		    time.sleep(interval)
		    print("work end:{0}".format(time.ctime()));
		
		
		if __name__ == "__main__":
		    p = multiprocessing.Process(target=worker, args=(3,))
		    p.daemon = True
		    p.start()
		    print("end!")  
		    
	输出结果：
	
		end!  


- 3 设置daemon执行完结束的方法  

	`注：等子程序结束后，主程序才结束`

		import multiprocessing
		import time
		
		
		def worker(interval):
		    print("work start:{0}".format(time.ctime()));
		    time.sleep(interval)
		    print("work end:{0}".format(time.ctime()));
		
		
		if __name__ == "__main__":
		    p = multiprocessing.Process(target=worker, args=(3,))
		    # 默认为p.daemon = True
		    p.daemon = True
		    p.start()
		    p.join()
		    print("end!")  
		    
		    
	输出结果：  
	
		work start:Sat Mar 23 10:11:50 2019
		work end:Sat Mar 23 10:11:53 2019
		end!


### Lock  

**当多个进程需要访问共享资源的时候，Lock可以用来避免访问的冲突，lock通过参数传到了每个进程中，但是我们知道进程之间是不共享内存的，所以我理解应该是每个进程获得的锁其实是不同的， 所以无法对写文件起到加锁的效果**

	import multiprocessing
	import time
	
	
	def worker_with(lock, f):
	    with lock:
	        fs = open(f, 'a+')
	        n = 10
	        while n > 1:
	            fs.write("Lockd acquired via with" + str(n) + "\n")
	            n -= 1
	        fs.close()
	
	
	def worker_no_with(lock, f):
	    # 加锁
	    lock.acquire()
	    try:
	        fs = open(f, 'a+')
	        n = 10
	        while n > 1:
	            fs.write("Lock acquired directly" + str(n) + "\n")
	            n -= 1
	        fs.close()
	    finally:
	        # 释放锁    
	        lock.release()
	
	
	if __name__ == "__main__":
	    # 创建锁
	    lock = multiprocessing.Lock()
	    f = "file.txt"
	    w = multiprocessing.Process(target=worker_with, args=(lock, f))
	    nw = multiprocessing.Process(target=worker_no_with, args=(lock, f))
	    w.start()
	    nw.start()
	
	    time.sleep(5)
	    with open(f) as file:
	        for fl in file:
	            print(fl)
	    print("end")  
	    
	    
输出结果：

	Lockd acquired via with10
	
	Lockd acquired via with9
	
	Lockd acquired via with8
	
	Lockd acquired via with7
	
	Lockd acquired via with6
	
	Lockd acquired via with5
	
	Lockd acquired via with4
	
	Lockd acquired via with3
	
	Lockd acquired via with2
	
	Lock acquired directly10
	
	Lock acquired directly9
	
	Lock acquired directly8
	
	Lock acquired directly7
	
	Lock acquired directly6
	
	Lock acquired directly5
	
	Lock acquired directly4
	
	Lock acquired directly3
	
	Lock acquired directly2
	
	end


### Semaphore  

`Semaphore用来控制对共享资源的访问数量，例如池的最大连接数`

	import multiprocessing
	import time
	
	
	def worker(s, i):
	    s.acquire()
	    print(multiprocessing.current_process().name + "acquire");
	    time.sleep(i)
	    print(multiprocessing.current_process().name + "release\n");
	    s.release()
	
	
	if __name__ == "__main__":
	    s = multiprocessing.Semaphore(2)
	    for i in range(5):
	        p = multiprocessing.Process(target=worker, args=(s, i * 2))
	        p.start()

输出结果：

	Process-1acquire
	Process-1release
	
	Process-2acquire
	Process-3acquire
	Process-2release
	
	Process-4acquire
	Process-3release
	
	Process-5acquire
	Process-4release
	
	Process-5release   
	
	
### Event  

`Event用来实现进程间同步通信`  

	def wait_for_event(e):
	    print("wait_for_event: starting")
	    e.wait()
	    print("wairt_for_event: e.is_set()->" + str(e.is_set()))
	
	
	def wait_for_event_timeout(e, t):
	    print("wait_for_event_timeout:starting")
	    e.wait(t)
	    print("wait_for_event_timeout:e.is_set->" + str(e.is_set()))
	
	
	if __name__ == "__main__":
	    e = multiprocessing.Event()
	    w1 = multiprocessing.Process(name="block",
	                                 target=wait_for_event,
	                                 args=(e,))
	
	    w2 = multiprocessing.Process(name="non-block",
	                                 target=wait_for_event_timeout,
	                                 args=(e, 2))
	    w1.start()
	    w2.start()
	
	    time.sleep(3)
	
	    e.set()
	    print("main: event is set")  
	    
	    
输出结果：

	wait_for_event: starting
	wait_for_event_timeout:starting
	wait_for_event_timeout:e.is_set->False
	main: event is set
	wairt_for_event: e.is_set()->True  
	
	
### Queue  

Queue的种类：

- FIFO：Queue.Queue(maxsize=0)

  FIFO即First in First Out,先进先出。Queue提供了一个基本的FIFO容器，使用方法很单,maxsize是个整数，指明了队列中能存放的数据个数的上限。一旦达到上限，插入会导致阻塞，直到队列中的数据被消费掉。如果maxsize小于或者等于0，队列大小没有限制  

- LIFO: Queue.LifoQueue(maxsize=0)
	
	LIFO即Last in First Out,后进先出。与栈的类似，使用也很简单,maxsize用法同上

- priority:class Queue.PriorityQueue(maxsize=0)
	
	构造一个优先队列。maxsize用法同上  
	
基本方法：

- Queue.Queue(maxsize=0)   FIFO， 如果maxsize小于1就表示队列长度无限
- Queue.LifoQueue(maxsize=0)   LIFO， 如果maxsize小于1就表示队列长度无限
- Queue.qsize()   返回队列的大小 
- Queue.empty()   如果队列为空，返回True,反之False 
- Queue.full()   如果队列满了，返回True,反之False
- Queue.get([block[, timeout]])   读队列，timeout等待时间 
- Queue.put(item, [block[, timeout]])   写队列，timeout等待时间 
- Queue.queue.clear()   清空队列

其他：  

- task\_done()：意味着之前入队的一个任务已经完成。由队列的消费者线程调用。每一个get()调用得到一个任务，接下来的task_done()调用告诉队列该任务已经处理完毕。如果当前一个join()正在阻塞，它将在队列中的所有任务都处理完时恢复执行（即每一个由put()调用入队的任务都有一个对应的task_done()调用）。

- join()：阻塞调用线程，直到队列中的所有任务被处理掉。只要有数据被加入队列，未完成的任务数就会增加。当消费者线程调用task_done()（意味着有消费者取得任务并完成任务），未完成的任务数就会减少。当未完成的任务数降到0，join()解除阻塞  

Queue是多进程安全的队列，可以使用Queue实现多进程之间的数据传递。put方法用以插入数据到队列中，put方法还有两个可选参数：blocked和timeout。如果blocked为True（默认值），并且timeout为正值，该方法会阻塞timeout指定的时间，直到该队列有剩余的空间。如果超时，会抛出Queue.Full异常。如果blocked为False，但该Queue已满，会立即抛出Queue.Full异常。
 
get方法可以从队列读取并且删除一个元素。同样，get方法有两个可选参数：blocked和timeout。如果blocked为True（默认值），并且timeout为正值，那么在等待时间内没有取到任何元素，会抛出Queue.Empty异常。如果blocked为False，有两种情况存在，如果Queue有一个值可用，则立即返回该值，否则，如果队列为空，则立即抛出Queue.Empty异常。Queue的一段示例代码：


	import multiprocessing
	
	
	def writer_proc(q):
	    try:
	        q.put(1, block=False)
	    except:
	        print("writer")
	
	
	def reader_proc(q):
	    try:
	        print(q.get(block=False))
	    except:
	        print("reader")
	
	
	if __name__ == "__main__":
	    q = multiprocessing.Queue()
	    writer = multiprocessing.Process(target=writer_proc, args=(q,))
	    writer.start()
	
	    reader = multiprocessing.Process(target=reader_proc, args=(q,))
	    reader.start()
	
	    reader.join()
	    writer.join()


### Pipe  

Pipe方法返回(conn1, conn2)代表一个管道的两个端。Pipe方法有duplex参数，如果duplex参数为True(默认值)，那么这个管道是全双工模式，也就是说conn1和conn2均可收发。duplex为False，conn1只负责接受消息，conn2只负责发送消息  
 
send和recv方法分别是发送和接受消息的方法。例如，在全双工模式下，可以调用conn1.send发送消息，conn1.recv接收消息。如果没有消息可接收，recv方法会一直阻塞。如果管道已经被关闭，那么recv方法会抛出EOFError  
	
	import multiprocessing
	import time
	
	
	def proc1(pipe):
	    while True:
	        for i in range(10000):
	            print("send: %s" % (i))
	            pipe.send(i)
	            time.sleep(1)
	
	
	def proc2(pipe):
	    while True:
	        print("proc2 rev:" + str(pipe.recv()))
	        time.sleep(1)
	
	
	def proc3(pipe):
	    while True:
	        print("PROC3 rev:" + str(pipe.recv()))
	        time.sleep(1)
	
	
	if __name__ == "__main__":
	    pipe = multiprocessing.Pipe()
	    p1 = multiprocessing.Process(target=proc1, args=(pipe[0],))
	    p2 = multiprocessing.Process(target=proc2, args=(pipe[1],))
	
	    p1.start()
	    p2.start()
	
	    p1.join()
	    p2.join()

输出结果：

	send: 0
	proc2 rev:0
	send: 1
	proc2 rev:1
	send: 2
	proc2 rev:2
	send: 3
	proc2 rev:3
	send: 4
	proc2 rev:4
	send: 5
	proc2 rev:5
	send: 6
	proc2 rev:6  
	
### Pool  

在利用Python进行系统管理的时候，特别是同时操作多个文件目录，或者远程控制多台主机，并行操作可以节约大量的时间。当被操作对象数目不大时，可以直接利用multiprocessing中的Process动态成生成多个进程，十几个还好，但如果是上百个，上千个目标，手动的去限制进程数量却又太过繁琐，此时可以发挥进程池的功效。**Pool可以提供指定数量的进程，供用户调用，当有新的请求提交到pool中时，如果池还没有满，那么就会创建一个新的进程用来执行该请求；但如果池中的进程数已经达到规定最大值，那么该请求就会等待，直到池中有进程结束，才会创建新的进程来它**  

- 使用进程池（非阻塞）

		import multiprocessing
		import time
		
		
		def func(msg):
		    print("msg:" + msg)
		    time.sleep(3)
		    print("end")
		
		
		if __name__ == "__main__":
		    pool = multiprocessing.Pool(processes=3)
		    for i in range(4):
		        msg = "hello" + str(i)
		        pool.apply_async(func, (msg,))  # 维持执行的进程总数为processes，当一个进程执行完毕后会添加新的进程进去
		
		    print("Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~")
		    pool.close()
		    pool.join()  # 调用join之前，先调用close函数，否则会出错。执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束
		    print("Sub-process(es) done.")


	输出结果：

		Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~
		msg:hello0
		msg:hello1
		msg:hello2
		end
		end
		end
		msg:hello3
		end
		Sub-process(es) done.  
		
	**函数解释**：
   - apply_async(func[, args[, kwds[, callback]]]) 它是非阻塞，apply(func[, args[, kwds]])是阻塞的（理解区别，看例1例2结果区别）
	- close()    关闭pool，使其不在接受新的任务。
	- terminate()    结束工作进程，不在处理未完成的任务。
	- join()    主进程阻塞，等待子进程的退出， join方法要在close或terminate之后使用。

	**执行说明**：创建一个进程池pool，并设定进程的数量为3，xrange(4)会相继产生四个对象[0, 1, 2, 4]，四个对象被提交到pool中，因pool指定进程数为3，所以0、1、2会直接送到进程中执行，当其中一个执行完事后才空出一个进程处理对象3，所以会出现输出“msg: hello 3”出现在"end"后。因为为非阻塞，主函数会自己执行自个的，不搭理进程的执行，所以运行完for循环后直接输出“mMsg: hark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~”，主程序在pool.join()处等待各个进程的结束  
	
- 使用进程池（阻塞）  


		import multiprocessing
		import time
		
		
		def func(msg):
		    print("msg:" + msg)
		    time.sleep(3)
		    print("end")
		
		
		if __name__ == "__main__":
		    pool = multiprocessing.Pool(processes=3)
		    for i in range(4):
		        msg = "hello" + str(i)
		        pool.apply(func, (msg,))  # 维持执行的进程总数为processes，当一个进程执行完毕后会添加新的进程进去
		
		    print("Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~")
		    pool.close()
		    pool.join()  # 调用join之前，先调用close函数，否则会出错。执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束
		    print("Sub-process(es) done.")  
		    
	输出结果：
 	    
		msg:hello0
		end
		msg:hello1
		end
		msg:hello2
		end
		msg:hello3
		end
		Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~
		Sub-process(es) done.

- 使用进程池，并关注结果   


		import multiprocessing
		import time
		
		
		def func(msg):
		    print("msg:" + msg)
		    time.sleep(3)
		    print("end")
		
		    return "done" + msg
		
		
		if __name__ == "__main__":
		    pool = multiprocessing.Pool(processes=4)
		    result = []
		    for i in range(3):
		        msg = "hello" + str(i)
		        result.append(pool.apply_async(func, (msg,)))
		    pool.close()
		    pool.join()
		    for res in result:
		        print(":::" + res.get())
		    print("Sub-process(es) done.")  
		    
	输出结果：
	
		msg: hello 0
		msg: hello 1
		msg: hello 2
		end
		end
		end
		::: donehello 0
		::: donehello 1
		::: donehello 2
		Sub-process(es) done.  
		
		
- 使用多个进程池  

		import multiprocessing
		import os, time, random
		
		
		def Lee():
		    print("\nRun task Lee-%s" % (os.getpid()))  # os.getpid()获取当前的进程的ID
		    start = time.time()
		    time.sleep(random.random() * 10)  # random.random()随机生成0-1之间的小数
		    end = time.time()
		    print('Task Lee, runs %0.2f seconds.' % (end - start))
		
		
		def Marlon():
		    print("\nRun task Marlon-%s" % (os.getpid()))
		    start = time.time()
		    time.sleep(random.random() * 40)
		    end = time.time()
		    print('Task Marlon runs %0.2f seconds.' % (end - start))
		
		
		def Allen():
		    print("\nRun task Allen-%s" % (os.getpid()))
		    start = time.time()
		    time.sleep(random.random() * 30)
		    end = time.time()
		    print('Task Allen runs %0.2f seconds.' % (end - start))
		
		
		def Frank():
		    print("\nRun task Frank-%s" % (os.getpid()))
		    start = time.time()
		    time.sleep(random.random() * 20)
		    end = time.time()
		    print('Task Frank runs %0.2f seconds.' % (end - start))
		
		
		if __name__ == '__main__':
		    function_list = [Lee, Marlon, Allen, Frank]
		    print("parent process %s" % (os.getpid()))
		
		    pool = multiprocessing.Pool(processes=4)
		    for func in function_list:
		        pool.apply_async(func)  # Pool执行函数，apply执行函数,当有一个进程执行完毕后，会添加一个新的进程到pool中
		
		    print('Waiting for all subprocesses done...')
		    pool.close()
		    pool.join()  # 调用join之前，一定要先调用close() 函数，否则会出错, close()执行后不会有新的进程加入到pool,join函数等待素有子进程结束
		    print('All subprocesses done.')  
		    
		    
	输出结果：
	
		parent process 33185
		Waiting for all subprocesses done...
		
		Run task Lee-33186
		
		Run task Marlon-33187
		
		Run task Allen-33188
		
		Run task Frank-33189
		Task Lee, runs 3.26 seconds.
		Task Frank runs 6.40 seconds.
		Task Marlon runs 21.81 seconds.
		Task Allen runs 23.53 seconds.
		All subprocesses done.
