# RTOS15
嵌入式实时操作系统15——优先级反转和死锁
## 1.信号量和互斥量的使用中的两个问题

**信号量在操作系统中用于实现任务同步**，通过同步机制可以实现多个任务合作，让多任务之间按照先后顺序执行。

**互斥量在操作系统中用于协调多任务使用共享共享资源**。当一些共享资源被正在一个任务使用时，其它准备使用这些资源的任务，只能等待资源使用者放弃使用权后才能使用该资源。

![在这里插入图片描述](https://img-blog.csdnimg.cn/791c8a31dfd8402a92ee451fe98652be.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)

信号量和互斥量广泛应用于操作系统中，正是由于这些机制使得我们可以构建功能丰富的，庞大的，移植性强的软件系统。但是如果使用不正确就会产生如下两个常见的问题：
**1、优先级反转。
2、死锁。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/5b1fa5bcdd804a898b3c55ab1bd30a54.png)

**2.优先级反转**
优先级反转是操作系统中使用信号量时，出现的一种优先级不合理的现象：
高优先级任务被低优先级任务阻塞，使得中等优先级的任务优先运行，而高优先级任务得不到调度。从运行现象上看，像是**中优先级任务**比**高优先级任务**具有更高的优先权。

导致优先级反转的原因是：当**高优先级**的任务正等待信号量A，该信号量A正被一个**低优先级**任务占有，此时一个介于这两个任务优先之间的**中优先级**任务抢占了**低优先级**任务的CPU使用权而运行。**使得高优先级任务等待一个低优先级任务，而低优先级任务却被中等优先级任务抢占无法执行。**

优先级反转运行图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/a734b8bd6e3342fa9452eba828f710ce.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)
**优先级反转后导致高优先级任务得不到调度，会影响系统的实时性，严重时甚至会产生错误结果。**
举例一个生活中的例子说明优先级反转：
现在有一个公司，公司内部有3个员工：公司老板，销售经理，销售员。这3个员工的等级是：**公司老板 > 销售经理 > 销售员。**
一天有一个重要的客户来到公司，此时老板需要打印一份公司的专利文件展示给客户看，于是**老板安排销售员去打印专利文件**，销售员来到打印室开始打印专利文件，销售员正在打印文件，**此时销售经理来到打印室打印一份很多页数销售合同**，由于销售经理是销售员的上级，因此销售员停止打印并将打印机让给销售经理打印合同。等待销售经理完成打印合同后，销售员继续打印专利文件，完成打印后销售员将专利文件交给老板。**由于等待的时间较久，导致客户和老板都很不愉快。**


![在这里插入图片描述](https://img-blog.csdnimg.cn/e277ff0c36264e07b4f532f563fb9fd4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

## 3.优先级继承

优先级反转后导致高优先级任务得不到调度，会影响系统的实时性，严重时甚至会产生错误结果。因此我们应该避免这种情况，**如何解决优先级反转？**

**优先级继承就是解决优先级反转问题的一种方法，优先级继承的工作原理是：**
当**低优先级任务获得信号**时候，此时如果有**高优先级任务正在等待该信号**时，操作系统**临时将低优先级任务**的等级**提高**为和高优先级任务一样的优先级，使得**低优先级任务能更快的执行**（不被中优先级任务抢占），释放信号后**低优先级任务恢复**其原来的**优先级**。

开启优先级继承后的运行图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/3f5d3dc4fb0843dfa592cf5ecf14d3bf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

回到上文中提到的例子：一个重要的客户来到公司，老板安排销售员去打印专利文件，销售员来到打印室开始打印专利文件，此时销售经理来到打印室打印一份页数很多销售合同，此时销售员说“**老板正在等我打印文件，你必须等我打印完成**”。**由于等待的时间很短暂，使得客户和老板能很愉快沟通打印文件中的内容。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/bc091223c6a74b0a9bc35318357be2ec.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)

**这就是优先级继承的工作原理，优先级继承保证了多任务系统中的实时性，保证多任务系统以正确合理的方式进行调度**。


## 4.死锁

死锁是指多个任务在运行过程中，由于**竞争**共享资源或者**通信**而造成的一种**阻塞**的现象。如果没有外力作用，它们都将处于互相等待的状态，这些的**任务永远无法得到调度**。
产生死锁的原因是：多个线程同时占用多个资源，但又在彼此等待对方释放资源，**导致这些任务都将处于无限等待中**。

《太空旅客》电影讲述了未来世界一艘搭载着五千余名乘客的移民飞船，飞向殖民星球的故事。电影中有一个叫休眠仓的设备，人类进入休眠仓后可以进入休眠状态。假设电影中的**男主角**进入了休眠仓等待女主角来唤醒，之后**女主角**也进入休眠仓等待男主角来唤醒，最终的结果是男女主角将进入无尽的等待，**永远无法唤醒**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/6613c7f44d0e48b88e23a8b724247724.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

现有两任务：每个任务都需要等待信号A和B，得到信号后执行相关操作，任务完成操作后就释放信号。任务的代码如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/be15e129fad24eb7bf8f63cb8dd7026c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

假设两个任务的运行过程为：
task1开始执行，task1获取信号A执行handle1()操作，此时切换到task2任务获取信号B执行handle3()操作，随后切换到task1，任务执行完成handle1()操作等待信号B（此时信号B为task2占有）进入休眠，task2任务完成handle3()操作等待等待信号A（此时信号A为task1占有）进入休眠。从此task1和task2进入死锁状态两个任务都永远得不到执行。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2b47d0e955dd49a1a114c41213e5daae.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

由此可见：在无外界干预的情况下，**处于死锁状态的任务将永远无法运行**。由于任务死锁无法运行将导致软件将失去相应的功能，因此我们必须避免在软件设计中出现死锁情况。

## 5.死锁的解决方法

死锁将对软件系统带来严重的影响，如何才能避免死锁？
我们可以使用以下3个方法避免死锁：
**1、每个任务增加信号等待超时限制
2、每个任务等待获取全部信号资源后再执行其它操作
3、每个任务避免嵌套使用信号资源**

![在这里插入图片描述](https://img-blog.csdnimg.cn/5fc74419822c40d6aae49a4f0aab76ef.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

**超时限制**
死锁任务在等待信号时可以设定一个超时时间，这个功能可以缓解死锁问题，避免任务出现无限等待的情况。当出现死锁情况时，任务在设定时间内没有得到信号资源，任务则会放弃等待继续执行。
回到《太空旅客》的例子，假设电影中的**男主角设定一个休眠时间**进入了休眠仓并等待女主角来唤醒，之后不管女主角是否来唤醒，**最终休眠仓也会自动将男主角唤醒**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/04f960dd815a4ce5ae360163c2c6f2be.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_17,color_FFFFFF,t_70,g_se,x_16)

**获取全部信号资源后再执行其它操作**
每个任务等待获取全部信号资源后再执行其它操作，这种方法可以避免出现锁死的情况，因为任务都获得了所有信号资源才开始进行执行，在执行过程中不会因为信号资源进入休眠，当任务执行完成后释放所有信号资源。优化后的代码如下：

```c
/* github: liyinuo2017	 author:liwei */
void task1(void *pv)
{
	for(;;)
	{
		/* 等待信号A */
		wait_signal(A); 			
		/* 等待信号B */
		wait_signal(B);
		/* 执行操作 */
		handle1(); 			
		/* 执行操作 */
		handle2(); 
		/* 释放信号A */
		release_signal(A);			
		/* 释放信号B */
		release_signal(B);
	}
}

void task2(void *pv)
{
	for(;;)
	{
		/* 等待信号B */
		wait_signal(B);
		/* 等待信号A */
		wait_signal(A);			
		/* 执行操作 */
		handle3(); 			
		/* 执行操作 */
		handle4(); 
		/* 释放信号B */
		release_signal(B);			
		/* 释放信号A */
		release_signal(A);

	}
}
```

**避免嵌套使用信号资源**
每个任务避免嵌套使用信号资源，当任务使用完信号后立即释放信号，保证每个任务在一个时刻尽量少的占有信号资源，做到“即用即放”。优化后的代码如下：

```c
/* github: liyinuo2017	 author:liwei */
void task1(void *pv)
{
	for(;;)
	{
		/* 等待信号A */
		wait_signal(A);
		/* 执行操作 */
		handle(); 
		/* 释放信号A */
		release_signal(A);			
		/* 等待信号B */
		wait_signal(B);
		/* 执行操作 */
		handle(); 
		/* 释放信号B */
		release_signal(B);

	}
}

void task2(void *pv)
{
	for(;;)
	{
		/* 等待信号B */
		wait_signal(B);
		/* 执行操作 */
		handle(); 
		/* 释放信号B */
		release_signal(B);			
		/* 等待信号A */
		wait_signal(A);
		/* 执行操作 */
		handle(); 
		/* 释放信号A */
		release_signal(A);

	}
}
```

> <font color=red>**未完待续…
  
实时操作系统系列将持续更新
  
创作不易希望朋友们点赞，转发，评论，关注。
  
您的点赞，转发，评论，关注将是我持续更新的动力
  
作者：李巍
  
Github：liyinuoman2017
  
CSDN：liyinuo2017
  
今日头条：程序猿李巍**
