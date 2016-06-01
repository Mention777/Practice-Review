# iOS开发笔记-多线程01
####理论部分:

一.进程

>1)概念:是指在系统中正在运行的一个应用程序,每个进程的存储空间是独立的(终端中输入top可以查看进程,按Q退出)
>
>**P.S**.**进程与应用程序的差别**:进程是有状态的,正在执行的
>
>2)进程是CPU进行资源分配与调度的基本单位

二.线程

>1)线程是CPU调度的基本单位(1个进程想执行任务,必须要有线程 ),一个进程中所有的任务都在线程中执行

>2)一个线程中的任务的执行是串行的

>**P.S.进程与线程的辨析** 
>
>>1)一个程序可以对应多个进程,一个进程中可以有多个线程,但至少要有一个线程,线程是进程的一条路径

>>2)同一个进程内的线程共享进程的资源

>>3)进程可以分配资源,而线程不可以

>3)多线程并发执行:其实是CPU快速地在多个线程之间调度(只是假象,因为同一时间,CPU只能处理一条线程),建议开辟线程数量为3-5条

>4)主线程:也称为UI线程,作用是:1.显示\刷新UI界面  2.处理UI事件  (所有和UI相关的操作都需要在主线程中执行)

>> 打印时,若显示的number等于1,则为主线程,其他数字,则为子线程

>**P.S.主线程注意点**

>>1.不要将耗时的任务放在主线程 

>>2.UI刷新操作要放在主线程中  

>>3.耗时的操作放在子线程中(后台线程)

三.pthread

```objc

说明：pthread的基本使用（需要包含头文件）
    //使用pthread创建线程对象
    pthread_t thread;
    //使用pthread创建线程
    //第一个参数：线程对象地址
    //第二个参数：线程属性
    //第三个参数：指向函数的指针
    //第四个参数：传递给该函数的参数
    pthread_create(&thread, NULL, run, NULL);

```

四.NSThread

>1.一个NSThread代表一条线程

>2.优先级高,cpu调用的概率高(影响的也是CPU调度到的概率)

```objc

1）NSThread创建线程的四种方式
        //第一种创建线程的方式：alloc initWithTarget.
        //特点：需要手动开启线程，可以拿到线程对象进行详细设置
            //创建线程
            /*
             第一个参数：目标对象
             第二个参数：选择器，线程启动要调用哪个方法
             第三个参数：前面方法要接收的参数（最多只能接收一个参数，没有则传nil）
             */
            NSThread *thread = [[NSThread alloc]initWithTarget:self selector:@selector(run:) object:@"hmx"];
             //启动线程
            [thread start];

        //第二种创建线程的方式：分离出一条子线程
        //特点：自动启动线程，无法对线程进行更详细的设置
            /*
             第一个参数：线程启动调用的方法
             第二个参数：目标对象
             第三个参数：传递给调用方法的参数
             */
            [NSThread detachNewThreadSelector:@selector(run:) toTarget:self withObject:@"我是分离出来的子线程"];

        //第三种创建线程的方式：后台线程
        //特点：自动启动线程，无法进行更详细设置
            [self performSelectorInBackground:@selector(run:) withObject:@"我是后台线程"];
        
        //第四种创建线程方法:alloc init
        //新建一个类 继承自NSThread,重写内部的main方法来封装任务
            NSThread *thread = [[NSThread alloc]init];
            //启动线程
            [thread start];

    2）设置线程的属性
        //设置线程的名称
        thread.name = @"线程A";

        //设置线程的优先级,注意线程优先级的取值范围为0.0~1.0之间，1.0表示线程的优先级最高,如果不设置该值，那么理想状态下默认为0.5
        thread.threadPriority = 1.0;

```

>3.线程状态相关问题:

>>     
    1.线程的各种状态：新建-就绪-运行-阻塞-死亡
    2.当线程的任务执行完毕之后就销毁了
    3.当线程放入可调度线程池中,CPU才会调度
    4.线程已经死了,是不能再重新打开的
    5.当线程解除阻塞状态时,会进入就绪状态,而不是运行状态

>4.线程安全相关问题:
>>      
        1. 互斥锁:@synchronized(self){//需要锁定的代码}(推荐使用self)
        2. 前提条件:多个线程可能会访问同一块资源
        3. 注意点:1)要注意加锁的位置
                 2)锁对象必须是对象,且全局唯一
                 3)加上互斥锁之后,就会使线程同步(即线程永远都是按一个顺序调用,例:一开始是2->1->3的顺序,之后依旧为2->1->3的顺序)
                 4)线程是需要消耗性能的
        4. 专业术语-线程同步
        5. 原子和非原子属性（是否对setter方法加锁）
           atomic 线程安全,需要消耗大量资源
           nonatomic 非线程安全,适合内存小的移动设备  (开发中声明为这个)

>5.计算代码段间的执行时间

```objc

        //第一种方法
            NSDate *start = [NSDate date];
            //2.根据url地址下载图片数据到本地（二进制数据）
            NSData *data = [NSData dataWithContentsOfURL:url];

            NSDate *end = [NSDate date];
            NSLog(@"第二步操作花费的时间为%f",[end timeIntervalSinceDate:start]);

        //第二种方法
            CFTimeInterval start = CFAbsoluteTimeGetCurrent();
            NSData *data = [NSData dataWithContentsOfURL:url];

            CFTimeInterval end = CFAbsoluteTimeGetCurrent();
            NSLog(@"第二步操作花费的时间为%f",end - start);

```
      
>6.回到主线程刷新UI
>
        //4.1 第一种方式
            [self performSelectorOnMainThread:@selector(showImage:) withObject:image waitUntilDone:YES];
        //4.2 第二种方式
            [self.imageView performSelectorOnMainThread:@selector(setImage:) withObject:image waitUntilDone:YES];
        //4.3 第三种方式
            [self.imageView performSelector:@selector(setImage:) onThread:[NSThread mainThread] withObject:image waitUntilDone:YES];
        }

五.PCD

>1.GCD基本知识
> 
    1） 两个核心概念:队列和任务:
        队列:用来存放任务(决定在哪个线程执行任务)
        任务:执行什么操作
    2） 同步函数和异步函数的区别:
        同步:1.只能在当前线程中执行,不具备开启新线程的能力
            2.执行任务的方式:当执行到我时必须等我执行完才能执行后面的任务
        异步:1.可以在新的线程中执行任务,具备开启新线程的能力
            2.执行任务的方式:可以不用等我执行完毕,就可以直接执行后面的任务
    3)  串行队列:1.取出一个任务后,等到该任务执行完毕之后,接着去第二个任务
                2.创建方式:a.自己创建  b.主队列
        并行队列:1.取出一个任务后,接着执行第二个任务
                2.创建方式:a.自己创建  b.全局并发队列
        主队列:1.在安排任务的时候,会先检查主线程的状态,如果主线程忙,那么久暂停调度直到空闲为止
              2.想要实现控制系统开几条线程时,只需要控制创建几个队列
              3.凡是放在主队列里的任务都在主线程完成
 2.GCD基本使用【重要】
>
    01 异步函数+并发队列：开启多条线程，并发执行任务
    02 异步函数+串行队列：开启一条线程，串行执行任务
    03 同步函数+并发队列：不开线程，串行执行任务
    04 同步函数+串行队列：不开线程，串行执行任务
    05 异步函数+主队列：不开线程，在主线程中串行执行任务
    06 同步函数+主队列：不开线程，串行执行任务（注意死锁发生）
       P.S若当前是子线程执行的同步函数加上主队列的方式,不会发生死锁
    07 注意同步函数和异步函数在执行顺序上面的差异
 3.GCD的线程间通信:  
 
```objc

        //0.获取一个全局的队列
        dispatch_queue_t queue = dispatch_get_global_queue(0, 0);

        //1.先开启一个线程，把下载图片的操作放在子线程中处理
        dispatch_async(queue, ^{

        //2.下载图片
            NSURL *url = [NSURL URLWithString:@"http://h.hiphotos.baidu.com/zhidao/pic/item/6a63f6246b600c3320b14bb3184c510fd8f9a185.jpg"];
            NSData *data = [NSData dataWithContentsOfURL:url];
            UIImage *image = [UIImage imageWithData:data];

            NSLog(@"下载操作所在的线程--%@",[NSThread currentThread]);

        //3.回到主线程刷新UI
            dispatch_async(dispatch_get_main_queue(), ^{
               self.imageView.image = image;
               //打印查看当前线程
                NSLog(@"刷新UI---%@",[NSThread currentThread]);
            });

        });

```

> 4.GCD相关注意点
>
    1. 开线程的两个条件:1.必须是异步函数 2.必须不是主队列(主线程)
    2. 放到主队列里的任务,必须要在主线程中执行(不一定在当前线程中执行,即即使在子线程中调用了主队列,还是子主线程中调用)

>5.GCD常用函数:

```objc
   
   1）栅栏函数（控制任务的执行顺序）
        dispatch_barrier_async(queue, ^{
            NSLog(@"--dispatch_barrier_async-");
        });
    /*
    P.S 注意点:1.栅栏函数可以控制线程的执行顺序
              2.栅栏函数不能使用全局并发队列
              3.栅栏函数在执行时是独占的
              4.dispatch_barrier_async == dispatch_async  这两个是等同的
    */
    2）延迟执行（延迟·控制在哪个线程执行）
          dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            NSLog(@"---%@",[NSThread currentThread]);
        });
    /*
    Q:GCD的延迟执行 是先等2秒再提交 OR 先提交再等2秒?
    A:先等2秒再提交(延迟提交)  因为任务提交到队列里就不好控制了
    */

    3）一次性代码（注意不能放到懒加载）
        -(void)once
        {
            //整个程序运行过程中只会执行一次
            //onceToken用来记录该部分的代码是否被执行过
            static dispatch_once_t onceToken;
            dispatch_once(&onceToken, ^{

                NSLog(@"-----");
            });
        }
    /*
    一次性代码的特点:1.整个程序运行过程中只会执行一次
                      2.它本身是线程安全的
    */

    4）快速迭代（开多个线程并发完成迭代操作）
           dispatch_apply(subpaths.count, queue, ^(size_t index) {
        });
     /*
     快速迭代:多个线程(子线程与主线程一起工作的)一起并发执行任务的--->对顺序没有要求时使用
     Q:快速迭代若不是从0开始,怎么处理
     A:初始值若不是0,则不需要使用GCD的快速迭代,使用for循环即可
     */
    
    5）队列组（同栅栏函数）--->调度组
        //创建队列组
        dispatch_group_t group = dispatch_group_create();
        //队列组中的任务执行完毕之后，执行该函数
        dispatch_group_notify(dispatch_group_t group,dispatch_queue_t queue,dispatch_block_t block);//这个方法本身也是异步的

    6）进入群组和离开群组
        dispatch_group_enter(group);//执行该函数后，后面异步执行的block会被gruop监听
        dispatch_group_leave(group);//异步block中，所有的任务都执行完毕，最后离开群组
       /*
       注意：dispatch_group_enter|dispatch_group_leave必须成对使用
            当需要监听多个任务时,则重复写一组即可
       */
       //死等方法:知道队列组中所有都执行完毕之后才过掉该方法(同步)
       //diepatch_group_wait(group,DISPATCH_TIME_FOREVER)
```
