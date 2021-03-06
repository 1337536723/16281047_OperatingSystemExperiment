



# <center>实验三 实验报告</center>

> <center>孙汉武	安全1601	16281047</center>

[TOC]

## Task 1 

### 1.1 实验要求

$\qquad$通过fork的方式，产生4个进程P1,P2,P3,P4，每个进程打印输出自己的名字，例如P1输出“I am the process P1”。要求P1最先执行，P2、P3互斥执行，P4最后执行。通过多次测试验证实现是否正确。

### 1.2 实验过程

1. 实验源码

   `Task1.c` 

   ```c++
   #include<stdio.h>
   #include<unistd.h>
   #include<stdlib.h>
   #include<pthread.h>
   #include<semaphore.h>
   #include<fcntl.h>
   int main()
   {
   	sem_t *P1_signal,*P2_signal,*P3_signal;
   	//主函数中的进程是P1
   	pid_t p2,p3,p4; 
   	P1_signal=sem_open("P1_signal",O_CREAT,0666,0);
   	P2_signal=sem_open("P2_signal",O_CREAT,0666,0);
   	P3_signal=sem_open("P3_signal",O_CREAT,0666,0);
   
   	p2=fork();//创建进程P2
   	if(p2<0)	
   	{
   		perror("创建进程p2出错！");
   	}
   	if(p2==0)
   	{
   		sem_wait(P1_signal);
   		printf("I am the process P2!\n");
   		sem_post(P1_signal);
   		sem_post(P2_signal);
   	}
   	if(p2>0)
   	{
   		p3=fork();
   		if(p3<0)	
   		{
   			perror("创建进程p出错！");
   		}
   		if(p3==0)
   		{
   			sem_wait(P1_signal);
   			printf("I am the process P3!\n");
   			sem_post(P1_signal);
   			sem_post(P3_signal);
   		}
   		if(p3>0)
   		{
   			printf("I am the process P1!\n");
   			sem_post(P1_signal);
   			p4=fork();
   			if(p4<0)	
   			{
   				perror("创建进程p4出错！");
   			}
   			if(p4==0)
   			{
   				sem_wait(P2_signal);
   				sem_wait(P3_signal);
   				printf("I am the process P4!\n");
   				sem_post(P2_signal);
   				sem_post(P3_signal);
   			}
   		}
   	}
   	sem_close(P1_signal);
   	sem_close(P3_signal);
   	sem_close(P2_signal);
   	sem_unlink("P1_signal");
   	sem_unlink("P2_signal");
   	sem_unlink("P3_signal");
   	return 0;
   }
   ```

2. 原理解释

   + 前趋图

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006tKfTcly1g1jizdjii3j30xm0gcq3w.jpg" width="600" /></div>

     

     

     前驱关系：`P1-->P2`、`P1-->P3`、`P2-->P4`、`P3-->P4`

   + 前驱关系实现

     题目要求产生的四个进程必须是P1最先执行，P2、P3在P1执行完后互斥执行，P4最后执行。于是根据要求有了上面的前驱关系和前驱图。但是如何实现这种进程间的前驱关系呢？比较自然的想到了是用信号量机制。如上面的代码所示，定义了三个信号量，P1_signal、P2_signal和P3_signal，其初值均为0

     ```c
     P1_signal=sem_open("P1_signal",O_CREAT,0666,0);
     P2_signal=sem_open("P2_signal",O_CREAT,0666,0);
     P3_signal=sem_open("P3_signal",O_CREAT,0666,0);
     ```

     P1进程执行完打印任务之后对P1_signal信号量进行V操作，产生一个资源让等待P1_signal的进程P2和P3其中之一可以执行。由于P2和P3都是等待P1_signal信号量，但是P1进程只产生一个单位的信号，所以P2和P3的执行是互斥的，这样就满足了题目要求。最后在P2和P3执行完打印任务后对信号量P2_signal和P3_signal进行V操作从各产生一个单位的信号量，而进程P4会等待P2_signal和P3_signal，所以知道当P2和P3进程都完成才能进行P4进程。通过控制这三个信号量，这四个进程之间的前驱关系就满足了题目要求。

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006tKfTcly1g1juc5uvifj30uk0ss778.jpg" width="600" /></div>

   + 进程产生实现

     根据题目要求，通过fork的方式产生四个进程。fork函数会从当前位置复制进程，并且在父进程中返回的pid为复制进程的真实pid，在子进程中返回的pid为0。了解这些知识之后可以得到如下的流程图：

     下面的流程图仅表示进程间的关系，前驱关系的实现请看上一小节.

     <div align="center"><img src="https://ws3.sinaimg.cn/large/006tKfTcly1g1jsiqayyyj30z60l0acn.jpg" width="600" /></div>

   + 进程树

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1u3ax2iraj30o20gywfa.jpg" width="400" ></div>

3. 编译源码

   通过下面的命令编译源码，得到可执行程序

   ```shell
   gcc -g task1.c -o task1 -lpthrea
   ```

### 1.3 实验结果

$\qquad$通过上面的实验已经得到满足实验要求的可执行程序task1,下面给出运行结果，经过多次测试，四个进程在屏幕上打印的顺序只有两种结果，分别如下：

1. 顺序1：`P1-->P2-->P3-->P4`

   <div align="center"><img src="https://ws3.sinaimg.cn/large/006tKfTcly1g1jiscj3k3j30oc02jq40.jpg" width="600" /></div>

2. 顺序2：`P1-->P3--P2-->P4`

<div align="center"><img src="https://ws3.sinaimg.cn/large/006tKfTcly1g1jit8cce5j30lu02imy5.jpg" width="600" /></div>

### 1.4 现象解释

$\qquad$测试的实验结果中出现两种执行顺序，通过1.2 节中的分析不难解释这种现象，由于P1是P2和P3的前驱，所以P1一定会在P2和P3之前执行，但是P2和P3是互斥关系，这两个进程谁先获得P1产生的信号量谁就先执行另一个进程等待。最后等P2和P3都执行完了再执行P4，所以会出现上面的两种执行顺序。

## Task 2

### 2.1 实验要求

$\qquad$火车票余票数ticketCount 初始值为1000，有一个售票线程，一个退票线程，各循环执行多次。添加同步机制，使得结果始终正确。要求多次测试添加同步机制前后的实验效果。

### 2.2 实验过程

#### 2.2.1未添加同步机制

1. 实验源码

   `task2_1.c`:

   ```c
   #include<stdio.h>
   #include<stdlib.h>
   #include<pthread.h>
   #include<semaphore.h>
   #include<sys/stat.h>
   #include<fcntl.h>
   #include<string.h>
   int ticketCount=1000;
   void *SaleThread(void *arg)
   {
   	int num,temp;
   	num=atoi(arg);
   	for(int i=0;i<num;i++)
   	{
   		if(i % 10 ==0)
   			printf("卖%d张票,剩余%d张票\n",i,ticketCount);
   		temp=ticketCount;
   		//放弃CPU，强制切换到另外一个进程
   		pthread_yield();
   		temp=temp-1;
   		pthread_yield();
   		ticketCount=temp;
   	}
   	return NULL;
   }
   
   void *RefundThread(void *arg)
   {
   	int num,temp;
   	num=atoi(arg);
   	for(int i=0;i<num;i++)
   	{
   		if(i % 10 ==0)
   			printf("退%d张票，剩余%d张票\n",i,ticketCount);
   		temp=ticketCount;
   		pthread_yield();
   		temp=temp+1;
   		pthread_yield();
   		ticketCount=temp;
   	}
   	return NULL;
   }
   int main(int argc,char *argv[])
   {
   	if(argc!=3)
   	{
   		printf("请正确输入参数！\n");
   		exit(0);
   	}
   	printf("初始票数为：%d\n",ticketCount);
   	pthread_t p1,p2;
   	/* printf("%s %s",argv[1],argv[2]); */
   	pthread_create(&p1,NULL,SaleThread,argv[1]);
   	pthread_create(&p2,NULL,RefundThread,argv[2]);
   	pthread_join(p1,NULL);
   	pthread_join(p2,NULL);
   	printf("最终票数为：%d\n",ticketCount);
   	return 0;
   }
   ```

2. 程序解释

   + 在main函数中创建两个线程，分别是模拟售票的线程`SaleThread`和模拟退票的线程`RefundThread`，两个进程并发执行，不添加任何的同步机制。
   + 模拟票数的变量ticketCount是全局变量
   + 程序运行需要输入两个参数，第一个是售票数量，第二个数退票数量

3. 程序运行结果

   编译上述程序，得到可执行程序`task2_1`

   ```shell
   gcc -g  task2_1.c -o task2_1 -lpthread
   ```

   多次测试运行，运行结果可以分为两种类型，一种是售票数量比退票数量多，另一种是售票数量比退票数量少。两种情况的结果分别如下：

   + 售票数量比退票数量多：

     初始票数：1000  	售票：100 	退票：40

     <div align="center"><img src="https://ws3.sinaimg.cn/large/006tKfTcly1g1jzmct47hj30ze0h4n0i.jpg" width="600" /></div>

     

   + 售票数量比退票数量少：

     初始票数：1000	售票：50	退票：80

     <div align="center"><img src="https://ws2.sinaimg.cn/large/006tKfTcly1g1jznrmxgxj30z60g2whl.jpg" width="600" /></div>

     

4. 实验现象归纳

   通过一系列的测试，归纳出的实现现象如下：

   + 当售票数量大于退票数量的时候，最终票数等于总票数减去售票数
   + 当售票数量小于退票数量的时候，最终票数等于总票数加上退票数

#### 2.2.2 添加同步机制

1. 实验源码

   `task2_2.c`

   ```c
   #include<stdio.h>
   #include<stdlib.h>
   #include<pthread.h>
   #include<semaphore.h>
   #include<sys/stat.h>
   #include<fcntl.h>
   #include<string.h>
   volatile int ticketCount=1000;
   sem_t *flag=NULL;
   void *SaleThread(void *arg)
   {
   	int num,temp;
   	num=atoi(arg);
   	for(int i=0;i<num;i++)
   	{
   		if(i % 10 ==0)
   			printf("卖%d张票,剩余%d张票\n",i,ticketCount);
   		sem_wait(flag);
   		temp=ticketCount;
   		//放弃CPU，强制切换到另外一个进程
   		pthread_yield();
   		temp=temp-1;
   		ticketCount-=1;
   		pthread_yield();
   		ticketCount=temp;
   		sem_post(flag);
   	}
   	return NULL;
   }
   
   void *RefundThread(void *arg)
   {
   	int num,temp;
   	num=atoi(arg);
   	for(int i=0;i<num;i++)
   	{
   		if(i % 10 ==0)
   			printf("退%d张票，剩余%d张票\n",i,ticketCount);
   		sem_wait(flag);
   		temp=ticketCount;
   		pthread_yield();
   		temp=temp+1;
   		ticketCount+=1;
   		pthread_yield();
   		ticketCount=temp;
   		sem_post(flag);
   	}
   	return NULL;
   }
   int main(int argc,char *argv[])
   {
   	if(argc!=3)
   	{
   		printf("请正确输入参数！\n");
   		exit(0);
   	}
   	flag=sem_open("flag",O_CREAT,0666,1);
   	printf("初始票数为：%d\n",ticketCount);
   	pthread_t p1,p2;
   	printf("%s %s",argv[1],argv[2]);
   	pthread_create(&p1,NULL,SaleThread,argv[1]);
   	pthread_create(&p2,NULL,RefundThread,argv[2]);
   	pthread_join(p1,NULL);
   	pthread_join(p2,NULL);
   	printf("最终票数为：%d\n",ticketCount);
   	sem_close(flag);
   	sem_unlink("flag");
   	return 0;
   
   }
   ```

2. 程序解释

   + task2_2.c在task2_1.c的基础上增加了同步机制，其他部分完全一致，通过信号量flag的控制，让售票线程和退票线程一次只能执行一个，在一个没有执行完成之前另一个不能进入执行，这样就保证了售票操作和退票操作的原子性，避免了脏数据的读取。

   + flag初始值为设置为1，表示每次只允许一个线程操作ticketCount这个数据

   + 售票线程和退票线程在进入操作之前都要sem_wait(flag)，等待信号量，在完成操作之后要sem_post(flag)，下图是售票线程中增加了信号量的操作：

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1md41scu2j30qi06z3za.jpg" width="600" /></div>

3. 程序运行结果

   编译上述程序，得到可执行程序task2_2

   ```shell
   gcc task2_2.c -o task2_2 -lpthread
   ```

   多次测试运行，测试主要分为两种类型，一种是售票数量比退票数量多，另一种是售票数量比退票数量少。两种情况的结果分别如下：

   + 售票数量比退票数量多：

     初始票数：1000  	售票：100 	退票：40

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1md9o8sw0j30q50dowh5.jpg" width="600" /></div>

   + 退票数量比售票数量多：

     初始票数：1000	售票：50	退票：80

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1mdaxj1kyj30q40cu40y.jpg" width="600" /></div>

4. 实验现象归纳

   - 在第一个测试样例中，初始票数为1000，售票100并且退票40，最终总票数为940，结果正确；

   - 在第二个测试样例中，初始票数为1000，售票50并且退票80，最终总票数为1030，结果正确。

   + 通过上面的测试结果可以看出，不论是售票数量比退票数量多还是少，都不会发生类似前面2.2.1的问题，最终的票数是期待得到的结果。

   + 上面的实验证实了增加了同步机制之后的多线程并发程序有效的解决了脏数据的读取问题

### 2.3 实验结果

 通过2.2节的对比实验可以看出，在执行多进程并发程序的时候，由于多进程的切换可能发生在某个进程的中间，会导致在一个进程处理的数据未写入ticketCount之前另外一个进程读取该数据，这样就导致了脏数据的读取，导致最终结果的不正确。

在2.2节的后半部分通过怎加同步机制，保证售票进程和退票进程的的原子性，就是指在某个进程操作的时候，在它完成操作之前另外一个进程无法操作共享变量ticketCount,这样就避免了脏数据的发生，得到了预期的正确结果。

### 2.4 现象解释

#### 2.4.1 现象解释1

在2.2.1节的实验中，以售票线程为例（代码如下图所示），没有添加同步机制，并且在进行`temp=temp-1`和`temp=ticketCount`的后面均加上了pthread_yield，这个函数的作用是放弃对CPU的使用权，切换到其他进程中，本实验中就是切换到退票进程中。

这样就能解释为什么2.2.1节中的实验现象，在2.2.1节中，不论售票数多还是退票数多，最终结果都是总票数加上或减去值比较大的那个数。通过分析可以得出解释，售票进程和退票进程同时进行，初始票数均为1000，售票进程完成一次是票数为999，售票进程开始下一次售票，但是在运行`temp=ticketCount`之前，退票进程处理的数据还没有写入到内存中，导致售票进程读取的还是自己之前计算的ticketCount值，而不是全局的值。退票进程也是同理。

但是为什么刚好就是总票数加上或减去值比较大的那个呢？按照道理来说因该售票进程执行temp=ticketCount在退票进程写入ticketCount值之前发生是存在一定概率的，但是在目前为止的所有测试结果全部都是在写入之前读取ticketCount值，对此的解释是由于ticketCount=temp和temp=ticketCount之间没有加pthread_yield操作，而现代的处理器运算速度足够快，在退票进程放弃CPU控制权的那个时间片已经完成了这两步操作，所以相当于售票进程读取的ticketCount一直是自己本身的值，退票进程处理的数据对售票进程并没有影响。

<div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1mg7l5m5jj30hu0ast9k.jpg" width="600" /></div>

为了验证上面的猜想，在=如下图所示代码，在ticketCount之后增加一行代码，pthread_yield，放弃当前进程对CPU的控制权，即售票进程放弃对CPU的控制权转而交给退票进程，这个时候退票进程处理的数据就能写入到内存中，而当售票进程再次处理temp=ticketCount的时候，读取的就是退票进程已经写入的数据。如果猜想正确的话，期待的最终票数因该还会发生错误，但并不是像第一中那种恰好等于总票数加减数值大的那个数。

<div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1mg5o8rzxj30ht0am0tk.jpg" width="600" /></div>

得到的结果如下，发现最终的票数不在是1000+50=1050，而是分布在1000~1050之间的数值。猜想得到验证。

<div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1mx3kkmlxj30hh05ndg8.jpg" width="600" /></div>

针对上面的猜想（CPU运算速度过快，导致ticketCount=temp和temp=ticketCount两步操作在一个进程的时间片内完成导致的数据错误），另外的一种验证方式是将初始票数和售票退票数设置的足够大，当数据足够大的时候，就会存在一定概率出现在一个进程的ticketCount=temp和temp=ticketCount两步操作之间切换进程的问题，得到的结果就不会类似2.2.1中的那样，而是类似在ticketCount=temp下面加了pthread_yield那样。

 <div align="center">
     <img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1pa2byvv1j30mp0hbtb2.jpg" width="600" div align="center"/></div>

再次运行，可以看到如下的实验结果：

<div align="center">      <img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1pagap9rwj30qm0do77w.jpg" width="600" div align="center"/></div>

#### 2.4.2 现象解释2

在2.2.2节中，当给售票进程和退票进程都加上同步机制后，保证了每个线程操作的原子性，每个线程操作的过程中其他的线程不能对共享的ticketCount变量进程修改，这样的话最终的结果就是正确的结果。

## Task 3

### 3.1 实验要求

$\qquad​$一个生产者一个消费者线程同步。设置一个线程共享的缓冲区， char buf[10]。一个线程不断从键盘输入字符到buf,一个线程不断的把buf的内容输出到显示器。要求输出的和输入的字符和顺序完全一致。（在输出线程中，每次输出睡眠一秒钟，然后以不同的速度输入测试输出是否正确）。要求多次测试添加同步机制前后的实验效果。

### 3.2 实验过程

1. 实验源码

   `task3.c`

   ```c
   #include<stdio.h>
   #include<stdlib.h>
   #include<pthread.h>
   #include<semaphore.h>
   #include<sys/stat.h>
   #include<fcntl.h>
   char buf[10];
   sem_t *empty=NULL;
   sem_t *full=NULL;
   void *worker1(void *arg)
   {
   	
   	for(int i=0;i<10;i++)
   	{
   		sem_wait(empty);
   		/* fflush(stdin); */
   		/* printf("输入："); */
   		scanf("%c",&buf[i]);
   		sem_post(full);
   		if(i==9)
   		{
   			i=-1;
   		}
   	}	
   	return NULL;
   }
   void *worker2(void *arg)
   {
   	for(int i=0;i<10;i++)
   	{
   		sem_wait(full);
   		printf("输出：%c\n",buf[i]);
   		sem_post(empty);
   		sleep(1);
   		if(i==9)
   		{
   			i=-1;		
   		}
   	}	
   	return NULL;
   }
   
   int main(int argc,char *argv[])
   {
   	empty=sem_open("empty_",O_CREAT,0666,10);
   	full=sem_open("full_",O_CREAT,0666,0);
   	pthread_t p1,p2;
   	pthread_create(&p1,NULL,worker1,NULL);
   	pthread_create(&p2,NULL,worker2,NULL);
   	pthread_join(p1,NULL);
   	pthread_join(p2,NULL);
   	sem_close(empty);
   	sem_close(full);
   	sem_unlink("empty_");
   	sem_unlink("full_");
   	return 0;
   }
   ```

2. 程序解释

   + work1是输入线程调用的函数，worker2是输出线程调用的函数。
   + 设置两个信号量empty和ful来控制程序的执行，其中empty信号量用于保证输入线程在写入数据到缓存的时候缓存中还有空余的位置，保证写入线程后写入的数据不会把前面写入但是为输出的数据给覆盖掉，其初始值为10，表示最开始缓存中有10个空余的位置供给写入线程写入数据；full信号量是用于保证输出线程有数据输出，避免在写入线程还没有写入数据的情况下输出线程输出随机数据，其初始值为0，表示初始状态下缓存中没有数据可以输出
   + 输入线程在写入一个数据前要等待empty信号量，进入后便消耗一个信号量；完成写入数据操作之后post一个full信号量，通知输出线程输出数据。
   + 输出线程在输出一个数据之前哟啊等待full信号量，进出输出操作后便消耗一个full信号量；完成输出操作后post一个empty信号量，通知写入线程缓存又多一个空余位置以供写入数据。
   + 输出线程每输出一个字符等待一秒钟，方便实验结果的查看。

3. 编译源代码

   ```shell
   gcc task3.c -o task3 -lpthread
   ```

### 3.3 实验结果

#### 3.3.1 实验运行现象

1. 随机输入字母和数字（10个以内）：124365abc

   <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1t6937h4hj31c80d2wg6.jpg" width="600" /></div>

2. 随机输入字母和数字(10个以上)：123456789abcdefg

<div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1t6andb8sj31ay0k60vf.jpg" width="600" /></div>

3. 不间断输入：

   <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1t6c6h7qjj31au0nujty.jpg" width="600" /></div>

   通过观察上面的实验现象，可以看到已经满足了实验题目的要求。

#### 3.3.2 实验现象解释

+ 在第一种类型的测试中，输入数据不大于10个字符的时候，由于empty的信号量初始值为10，所以输入进程会一直连续不断的向缓存中写入数据，每写入一个数据，便post一个full信号量，输出线程便能按序输出字符。
+ 在第二种类型的测试中，输入数据大于10个字符的时候，由于empty的初始值为10，所以输入的字符中开始的时候只有前10个字符被写入缓存中，其他的在I/O缓冲区等待输入，当输出线程接收到输入线程post的信号量的时候便会开始输出，每输出一个字符便会post一个empty信号量，当输入线程接收到empty信号量的时候有开始从I/O缓冲区读取数据写入到缓存中。

+ 第三种测试和第二种类似，在输出的过程中间输入数据，原理其实是一样的。

## Task 4

### 4.1 实验要求

1. 通过实验测试，验证共享内存的代码中，receiver能否正确读出sender发送的字符串？如果把其中互斥的代码删除，观察实验结果有何不同？如果在发送和接收进程中打印输出共享内存地址，他们是否相同，为什么？

2. 有名管道和无名管道通信系统调用是否已经实现了同步机制？通过实验验证，发送者和接收者如何同步的。比如，在什么情况下，发送者会阻塞，什么情况下，接收者会阻塞？
3. 消息通信系统调用是否已经实现了同步机制？通过实验验证，发送者和接收者如何同步的。比如，在什么情况下，发送者会阻塞，什么情况下，接收者会阻塞？



### 4.2 实验过程

> 实验过程根据实验要求的三个部分，对应的过程也分为三个部分，具体如下所示

#### 4.2.1 内存共享

1. 实验源码

   > 内存内存共享实验的源码分为两个部分，分别是Sender.c和Receive.c,

   `Sender.c`：

   ```c
   /*
    * Filename: Sender.c
    * Description: 
    */
   
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>
   #include <sys/sem.h>
   #include <sys/ipc.h>
   #include <sys/shm.h>
   #include <sys/types.h>
   #include <string.h>
   
   int main(int argc, char *argv[])
   {
       key_t  key;
       int shm_id;
       int sem_id;
       int value = 0;
       //1.Product the key
       key = ftok(".", 0xFF);
       //2. Creat semaphore for visit the shared memory
       sem_id = semget(key, 1, IPC_CREAT|0644);
       if(-1 == sem_id)
       {
           perror("semget");
           exit(EXIT_FAILURE);
       }
       //3. init the semaphore, sem=0
       if(-1 == (semctl(sem_id, 0, SETVAL, value)))
       {
           perror("semctl");
           exit(EXIT_FAILURE);
       }
       //4. Creat the shared memory(1K bytes)
       shm_id = shmget(key, 1024, IPC_CREAT|0644);
       if(-1 == shm_id)
       {
           perror("shmget");
           exit(EXIT_FAILURE);
       }
       //5. attach the shm_id to this process
       char *shm_ptr;
       shm_ptr = shmat(shm_id, NULL, 0);
       if(NULL == shm_ptr)
       {
           perror("shmat");
           exit(EXIT_FAILURE);
       }
       //6. Operation procedure
       struct sembuf sem_b;
       sem_b.sem_num = 0;      //first sem(index=0)
       sem_b.sem_flg = SEM_UNDO;
       sem_b.sem_op = 1;           //Increase 1,make sem=1
       
       while(1)
       {
           if(0 == (value = semctl(sem_id, 0, GETVAL)))
           {
               printf("\nNow, snd message process running:\n");
               printf("\tInput the snd message:  ");
               scanf("%s", shm_ptr);
   
               if(-1 == semop(sem_id, &sem_b, 1))
               {
                   perror("semop");
                   exit(EXIT_FAILURE);
               }
           }
           //if enter "end", then end the process
           if(0 == (strcmp(shm_ptr ,"end")))
           {
               printf("\nExit sender process now!\n");
               break;
           }
       }
       shmdt(shm_ptr);
       return 0;
   }
   
   ```

   `Receiver.c`

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>
   #include <sys/sem.h>
   #include <sys/ipc.h>
   #include <sys/shm.h>
   #include <sys/types.h>
   #include <string.h>
   
   int main(int argc, char *argv[])
   {
       key_t  key;
       int shm_id;
       int sem_id;
       int value = 0;
       //1.Product the key
       key = ftok(".", 0xFF);
       //2. Creat semaphore for visit the shared memory
       sem_id = semget(key, 1, IPC_CREAT|0644);
       if(-1 == sem_id)
       {
           perror("semget");
           exit(EXIT_FAILURE);
       }
       //3. init the semaphore, sem=0
       if(-1 == (semctl(sem_id, 0, SETVAL, value)))
       {
           perror("semctl");
           exit(EXIT_FAILURE);
       }
       //4. Creat the shared memory(1K bytes)
       shm_id = shmget(key, 1024, IPC_CREAT|0644);
       if(-1 == shm_id)
       {
           perror("shmget");
           exit(EXIT_FAILURE);
       }
       //5. attach the shm_id to this process
       char *shm_ptr;
       shm_ptr = shmat(shm_id, NULL, 0);
       if(NULL == shm_ptr)
       {
           perror("shmat");
           exit(EXIT_FAILURE);
       }
   
       //6. Operation procedure
       struct sembuf sem_b;
       sem_b.sem_num = 0;      //first sem(index=0)
       sem_b.sem_flg = SEM_UNDO;
       sem_b.sem_op = -1;           //Increase 1,make sem=1
       
       while(1)
       {
           if(1 == (value = semctl(sem_id, 0, GETVAL)))
           {
               printf("\nNow, receive message process running:\n");
               printf("\tThe message is : %s\n", shm_ptr);
   
               if(-1 == semop(sem_id, &sem_b, 1))
               {
                   perror("semop");
                   exit(EXIT_FAILURE);
               }
           }
           //if enter "end", then end the process
           if(0 == (strcmp(shm_ptr ,"end")))
           {
               printf("\nExit the receiver process now!\n");
               break;
           }
       }
       shmdt(shm_ptr);
       //7. delete the shared memory
       if(-1 == shmctl(shm_id, IPC_RMID, NULL))
       {
           perror("shmctl");
           exit(EXIT_FAILURE);
       }
       //8. delete the semaphore
       if(-1 == semctl(sem_id, 0, IPC_RMID))
       {
           perror("semctl");
           exit(EXIT_FAILURE);
       }
       return 0;
   }
   ```

2. 程序解释

   > 下面以sender.c为例解释一下如何创建共享内存并通过信号量机制实现互斥访问从而达到进程间通信的目的。

   + 创建一个共享内存的ID,就是代码中的key

     ```c
      key_t  key;
      key = ftok(".", 0xFF);
     ```

     >  通过ftok函数创建一个key_t类型的变量，作为共享内存的key，ftok函数的两个参数分别是文档名(一个存在的路径),上例中的路径是`.`表示当前路径，另一个参数是子序号

   + 创建并初始化信号量

     ```c
     int sem_id;
     sem_id = semget(key, 1, IPC_CREAT|0644);
     if(-1 == sem_id)
     {
         perror("semget");
         exit(EXIT_FAILURE);
     }
     if(-1 == (semctl(sem_id, 0, SETVAL, value)))
     {
         perror("semctl");
         exit(EXIT_FAILURE);
     }
     ```

     > 通过semget()函数创建一个信号量，初始值为1，再通过semctl()函数初始化该信号量

   + 创建共享内存并挂载在进程中

     ```c
     //4. Creat the shared memory(1K bytes)
     shm_id = shmget(key, 1024, IPC_CREAT|0644);
     if(-1 == shm_id)
     {
         perror("shmget");
         exit(EXIT_FAILURE);
     }
     //5. attach the shm_id to this process
     char *shm_ptr;
     shm_ptr = shmat(shm_id, NULL, 0);
     if(NULL == shm_ptr)
     {
         perror("shmat");
         exit(EXIT_FAILURE);
     }
     ```

     > 在这部分代码中，首先通过shmget()函数创建了一个大小为1000B的共享内存，然后通过shmat函数，将刚刚创建的共享内存以可读写的方式挂载在进程上，并且指定系统将自动选择一个合适的地址给共享内存，将挂载的共享内存地址赋值给char型指针shm_ptr

   + Sender主循环

     ```c
     while(1)
     {
         if(0 == (value = semctl(sem_id, 0, GETVAL)))
         {
             printf("\nNow, snd message process running:\n");
             printf("\tInput the snd message:  ");
             scanf("%s", shm_ptr);
     
             if(-1 == semop(sem_id, &sem_b, 1))
             {
                 perror("semop");
                 exit(EXIT_FAILURE);
             }
         }
         //if enter "end", then end the process
         if(0 == (strcmp(shm_ptr ,"end")))
         {
             printf("\nExit sender process now!\n");
             break;
         }
     }
     ```

     > 主循环中首先判断表示共享内存访问情况的信号量是否为0(为0表示共享内存空闲)，如果为0的话提示用户输入想要输入的消息，并将用户输入的消息写入共享内存中，写完后通过semop函数将信号量加一，通知receiver读取消息。并且定义一个`end`命令表示退出当前进程。循环退出的时候取消共享内存的挂载

   + Receiver主循环

     ```c
     while(1)
     {
         if(1 == (value = semctl(sem_id, 0, GETVAL)))
         {
             printf("\nNow, receive message process running:\n");
             printf("\tThe message is : %s\n", shm_ptr);
     
             if(-1 == semop(sem_id, &sem_b, 1))
             {
                 perror("semop");
                 exit(EXIT_FAILURE);
             }
         }
         //if enter "end", then end the process
         if(0 == (strcmp(shm_ptr ,"end")))
         {
             printf("\nExit the receiver process now!\n");
             break;
         }
     }
     ```

     > 主循环中首先判断表示共享内存访问情况的信号量是否为1(为1表示共享内存已经写入消息，可以读取)，如果为1的话输出该消息，输出后通过semop函数将信号量减1，通知Sender可以再次写入消息。并且定义一个`end`命令表示退出当前进程。循环退出的时候取消共享内存的挂载

3. 实验现象

   将上述源码编译后进行测试，得到下面的结果。

   <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1t9bsd3s5j31uq0fa0xm.jpg" width="800" /></div>

   > 可以看到sender进程发出的消息receiver进程均准确无误的收到

4. 删除互斥访问相关的代码

   程序主要的代码没有变化，只是在Sender和Receiver进程的主循环中将用于控制互斥访问共享内存的相关代码删除，注释后的结果如下：

   `Sender_2.c:`

   ```c
   while(1)
   {
       printf("\nNow, snd message process running:\n");
       printf("\tInput the snd message:  ");
       scanf("%s", shm_ptr);
       //if enter "end", then end the process
       if(0 == (strcmp(shm_ptr ,"end")))
       {
           printf("\nExit sender process now!\n");
           break;
       }
   }
   ```

   `Receiver_2.c:`

   ```CQL
   while(1)
   {
       printf("\nNow, receive message process running:\n");
       printf("\tThe message is : %s\n", shm_ptr);
   
       //if enter "end", then end the process
       if(0 == (strcmp(shm_ptr ,"end")))
       {
           printf("\nExit the receiver process now!\n");
           break;
       }
       sleep(3);
   }
   ```

   > 最后加一个sleep(1)用于控制打印的速度，便于观察现象

5. 删除互斥访问后的实验现象

   <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1t9sdljhhj31uq0n8799.jpg" width="800" /></div>

   > 实验现象解释：当删除互斥访问之后，两个进程便没有限制的访问共享内存，Sender进程由于受限于用户输入的速度，会停留一直等待用户输入数据，但是Receiver进程会一直输出共享内存中的消息。

6. 打印Sender和Receiver进程中共享内存的地址

   在原始代码的基础上修改，具体代码文件分别是`Sender_3.c`和`Receiver_3.c`，具体修改就是如下：

   <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1ta8ocwdvj30wy044t9i.jpg" width="600" /></div>

   在挂载共享内存后打印挂载后的地址

7. 打印共享内存地址实验现象

   <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1ta7ecrb4j31uq0fe7b0.jpg" width="800" /></div>

   > 可以看到实验现象，在两个进程中共享内存的地址不一样

   ==**现象解释：**==

   通过上面的现象可以看到共享内存在不同进程中是不相同的，总结有以下的原因导致共享内存在不同进程中的地址不一样：

   + 进程在挂载内存的时候使用的`shmat()`函数中的第二个参数使用的是NULL，NULL参数的含义是进程让系统分配给共享内存合适的地址。在`shmat()`函数中，第二个参数有三种选择，分别是：

   | 参数值 |             NULL             |                             addr                             |                             addr                             |
   | :----: | :--------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
   |  含义  | 系统将自动选择一个合适的地址 | 如果shmaddr非0 并且指定了SHM_RND 则此段连接到shmaddr -（shmaddr mod SHMLAB)所表示的地址上。 | 第三个参数如果在flag中指定了SHM_RDONLY位，则以只读方式连接此段，否则以读写的方式连接此 段。 |

   ​	可以看到，当addr有具体的值的时候，便将共享内存挂载到指定的地址上

   + 现代操作系统中都存在ASLR(地址空间随机化)，ASLR是⼀种针对缓冲区溢出的安全保护机制，具有ASLR机制的操作系统每次加载到内存的程序起始地址会随机变化。系统的这个随机化操作可能导致共享内存的地址不一致。

   ==**验证：**==

   1. 指定Sender_4.c和Receiver_4.c中共享内存的挂载地址为`0x7fcc2c0bb000`

      + 修改具体的代码如下：

        <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1taz7pfxdj30x408w404.jpg" width="600" /></div>

      + 运行结果：

        <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1tb56qlhmj31us0d0jx1.jpg" width="800" /></div>

        > 实验结果结果佐证了上面的现象解释，通过指定挂载共享内存的地址，可以使共享内存的地址一致，可以随意指定改地址

   2. 关闭系统的ASLR操作

      + 具体的关闭命令如下：

      ```
      sudo su
      sysctl -w kernel.randomize_va_space=0
      ```

      + 运行结果：

        <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1tb3biwvpj31uw0cc794.jpg" width="800" /></div>

        > 这个实验现象也佐证了系统的ASLR也对导致挂载的共享内存地址不一样

#### 4.2.2 管道通信

##### （1）无名管道

1. 实验源码

   `pipe.c:`

   ```c
   #include <stdio.h>
   #include <unistd.h>     //for pipe()
   #include <string.h>     //for memset()
   #include <stdlib.h>     //for exit()
   int main()
   {
       int fd[2];
       char buf[20];
       if(-1 == pipe(fd))
       {
           perror("pipe");
           exit(EXIT_FAILURE);
       }
       write(fd[1], "hello,world", 12);
       memset(buf, '\0', sizeof(buf));
       read(fd[0], buf, 12);
       printf("The message is: %s\n", buf);
       return 0;
   }
   ```

2. 程序解释

   + 通过pipe函数创建管道，函数传递一个整形数组fd，fd的两个整形数表示的是两个文件描述符，其中第一个用于读取数据，第二个用于写数据。两个描述符相当远管道的两端，一段负责写数据，一段负责读数据。
   + pipe管道是半双工的工作模式，某一时刻只能读或者只能写
   + 读写管道就和读写普通文件一样，使用write和read

3. 实验现象

   <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1tw0wq5j8j30xq0323zh.jpg" width="600" ></div>

4. 无名管道同步机制验证

   > 为了验证无名管道的同步机制，在上述代码的基础上进行修改，得到如下的代码

   `pipe_2.c:`

   ```c
   #include <stdio.h>
   #include <unistd.h>     //for pipe()
   #include <string.h>     //for memset()
   #include <stdlib.h>     //for exit()
   int main()
   {
       int fd[2];
       char buf[200]={0};
   	pid_t child;
       //创建管道
   	if(-1 == pipe(fd))
       {
           perror("pipe");
           exit(EXIT_FAILURE);
       }
   	//创建子进程
   	child=fork();
   	if(child==-1)
   	{
   		perror("fork");
   		exit("EXIT_FAILURE");
   	}
   	if(child==0)
   	{
   		//关闭子进程中不需要的写描述符
   		close(fd[1]);
   		while(1)
   		{
   			if(read(fd[0],buf,sizeof(buf))>0)
   					printf("子进程接收的消息是:%s\n",buf);
   			else
   					printf("子进程:管道中没有数据\n");
   			sleep(2);		
   			if(strcmp(buf,"end")==0)
   					break;
   			memset(buf,0,sizeof(buf));
   		}
   	}
   	if(child>0)
   	{
   		close(fd[0]);
   		while(1)
   		{
   			printf("父进程中-请输入消息:");
   			scanf("%s",buf);
   			write(fd[1],buf,strlen(buf));
   			if(strcmp(buf,"end")==0)
   				break;
   		}	
   	}
       return 0;
   }
   ```

   > 对于上述代码做出如下解释：父进程是消息的发送者，在父进程中创建了两个文件描述符，fork一个子进程的时候会复制这两个管道文件描述符，因此父进程和子进程都会将自己的那个用不到的文件描述符关闭。父进程中会持续向管道中写入用户输入的消息，子进程会一直输出管道中的消息，如果管道中没有消息就会阻塞等待。

5. 无名管道同步机制实验现象

   <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1u1cu3kutj30xm0ny44z.jpg" width="600" ></div>

   > 可以看到输出进程是按照输入进程输入的顺序输出数据，并且当输入进程没有数据输入，即管道中没有数据的时候，输出进程会阻塞。因此无名管道通信系统调用的时候已经yijing实现了同步机制

6. 无名管道同步机制原理

   通过上面的实验和查阅相关资料，得到无名管道如下的同步机制：

   + 管道的读写通过两个系统调用write和read实现

   + 发送者在向管道内存中写入数据之前，首先**检查内存是否被读进程锁定**和**内存中是否还有剩余空间**，如果这两个要求都满足的话write函数会对内存上锁，然后进行写入数据，写完之后解锁；否则就会等待(阻塞)。
   + 写进程在读取管道中的数据之前，也会**检查内存是否被读进程锁定**和**管道内存中是否有数据**，如果满足这两个条件，read函数会对内存上锁，读取数据后在解锁；否则会等到(阻塞)

##### （2）有名管道

1. 实验代码

   > 有名管道实验中设计两个代码文件`fifo_send.c`和`fifo_rcv.c`

   `fifo_send.c:`

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>
   #include <sys/stat.h>
   #include <sys/ipc.h>
   #include <fcntl.h>
   #define FIFO "./my_fifo"
   
   int main()
   {
       char buf[] = "hello,world";
       //1. check the fifo file existed or not
       int ret;
       ret = access(FIFO, F_OK);
       if(ret != 0)    //file /tmp/my_fifo existed
       {
       	if(-1 == mkfifo(FIFO, 0766))
       	{
       	    perror("mkfifo");
       	    exit(EXIT_FAILURE);
       	}
       }
   
       //3.Open the fifo file
       int fifo_fd;
       fifo_fd = open(FIFO, O_WRONLY);
       if(-1 == fifo_fd)
       {
           perror("open");
           exit(EXIT_FAILURE);
   
       }
       //4. write the fifo file
       int num = 0;
       num = write(fifo_fd, buf, sizeof(buf));
       if(num < sizeof(buf))
       {
           perror("write");
           exit(EXIT_FAILURE);
       }
       printf("write the message ok!\n");
   
       close(fifo_fd);
   
       return 0;
   }
   ```

   `fifo_rcv.c:`

   ```c
   /*
    *File: fifo_rcv.c
    */
    
   #include <stdio.h>
   #include <string.h>
   #include <stdlib.h>
   #include <unistd.h>
   #include <sys/stat.h>
   #include <sys/ipc.h>
   #include <fcntl.h>
   
   
   #define FIFO "./my_fifo"
   
   int main()
   {
       char buf[20] ;
       memset(buf, '\0', sizeof(buf));
   
       //`. check the fifo file existed or not
       int ret;
       ret = access(FIFO, F_OK);
       if(ret != 0)    //file /tmp/my_fifo existed
       {
           if(-1==mkfifo(FIFO,0766))
           {
               perror("mkfifo"); 
               exit("EXIT_FAILURE");
           }
       }
   
   //	2.Open the fifo file
       int fifo_fd;
       fifo_fd = open(FIFO, O_RDONLY);
       if(-1 == fifo_fd)
       {
           perror("open");
           exit(EXIT_FAILURE);
       }
       //4. read the fifo file
       int num = 0;
       num = read(fifo_fd, buf, sizeof(buf));
       printf("Read %d words: %s\n", num, buf);
       close(fifo_fd);
       return 0;
   }
   ```

2. 程序解释

   + 写进程fifo_send分为四个步骤执行，首先判断当前目录下是否已经存在my_fifo文件，不存在的话在当前目录下通过mkfifo()函数创建FIFO类型的文件my_fifo；再通过open()函数打开my_fifo文件，最后向文件中写入消息；
   + 读进程的过程和写进程的类似，只没有了创建fifo文件的过程而已

3. 实验现象

   <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1u29b5rfrj31vi05cgpq.jpg" width="800" ></div>

   > 现象描述：在仅仅只运行fifo_send进程的时候，没有任何输出，进程一直阻塞，直到fifo_rcv进程运行，两个进程才开始输出信息。

   当写进程和读进程都设置成阻塞状态的时候，不论先执行那个进程，先执行的进程都会阻塞等待，待另一个进程执行后两个进程才正常执行。

4. 探究有名管道的同步和阻塞机制

   通过`fifo_fd=open(FIFO,O_RDONLY | O_NONBLOCK)`设置为非阻塞状态，`fifo_fd=open(FIFO,O_RDONLY)`设置为阻塞状态，对应四个进程分别为fifo_send(阻塞)、fifo_rcv(阻塞)、fifo_send_1(非阻塞)、fifo_rcv_1(非阻塞)

   + 读进程阻塞、写进程阻塞

     + 先执行fifo_send后执行fifo_rcv，结果正确

       截图请见上面的实验现象

     + 先执行fifo_rcv后执行fifo_send，结果正确

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1u7bac7ewj31vg05qgp1.jpg" width="800" ></div>

     具体的原因是读进程在open FIFO的时候由于没有s

     通过查阅资料得到了FIFO管道的阻塞机制如下：

     对于设置了阻塞的读进程而言：

     > 1. 读进程阻塞的原因有三种：FIFO 中没有数据、有其他的读进程正在读取这些数据、没有写进程打开FIFO文件
     > 2. 不论是哪种原因引起的阻塞，解开阻塞的原因都是FIFO有新的数据写入
     > 3. 如果一个读进程有多个read操作，那么只会阻塞第一个read，其他的不会发生阻塞

     对于设置了阻塞的写进程而言：

     > 1. 当写入的数据量小于PIPE_BUF时，Linux保证写入原子性。如果此时管道中的空闲位置不足以容纳要写入的数据，泽写进程阻塞，直到管道中空间足够，一次性写入所有数据
     > 2. 当写入的数据量大于PIPE_BUF时，Linux不再保证写入的原子性。一旦管道中有空闲位置便尝试写入数据，直到所有数据写入完成后返回。

   + 读进程阻塞，写进程非阻塞

     + 先执行fifo_send_1后执行fifo_rcv，写进程open函数返回-1

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1u7jxe9xej31vg082gq1.jpg" width="800" ></div>

     + 先执行fifo_rcv后执行fifo_send_1，结果正常

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1u7lnstdkj31vi08cn0l.jpg" width="800" ></div>

   + 读进程非阻塞，写进程阻塞

     + 先执行fifo_send后执行fifo_rcv_1,结果正常

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1u7s1xalwj31vg07sadq.jpg" width="800" ></div>

     + 先执行fifo_rcv_1后执行fifo_send，程序崩溃

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1u7vg5lzsj31vk07mdk6.jpg" width="800" ></div>

   + 读写进程都是非阻塞

     + 先执行fifo_send_1后执行fifo_rcv_1

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1u7ymttkbj31ve062n0z.jpg" width="800" ></div>

     + 先执行fifo_rcv_1后执行fifo_send_1

     <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1u811kn01j31ve06utc6.jpg" width="800" ></div>

#### 4.2.3 消息队列

1. 实验代码

   > 本实验代码文件分为Server.c和Client.c两个

   `Server.c:`

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <string.h>
   #include <unistd.h>
   #include <sys/types.h>
   #include <sys/msg.h>
   #include <sys/ipc.h>
   #include <signal.h>
   
   #define BUF_SIZE 128
   
   //Rebuild the strcut (must be)
   struct msgbuf
   {
       long mtype;
       char mtext[BUF_SIZE];
   };
   int main(int argc, char *argv[])
   {
       //1. creat a mseg queue
       key_t key;
       int msgId;
       
       key = ftok(".", 0xFF);
       msgId = msgget(key, IPC_CREAT|0644);
       if(-1 == msgId)
       {
           perror("msgget");
           exit(EXIT_FAILURE);
       }
   
       printf("Process (%s) is started, pid=%d\n", argv[0], getpid());
   
       while(1)
       {
           alarm(0);
           alarm(600);     //if doesn't receive messge in 600s, timeout & exit
           struct msgbuf rcvBuf;
           memset(&rcvBuf, '\0', sizeof(struct msgbuf));
           msgrcv(msgId, &rcvBuf, BUF_SIZE, 1, 0);                
           printf("Receive msg: %s\n", rcvBuf.mtext);
           
           struct msgbuf sndBuf;
           memset(&sndBuf, '\0', sizeof(sndBuf));
   
           strncpy((sndBuf.mtext), (rcvBuf.mtext), strlen(rcvBuf.mtext)+1);
           sndBuf.mtype = 2;
   
           if(-1 == msgsnd(msgId, &sndBuf, strlen(rcvBuf.mtext)+1, 0))
           {
               perror("msgsnd");
               exit(EXIT_FAILURE);
           }
               
           //if scanf "end~", exit
           if(!strcmp("end~", rcvBuf.mtext))
                break;
       }     
       printf("THe process(%s),pid=%d exit~\n", argv[0], getpid());
       return 0;
   }
   ```

   `Client.c:`

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <string.h>
   #include <unistd.h>
   #include <sys/types.h>
   #include <sys/msg.h>
   #include <sys/ipc.h>
   #include <signal.h>
   
   #define BUF_SIZE 128
   
   //Rebuild the strcut (must be)
   struct msgbuf
   {
       long mtype;
       char mtext[BUF_SIZE];
   };
   
   
   int main(int argc, char *argv[])
   {
       //1. creat a mseg queue
       key_t key;
       int msgId;
       
       printf("THe process(%s),pid=%d started~\n", argv[0], getpid());
   
       key = ftok(".", 0xFF);
       msgId = msgget(key, IPC_CREAT|0644);
       if(-1 == msgId)
       {
           perror("msgget");
           exit(EXIT_FAILURE);
       }
   
       //2. creat a sub process, wait the server message
       pid_t pid;
       if(-1 == (pid = fork()))
       {
           perror("vfork");
           exit(EXIT_FAILURE);
       }
   
       //In child process
       if(0 == pid)
       {
           while(1)
           {
               alarm(0);
               alarm(100);     //if doesn't receive messge in 100s, timeout & exit
               struct msgbuf rcvBuf;
               memset(&rcvBuf, '\0', sizeof(struct msgbuf));
               msgrcv(msgId, &rcvBuf, BUF_SIZE, 2, 0);                
               printf("Server said: %s\n", rcvBuf.mtext);
           }
           
           exit(EXIT_SUCCESS);
       }
   
       else    //parent process
       {
           while(1)
           {
               usleep(100);
               struct msgbuf sndBuf;
               memset(&sndBuf, '\0', sizeof(sndBuf));
               char buf[BUF_SIZE] ;
               memset(buf, '\0', sizeof(buf));
               
               printf("\nInput snd mesg: ");
               scanf("%s", buf);
               
               strncpy(sndBuf.mtext, buf, strlen(buf)+1);
               sndBuf.mtype = 1;
   
               if(-1 == msgsnd(msgId, &sndBuf, strlen(buf)+1, 0))
               {
                   perror("msgsnd");
                   exit(EXIT_FAILURE);
               }            
               //if scanf "end~", exit
               if(!strcmp("end~", buf))
                   break;
           }
           
           printf("THe process(%s),pid=%d exit~\n", argv[0], getpid());
       }
       return 0;
   }
   ```

2. 程序解释

   + 程序分为服务器端和客户端，客户端向服务器发起通信，服务器端收到数据后将一模一样的数据返回

   + 通过mgsrcv函数读取客户端传过来的消息，msgrcv的参数列表见下面。

     `int msgrcv(int msqid, void  *ptr, size_t  length, long  type, int  flag);`

   | 参数 |     msgid      |      ptr       |    length    |                             type                             |                          flag                          |
   | :--: | :------------: | :------------: | :----------: | :----------------------------------------------------------: | :----------------------------------------------------: |
   | 含义 | 消息队列标识符 | 消息缓冲区指针 | 消息数据长度 |                 决定从队列中返回那一条下消息                 |                        阻塞与否                        |
   | 备注 |                |                |              | =0 返回消息队列中第一条消息<br/>>0 返回消息队列中等于mtype 类型的第一条消息。
<0 返回mtype<=type 绝对值最小值的第一条消息。 | msgflg 为０表示阻塞方式，设置IPC_NOWAIT 表示非阻塞方式 |

   + 通过msgsnd函数向消息队列中加入消息，msgsnd的参数列表见下面。

   `int msgsnd(int  msqid, const  void   *ptr, size_t    length, int   flag);`

   | 参数 |     msgid      |      ptr       |    length    |                          flag                          |
   | :--: | :------------: | :------------: | :----------: | :----------------------------------------------------: |
   | 含义 | 消息队列标识符 | 消息缓冲区指针 | 消息数据长度 |                        阻塞与否                        |
   | 备注 |                |                |              | msgflg 为０表示阻塞方式，设置IPC_NOWAIT 表示非阻塞方式 |

   + 客户端的子进程主要负责消息的接受，父进程主要负责消息的发送；
   + 通过分析上面的代码可以知道，客户端和服务器端都是以阻塞的方式读取和写入消息

3. 程序运行结果

   <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1u9ayehs2j31vg0iogu3.jpg" width="800" ></div>

4. 探究消息队列的同步和阻塞机制

   > 通过上面的程序解释中可以看出，消息队列通过msgrcv和msgsnd两个函数的flag参数控制是否阻塞，将其设置为IPC_NOWAIT表示不阻塞；如果客户端和服务器端都设置阻塞话，就可以达到同步的目的

   现在做出如下探究：

   + 客服端不阻塞(代码为Client_1.c),服务器端阻塞，得到结果如下。

   <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1uaek4005j31ve06ygou.jpg" width="800" ></div>

   > 可以看到当客户端不阻塞的话在客户端接受服务器端消息的时候会无限制的打印消息队列中的空消息，哪怕消息队列中没有任何消息

   + 客户端阻塞，服务器端不阻塞(代码为Server_1.c)

   <div align="center"><img src="https://ws1.sinaimg.cn/large/006CotQ3ly1g1uaisf6n7j31vc0l6463.jpg" width="800" ></div>

   > 可以看到当服务器端没有设置阻塞的时候，服务器端会一直接受消息队列中的空消息并向客户端转发。 

## Task 5

> 本实验分析进程上下文切换的代码，说明实现的保存和恢复的上下文内容以及进程切换的工作流程。

我们首先从`devices/timer.c`文件中的timer_sleep函数开始 分析，下面是该函数的具体代码：

```c
/* Sleeps for approximately TICKS timer ticks.  Interrupts must
   be turned on. */
void timer_sleep (int64_t ticks) 
{
  int64_t start = timer_ticks ();
  ASSERT (intr_get_level () == INTR_ON);
  while (timer_elapsed (start) < ticks) 
    thread_yield ();
}
```

下面开始逐行分析这个函数，第5行的`timer_ticks`函数也在timer.c文件中，跳转到该函数中：

```c
/* Returns the number of timer ticks since the OS booted. */
int64_t timer_ticks (void) 
{
  enum intr_level old_level = intr_disable ();
  int64_t t = ticks;
  intr_set_level (old_level);
  return t;
}
```

`timer_ticks`函数中第4行涉及一个名为intr_disable()的函数，该函数的具体定义在`devices/interrupt.c`文件中。

```c
/* Disables interrupts and returns the previous interrupt status. */
enum intr_level intr_disable (void) 
{
  enum intr_level old_level = intr_get_level ();

  /* Disable interrupts by clearing the interrupt flag.
     See [IA32-v2b] "CLI" and [IA32-v3a] 5.8.1 "Masking Maskable
     Hardware Interrupts". */
  asm volatile ("cli" : : : "memory");
  return old_level;
}
```

在看看返回值`intr_level`是个什么结构,代码在`devices/interrupt.h`中：

```c
/* Interrupts on or off? */
enum intr_level 
  {
    INTR_OFF,             /* Interrupts disabled. */
    INTR_ON               /* Interrupts enabled. */
  };
```

可以发现，intr_level这个枚举类型表示的是是否允许中断。于是分析得到`intr_disable`函数做了两件事。1. 调用`intr_old_level`函数 2. 直接执行汇编代码保证这个线程不能被中断。之后返回调用`intr_old_level`函数的返回值。

再看看`intr_get_level`函数的实现细节，该函数的定义也在`devices/interrupt.c`文件中

```c
/* Returns the current interrupt status. */
enum intr_level intr_get_level (void) 
{
  uint32_t flags;
  /* Push the flags register on the processor stack, then pop the
     value off the stack into `flags'.  See [IA32-v2b] "PUSHF"
     and "POP" and [IA32-v3a] 5.8.1 "Masking Maskable Hardware
     Interrupts". */
  asm volatile ("pushfl; popl %0" : "=g" (flags));
  return flags & FLAG_IF ? INTR_ON : INTR_OFF;
}
```

通过注释信息和分析汇编代码可以知道，`intr_get_level`这个函数的作用是返回当前的中断状态。`intr_get_level`函数弄清楚了之后，返回上一层函数中，到了`intr_disable`函数中，这样就可以清楚的知道`intr_disable`函数的作用：

+ 获取当前中断状态
+ 将当前中断状态更改为不可中断
+ 返回先前的中断状态

弄清楚了`intr_disable`函数，接着看`timer_ticks`函数的5、6、7行

+ 第5行通过一个int64_t类型的变量t获取全局变量ticks的值；
+ 第6行`intr_set_level(old_level)`表示将当前中断状态设置为之前的中断状态。

+ 第7行返回t

这样，函数`timer_ticks`的含义也就弄清楚了。其实`timer_ticks`函数的作用很简单，就是想获取当前系统的ticks值而已，而上面通过这么大篇幅的介绍`timer_ticks`函数的4、6两行的作用，原因是第4行和第6行通过先关闭中断，待t获取到ticks值之后载恢复之前的中断状态，来保证操作的原子性，简单的说就是在t获取全局变量ticks的值的时候，不能被打断。

然后接着分析`timer_sleep`函数的第6行`ASSERT (intr_get_level () == INTR_ON);`这里是一个断言，当`intr_get_lvel`函数获取的当前中断状态不是`INTR_ON`的时候发生警告且退出。

`timer_sleep`函数剩下的就是一个循环了：

```c
while (timer_elapsed (start) < ticks) 
    thread_yield ();
```

通过分析不难得出`timer_elapsed()`函数的作用是计算当前的系统ticks减去之前得到的start的差值，如果这个差值小于函数参数ticks的话一直执行thread_yield()函数。

再看看`thread_yield`函数的具体定义（在`thread/thread.c文件中`），分析一下该函数的作用：

```c
/* Yields the CPU.  The current thread is not put to sleep and
   may be scheduled again immediately at the scheduler's whim. */
void thread_yield (void) 
{
  struct thread *cur = thread_current ();
  enum intr_level old_level;
  ASSERT (!intr_context ());
  old_level = intr_disable ();
  if (cur != idle_thread) 
    list_push_back (&ready_list, &cur->elem);
  cur->status = THREAD_READY;
  schedule ();
  intr_set_level (old_level);
}
```

`thread_yield`函数第5行顾名思义，作用就是返回当前正在运行的线程，通过一个thread类型的结构体指针接受该函数返回值。

`thread_yield`函数的第7行通过断言的方式判断中断类型，如果是由于I/O等引起的硬中断则退出，如果是软中断的话正常运行。

再看第8行和第13行的之前也分析过，这是保证9-12行操作的原子性。

再分析9-12行：

+ 9-10行：如何当前线程不是空闲的线程就调用list_push_back把当前线程的元素扔到就绪队列里面， 
+ 11行：把线程改成THREAD_READY状态
+ 12行：调用schedule函数

再深入`schedule`函数(`thread/thread.c文件`)看看

```c
/* Schedules a new process.  At entry, interrupts must be off and
   the running process's state must have been changed from
   running to some other state.  This function finds another
   thread to run and switches to it.
   It's not safe to call printf() until thread_schedule_tail()
   has completed. */
static void schedule (void) 
{
  struct thread *cur = running_thread ();
  struct thread *next = next_thread_to_run ();
  struct thread *prev = NULL;

  ASSERT (intr_get_level () == INTR_OFF);
  ASSERT (cur->status != THREAD_RUNNING);
  ASSERT (is_thread (next));

  if (cur != next)
    prev = switch_threads (cur, next);
  thread_schedule_tail (prev);
}
```

`schedule`函数首先获取当前正在运行的线程指针cur和下一个运行的线程next，之后是三个断言。

+ `ASSERT (intr_get_level () == INTR_OFF)`：保证中断状态是开启的
+ `ASSERT (cur->status != THREAD_RUNNING)`：保证当前运行的线程是RUNNING_THREAD的
+ `ASSERT (is_thread (next))`：保证下一个线程有效

17-18行的作用是：如果当前线程和下一个要跑的线程不是同一个的话调用switch_threads返回给prev

下面再看看`switch_threads`函数(在`threads/switch.S`中)这是一个完全由汇编语言编写的函数

```assembly
#include "threads/switch.h"

#### struct thread *switch_threads (struct thread *cur, struct thread *next);
####
#### Switches from CUR, which must be the running thread, to NEXT,
#### which must also be running switch_threads(), returning CUR in
#### NEXT's context.
####
#### This function works by assuming that the thread we're switching
#### into is also running switch_threads().  Thus, all it has to do is
#### preserve a few registers on the stack, then switch stacks and
#### restore the registers.  As part of switching stacks we record the
#### current stack pointer in CUR's thread structure.

.globl switch_threads
.func switch_threads
switch_threads:
	# Save caller's register state.
	#
	# Note that the SVR4 ABI allows us to destroy %eax, %ecx, %edx,
	# but requires us to preserve %ebx, %ebp, %esi, %edi.  See
	# [SysV-ABI-386] pages 3-11 and 3-12 for details.
	#
	# This stack frame must match the one set up by thread_create()
	# in size.
	pushl %ebx
	pushl %ebp
	pushl %esi
	pushl %edi

	# Get offsetof (struct thread, stack).
.globl thread_stack_ofs
	mov thread_stack_ofs, %edx

	# Save current stack pointer to old thread's stack, if any.
	movl SWITCH_CUR(%esp), %eax
	movl %esp, (%eax,%edx,1)

	# Restore stack pointer from new thread's stack.
	movl SWITCH_NEXT(%esp), %ecx
	movl (%ecx,%edx,1), %esp

	# Restore caller's register state.
	popl %edi
	popl %esi
	popl %ebp
	popl %ebx
        ret
.endfunc

.globl switch_entry
.func switch_entry
switch_entry:
	# Discard switch_threads() arguments.
	addl $8, %esp

	# Call thread_schedule_tail(prev).
	pushl %eax
.globl thread_schedule_tail
	call thread_schedule_tail
	addl $4, %esp

	# Start thread proper.
	ret
.endfunc
```

分析这段汇编代码，首先将4个寄存器的值压栈保护寄存器状态，这四个寄存器的值是`switch_threads_frame`的成员，`switch_threads_frame`结构的具体定义如下(`thread/switch.h`中定义)：

```c
/* switch_thread()'s stack frame. */
struct switch_threads_frame 
  {
    uint32_t edi;               /*  0: Saved %edi. */
    uint32_t esi;               /*  4: Saved %esi. */
    uint32_t ebp;               /*  8: Saved %ebp. */
    uint32_t ebx;               /* 12: Saved %ebx. */
    void (*eip) (void);         /* 16: Return address. */
    struct thread *cur;         /* 20: switch_threads()'s CUR argument. */
    struct thread *next;        /* 24: switch_threads()'s NEXT argument. */
  };
```

全局变量`thread_stack_ofs`记录线程和栈之间的间隙，下面我们来看线程切换中保存现场的过程。

+ 35-36行：先把当前的线程指针放到eax中， 并把线程指针保存在相对基地址偏移量为edx的地址中

+ 40-41: 切换到下一个线程的线程栈指针， 保存在ecx中， 再把这个线程相对基地址偏移量edx地址（上一次保存现场的时候存放的）放到esp当中继续执行。

  > 这里ecx, eax起容器的作用， edx指向当前现场保存的地址偏移量。简单来说就是保存当前线程状态， 恢复新线程之前保存的线程状态。

由此我们可以看出`schedule`函数是先将当前线程放入就绪队列，如果下一个线程和当前线程不一样的话切换到下一个线程。

再看看`shcedule`函数最后一行执行的操作，最后一行调用`thread_schedule_tail`函数，下面详细分析一下这个函数（`thread/thread.c`文件中）。

```c
void thread_schedule_tail (struct thread *prev)
{
  struct thread *cur = running_thread ();
  
  ASSERT (intr_get_level () == INTR_OFF);

  /* Mark us as running. */
  cur->status = THREAD_RUNNING;

  /* Start new time slice. */
  thread_ticks = 0;

#ifdef USERPROG
  /* Activate the new address space. */
  process_activate ();
#endif

  /* If the thread we switched from is dying, destroy its struct
     thread.  This must happen late so that thread_exit() doesn't
     pull out the rug under itself.  (We don't free
     initial_thread because its memory was not obtained via
     palloc().) */
  if (prev != NULL && prev->status == THREAD_DYING && prev != initial_thread) 
    {
      ASSERT (prev != cur);
      palloc_free_page (prev);
    }
}
```

首先是获得当前线程的的cur(切换之后的线程)，然后将cur的状态改为`THREAD_RUNNING`，然后thread_ticks清零开始新的线程切换时间片。然后调用diaoyong`process_activate`函数申请新的地址空间，再分析`process_active`函数(在`useruserprog/process.c`文件中定义)

```c
/* Sets up the CPU for running user code in the current
   thread.
   This function is called on every context switch. */
void process_activate (void)
{
  struct thread *t = thread_current ();
  /* Activate thread's page tables. */
  pagedir_activate (t->pagedir);
  /* Set thread's kernel stack for use in processing
     interrupts. */
  tss_update ();
}
```

关键的就是`pagedir_activate()`函数和`tss_update`函数，这两个函数分别位于`userprog/pagedir.c`和`userprog/tss.c`文件中

下面再进入`pagedir_activate()`函数中查看。

```c
/* Loads page directory PD into the CPU's page directory base
   register. */
void pagedir_activate (uint32_t *pd) 
{
  if (pd == NULL)
    pd = init_page_dir;

  /* Store the physical address of the page directory into CR3
     aka PDBR (page directory base register).  This activates our
     new page tables immediately.  See [IA32-v2a] "MOV--Move
     to/from Control Registers" and [IA32-v3a] 3.7.5 "Base
     Address of the Page Directory". */
  asm volatile ("movl %0, %%cr3" : : "r" (vtop (pd)) : "memory");
}
```

这个汇编指令将当前线程的页目录指针存储到CR3（页目录表物理内存基地址寄存器）中，也就是说这个函数更新了现在的页目录表

再进入`tss_update`函数中：

```c
/* Sets the ring 0 stack pointer in the TSS to point to the end
   of the thread stack. */
void tss_update (void) 
{
  ASSERT (tss != NULL);
  tss->esp0 = (uint8_t *) thread_current () + PGSIZE;
}
```

tss指的是 task state segment， 叫任务状态段， 任务（进程）切换时的任务现场信息。这里其实是把TSS的一个栈指针指向了当前线程栈的尾部， 也就是更新了任务现场的信息和状态。

到此`process_activate`函数的分析完毕，它做了两件事：

+ 更新页目录表
+ 更新任务现场信息（tss）

在继续看`thread_schedule_tail`函数的最后4行：

```c
if (prev != NULL && prev->status == THREAD_DYING && prev != initial_thread) 
{
    ASSERT (prev != cur);
    palloc_free_page (prev);
}
```

这里是说如果我们切换的线程状态是THREAD_DYING（代表欲要销毁的线程）的话， 调用palloc_free_page（`thread/palloc.c`文件中定义）：

```c
/* Frees the page at PAGE. */
void palloc_free_page (void *page) 
{
  palloc_free_multiple (page, 1);
}
```

简单而言作用就是释放PAGE参数中的页面

到此，`thread_schedule_tail`函数分析完毕，其作用就是分配恢复之前执行的状态和现场， 如果当前线程死了就清空资源。

`schedule`函数的作用就是拿下一个线程切换过来继续运行。`thread_yield`函数的作用是shi把当前进程放在就绪队列里，调用`schedule`切换到下一个进程。

最后返回到最顶层的`timer_sleep`函数，他的作用就是在ticks的时间内nei，如果线程处于running状态就不断的把它放在就绪队列不让它执行。