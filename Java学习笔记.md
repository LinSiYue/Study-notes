Java学习笔记

## 一、锁

### 1、死锁

#### 1.1概念

在多道程序环境中，多个进程可以竞争（不是抢占）有限数量的资源。当一个进程申请资源时，如果这时没有可用资源，那么这个进程进入等待状态。有时，如果所申请的资源被其他等待进程占有，那么该等待进程有可能再也无法改变状态。这种情况称为**死锁**。

<img src="https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%AD%BB%E9%94%81/%E8%B5%84%E6%BA%90%E5%88%86%E9%85%8D%E5%9B%BE.png?raw=true"/>

​                                                                                     图1 资源分配图

![1570495710537](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%AD%BB%E9%94%81/%E6%8A%BD%E8%B1%A1%E5%9B%BE.png?raw=true)

​                                                                                      图2 抽象图

图1中，形成环P1—> R1 —> P2 一> R3 —> P3 —> R2 —> P1形成死锁。

> 在正常操作模式下，进程只能按如下顺序使用资源：
> * 申请：进程请求资源。如果申请不能立即被允许（例如，申请的资源正在被其他进程使用)，那么申请进程应等待，直到它能获得该资源为止。
> * 使用：进程对资源进行操作（例如，如果资源是打印机，那么进程就可以在打印机上打印了）。
> * 释放：进程释放资源。

#### 1.2死锁特征

必要条件
> 如果在一个系统中以下四个条件同时成立，那么就能引起死锁：
> 1) 互斥：至少有一个资源必须处于非共享模式，即一次只有一个进程可使用。如果另一进程申请该资源，那么申请进程应等到该资源释放为止。
> 2) 占有并等待：—个进程应占有至少一个资源，并等待另一个资源，而该资源为其他进程所占有。
> 3) 非抢占：资源不能被抢占，即资源只能被进程在完成任务后自愿释放。
> 4) 循环等待：有一组等待进程 {P0，P1，…，Pn}，P0 等待的资源为 P1 占有，P1 等待的资源为 P2 占有，……，Pn-1 等待的资源为 Pn 占有，Pn 等待的资源为 P0 占有。
我们强调所有四个条件必须同时成立才会出现死锁。循环等待条件意味着占有并等待条件，这样四个条件并不完全独立。

我们强调所有四个条件必须同时成立才会出现死锁。循环等待条件意味着占有并等待条件，这样四个条件并不完全独立。

代码示例：

```java
package com.lsy.sychronizedTest;

public class DeadLockTest {

    public static void main(String[] args) {
        A a = new A();

        B b = new B();

        new Thread(() ->{
            synchronized (a) {
                a.print();
                synchronized (b) {
                    b.print();
                }
            }
        }, "A").start();

        new Thread(() ->{
            synchronized (b) {
                b.print();
                synchronized (a) {
                    a.print();
                }
            }
        }, "B").start();
    }
}

class A{
    void print(){
        System.out.println("A");
    }
}

class B{
    void print(){
        System.out.println("B");
    }
}
```

#### 1.3解决死锁的基本方法

> 预防死锁：
> * 资源一次性分配：一次性分配所有资源，这样就不会再有请求了：（破坏请求条件）
> * 只要有一个资源得不到分配，也不给这个进程分配其他的资源：（破坏请保持条件）
> * 可剥夺资源：即当某进程获得了部分资源，但得不到其它资源，则释放已占有的资源（破坏不可剥夺条件）
> * 资源有序分配法：系统给每类资源赋予一个编号，每一个进程按编号递增的顺序请求资源，释放则相反（破坏环路等待条件）

1)、以确定的顺序获得锁

如果必须获取多个锁，那么在设计的时候需要充分考虑不同线程之前获得锁的顺序。针对两个特定的锁，开发者可以尝试按照锁对象的hashCode值大小的顺序，分别获得两个锁，这样锁总是会以特定的顺序获得锁，那么死锁也不会发生。问题变得更加复杂一些，如果此时有多个线程，都在竞争不同的锁，简单按照锁对象的hashCode进行排序（单纯按照hashCode顺序排序会出现“环路等待”），可能就无法满足要求了，这个时候开发者可以使用银行家算法，所有的锁都按照特定的顺序获取，同样可以防止死锁的发生。

2)、超时放弃

当使用synchronized关键词提供的内置锁时，只要线程没有获得锁，那么就会永远等待下去，然而Lock接口提供了boolean tryLock(long time, TimeUnit unit) throws InterruptedException方法，该方法可以按照固定时长等待锁，因此线程可以在获取锁超时以后，主动释放之前已经获得的所有的锁。通过这种方式，也可以很有效地避免死锁。

#### 1.4避免死锁

> * 预防死锁的几种策略，会严重地损害系统性能。因此在避免死锁时，要施加较弱的限制，从而获得 较满意的系统性能。由于在避免死锁的策略中，允许进程动态地申请资源。因而，系统在进行资源分配之前预先计算资源分配的安全性。若此次分配不会导致系统进入不安全的状态，则将资源分配给进程；否则，进程等待。其中最具有代表性的避免死锁算法是银行家算法。
> * 银行家算法：首先需要定义状态和安全状态的概念。系统的状态是当前给进程分配的资源情况。因此，状态包含两个向量Resource（系统中每种资源的总量）和Available（未分配给进程的每种资源的总量）及两个矩阵Claim（表示进程对资源的需求）和Allocation（表示当前分配给进程的资源）。安全状态是指至少有一个资源分配序列不会导致死锁。当进程请求一组资源时，假设同意该请求，从而改变了系统的状态，然后确定其结果是否还处于安全状态。如果是，同意这个请求；如果不是，阻塞该进程知道同意该请求后系统状态仍然是安全的。

#### 1.5检测死锁

> * 首先为每个进程和每个资源指定一个唯一的号码；
> * 然后建立资源分配表和进程等待表。
>
> ```shell
> # 查看进程以及进程编号
> jps -l
> 
> # 查看进程详情
> jstack <PID>
> ```
>
> 

#### 1.6解除死锁

当发现有进程死锁后，便应立即把它从死锁状态中解脱出来，常采用的方法有：

> * 剥夺资源：从其它进程剥夺足够数量的资源给死锁进程，以解除死锁状态；
> * 撤消进程：可以直接撤消死锁进程或撤消代价最小的进程，直至有足够的资源可用，死锁状态.消除为止；所谓代价是指优先级、运行代价、进程的重要性和价值等。

#### 1.7银行家算法

1) 算法思想：银行家算法是从当前状态出发，按照系统各类资源剩余量逐个检查各进程需要申请的资源量，找到一个各类资源申请量均小于等于系统剩余资源量的进程P1。然后分配给该P1进程所请求的资源，假定P1完成工作后归还其占有的所有资源，更新系统剩余资源状态并且移除进程列表中的P1，进而检查下一个能完成工作的客户，......。如果所有客户都能完成工作，则找到一个安全序列，银行家才是安全的。若找不到这样的安全序列，则当前状态不安全。

2) 相关数据结构

> * 可利用资源向量Available。这是一个含有m个元素的数组，其中的而每一个元素代表一类可利用资源数目，其初始值是系统中所配置的该类全部可用资源的数目，其数值随该类资源的分配和回收而动态的改变。如果Available[j]=K,则表示系统中现有Rj类资源K个。
> * 最大需求矩阵Max。这是一个n*m的矩阵，它定义了系统中n个进程中的每一个进程对m类资源的最大需求。如果Max[i,j]=K；则表示进程i需要Rj类资源的最大数目为K。
> * 分配矩阵Allocation。这也是一个n*m的矩阵，它定义了系统中每一类资源当前已分配给每一进程的资源数。如果Allocation[i,j]=K，则表示进程i当前已分得Rj类资源的数目为K。
> * 需求矩阵Need。这也是一个n*m的矩阵，用以表示每一个进程尚需的各类资源数。如果Need[i,j]=K,则表示进程i还需要Rj类资源K个，方能完成任务。

上述三个矩阵间存在下述关系：Need[i,j]=Max[i,j]-Allocation[i,j]

最后得出的分配顺序：（work是剩余资源，即Available）

![1570495786024](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%AD%BB%E9%94%81/%E9%93%B6%E8%A1%8C%E5%AE%B6%E7%AE%97%E6%B3%95.png?raw=true)

全为true，则表示没有死锁，可避免死锁。

### 2、锁

java中的锁，本次介绍的锁如下：

- 公平锁/非公平锁
- 可重入锁
- 独享锁/共享锁
- 互斥锁/读写锁
- 乐观锁/悲观锁
- 分段锁
- 偏向锁/轻量级锁/重量级锁
- 自旋锁

#### 2.1公平锁/非公平锁

**公平锁**是指多个线程按照**申请锁的顺序**来获取锁。
**非公平锁**是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。
对于Java `ReentrantLock`而言，通过构造函数指定该锁是否是公平锁，**默认是非公平锁**。

```
//默认是不公平锁，传入true为公平锁，否则为非公平锁
ReentrantLock reentrantLock =  new ReetrantLock();
```
非公平锁的优点在于吞吐量比公平锁大。
对于`Synchronized`而言，也是一种非公平锁。由于其并不像`ReentrantLock`是通过AQS的来实现线程调度，所以并没有任何办法使其变成公平锁。

#### 2.2共享锁和独享锁

**独享锁**是指该锁一次只能被一个线程所持有。
**共享锁**是指该锁可被多个线程所持有。

对于Java `ReentrantLock`而言，其是独享锁。但是对于Lock的另一个实现类`ReadWriteLock`，其**读锁是共享锁，其写锁是独享锁**。
读锁的共享锁可保证并发读是非常高效的，读写，写读 ，写写的过程是互斥的。
独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。
对于`Synchronized`而言，当然是独享锁。

#### 2.3互斥锁/读写锁

独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。
互斥锁在Java中的具体实现就是`ReentrantLock`
读写锁在Java中的具体实现就是`ReadWriteLock`

#### 2.4乐观锁和悲观锁

乐观锁与悲观锁不是指具体的什么类型的锁，而是指看待并发同步的角度。
**悲观锁**认为对于同一个数据的并发操作，一定是会发生修改的，哪怕没有修改，也会认为修改。因此对于同一个数据的并发操作，悲观锁采取加锁的形式。<u>悲观的认为，不加锁的并发操作一定会出问题。</u>
**乐观锁**则认为对于同一个数据的并发操作，是不会发生修改的。在更新数据的时候，会采用尝试更新，不断重新的方式更新数据。<u>乐观的认为，不加锁的并发操作是没有事情的。</u>基于CAS算法实现。

**悲观锁适合写操作非常多的场景，乐观锁适合读操作非常多的场景，不加锁会带来大量的性能提升**。
悲观锁在Java中的使用，就是利用各种锁。
乐观锁在Java中的使用，是无锁编程，常常采用的是CAS算法，典型的例子就是原子类，通过CAS自旋实现原子操作的更新。

#### 2.5分段锁

**分段锁**其实是一种锁的设计，并不是具体的一种锁，对于`ConcurrentHashMap`而言，其并发的实现就是通过分段锁的形式来实现高效的并发操作。
`ConcurrentHashMap`中的分段锁称为Segment，它即类似于HashMap（JDK7与JDK8中HashMap的实现）的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表；同时又是一个ReentrantLock（Segment继承了ReentrantLock)。
当需要put元素的时候，并不是对整个hashmap进行加锁，而是先通过hashcode来知道他要放在那一个分段中，然后对这个分段进行加锁，所以当多线程put的时候，只要不是放在一个分段中，就实现了真正的并行的插入。
但是，在统计size的时候，可就是获取hashmap全局信息的时候，就需要获取所有的分段锁才能统计。
分段锁的设计目的是细化锁的粒度，当操作不需要更新整个数组的时候，就仅仅针对数组中的一项进行加锁操作。

#### 2.6偏向锁/轻量级锁/重量级锁

这三种锁是指锁的状态，并且是针对`Synchronized`。在Java 5通过引入锁升级的机制来实现高效`Synchronized`。这三种锁的状态是通过对象监视器在对象头中的字段来表明的。
**偏向锁**是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁。降低获取锁的代价。
**轻量级锁**是指当锁是<u>偏向锁</u>的时候，<u>被另一个线程所访问</u>，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。乐观锁。多个线程在自己的线程栈中生成Lock Record的指针，然后竞争对象上的锁，当对象上有指向某个线程的Lock Record指针，则代表该线程获得锁成功。
**重量级锁**是指当锁为<u>轻量级锁</u>的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当<u>自旋一定次数（超过10次）的时候，或者自旋线程数超过CPU核数的一半，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁</u>。重量级锁会让其他申请的线程进入阻塞，性能降低。

锁的升级叫做锁的膨胀，膨胀只能由低到高，最后重新回到偏向锁重新膨胀。

#### 2.7自旋锁

在Java中，自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用**循环的方式去尝试获取锁**，这样的好处是<u>减少线程上下文切换的消耗，缺点是循环会消耗CPU</u>。

```java
public class SpinLock {

  private AtomicReference<Thread> sign =new AtomicReference<>();

  public void lock(){
    Thread current = Thread.currentThread();
    while(!sign .compareAndSet(null, current)){
    }
  }

  public void unlock (){
    Thread current = Thread.currentThread();
    sign.compareAndSet(current, null);
  }
}
```

#### 2.8可重入锁

**可重入锁**又名<u>递归锁</u>，是指在同一个线程<u>在外层方法获取锁的时候，在进入内层方法会自动获取锁</u>。说的有点抽象，下面会有一个代码的示例。
对于Java `ReentrantLock`而言, 他的名字就可以看出是一个可重入锁，其名字是`Re entrant Lock`重新进入锁。
对于`Synchronized`而言,也是一个可重入锁。可重入锁的一个好处是可一定程度避免死锁。

```
synchronized void setA() throws Exception{
    Thread.sleep(1000);
    setB();
}

synchronized void setB() throws Exception{
    Thread.sleep(1000);
}
```

上面的代码就是一个可重入锁的一个特点，如果不是可重入锁的话，setB可能不会被当前线程执行，可能造成死锁。

### 3、synchronized是锁住对象

* Synchronized锁的是对象，static synchronized锁的是类，上锁就是改变对象的对象头。
* 实现过程：
  * java代码：sychronized
  * 字节码：monitorenter    monitorexit
  * 执行过程中自动升级
  * lock comxchg

```java
class Lock {
	//同把锁，无论创建多少对象去调用方法，都是同把锁，因为静态同步方法锁的是类
	// a b
	static synchronized void a(){
		System.out.println("a");
	}
	static synchronized void b(){
		System.out.println("b");
	}
	
	// 不同锁，同个对象或不同对象都用不同锁
	// d c
	stactic synchronized void c(){
		System.out.println("c");
	}
	synchronized void d(){
		System.out.println("d");
	}
	
	// 同个对象则同把锁，不同对象则不同锁
	// 同对象：e f；不同对象：f e
	synchronized void e(){
		System.out.println("e");
	}
	synchronized void f(){
		System.out.println("f");
	}
}
```

### 4. Sychronized和ReentrantLock

```java
// Sychronized
class Count{
    static volatile int m = 0;

    synchronized void add() throws InterruptedException {
        while (m!=0){
            this.wait();
        }
        m++;
        System.out.println(m);
        this.notifyAll();
    }

    synchronized void desc() throws InterruptedException {
        while (m==0){
            this.wait();
        }
        m--;
        System.out.println(m);
        this.notifyAll();
    }
}
```

```java
// ReentrantLock搭配condition
class Operation {
    ReentrantLock lock = new ReentrantLock();
    Condition condition1 = lock.newCondition();
    Condition condition2 = lock.newCondition();

    static volatile int m = 0;

    void add() throws InterruptedException {
        lock.lock();
        while (m != 0) {
            condition1.await();
        }
        m++;
        System.out.println(m);
        condition2.signal();
        lock.unlock();
    }

    void desc() throws InterruptedException {
        lock.lock();
        while (m == 0) {
            condition2.await();
        }
        m--;
        System.out.println(m);
        condition1.signal();
        lock.unlock();
    }
}
```

```java
public class SynchronizedCountTest {
	
    // 完成一个线程+1，一个线程-1的操作
    public static void main(String[] args) {
		
        Count count = new Count();
        // Operation op = new Operation();

        for (int i = 0; i < 10; i++) {
            new Thread(() ->{
                try {
                    count.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "A").start();
        }

        for (int i = 0; i < 10; i++) {
            new Thread(() ->{
                try {
                    count.desc();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "B").start();
        }
    }

}
```

## 二、十大经典排序算法

**术语说明**
* **稳定**：如果a原本在b前面，而a=b，排序之后a仍然在b的前面；

* **不稳定**：如果a原本在b的前面，而a=b，排序之后a可能会出现在b的后面；

* **内排序**：所有排序操作都在内存中完成；

* **外排序**：由于数据太大，因此把数据放在磁盘中，而排序通过磁盘和内存的数据传输才能进行；

* **时间复杂度：** 一个算法执行所耗费的时间。

* **空间复杂度**：运行完一个程序所需内存的大小。

  **算法总结**
  ![clip_image001.jpg](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image001.png?raw=true)

**图片名词解释：**
* n: 数据规模
* k: “桶”的个数
* In-place: 占用常数内存，不占用额外内存
* Out-place: 占用额外内存

**算法分类**

![clip_image002.jpg](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image002.png?raw=true)

####  

比较和非比较的区别

常见的**快速排序、归并排序、堆排序、冒泡排序**等属于**比较排序**。**在排序的最终结果里，元素之间的次序依赖于它们之间的比较。每个数都必须和其他数进行比较，才能确定自己的位置。**
 在**冒泡排序**之类的排序中，问题规模为n，又因为需要比较n次，所以平均时间复杂度为O(n²)。在**归并排序、快速排序**之类的排序中，问题规模通过**分治法**消减为logN次，所以时间复杂度平均**O(nlogn)**。
 比较排序的优势是，适用于各种规模的数据，也不在乎数据的分布，都能进行排序。可以说，**比较排序适用于一切需要排序的情况。**

**计数排序、基数排序、桶排序**则属于**非比较排序**。非比较排序是通过确定每个元素之前，应该有多少个元素来排序。针对数组arr，计算arr[i]之前有多少个元素，则唯一确定了arr[i]在排序后数组中的位置。
 非比较排序只要确定每个元素之前的已有的元素个数即可，所有一次遍历即可解决。算法时间复杂度**O(n)**。
 **非比较排序时间复杂度底，但由于非比较排序需要占用空间来确定唯一位置。所以对数据规模和数据分布有一定的要求。**



### **算法**
### 1、冒泡排序（Bubble Sort）
#### 1.1 算法描述
* 比较相邻的元素。如果第一个比第二个大，就交换它们两个；
* 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对，这样在最后的元素应该会是最大的数；
* 针对所有的元素重复以上的步骤，除了最后一个；
* 重复步骤1~3，直到排序完成。

#### 1.2 代码实现
```java
/**
* 冒泡排序
* @param array
* @return
*/
public static int[] bubbleSort(int[] array) {
    if (array.length == 0)
		return array;
    for (int i = 0; i < array.length; i++) {
        for (int j = 0; j < array.length - 1 - i; j++) {
            if (array[j + 1] < array[j]) {
                int temp = array[j + 1];
                array[j + 1] = array[j];
                array[j] = temp;
            }                
        }
    }
    return array;
}
```

#### 1.3 算法分析

**最佳情况：**T(n) = O(n)   **最差情况：**T(n) = O(n^2)   **平均情况：**T(n) = O(n^2)

### 2、选择排序（Selection Sort）

表现**最稳定的排序算法之一**，因为无论什么数据进去都是O(n2)的时间复杂度，所以用到它的时候，数据规模越小越好。唯一的好处可能就是不占用额外的内存空间了吧。

#### 2.1 算法描述
n个记录的直接选择排序可经过n-1趟直接选择排序得到有序结果。具体算法描述如下：
* 初始状态：无序区为R[1..n]，有序区为空；
* 第i趟排序(i=1,2,3…n-1)开始时，当前有序区和无序区分别为R[1..i-1]和R(i..n）。该趟排序从当前无序区中-选出关键字最小的记录 R[k]，将它与无序区的第1个记录R交换，使R[1..i]和R[i+1..n)分别变为记录个数增加1个的新有序区和记录个数减少1个的新无序区；
* n-1趟结束，数组有序化了。
#### 2.2 代码实现
```java
/**
* 选择排序
* @param array
* @return
*/
public static int[] selectionSort(int[] array) {
	if (array.length == 0)
		return array;
	for (int i = 0; i < array.length; i++) {
		int minIndex = i;
		for (int j = i; j < array.length; j++) {
			if (array[j] < array[minIndex]) //找到最小的数
 				minIndex = j; //将最小数的索引保存
		}
        int temp = array[minIndex];
        array[minIndex] = array[i];
        array[i] = temp;
	}
 	return array;
}
```

#### 2.3 算法分析

最佳情况：T(n) = O(n^2)  最差情况：T(n) = O(n^2)  平均情况：T(n) = O(n^2)

### 3、插入排序（Insertion Sort）

#### 3.1 算法描述
一般来说，插入排序都采用in-place在数组上实现。具体算法描述如下：
* 从第一个元素开始，该元素可以认为已经被排序；
* 取出下一个元素，在已经排序的元素序列中从后向前扫描；
* 如果该元素（已排序）大于新元素，将该元素移到下一位置；
* 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置；
* 将新元素插入到该位置后；
* 重复步骤2~5。
#### 3.2 代码实现
```java
/**
* 插入排序
* @param array
* @return
*/
public static int[] insertionSort(int[] array) {
	if (array.length == 0)
		return array;
	int current;
	for (int i = 0; i < array.length - 1; i++) {
		current = array[i + 1];
		int preIndex = i;
		while (preIndex >= 0 && current < array[preIndex]) {
			array[preIndex + 1] = array[preIndex];
			preIndex--;
        }
		array[preIndex + 1] = current;
    }
	return array;
}
```
#### 3.3 算法分析
最佳情况：T(n) = O(n)   最坏情况：T(n) = O(n^2)   平均情况：T(n) = O(n^2)

### 4、快速排序（Quick Sort）

#### 4.1 算法描述

快速排序使用分治法来把一个串（list）分为两个子串（sub-lists）。具体算法描述如下：
* 从数列中挑出一个元素，称为 “基准”（pivot）；
* 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作；
* 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

#### 4.2 代码实现
```java
/**
* 快速排序（挖坑填数+分治）
* @param array
* @return
*/
public void sort(int[] array) {
	quickSort(0, array.length - 1, array);
}

public void quickSort(int left, int right, int[] array) {
	if (left < right) {
		int x = left, y = right, s = array[x];
		while (x < y) {
			while (x < y && array[y] >= s) {
				y--;
			}
			if (x < y) {
				array[x++] = array[y];
			}
			while (x < y && array[x] <= s) {
				x++;
			}
			if (x < y) {
            	array[y--] = array[x];
            }
		}
        array[x] = s;
        quickSort(left, x - 1, array);
        quickSort(x + 1, right, array);
	}
}
```
#### 4.3 算法分析
最佳情况：T(n) = O(nlogn)   最差情况：T(n) = O(n^2)   平均情况：T(n) = O(nlogn)　

### 5、希尔排序（Shell Sort）

希尔排序也是一种插入排序，它是简单插入排序经过改进之后的一个更高效的版本，也称为缩小增量排序，同时该算法是冲破O(n2）的第一批算法之一。

希尔排序是把记录按下表的一定增量分组，对每组使用直接插入排序算法排序；随着增量逐渐减少，每组包含的关键词越来越多，当增量减至1时，整个文件恰被分成一组，算法便终止。

#### 5.1 算法描述

我们来看下希尔排序的基本步骤，在此我们选择增量gap=length/2，缩小增量继续以gap = gap/2的方式，这种增量选择我们可以用一个序列来表示，{n/2,(n/2)/2...1}，称为增量序列。希尔排序的增量序列的选择与证明是个数学难题，我们选择的这个增量序列是比较常用的，也是希尔建议的增量，称为希尔增量，但其实这个增量序列不是最优的。此处我们做示例使用希尔增量。

先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，具体算法描述：
* 选择一个增量序列t1，t2，…，tk，其中ti>tj，tk=1；
* 按增量序列个数k，对序列进行k 趟排序；
* 每趟排序，根据对应的增量ti，将待排序列分割成若干长度为m 的子序列，分别对各子表进行直接插入排序。仅增量因子为1 时，整个序列作为一个表来处理，表长度即为整个序列的长度。

#### 5.2 过程演示

![clip_image003.jpg](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image003.jpg?raw=true)

#### 5.3 代码实现
```java
/**
* 希尔排序
* @param array
* @return
*/
public static int[] ShellSort(int[] array) {
    int len = array.length;
    int temp, gap = len / 2;
    while (gap > 0) {
        for (int i = gap; i < len; i++) {
        	temp = array[i];
        	int preIndex = i - gap;
            while (preIndex >= 0 && array[preIndex] > temp) {
            	array[preIndex + gap] = array[preIndex];
            	preIndex -= gap;
        	}
        	array[preIndex + gap] = temp;
        }
    	gap /= 2;
    }
    return array;
}
```
#### 5.4 算法分析
最佳情况：T(n) = O(nlog2 n)  最坏情况：T(n) = O(nlog2 n)  平均情况：T(n) =O(nlog2n)

### 6、归并排序（Merge Sort）

和选择排序一样，归并排序的性能不受输入数据的影响，但表现比选择排序好的多，因为始终都是O(n log n）的时间复杂度。代价是需要额外的内存空间。

#### 6.1 算法描述
* 把长度为n的输入序列分成两个长度为n/2的子序列；
* 对这两个子序列分别采用归并排序；
* 将两个排序好的子序列合并成一个最终的排序序列。

#### 6.2 代码实现
```java
/**
* 归并排序
* @param array
* @return
*/
public static int[] MergeSort(int[] array) {
    if (array.length < 2) return array;
    int mid = array.length / 2;
    int[] left = Arrays.copyOfRange(array, 0, mid);
    int[] right = Arrays.copyOfRange(array, mid, array.length);
    return merge(MergeSort(left), MergeSort(right));
}

/**
* 归并排序——将两段排序好的数组结合成一个排序数组
* @param left
* @param right
* @return
*/
public static int[] merge(int[] left, int[] right) {
    int[] result = new int[left.length + right.length];
    for (int index = 0, i = 0, j = 0; index < result.length; index++) {
        //当左边数组全部放到result数组时，将右边数组剩下的全部放入
        if (i >= left.length)
        	result[index] = right[j++];
        //当右边数组全部放到result数组时，将左边数组剩下的全部放入
        else if (j >= right.length) 
        	result[index] = left[i++];
        else if (left[i] > right[j]) //哪边的数组值小就放入result里面
        	result[index] = right[j++];
        else
        	result[index] = left[i++];
    }
    return result;
}
```
#### 6. 3 算法分析
最佳情况：T(n) = O(n)  最差情况：T(n) = O(nlogn)  平均情况：T(n) = O(nlogn)

### 7、堆排序（Heap Sort）

堆排序（Heapsort）是指利用堆这种数据结构所设计的一种排序算法。堆积是一个近似完全二叉树的结构，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。

完全二叉树有个特性：左边子节点位置 = 当前父节点的两倍 + 1，右边子节点位置 = 当前父节点的两倍 + 2。

#### 7.1 算法描述
* 将初始待排序关键字序列(R1,R2….Rn)构建成大顶堆，此堆为初始的无序区；
* 将堆顶元素R[1]与最后一个元素R[n]交换，此时得到新的无序区(R1,R2,……Rn-1)和新的有序区(Rn),且满足R[1,2…n-1]<=R[n]；
* 由于交换后新的堆顶R[1]可能违反堆的性质，因此需要对当前无序区(R1,R2,……Rn-1)调整为新堆，然后再次将R[1]与无序区最后一个元素交换，得到新的无序区(R1,R2….Rn-2)和新的有序区(Rn-1,Rn)。不断重复此过程直到有序区的元素个数为n-1，则整个排序过程完成。

#### 7.2 图片演示

![clip_image004.jpg](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image004.jpg?raw=true)

![clip_image005.jpg](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image005.jpg?raw=true)

![clip_image006.jpg](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image006.jpg?raw=true)

#### 7.3 代码实现
```java
//声明全局变量，用于记录数组array的长度；
static int len;

/**
* 堆排序算法
* @param array
* @return
*/
public static int[] HeapSort(int[] array) {
    len = array.length;
    if (len < 1) return array;
    //1.构建一个最大堆
    buildMaxHeap(array);
    //2.循环将堆首位（最大值）与末位交换，然后在重新调整最大堆
    while (len > 0) {
        swap(array, 0, len - 1);
        len--;
        adjustHeap(array, 0);
    }
    return array;
}

/**
* 建立最大堆
* @param array
*/
public static void buildMaxHeap(int[] array) {
    //从最后一个非叶子节点开始向上构造最大堆
    for (int i = (len/2 - 1); i >= 0; i--) {
    	adjustHeap(array, i);
    }
}

/**
* 调整使之成为最大堆
* @param array
* @param i
*/
public static void adjustHeap(int[] array, int i) {
    int maxIndex = i;
    //如果有左子树，且左子树大于父节点，则将最大指针指向左子树
    if (i * 2 < len && array[i * 2] > array[maxIndex])
    	maxIndex = i * 2;
    //如果有右子树，且右子树大于父节点，则将最大指针指向右子树
    if (i * 2 + 1 < len && array[i * 2 + 1] > array[maxIndex])
    	maxIndex = i * 2 + 1;
    //如果父节点不是最大值，则将父节点与最大值交换，并且递归调整与父节点交换的位置。
    if (maxIndex != i) {
    	swap(array, maxIndex, i);
    	adjustHeap(array, maxIndex);
    }
}

public static void swap(int[] array, int maxIndex, int i){
    int temp = array[maxIndex];
    array[maxIndex] = array[i];
    array[i] = temp;
}
```
#### 7.4 算法分析

最佳情况：T(n) = O(nlogn) 最差情况：T(n) = O(nlogn) 平均情况：T(n) = O(nlogn)

### 8、计数排序（Counting Sort）

计数排序(Counting sort)是一种稳定的排序算法。计数排序的核心在于将输入的数据值转化为键存储在额外开辟的数组空间中。 作为一种线性时间复杂度的排序，计数排序要求输入的数据必须是有确定范围的整数。

#### 8.1 算法描述
* 找出待排序的数组中最大和最小的元素；
* 统计数组中每个值为i的元素出现的次数，存入数组C的第i项；
* 对所有的计数累加（从C中的第一个元素开始，每一项和前一项相加）；
* 反向填充目标数组：将每个元素i放在新数组的第C(i)项，每放一个元素就将C(i)减去1。

#### 8.2 代码实现
```java
/**
* 计数排序
* @param array
* @return
*/
public static int[] CountingSort(int[] array) {
    if (array.length == 0) return array;
    int bias, min = array[0], max = array[0];
    for (int i = 1; i < array.length; i++) {
        if (array[i] > max)
        max = array[i];
        if (array[i] < min)
        min = array[i];
	}
    bias = 0 - min;
    int[] bucket = new int[max - min + 1];
    Arrays.fill(bucket, 0);
    for (int i = 0; i < array.length; i++) {
    	bucket[array[i] + bias]++;
	}
    int index = 0, i = 0;
    while (index < array.length) {
        if (bucket[i] != 0) {
        array[index] = i - bias;
        bucket[i]--;
        index++;
    } else
    	i++;
    }
    return array;
}
```
#### 8.4 算法分析

当输入的元素是n 个0到k之间的整数时，它的运行时间是 O(n + k)。计数排序不是比较排序，排序的速度快于任何比较排序算法。由于用来计数的数组C的长度取决于待排序数组中数据的范围（等于待排序数组的最大值与最小值的差加上1），这使得计数排序对于数据范围很大的数组，需要大量时间和内存。

最佳情况：T(n) = O(n+k)  最差情况：T(n) = O(n+k)  平均情况：T(n) = O(n+k)

### 9、桶排序（Bucket Sort）
桶排序 (Bucket sort)的工作的原理：假设输入数据服从均匀分布，将数据分到有限数量的桶里，每个桶再分别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排序。

#### 9.1 算法描述
* 人为设置一个BucketSize，作为每个桶所能放置多少个不同数值（例如当BucketSize==5时，该桶可以存放｛1,2,3,4,5｝这几种数字，但是容量不限，即可以存放100个3）；
* 遍历输入数据，并且把数据一个一个放到对应的桶里去；
* 对每个不是空的桶进行排序，可以使用其它排序方法，也可以递归使用桶排序；
* 从不是空的桶里把排好序的数据拼接起来。 

**注意，如果递归使用桶排序为各个桶排序，则当桶数量为1时要手动减小BucketSize增加下一循环桶的数量，否则会陷入死循环，导致内存溢出。**

#### 9.2 图片演示

![clip_image007.png](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image007.png?raw=true)

#### 9.3 代码实现
```java
/**
* 桶排序
* @param array
* @param bucketSize
* @return
*/
public static ArrayList<Integer> BucketSort(ArrayList<Integer> array, int bucketSize) {
    if (array == null || array.size() < 2)
    	return array;
    int max = array.get(0), min = array.get(0);
    // 找到最大值最小值
    for (int i = 0; i < array.size(); i++) {
    	if (array.get(i) > max)
        	max = array.get(i);
        if (array.get(i) < min)
        	min = array.get(i);
        }
        //当数组里都是相同的数值时
        if(max == min){
        	return array;
    	}
        //定义桶的个数，bucketSize是每个小桶能放置的不同数值的个数；
        //例如当BucketSize==5时，该桶可以存放｛1,2,3,4,5｝这几种数字，但是容量不
        //限，即可以存放100个3）
        int bucketCount = (max - min) / bucketSize + 1;
		//建一个能容纳这些桶的大桶
        ArrayList<ArrayList<Integer>> bucketArr = new ArrayList<>(bucketCount);
        ArrayList<Integer> resultArr = new ArrayList<>();
        //在大桶中放入空桶
        for (int i = 0; i < bucketCount; i++) {
        	bucketArr.add(new ArrayList<Integer>());
		}
        //将数据放入对应的小桶，实际值-最小值就能匹配0为初始值的桶，除以小桶能放置的
        //不同数值的个数就能得出应该放在哪个小桶中
        for (int i = 0; i < array.size(); i++) {
        	bucketArr.get((array.get(i) - min) / bucketSize).add(array.get(i));
        }
        for (int i = 0; i < bucketCount; i++) {
            if (bucketSize == 1) { 
                for (int j = 0; j < bucketArr.get(i).size(); j++)
                	resultArr.add(bucketArr.get(i).get(j));
            } else {
                if (bucketCount == 1)
                	bucketSize--;
                ArrayList<Integer> temp = BucketSort(bucketArr.get(i), bucketSize);
                for (int j = 0; j < temp.size(); j++)
                	resultArr.add(temp.get(j));
		}
	}
	return resultArr;
}
```
#### 9.4 算法分析

桶排序最好情况下使用线性时间O(n)，桶排序的时间复杂度，取决与对各个桶之间数据进行排序的时间复杂度，因为其它部分的时间复杂度都为O(n)。很显然，桶划分的越小，各个桶之间的数据越少，排序所用的时间也会越少。但相应的空间消耗就会增大。 

最佳情况：T(n) = O(n+k)   最差情况：T(n) = O(n+k)   平均情况：T(n) = O(n^2)　

### 10、基数排序（Radix Sort）

基数排序也是非比较的排序算法，对每一位进行排序，从最低位开始排序，复杂度为O(kn),为数组长度，k为数组中的数的最大的位数；

基数排序是按照低位先排序，然后收集；再按照高位排序，然后再收集；依次类推，直到最高位。有时候有些属性是有优先级顺序的，先按低优先级排序，再按高优先级排序。最后的次序就是高优先级高的在前，高优先级相同的低优先级高的在前。基数排序基于分别排序，分别收集，所以是稳定的。

#### 10.1 算法描述
* 取得数组中的最大数，并取得位数；
* arr为原始数组，从最低位开始取每个位组成radix数组；
* 对radix进行计数排序（利用计数排序适用于小范围数的特点）；

#### 10.2 代码实现
```java
/**
* 基数排序
* @param array
* @return
*/
public static int[] RadixSort(int[] array) {
    if (array == null || array.length < 2)
    	return array;
    // 1.先算出最大数的位数；
    int max = array[0];
    for (int i = 1; i < array.length; i++) {
    	max = Math.max(max, array[i]);
    }
    int maxDigit = 0;
    while (max != 0) {
    	max /= 10;
    	maxDigit++;
    }
    int mod = 10, div = 1;
    ArrayList<ArrayList<Integer>> bucketList = new ArrayList<ArrayList<Integer>>();
    for (int i = 0; i < 10; i++)
    	bucketList.add(new ArrayList<Integer>());
    for (int i = 0; i < maxDigit; i++, mod *= 10, div *= 10) {
        for (int j = 0; j < array.length; j++) {
            int num = (array[j] % mod) / div;
            bucketList.get(num).add(array[j]);
        }
        int index = 0;
        for (int j = 0; j < bucketList.size(); j++) {
            for (int k = 0; k < bucketList.get(j).size(); k++)
            	array[index++] = bucketList.get(j).get(k);
            bucketList.get(j).clear();
        }
    }
    return array;
}
```
#### 10.4 算法分析
最佳情况：T(n) = O(n \* k)   最差情况：T(n) = O(n \* k)   平均情况：T(n) = O(n \* k)

基数排序有两种方法：
MSD 从高位开始进行排序 LSD 从低位开始进行排序

### 11、比较器

代码示例：

```java
List<CityCount> list = new ArrayList<CityCount>(cityCountMap.values());
Collections.sort(list, new Comparator<CityCount>() {
    public int compare(CityCount cityCount1,
                       CityCount cityCount2) {
        return (cityCount2.getNum() - cityCount1.getNum());
    }
});
```

## 三、JVM

原文地址： https://baijiahao.baidu.com/s?id=1632503816780923946&wfr=spider&for=pc 

 https://blog.csdn.net/u013541140/article/details/97375248 

### 1. 类的加载

* 加载器：从上到下

  * Bootstrap Class Loader 启动类加载器（C++）：jdk自带的类，用的是bootstrap class loader
  * Extension Class Loader 扩展类加载器（Java）
  * App Class Loader 应用程序加载器：自定义的类，用的是App Class Loader

类的加载过程是将java编译后的class文件读入内存中，在堆区创建java.lang.Class对象，用于封装类在方法区内的数据结构。类加载的最终目的是封装类在方法区内的数据结构，提供外部访问方法区数据的接口。

类加载的生命周期有5个阶段：加载、连接、初始化、使用、卸载。

* 加载：类的加载过程主要完成3件事，
  1. 通过类的全限定名获取类的二进制字节流。
  2. 将类字节流代表的静态存储结构转化为方法区的运行时数据结构。
  3. 在堆中生成一个代表此类的java.lang.Class对象，作为访问方法区这些数据结构的入口。
* 连接：这个过程分3个阶段完成，分别是校验、准备、解析。
  1. 校验class文件包含的信息是否符合jvm规范。
  2. 为类变量分配内存和初始化。
  3. 把类型中的符号引用转换为直接引用。具体解析有4种：类或接口、字段、类方法、接口方法。
* 初始化：即执行类的构造器方法的过程。有5种方法可以完成初始化。
  1. 调用new方法。
  2. 使用Class类的newInstance方法（反射机制）。
  3. 使用Constructor类的newInstance方法（反射机制）。
  4. 使用Clone方法创建对象。
  5. 使用（反）序列化机制创建对象。
* 使用：对类的初始化完成之后，就可以对类进行实例化，在程序中使用。
* 卸载：当类被加载，连接和初始化之后，类的生命周期就开始了，当代表类的class对象不被引用时，class对象就会结束生命周期，类在方法区的数据就会被卸载。

### 2. jvm内存结构

 ![img](https://pics0.baidu.com/feed/a6efce1b9d16fdfa1c6aa223708d4b5095ee7b84.jpeg?token=9ad03500fdf465156ff084e49fd9eb07&s=B132C8324F0E46491C581A7C0300E072) 

Java内存分配：

具体划分为以下5个内存空间：

1. 栈（私有）：存放局部变量，8种基本类型的变量，对象的引用变量，实例方法。
2. 堆（公有）：存放所有new出来的东西，存放实例变量。
3. 方法区（公有）：被虚拟机加载的类信息、常量、静态常量等，存放类的结构信息。
4. 程序计数器（私有）：记录方法之间的调用和执行情况，存储指向下一条指令的地址，还有即将要执行的指令代码，是当前线程所执行的字节码的行号指示器。
5. 本地方法栈（私有）：存储本地方法

对象是怎么存储的：

​	对象实例存储在堆空间，对象元数据存储在方法区（元数据区），对象的引用存储在栈空间。

### 3. 堆内存

-Xms 堆内存初始大小，默认内存大小：系统**1/64**

-Xmx 堆内存最大值，默认内存大小：系统**1/4**

堆内存逻辑上分为：新生代 + 老年代 + 原空间（java8）/ 永久代（java7）

物理上分为：新生代 + 老年代，默认1:2

新生代：Eden + survivor0（from） + survivor1（to），默认8:1:1

当Eden内存满了之后，第一次发生Young GC，将Eden基本清空，幸存的对象放到survivor0（from）区；当再次触发GC，会扫描Eden区和from区，对这两个区进行垃圾回收，经过这次回收还存活的对象，则直接复制到to区，之后在幸存区每GC一次存活对象年龄都加1，from区和to区在survivor0和survivor1互相转换，（即上一次复制到的to区这次变为from，因为需要从有对象的区域复制到空的区域，所以from和to区一直交替变换）经过**15**次（CMS算法则默认**6**次）【对象头中用4字节标记，则二进制最大1111，即15最大】GC之后，标记达到15，则放入Old区。Old区满了，则触发Full GC，当Full GC已经腾不出内存，则报OOM（OutOfMemoryError）堆内存溢出。GC会导致STW（stop the world），暂停程序执行。

### 4. GC算法和垃圾回收器

* 常见的垃圾回收算法：引用计数法、标记-清除、标记-压缩（整理）、复制算法。

  * 引用计数法：对象每被引用一次则计数加1，引用结束则计数减1，当计数为0时，需要被GC回收。

    【缺点：1. 每次对对象赋值时均要维护引用计数器，且计数器本身也有一定的消耗；           2. 较难处理循环引用】

  * 复制算法：年轻代使用的垃圾回收算法，从根集合（GC root）开始，通过Tracing从From中找到存活对象，拷贝到to；from和to交换身份，下次内存分配从to开始。

    【优点：1. 不会产生内存碎片，可以利用bump-the-pointer实现快速内存分配。  2. 没有标记清除过程，效率高。

  ​           缺点：1. 需要双倍空间。 2. 当对象存活率高时，需要复制的对象很多，效率就变低了。】

  * 标记-清除：老年代使用的垃圾回收算法。先标记要清除的对象，之后进行清除。

    【优点：不需要额外空间。

    ​    缺点：1. 两次扫描，耗时严重。 2. 会产生内存碎片。】

  * 标记-压缩（整理）：老年代使用的垃圾回收算法。在标记清除之后，再次扫描，并往一端滑动存活对象。

    后面出的一种折中实现，在标记清除之后，经过几次GC再进行整理。

    【优点：没有内存碎片。

    ​    缺点：需要移动对象的成本，花费的时间变长。】

* 没有最好的算法，只能根据不同区域使用不同算法，称为分代收集算法，新生代用复制算法，老年代用标记-清除或标记-清除和标记-压缩的混合实现。

  * 内存效率（比时间）：复制 > 标记清除 > 标记整理
  * 内存整齐度：复制 = 标记整理 > 标记清除
  * 内存利用率：标记清除 = 标记整理 > 复制

* 常见的垃圾收集器：串行垃圾回收器（Serial）、并行垃圾回收器（Parallel）、并发垃圾回收器（Cms）、G1垃圾回收器。

  * 串行垃圾回收器（Serial）：为单线程环境设计且只使用一个线程进行垃圾回收，会暂停所有的用户线程。

    ​              ------->                               --------->

    Serial    ------->   .....GC线程.....>   --------->

    ​              ------->                               --------->

  * 并行垃圾回收器（Parallel）：Java8 默认垃圾回收器（parallel scavenge 和parallel Old），多个垃圾回收线程并行工作，此时用户线程是暂停的。

    ​              ------->   .....GC线程.....>   --------->

    Parallel ------->   .....GC线程.....>   --------->

    ​              ------->   .....GC线程.....>   --------->

  * 并发垃圾回收器（Concurrent Mark Sweep）：用户线程和垃圾收集线程同时执行（不一定并行，可能交替执行），不用停顿或长时间停顿用户线程。使用标记-清除算法。工作有四阶段，1. 初始标记，标记GC Roots能直接关联的对象，速度很快，但需要暂停所有工作线程。 2. 并发标记，进行GC Roots的跟踪过程，不需要暂停。 3. 最终标记，修正在并发期间，因用户线程运行导致标记发生变动的那一部分对象的记录，需暂停线程。 4.  筛选回收，清除GC Roots不可达的对象，不需要暂停线程。

    【优点：并发收集低停顿。   缺点：1.并发执行，对CPU资源压力大。 2. 采用标记-清除算法会导致大量碎片。】

    ​           ----->  1....GC线程...>   ------>  2...GC线程...>  3...GC线程...>  ------> 4...GC线程....>

    Cms   ----->  1....GC线程...>   ------>  2.--------------->  3...GC线程...>  ------> 4.--------------->

    ​           ----->  1....GC线程...>   ------>  2.--------------->  3...GC线程...>  ------> 4.--------------->

  * G1：java9默认垃圾回收器，将堆内存分割成不同的区域，然后并发的对其进行垃圾回收。

    * Young GC：1. 根扫描。 2. 更新RS。 3. 处理RS。 4. 对象拷贝。 5. 处理引用队列。
    * Mix GC：1. 全局并发标记。 2. 拷贝存活对象。

    1. 初始标记，标记GC Roots能直接关联的对象，速度很快，但需要暂停所有工作线程。 2. 并发标记，进行GC Roots的跟踪过程，不需要暂停。 3. 最终标记，修正在并发期间，因用户线程运行导致标记发生变动的那一部分对象的记录，需暂停线程。 4. 筛选回收，清除GC Roots不可达的对象，不需要暂停线程。

* 查看默认的垃圾收集器：java -XX:+PrintCommandLineFlags -version

  ```shell
  # 查看进程编号
  jps
  
  # 查看编号进程是否使用某个垃圾回收器
  # 返回-XX:-UseSerialGC表示没使用
  # 返回-XX:+UseSerialGC表示使用
  jinfo -flag UseSerialGC 5872
  ```

* gc回收的种类（源码写的6种，不包括括号，实际7种）：

  ![image-20200914183958102](C:\Users\lsy\AppData\Roaming\Typora\typora-user-images\image-20200914183958102.png)

  * -XX:+UseConcMarkSweepGC：Concurrent Mark Sweep收集器，当CMS未在资源耗尽时完成垃圾回收，将发生STW，然后使用Serial Old收集器进行垃圾回收

  ![image-20200914215023577](C:\Users\lsy\AppData\Roaming\Typora\typora-user-images\image-20200914215023577.png)

  > DefNew：Default New Generation
  >
  > Tenured：Old
  >
  > ParNew：Parallel New Generation
  >
  > PsYoungGen：Parallel Scavenge
  >
  > ParOldGen：Parallel Old Generation

* 垃圾回收器的选择

  * 单CPU或小内存，单机程序：-XX:+UseSerialGC
  * 多CPU，需要最大吞吐量，如后台计算型应用：-XX:+UseParallelGC 或者 -xx:+UseParallelOldGC
  * 多CPU，追求低停顿时间，需快速响应如互联网应用：-XX:+UseConcMarkSweepGC 或 -XX:+UseParNewGC

| 参数                                       | 新生代垃圾收集器                       | 新生代算法              | 老年代垃圾收集器                                          | 老年代算法 |
| ------------------------------------------ | -------------------------------------- | ----------------------- | --------------------------------------------------------- | ---------- |
| -XX:+UseSerialGC                           | SerialGC                               | 复制                    | SerialOldGC                                               | 标记整理   |
| -XX:+UseParNewGC                           | ParNew                                 | 复制                    | SerialOldGC                                               | 标记整理   |
| -XX:+UseParallelGC / -XX:+UseParallelOldGC | Parallel [Scavenge]                    | 复制                    | Parallel Old                                              | 标记整理   |
| -XX:+UseConcMarkSweepGC                    | ParNew                                 | 复制                    | CMS+Serial Old组合（Serial Old作为CMS出错后的后备收集器） | 标记清除   |
| -XX:+UseG1GC                               | G1，局部通过复制算法，不会产生内存碎片 | 整体上采用标记-整理算法 |                                                           |            |

### 5. jvm调优

![image-20200914114309617](C:\Users\lsy\AppData\Roaming\Typora\typora-user-images\image-20200914114309617.png)

![image-20200914114211439](C:\Users\lsy\AppData\Roaming\Typora\typora-user-images\image-20200914114211439.png)

1）参数：

JVM内存大小 = 持久代大小 + 年轻代大小 + 老年代大小

* -Xmx3550m：设置JVM最大内存为3550M
* -Xms3550m：设置JVM初始化内存3550M
* -XX:NewSize=n：设置年轻代大小
* -XX:NewRatio=n：设置老年代和年轻代的比值，如3，比值为3:1。
* -XX:SurvivorRatio=n：设置年轻代中Eden区和Survivor区（from和to）的比值。如3，则比值为3:2。
* -XX:MaxPermSize=n：设置持久代的大小。
* -XX:printGCDetails：打印堆内存详情 
* -XX:MaxTenuringThreshold：设置对象在新生代中存活的次数
* -XX:+UserSerialGC：使用Serial垃圾收集器
* -XX:CMSFullGCsBeForeCompaction：指定多少次CMS收集之后进行一次整理。

（一般年轻代和老年代比例为1:2；年轻代中的Eden区和survivor区的比例为8:2；survivor区中的from和to比例为1:1）

* 实例：-Xms1024m -Xmx2048m -XX:MaxPermSize=512m

2）有如下建议：

* 初始化内存尽量与最大内存保持一致，避免内存不够继续扩充。最大内存不要超过物理内存。
* gc频率不要太高，每次gc时间不要太长，根据系统应用来定。

3）常用的调优工具：

* jconsole：jdk自带工具，可查看资源占用、GC等情况，安装jdk之后，命令行窗口jconsole即可开启。
* jvisualvm：jdk自带工具，可查看资源占用等情况，图形化界面比较直观，要查看GC情况，需要安装GC插件。安装jdk之后，命令行窗口jvisualvm即可开启。
* jps：查看所有jvm进程，包括进程ID，进程启动路径等。
* jstack:观察jvm中当前所有线程的运行情况和状态。
* jstat：利用jvm内建的指令对java应用程序的资源和性能进行实时的命令行的监控，包括对进程的classloader、compiler、gc情况。
* jmap：监视进程中的jvm物理内存占用情况，该进程内所有对象的情况。

### 6. JMM-Java内存模型

volatile

* 可见性、原子性、有序性
* 每个线程创建时，都会创建一个工作内存（私有），Java内存模型中规定所有变量都存储在主内存，主内存是共享的。线程对变量的读和赋值操作必须在工作内存中进行。首先将变量拷贝到线程自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存，不能直接操作主内存中的变量，各个线程的工作内存存的是主内存中变量的副本拷贝，线程间无法相互访问工作内存，线程间通信（传值）需通过主内存来完成。

### 7. jvm常见面试问题

* jvm的初始化步骤？

  1. 假如这个类还没有加载和连接，则程序先加载和连接该类；
  2. 假如这个类的直接父类还没有初始化，则先初始化该类的直接父类；
  3. 假如这个类有初始化语句，则系统依次执行这些初始化语句。

* 类的加载过程？

  1. 通过类的全限定名获取类对应的二进制字节流；
  2. 将类字节流的存储结构转化为方法区运行时的数据结构；
  3. 在堆中创建一个java.lang.Class对象，作为方法区该类的数据访问入口。

* 什么时候会加载类？

  6种主动使用方式：

  1. new对象
  2. 调用类的静态方法。
  3. 调用类的静态变量。
  4. 初始化该类的子类。
  5. 反射。
  6. 被jvm启动时标为启动类的类。

* jvm的内存结构？

  分为方法区、堆、java栈（栈）、本地方法栈、程序计数器。

* 每块内存分别存的什么信息？

  【公有】

  堆：存放所有new的对象。

  方法区：存放已被虚拟机加载的类信息、常量、静态变量等。

  【私有】

  java栈（栈）：虚拟机栈中执行每个方法时，都会创建一个栈用于存储局部变量表、操作数栈、动态链接、方法出口等信息。

  本地方法栈：与java栈相似，只不过本地方法栈是在执行每个本地方法的时候创建信息。

  程序计数器：指示虚拟机下一条要执行的字节码指令。

* 双亲委派模型？

  原理：当类加载器接收到一个类的类加载请求时，它不会自己进行类加载，而是由父加载器负责加载，只有当父加载器在自己的搜索范围内找不到指定的类时，子加载器才会自己加载。

   ![img](https://upload-images.jianshu.io/upload_images/2154124-d2f7f6206935de2b?imageMogr2/auto-orient/strip%7CimageView2/2) 

* GC算法？

  1. 引用计数：当一个对象有一个引用时，计数加一，删除一个引用则计数减一。垃圾回收只回收计数为0的对象。此算法无法处理循环引用的问题。
  2. 标记-清除：第一阶段从引用的根节点开始标记所有被引用的对象；第二阶段，遍历整个堆，把未标记的对象清除。此算法需要停止整个应用，同时会产生内存碎片。
  3. 复制：此算法会把空间划分为两个相等的区域，每次只使用其中的一个区域。垃圾回收时，遍历当前正在使用的区域，把正在使用中的对象复制到另外一个区域。此算法每次只处理正在使用中的对象，因此复制成本较小，同时复制过去之后还能整理内存，不会出现内存碎片，但是需要两倍内存空间。
  4. 标记-整理：此算法结合了“标记-清除”和“复制”两个算法的优点。第一阶段从根节点开始标记所有被引用的对象；第二阶段遍历整个堆，清除未标记的对象，并把存活的对象压缩到堆的其中一块，按顺序排放。此算法避免了碎片问题，也避免了空间问题。

  【现在都是使用分代收集：基于对象生命周期分析后得出的垃圾回收算法。把对象分为年青代、年老代、持久代，对不同生命周期的对象使用不同的算法（上述方法中的一个）进行回收。】

* 如何调优？

  1. 初始化内存尽量与最大内存保持一致（-Xms和-Xmx），避免内存不够继续扩充。最大内存不要超过物理内存。

  2. gc频率不要太高，每次gc时间不要太长，根据系统应用来定。

* FULL GC的条件 ？

  1. 直接调用System.gc()，调用后并不会马上FGC，会在某个时间点发生。
  2. 老年代的可用空间不足时；
  3. 方法区空间不足时；
  4. 通过Monitor GC后进入老年代的平均大小大于老年代的可用内存时。
  5. concurrent mode failure

## 四、Redis

### 1. 数据类型

* String
* Hash
* list
* Set
* Zset

### 2. 单线程Redis性能高

* 基于内存

   ![img](http://p3.pstatp.com/large/2dd480002f657a6d59e09) 

  * 数据查询类场景：内存中有全量的数据，可以直接从内存中取得；
  * 数据写入类场景：如果配置的是同步持久化，写入内存的同时，也会写入磁盘，性能有所降低，但是由于Redis使用的是IO多路复用，同时没有线程竞争，因此IO利用率很高。
  * 数据写入类场景：如果配置的是异步持久化，写入内存成功，即响应成功，不用等待磁盘的写入，性能很高。

* 单线程，多路IO复用

  * 多线程存在线程竞争，且有锁的问题，多线程代码逻辑复杂，复杂的代码逻辑通常会带来一定的性能损耗。

  * Redis虽然是单线程，但它的“IO多路复用”模型的IO利用率很高。是采用效率最高的epoll模式，单线程却实现了多客户端接入。将接入的客户端生成一个队列。

     ![img](http://p9.pstatp.com/large/2dd450002f4f251ffa31a) 

* 数据结构

  * Redis全程采用了Hash结构，因此存取效率非常高；
  * 对于数据的存储内容也进行了压缩，节省了空间占用，也减少了io带宽。

### 3. 持久化

* RDB（快照）
* AOF

## 五、异常和错误

Throwable:

* Error：程序中发生严重的错误Error，无法处理的错误，只能事先避免；如内存溢出等。

【栈内存溢出（java.lang.StackOverflowError）：使用递归时，没有递归出口，导致方法不断压栈，最后内存溢出。】

【堆内存溢出（java.lang.OutOfMemoryError: Java heap space）：不断创建新对象，或者一次性创建大对象，把对内存挤爆。byte b = new byte[1024*1024]】

* Exception：异常发生后程序员可以通过代码的方式纠正，使程序继续运行，是必须要处理的。

### 1. 运行时异常

* 类型转换异常（ClassCastException）
* 数组下标越界（IndexOutofBoundsException）
* 空指针（NullPointerException）
* 数字格式异常（NumberFormatException）
* 算术运算异常 （ArithmeticException）
* 非法参数异常 （IllegaArguementException）
* 非法监听状态异常（java.lang.IllegalMonitorStateException）：使用condition的await方法和Object的wait方法，没有搭配lock.lock()或者synchronized，或者线程unlock不是自己的锁。
* 并发修改异常（java.util.ConcurrentModificationException）：使用多线程修改同一个数组时发生。
* 非法线程状态异常（java.lang.IllegalThreadStateException）：Thread使用了两次start方法，第一次使用已经将状态变为0，调用start方法判断非0状态则抛出异常。

### 2. 编译时异常

* 输入输出异常（IOException）
* 文件未找到（FileNotFoundException）
* 时间格式化异常（ParseException）
* 指定类不存在（ClassNotFoundException）
* 克隆不支持（CloneNotSupportedException）：出现在原型模式或者重写Object的clone方法时。

## 六、注解

### 1. @ComponentScan

```java
@Configuration
// 排除扫包; 包含包includeFilters
//@ComponentScan(value = "com.lsy",excludeFilters = {
//    @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
//})
// 可以配置多个扫描规则
@ComponentScans(value = {
//    @ComponentScan(value = "com.lsy",excludeFilters = {
//    	@ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class}),
//      @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {CarService.class})
//    }),
    // 使用includeFilters时，需要禁用默认扫描规则才能生效
	@ComponentScan(value = "com.lsy",includeFilters = {
//      @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
        @ComponentScan.Filter(type = FilterType.CUSTOM, classes = {MyTypeFilter.class})
    }, useDefaultFilters = false)
})
// @ComponentScan value：指定要扫描的包
// excludeFilters = Filter[]：指定扫描的时候按照什么规则排除那些组件
// includeFilters = Filter[]：指定扫描的时候只需要包含哪些组件
// FilterType.ANNOTATION：按照注解的形式
// FilterType.ASSIGNABLE_TYPE：按照给定的类型
// FilterType.ASPECTJ：使用ASPECTJ表达式
// FilterType.REGEX：正则表达式
// FilterType.CUSTOM：正则表达式
```

### 2.@Bean

```java
// 给容器中注册一个Bean，类型为返回值的类型，id默认是方法名
@Bean(value = "car")
```

### 3.@Scope

```java
/**
     * @Scope默认单例
     * ConfigurableBeanFactory
     * singleton：单例（默认值），IOC容器启动会调用方法创建对象放到IOC容器中，以后每次获取就是
     *            直接从容器（map.get()）中拿
     * prototype：多例，IOC容器启动不会调用方法创建对象放在容器中，每次获取的时候才会调用方法
     *            创建对象
     * request：同一次请求创建同一个实例
     * session：同一个session创建一个实例
*/
@Scope("prototype")
```

### 4. @Lazy

```java
/**
     *  Lazy + 单例，容器只在获取的时候才调用方法创建对象，而且只创建一次
     * @return
*/
@Lazy
```

### 5. *@Conditional

```java
// 条件注解，根据条件类返回对应的类
// 条件类继承Condition，实现matches
@Conditional({LinuxCondition.class})
```

### 6. *@Import

```java
/**
	*  给容器中注册组件：
	*  1、包扫描+组件标注注解（@Controller/@Service/@Repository
	/@Component）[自己写的类]
	*  2、@Bean[导入的第三方包里面的组件]
	*  3、@Import[快速给容器中导入一个组件]
	*    1)、@Import（要导入到容器中的组件）；容器中就会自动注册这个组件，id默认是全类名
	*    2)、ImportSelector：返回需要导入的组件的全类名数组；
		 3)、ImportBeanDefinitionRegistrar：手动注册bean到容器中
*/
@Import
```

给容器添加Bean的几种方式：
	给容器中注册组件：

1. 包扫描+组件标注注解（@Controller/@Service/@Repository/@Component）[自己写的类]

2. @Bean[导入的第三方包里面的组件]

3. @Import[快速给容器中导入一个组件]
   1)、@Import（要导入到容器中的组件）；容器中就会自动注册这个组件，id默认是全类名
   2)、ImportSelector：返回需要导入的组件的全类名数组；
   3)、ImportBeanDefinitionRegistrar：手动注册bean到容器中

4. 使用Spring提供的FactoryBean（工厂Bean）；

   1)、默认获取到的是工厂bean调用getObject创建的对象

   2)、要获取工厂Bean本身，我们需要给id前面加一个&

   ​		&colorFactoryBean

### 7. Bean生命周期

   * bean的创建---初始化---销毁过程；
   * 容器管理bean的生命周期；
   * 我们可以自定义初始化和销毁方法；容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法；
     * 构造（对象创建）
       * 单实例：在容器启动的时候创建对象
       * 多实例：在每次获取的时候创建对象
       
     * BeanPostProcessor.postProcessBeforeInitalization
     * 初始化：
       
       * 对象创建完成，并赋值好，调用初始化方法
       
     * BeanPostProcessor.postProcessAfterInitalization
     * 销毁：
	  * 单实例：容器关闭的时候
       * 多实例：容器不会管理这个bean；容器不会调用销毁方法
     1. 指定初始化和销毁方法；
     
        * 通过@Bean指定init-method和destroy-method
        
     2. 通过让Bean实现InitializingBean(定义初始化逻辑)
        
     * DisposableBean(定义销毁逻辑)
       
     3. 使用JSR250
       
        * @PostConstruct：在bean创建完成并且属性赋值完成；来执行初始化方法
        
        * @PreDestroy：在容器销毁bean之前通知我们进行清理工作
         BeanPostProcessor【interface】
     
     4. 在bean初始化前后进行一些处理工作
     
        * postProcessBeforeInitialization：在初始化之前工作
     * postProcessAfterInitalization：在初始化之后工作
       
        =>AnnotationConfigAppicationContext
        
        =>refresh
        
        =>finishBeanFactoryInitialization（初始化）=>getBean/createBean
        
        BeanPostProcessor原理：
        
        =>populateBean(给bean属性赋值)
        
        =>initializeBean(初始化)
        
        ​	【遍历得到容器中所有的BeanPostProcessor：挨个执行beforeInitialization，
        
        ​	一旦返回null，则跳出for循环，不会执行后面的
        
        ​	BeanPostProcessor.postProcessorsBeforeInitialization】
        
        ​	=>applyBeanPostProcessorBeforeInitialization
        
        ​	=>invokeInitMethods
        
        ​	=>applyBeanPostProcessorAfterInitialization
        
     5. Spring底层对BeanPostProcessor的使用：
     
        bean赋值，注入其他组件，@Autowired，生命周期注解功能，@Async，xxx BeanPostProcessor；
     

### 8. @Value和@PropertySource

```java
// 可以给实体类属性赋值
@Value("张三")
private String name;

@Value("${person.age}")
private Integer age;

// 导入外部配置文件中的k/v保存到运行的环境变量中；加载完之后用${}取出配置变量
@PropertySource(value={"classpath:/person.properties"})
```

### 9. @Autowired和@Qualifier

```java
// 默认按属性名装配
@Autowired

// 直接指定需要装配的组件的id，而不是使用属性名
@Qualifier
```

* @Autowired：自动注入：

​	1）、默认优先按照类型去容器中找对应的组件：applicationContext.getBean(CarDao.class);

​	2）、如果找到多个相同类型的组件，再将属性的名称作为组件的id，而不是使用属性名

​	3）、@Qualifier("carDao")：使用@Qualifier指定需要装配的组件的id，而不是使用属性名

​	4）、自动装配默认一定要将属性赋值好，没有就报错

​		可以使用@Autowired(required=false )

​	5）、@Primary：让Spring进行自动装配的时候，默认使用首选的bean；

​		也可以继续使用@Qualifier

* @Autowired：构造器、参数、方法、属性；都是从容器中获取参数组件值

  1）、标注在方法位置：@Bean+方法参数；参数从容器中获取参数组件的值

  2）、标注在构造器上：如果组件只有一个有参构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取

  3）、放在参数位置

### 10. @Resource和@Inject

Spring还支持使用@Resource(JSR250)和@Inject(JSR330)[java规范的注解]：

* @Resource：

  可以和@Autowired一样实现自动装配功能；默认是按照组件名称进行装配的；

  没有能支持@Primary功能没有支持@Autowired(required=false);

* @Inject：

  需要导入javax.inject的包，和Autowired的功能一样。没有required=false的功能；

@Autowired：Spring定义的，在Service调用的dao和直接获取的dao是同一个；@Resource和@Inject都是java规范。

AutowiredAnnotationBeanPostProcessor

### 11. Aware原理

自定义组件想要使用Spring容器底层的一些组件（ApplicationContext，BeanFactory，xxx）

自定义组件实现xxxAware；在创建对象的时候，会直接调用接口规定的方法注入相关组件；Aware；

把Spring底层一些组件注入到自定义的Bean中；

xxxAware；功能使用xxxProcessor；

​	ApplicationContextAware ==> ApplicationContextAwareProcessor；

### 12. Profile

Profile：

​	Spring为我们提供的可以根据当前环境，动态的激活和切换一系列组件的功能；

开发环境、测试环境、生产环境；

数据源：（/A）（/B）（/C）

```java
@Configuration
public class MainConfigOfProfile {
	
    @Profile("test")
    @Bean("testDataSource")
    public DataSource dataSourceTest() throws Exception {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("123456");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }
    
    @Profile("dev")
    @Bean("devDataSource")
    public DataSource dataSourceDev() throws Exception {
        ...
    }
    
    @Profile("prod")
    @Bean("prodDataSource")
    public DataSource dataSourceProd() throws Exception {
        ...
    }
    
}
```

@Profile：指定组件在哪个环境的情况下才能被注册到容器中，任何环境下都能注册这个组件

1. 使用命令行动态参数：在虚拟机参数位置加载

   ```shell
   -Dspring.profiles.active=test
   ```

2. 使用代码

   * 创建一个applicationContext
   * 设置需要激活的环境
   * 注册主配置类
   * 启动刷新容器

* 加了标识的bean，只有这个环境被激活的时候才能注册到容器中，默认是default环境

* 写在配置类上，只有是指定的环境的时候，整个配置类里面的所有配置才能开始生效
* 没有标注环境标识的bean在任何环境下都是加载的

### 13. AOP

实现原理： 动态代理

1、导入aop模块；Spring AOP(spring-aspects)

2、定义一个业务逻辑类（MathCalculator）；在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、方法出现异常等等）

3、定义一个日志切面类（LogAspects）；切面类里面的方法需要动态感知MathCalculator.div运行到哪里然后执行。

​	通知方法：

* 前置通知：logStart，在目标方法运行之前运行
* 后置通知：logEnd，在目标方法运行结束之后运行
* 返回通知：logReturn，在目标方法正常返回之后运行
* 异常通知：logException，在目标方法出现异常之后运行
* 环绕通知：动态代理，手动推进目标方法运行（joinPoint.procced()）

4、给切面类的目标方法标注何时何地运行（通知注解）

5、将切面类和业务逻辑类（目标方法所在类）都加入到容器中

6、必须告诉Spring哪个是切面类（给切面类标注一个注解@Aspect）

7、给配置类中加@EnableAspectJAutoProxy【开启基于注解的aop模式】

​	在Spring中很多的@Enablexxx

```java
@Aspect
public class LogAspects {

    @Pointcut("execution(public int com.lsy.aop.MathCalculator.*(..))")
    public void pointCut() {

    }
    
    // JoinPoint可以拿到方法名和参数，要在第一个参数位置，否则失效
    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        System.out.println(joinPoint.getSignature().getName() + "运行。。。参数:" + Arrays.asList(args));
    }

    @After("com.lsy.aop.LogAspects.pointCut()")
    public void logEnd(JoinPoint joinPoint){
        System.out.println(joinPoint.getSignature().getName() + "结束。。。");
    }

    @AfterReturning(value = "pointCut()", returning = "result")
    public void logReturn(JoinPoint joinPoint, Object result){
        System.out.println(joinPoint.getSignature().getName() + "正常返回。。。运行结果是：" + result);
    }

    @AfterThrowing(value = "pointCut()", throwing = "exception")
    public void logException(JoinPoint joinPoint, Exception exception){
        System.out.println(joinPoint.getSignature().getName() + "异常。。。异常欣喜：" + exception);
    }
}
```

三步：

* 将业务逻辑组件和切面类都加入到容器中；告诉Spring哪个是切面类（@Aspect）
* 在切面类上的每一个通知方法上标注通知注解，告诉Spring何时何地运行（切入点表达式）
* 开启基于注解的aop模式，@EnableAspectJAutoProxy

## 七、多线程

### 1. 线程

* 线程的六种状态

new：初始，新建

runnable：运行，将就绪（ready）和运行中（running）两种状态笼统称为运行。

blocked：阻塞

waiting：等待（一直等待）

timed_waiting：超时等待（超时退出）

terminated：终止

![img](http://images2015.cnblogs.com/blog/527668/201603/527668-20160318095911506-2062217765.jpg)

**常见问题：**

1、造成线程阻塞的方法？

阻塞线程的方法：join、yield、sleep 和Object的wait()方法，LockSupport.park()；

【wait：Object类的方法，会让出cpu资源和对象锁。需要nodify()随机唤醒或者nodifyAll()进行唤醒。

sleep：让出cpu资源，但是不释放锁。

join：会等待该线程执行完成再往下执行。

yield：暂停当前线程，使当前线程回到可执行状态，不会释放锁，可能会又马上被执行。】

【Thread.currentThread().interrupt()线程打断并不是终止线程运行，不会影响线程继续执行，是设置一个中断标志，用来打断线程阻塞状态，例如sleep、wait、join等，打断这些会抛出异常，然后将interrupt重新设置为false。】

2、Java的守护进程（后台进程）？

设置线程为后台进程运行：setDaemon(true) 如果一个进程中只有后台线程在运行，这个进程就会结束。

3、造成线程阻塞后，线程回到哪个状态了？

通过join、yield、sleep造成线程阻塞后是回到了就绪状态

3、哪些状态之后是回到就绪状态？

 a)通过join、yield、sleep造成线程阻塞后是回到了就绪状态

 b)遇到synchronized后

 c)遇到Object的等待wait方法后

4、sleep会释放锁吗？

 sleep不会释放锁【它会抱着锁睡觉】

5、线程都有哪些状态？具体是怎么运行的？

线程有：创建、就绪、运行、阻塞、终止。5种状态

1.通过new关键字创建后，进入到新生状态

2.调用start后进入就绪状态

3.CPU调度到本线程后，本线程开始执行。进入到运行状态

4.运行中遇到join，yield，sleep造成阻塞，进入阻塞状态。阻塞完成后，又回到就绪状态

5.线程正常执行完，或者遇到异常终止后，进入死亡状态

6、终止线程有哪几种方法？

 线程调用 stop()方法、destory()方法或 run()方法执行结束后，线程即处于死亡状态。处于死亡状态的线程不具有继续运行的能力。

 推荐使用boolen标识来停止线程



* 1. 高聚低耦合前提下，线程操作资源类
  2. 判断/干活/通知
  3. 防止虚假唤醒：调用wait会释放资源，进入等待池，被唤醒之后，会继续执行未执行完的代码，所以用if的话，因为之前判断过了，会直接跳过，直接执行下一步，而while会继续判断。

  ```java
  class Operation {
  	private int number = 0;
  	public synchronized void increase() throws InterruptedException{
  		while (number != 0) {
              this.wait();
          }
          number ++;
          this.nodifyAll();
  	}
      
      public synchronized void decrease() throws InterruptedException {
  		while (number == 0) {
              this.wait();
          }
          number --;
          this.nodifyAll();
  	}
  }
  ```

  

### 2. 初始化线程并执行

* 继承Thread类，重写run方法

```java
	Thread thread = new Thread(new Thread01());
	thread.start();

	public class Thread01 extends Thread {

        @Override
        public void run() {
            System.out.println("继承Thread类");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
```

* 实现Runnable接口，重写run方法

```java
    Thread thread02 = new Thread(new Runnable01());
    thread02.start();
        
    public class Runnable01 implements Runnable {

        @Override
        public void run() {
            System.out.println("实现Runnable接口");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
```

* 实现Callable接口，重写call方法，使用FutureTask搭配使用获取返回值。

```java
	FutureTask<Integer> future = new FutureTask<>(new Callable01());
    Thread thread = new Thread(future);
    thread.start();
    try {
    	System.out.println(future.get());
    } catch (Exception e) {
    	e.printStackTrace();
    }
	
	public class Callable01 implements Callable<Integer>{

        @Override
        public Integer call() throws Exception {
            int i = 10 /2;
            return i;
        }
    }
```

* 线程池

  * 分配线程数：Runtime.getRuntime().availableProcessors() 查看cpu核心数

    * cpu密集型（计算while,if这些一直运算）：核心数+1

    * io密集型：1. 任务线程不是一直执行，cpu核数 * 2;   

      ​                   2. 大部分线程阻塞，cpu核数/（1 - 阻塞系数），阻塞系数（0.8~0.9之间），如8核： 8 / （1 - 0.9） = 80

```java
        /*
         * 七个参数：
         * @param corePoolSize：核心线程数
         * @param maximumPoolSize：最大线程数
         * @param keepAliveTime：存活时间，即除核心线程外的线程空闲多长时间后回收
         * @param unit：时间单位
         * @param workQueue：任务队列，阻塞队列，在核心线程数都满了之后，再进来的任务则进入任务队列
         *                  LinkedBlockingQueue默认最大为Integer最大值，会导致内存被占满
         * @param threadFactory：线程创建工厂
         * @param handler：如果队列和最大线程都执行满了，则按照指定的拒绝策略拒绝执行任务RejectedExecutionHandler，
         *                 默认AbortPolicy，直接丢弃新来的任务，并抛出异常
         *
         * 题：corePoolSize 7, maximumPoolSize 20，queue 50，task 100；
         *    7个会立即得到执行，50个任务进入队列，再开13个线程进行执行，剩下的30个执行拒绝策略。如果不想丢弃任务，可以使用
         *    CallerRunsPolicy策略，直接执行任务的run方法进行执行。
         */
         ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(5, 10, 200, TimeUnit.SECONDS, new LinkedBlockingQueue<>(25), Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());
         
         for (int i = 0; i < 15; i++) {
            Runnable01 runnable01 = new Runnable01(i);
            threadPoolExecutor.execute(runnable01);
         }
         try {
            Thread.sleep(40000);
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
```

### 3. 异步编排

```java
	ExecutorService executor = Executors.newFixedThreadPool(5);
        
    System.out.println("main ---- start");

//        CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
//            System.out.println("当前线程：" + Thread.currentThread().getId());
//            int i = 10 / 2;
//            System.out.println("运行结果:" + i);
//        }, executor);

    CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
        System.out.println("当前线程：" + Thread.currentThread().getId());
        int i = 10 / 2;
        System.out.println("运行结果:" + i);
        return i;
    }, executor);

    System.out.println("main ---- end" + future.get());
```

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            System.out.println("当前线程：" + Thread.currentThread().getId());
            int i = 10 / 2;
            System.out.println("运行结果:" + i);
            return i;
        }, executor)
                // 任务完成回调方法
                .whenComplete((res, exception)-> System.out.println("异步任务完成，返回结果是：" + res + "；异常是：" + exception))
                // 异常之后修改值
                .exceptionally(throwable -> {
                    return 10;
                });
```

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            System.out.println("当前线程：" + Thread.currentThread().getId());
            int i = 10 / 0;
            System.out.println("运行结果:" + i);
            return i;
        }, executor).handle((res, throwable) -> {
            if (res != null) {
                return res * 2;
            }

            if (throwable != null) {
                return -1;
            }
            return 0;
        });
        System.out.println("main ---- end" + future.get());
```

* 串行化

thenApply：接收上个任务返回值，并返回当前任务返回值。

thenApplyAsync：新建一个线程，接收上个任务返回值，并返回当前任务返回值。

thenAccept：接收上个任务返回值，处理，不返回结果。

thenAcceptAsync：新建一个线程，接收上个任务返回值，处理，不返回结果。

thenRun：上个任务执行完，执行这个任务，不返回结果。

thenRunAsync：新建线程，上个任务执行完，执行这个任务，不返回结果。

* 两任务组合，两个全部完成之后才执行

```java
		// 1.
        future01.runAfterBothAsync(future02, () -> {
            System.out.println("任务三开始");
        }, executor);
		
		// 2.在两个任务结束之后执行，可以获取future01和future02的返回值
        future01.thenAcceptBothAsync(future02, (res1, res2) -> {
            System.out.println("任务三开始" + res1 + res2);
        }, executor);
		
        // 3.
		CompletableFuture<String> combineAsync = future.thenCombineAsync(future02, (f1, f2) -> {
            System.out.println("任务三开始" + f1 + f2);
            return "任务三开始" + f1 + f2;
        }, executor);

```

* 两任务组合，其中一个完成就执行
* 多任务组合

allOf：参数传入多个任务future，allof.get()阻塞，等待所有任务完成才结束。

anyOf：参数传入多个任务future，anyof.get()阻塞，等待有一个任务完成才结束，可以用anyof.get()获取结束的任务的数据。

### 4. volatile和CAS算法

volatile：可见性，不保证原子性，轻量级的同步机制，可以禁止指令重排。当线程修改完数据之后，马上申请写回主内存，并通知其他线程，再继续执行下面的操作。

【按缓存行为单位，一个缓存行64字节，当两个对象不存在同一个缓存行，则volatile修改数据之后不会互相通知，速度变快增加。】

```java
public class volatileTest {
	public static T[] arr = new T[2];

    static {
        arr[0] = new T();
        arr[1] = new T();
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() ->{
            for (long i = 0; i < 1000_0000L; i++) {
                arr[0].x = i;
            }
        }, "t1");

        Thread t2 = new Thread(() ->{
            for (long i = 0; i < 1000_0000L; i++) {
                arr[1].x = i;
            }
        }, "t2");

        final long start = System.nanoTime();

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println((System.nanoTime() - start)/100_0000);
    }

}

class Padding{
    // 缓存行对齐
    private volatile long p1,p2,p3,p4,p5,p6,p7;
}

// 继承Padding之后，两个T的缓存行将不同，因为一个T超过64字节，则导致做修改不会相互通知
class T extends Paddings{
    public volatile long x = 0L;
}
```

```shell
// T继承Padding
120

// T未继承Padding
239
```

* 代码重排序

  ```java
  	static int a,b,x,y;
  
      public static void main(String[] args) throws InterruptedException {
          int i = 0;
  
          for (;;){
              i++;
              x = 0; y = 0;
              a = 0; b = 0;
              Thread t1 = new Thread(() ->{
                  a = 1;
                  x = b;
              }, "t1");
  
              Thread t2 = new Thread(() ->{
                  b = 1;
                  y = a;
              }, "t2");
  
              t1.start();
              t2.start();
              t1.join();
              t2.join();
              if (x == 0 && y ==0 ){
                  System.err.println("第" + i + "次" + "x=0,y=0");
                  break;
              } else {
  
              }
          }
      }
  // 无论代码怎么排序，如果按顺序，不可能出现x=0,y=0，如果出现，证明代码重排序了
  // 运行结果：
  //    第310954次x=0,y=0
  ```

  

##### **CAS**

CAS是乐观锁的一种实现，比较当前值，当前值是期望值，则进行修改，否则不修改。

compareAndSet方法，底层调用Unsafe类的compareAndSwapInt / compareAndSwapObject / compareAndSwapLong，取得对象的valueoffset（内存偏移量），进行原子操作。CAS是cpu并发原语，是完全于依赖硬件的功能，执行是连续的。

原理：自旋 + Unsafe类，C++底层通过lock cmpxchg修改变量值

```java
/**
* var1：当前对象，就是调用这个方法的对象本身，如AtomicInteger
* var2：当前对象内存偏移量，地址
* var4：修改的值
* var5：获取当前对象的值
*/
public final int getAndSetInt(Object var1, long var2, int var4) {
    int var5;
    do {
    	var5 = this.getIntVolatile(var1, var2);
        // 当当前对象的值为获取到的值var5时，表示还没被修改，可以进行修改，修改完成则跳出循环，没完成就继续获取值并判断修改
    } while(!this.compareAndSwapInt(var1, var2, var5, var4));

    return var5;
}
```

CAS缺点：1. 循环时间长，开销大。 2. 只能保证一个共享变量的原子操作。 3. ABA问题。

ABA问题解决：AtomicStampedReference，根据版本号修改，参数（期望值，最新值，版本号，最新版本号）。

### 5. AQS

* java.util.concurrent.locks.AbstractQueuedSynchronizer：抽象队列同步器，ReentrantLock里面的抽象静态内部类Sync就是继承了AbstractQueuedSynchronizer。

* 自旋 + park + CAS，AQS结构是Node双链表，Node包含属性有前节点、后节点、Thread、waitStatus等。

* 如果是单线程情况下，不需要涉及到队列，队列还没有生成，改变state值，当state值从0变为1，表示拿到锁。

* 多线程：
  * 加锁：第一个线程加锁，修改state值。之后第二个线程竞争锁，先看锁的状态，如果锁的状态是自由状态，再判断需不需要排队。锁被占用，需要排队会生成队列，先设置头节点中的thread为null，之后线程判断前节点是否为头节点，如果是并且队列中不止一个节点，当前线程是第一个排队的线程，则开始尝试获取锁，否则直接park，这里是自旋获取锁，最大次数为2次，如果两次没获取到锁，则进行阻塞park。（自旋获取锁过程，获取锁，不成功则进入下一步，判断前节点的waitStatus是否等于signal，signal默认为-1，而waitStatus默认为0，所以第一次自旋不相等，将前节点的waitStatus用CAS设置为signal的值，第二次自旋获取锁，获取不成功，继续进入判断waitStatus，此时waitStatus等于signal，返回true，下一步进行park，阻塞当前线程。【为什么是改上一个节点的waitStatus呢？因为上一个节点自己park之后，执行不了代码，所以改不了自己的状态，如果在park前修改状态，而修改状态和park不是原子性，可能会出现问题。就像一个人不知道自己什么时候睡着，需要别人判断你是否睡着。】）
  * 解锁：遵循获得锁的线程不在队列中，不参与排队的原则。当第一个线程解锁，则判断队列中头节点(Thread为null的节点)的waitStatus不等于0，如果waitStatus小于0，将他设置为0，然后将头节点的下一个节点（也就是第二个线程）unpark，唤醒。然后第二个线程就开始进行自旋尝试获取锁，一般都能获取到。然后将头节点设置为第二个线程的节点，然后将节点的Thread设置为null（为了符合获得锁的线程不在队列中的原则），把之前的头节点引用设置为null（出队），方便GC。

## 八、Mysql数据库

DQL：数据库查询语言，select

DML：数据库操纵语言，insert、update、delete

DDL：数据库定义语言，create

DCL：数据库控制语言，grant、rollback、commit

### 1. 数据库引擎：

|              | Innodb（5.0以后默认）              | myisam（5.0前默认）                                          |
| ------------ | ---------------------------------- | ------------------------------------------------------------ |
| 事务         | 支持                               | 不支持                                                       |
| 锁           | 行锁（主键索引）、表锁             | 表锁                                                         |
| 外键         | 支持                               | 不支持                                                       |
| 存放索引方式 | 聚集索引，数据文件和索引绑在一起   | 非聚集索引，数据文件分离                                     |
| 存放文件     | frm，ibd（表索引，叶子节点存数据） | frm（表结构信息），MYI（表索引，叶子节点存数据地址），MYD（表数据） |
| 查询具体行数 | 不保存表的具体行数                 | 用一个变量保存了整个表的行数                                 |
| 全文索引     | 不支持                             | 支持                                                         |

【Innodb：非主键索引，叶子节点存储的是数据的主键。一般都需要有主键，因为主键索引维护的是一整张表的数据。如果没有设置主键，Innodb会自己寻找没有重复的一列作为主键索引来维护索引树，如果找不到，则自己建一列隐藏列，可唯一识别的列，来作为主键索引维护索引树。自增主键为了减少维护索引的效率，如果再插入小的数，有时需要树分裂或者重新平衡。】

【B树（B-树）：在非叶子节点上存储索引和数据。

B+树：非叶子节点存储索引，叶子节点存储数据，叶子节点使用双向链表。为了每个高度存储更多的索引。设置每个节点最大存放数据数，满了则拆分，将改节点中间的数据（偶数则取中间偏右）往上一层放，左右两边拆成两个节点，以此类推。一般主键用自增，是为了在维护索引B+树时，直接在最后插入，不用重排序。

Hash：精确查找很快，但是范围查找和部分查找（只传联合索引中的一个值）非常慢甚至不支持。】

### 2. Mybatis缓存：

默认开启一级缓存：

以下情况可以命中：

1. 相同的sqlSession

2. 相同的sql语句和参数

3. 调用相同的方法

4. 必须相同的namespace（也就是mapper）

5. 不能在查询之前执行clearCache()

6. 不能执行增删改语句

 

一级缓存原理：（保存的是对象）

1. 客户端调用

2. 动态代理

3. 会话调用DefaultSqlSession

4. 拼装缓存key 

5. 查询是否有二级缓存

6. 执行一级缓存（是否清空，判断缓存是否存在数据）

7. 获取一级缓存数据

 

一级缓存PUT流程：

1. 先查缓存，如果没有就去查数据库

2. 查数据库

3. 设置到缓存当中去

 

缓存查询顺序：二级缓存 -> 一级缓存 -> 数据库

查询默认将对象保存到一级缓存，只有当提交或者关闭会话，才会将一级缓存的数据存入二级缓存。

 

二级缓存命中条件：（原理Map，保存的是数据，之后自动封装为对象返回，所以命中二级缓存，返回的对象与第一次查询的对象不同）

1. 在xml中配置<cache />
2. put缓存之前要关闭会话
3. 调用相同的方法
4. 必须相同的namespace（也就是mapper），需要自定义的sql，即在xml自己声明的sql
5. 不能在查询之前执行clearCache()
6. 不能执行增删改语句

【二级缓存有可能出现脏读，当两个不同的mapper都操作同一个数据库表时，其中一个修改了表内容，另一个mapper对应的namespace并没有更新缓存，会导致脏读。】

### 3. SQL优化：

思路：

1. 抓慢查询sql语句

   ```shell
   # 1. 重要且紧急
   # 登录mysql
   show full processlist;
   
   # 在mysql外使用
   mysql -u<user> -p<password> -S /data/3306/mysql.sock -e "show full processlist;" |egrep -vi "sleep"; 
   
   # 重要不紧急
   # 2.配置文件配置记录日志
   long_query_time = 2  # 查询超过2秒就记录日志
   log_queries_not_using_indexes  # 没有使用索引
   log_slow_queries = /data/3306/slow.log  # 记录日志的文件
   ```

2. explain语句检查索引执行情况（又称执行计划）

   需要关注的要素：

   * type：本次查询表的联接类型，如all全表扫描，index索引扫描等
   * key：最终使用的索引
   * key_len：本次查询用于结果过滤的索引实际长度，如int为4，通过长度可以判断联合索引有多少列被选择了。
   * rows：预计扫描的记录数
   * Extra：额外附加信息，出现Using filesoft排序等

   ```sql
   explain select * from test where id=1;
   
   explain select SQL_NO_CACHE * from test where id=1;
   ```

3. 对需要建索引的条件列建立索引，生产场景，大表不能高峰期建立索引，例如300万记录。

4. 分析慢查询SQL的工具mysqlsla（每天早晨发邮件）

* limit和offset，减少使用**limit x,y** 和 **limit x offset y**，因为会进行全表扫描，在数据量大的情况耗时很长，可以使用指针式的方法，如**where id > x limit y**。
* explain查看是否使用索引

优化：

1. 考虑在where和order by涉及的列上建立索引。

2. 避免where使用 **!=** 和 **<>** 操作符，否则进行全表扫描。

3. 避免在where中对字段进行null值判断，否则进行全表扫描。

4. 避免在where中使用or来连接条件，可以用union all 进行替换，否则进行全表扫描。

5. 避免在like中，把%放在最前面，否则进行全表扫描。

6. 避免使用in和not in，否则全表扫描。

7. 避免在where语句中使用参数，否则会全表扫描；

   如：select id from t where num = @num，强制使用索引

   select id from t with(index(索引名)) where num = @num

8. 避免在where语句中对字段进行表达式操作，否则全表扫描，如select id from t where num/2 = 100;改为select id from t where num = 100*2;

9. 避免在where中对字段进行函数操作，否则全表扫描。

10. 不要在where中的 “=” 左边进行函数、算术或者其他表达式运算，否则可能无法正确使用索引。

11. 使用索引字段作为条件时，若该索引是复合索引，则必须使用到该索引的第一个字段作为条件才能保证系统使用该索引，并且尽可能保持字段顺序与索引顺序一致。

12. 不要写一些没有意义的查询。

13. 使用exists代替in。

14. 当索引有大量数据重复时，SQL查询可能不会利用索引。

15. 索引不是越多越好，因为更新数据时，会重新排序和维护索引。

16. 尽量使用数字型字段，只含数值信息的字段尽量设计为数字型。

17. 尽量避免更新clustered索引数据列，因为聚集索引数据列的顺序就是表记录的物理存储顺序，一旦列值改变，将导致整个表记录顺序的调整。

18. 尽可能用varchar代替char，节省空间，相对较小字段内搜索效率更高。

19. 尽量避免使用*作为查询结果。

20. 尽量使用表变量来代替临时表。

21. 避免频繁创建和删除临时表

22. 避免游标。

23. 主键尽量使用自增字段，因为在插入时，建立索引可以直接插入到最后，避免重新排序。

### 4. 索引

是排好序的数据结构。

* 索引优缺点

  * 优点

    1.大大加快数据的检索速度;

    2.创建唯一性索引，保证数据库表中每一行数据的唯一性;

    3.加速表和表之间的连接;

    4.在使用分组和排序子句进行数据检索时，可以显著减少查询中分组和排序的时间。

  * 缺点

    1.索引需要占物理空间。

    2.当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，降低了数据的维护速度。
  
* 联合索引：

  * 多个值联合的索引，B+数索引排序按照索引值从左到右依次对比排序。如（name, age, position），先比name，再比age...
  * 最左前缀，当查询条件没用到联合索引的第一个值时，不使用联合索引。原因是，他优先按照联合索引第一个值排序，然后用第二个值查找的话，其实是属于乱序，前后都可能出现符合条件的值，没办法定位，需要全表扫描。

## 九、事务

@Transactional

### 1. 特性：ACID

* 原子性（Automicity）：一系列操作不可分割，要么全部成功，要么全部失败。
* 一致性（Consistency）：数据在事务的前后，业务整体一致；如转账，一定是有一方扣钱，一方加钱。
* 隔离性（Isolation）：事务之间相互隔离；一个事务的回滚，不会影响另一个事务。
* 持久性（Durabilily）：一旦事务成功，数据一定会落盘在数据库。即**不会出现**事务通知事务成功，但因数据库断电或者其他原因导致数据未保存到数据库。

### 2. 隔离级别

* 读未提交：在一个事务执行的操作就算不提交，别的事务也能看到。会导致脏读，读取别人没有提交的数据，出现脏数据。
* 读已提交：Oracle和SQL Server默认的隔离级别。在一个事务提交之后，其他事务才能看到事务的修改。在同个事务中执行相同的查询语句可能读到不同的数据，即不可重复读。
* 可重复读：Mysql默认的隔离级别，在同一个事务中，select的结果是事务刚开始时的状态，读到的结果会一致。
* 序列化：该隔离级别下事务都是串行顺序执行的，Mysql的InnoDB引擎会给读操作隐式的加一把读共享锁，从而避免脏读、不可重复读和幻读。

### 3. 事务传播行为

| 事务行为                  | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。 |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行。         |
| PROPAGATION_MANDATORY     | 支持当前事务，假设当前没有事务，就抛出异常。                 |
| PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，把当前事务挂起。                 |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式运行操作。如果当前存在事务，就把当前事务挂起。   |
| PROPAGATION_NEVER         | 以非事务方式运行，如果当前存在事务，则抛出异常。             |
| PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |

## 十、集合，数据结构

物理上，数据结构分为数组和链表两种；逻辑上，树，二叉树栈，队列等都是基于这两者实现。

### 1. ArrayList

* new ArrayList底层是Object[]，（提示：add方法可以装String，int...）

* 初始值是多少？ 10。（提示：jdk1.7初始10，而jdk1.8默认为空引用，等到第一个add才开始扩容到10）

* 扩容？扩容为（原数值 + 原数值>>1），近似1.5倍，向下取整。

* 线程不安全？

1. Vector 
2. Collections.synchronizedList(new ArrayList<>())
3. JUC包下的CopyOnWriteArrayList：没加锁，原理是用了两个数组，在add的时候，用Arrays.copyOf(element, element.length)复制旧的数组，然后在新数组执行下一位写入数据，最后将新数组赋值给旧数组。

### 2. Set

* HashSet底层使用HashMap。add方法调用map的put方法，

  add(E e) -> put(e, new Object()); 此处的object是伪值，以便与后备映射中的对象相关联。

* 线程不安全？
  1. Collections.synchronizedSet(new HashSet<>());
  2. CopyOnWriteArraySet<>();

### 3.HashMap和concurrentHashMap：

* 线程不安全？ConcurrentHashMap<>();

* HashMap：数组+链表（1.8版本红黑树）【当链表长度达到8，散列表长度达到64，则变为红黑树；小于等于6则变为链表】

【HashMap默认长度为16，size达到阈值乘以长度就扩容，扩容则（原数值<<1），近似2倍】

计算元素位置：hash值 & （数组长度-1），Node中的hash值是key.hashCode()返回值加工结果，hashcode^（异或）高16位。

* concurrentHashMap：数组+segmentTable【默认HashEntry长度为2】

【1.7使用CAS+ReentranteLock实现并发安全；1.8之后用CAS+synchronized】

CAS算法是乐观锁的一种实现方式

 

补充：位运算：

* 与&：

相同位都为1，则为1；有一个是0，则为0。

* 或|：

相同位有一个为1，则为1.

* 异或^：

相同位不同则为1；相同则为0。

### 4. 遍历

* HashMap

  ```java
  // 迭代器遍历，也是最常使用的遍历方式
  Iterator iterator = map.entrySet().iterator();
  while(iterator.hasNext()){
  	System.out.println(iterator.next());
  }
  
  // for
  for (Map.Entry entry : map.entrySet()){
      System.out.println(entry.getKey() + " " + entry.getValue());
  }
  
  // 通过查找到map的key，再进行查找对应的值
  for (String key : map.keySet()) {
      System.out.println(key + " " + map.get(key));
  }
  ```

### 5. 红黑树

* 性质： 以下性质约束了红黑树从根节点到叶子节点的最长路径不多于最短路径的2倍。
  * 节点是红色或黑色
  * 根节点是黑色
  * 叶子节点（NIL）是黑色
  * 每个红色节点的子节点都是黑色，不能有连续的红色节点
  * 从任一节点到其每个叶子节点的所有路径都包含相同数目的黑色节点
* 旋转（所有插入的节点默认为红色）
  * 变颜色：当插入节点的父节点是红色且其叔叔节点是红色时，改变颜色，其父节点和叔叔节点变为黑色，爷爷节点变为红色。做完操作后，看那个地方破坏了平衡，则破坏平衡的地方变为插入节点。
  * 左旋：当插入节点的父节点是红色，且其叔叔节点是黑色时，需要旋转。插入节点为右节点，则发生左旋。以插入节点的父节点为轴逆时针旋转，插入节点到前父节点位置（即类似于根节点位置），前父节点到前爷爷节点位置，前爷爷节点变为前父节点左子树，前父节点的左子树变为前爷爷节点的右子树。做完操作后，看那个地方破坏了平衡，则破坏平衡的地方变为插入节点。
  
  ![img](https://img-blog.csdnimg.cn/20181125155223734.gif)
  
  * 右旋：当插入节点的父节点是红色，且其叔叔节点是黑色时，需要旋转。插入节点为左节点，则发生右旋。以插入节点的父节点为轴顺时针旋转，插入节点到前父节点位置（即类似于根节点位置），前父节点到前爷爷节点位置，前爷爷节点变为前父节点左子树，最后将他的前父节点E颜色变为黑色，前爷爷节点S变为红色。做完操作后，看那个地方破坏了平衡，则破坏平衡的地方变为插入节点。
  
    ![img](https://img-blog.csdnimg.cn/20181125160017452.gif)

## 十一、分布式

### 1. CAP定理

* 一致性C
* 可用性A
* 分区容错性P

 一般满足P，C或A只能2选一。

### 2. Raft算法（实现CP定理）：

* 每个节点有3个状态：leader、follower和candidate。

* 日志复制和领导选举：保证一致性

选举超时时间：自旋时间，每个节点的自旋时间不同，随机。每次收到消息就重置，自旋时间结束，若没有收到消息，变为候选者。候选者会发出投票信息，让其他节点给他投票，每个节点投票一次，超过半数则成为leader节点。

心跳时间：在leader节点维持的正常时，会维持心跳。

* 脑裂现象：

一轮投票结束，平票，则重新自旋时间，重新投票，期间随从节点先自旋结束并没有收到信息的话可以变为候选者节点，上一轮候选者在这轮自旋慢并收到信息的话，变为随从节点。

* 日志复制

当写操作发给主节点，主节点收到请求后会记录日志，然后将日志发给从节点，从节点收到后发送确认，主节点收到大多数节点的确认后，提交写请求，并在下一个心跳时间（信息都是在心跳时间进行通讯的）发送提交信息给从节点，从节点开始提交。

* 分区容忍

分区时，各区各自选举各区领导，如果某区只有2个节点，1个领导节点，一个从节点，那么各种保存请求都得不到半数以上回应，不会保存。当分区结束，恢复通信，则由高位选举出的领导节点成为新的领导节点，其他各区的领导节点变为随从节点，并开始复制领导节点日志，形成一致。

### 3. BASE理论

* 基本可用：系统出现故障时，允许损失部分可用性；响应时间上损失、功能上损失。
* 软状态：允许系统存在中间状态，如正在保存中。
* 最终一致性：系统数据副本经过一定时间后，最终达到一致状态。弱一致性则表示最终可以允许后续部分访问不到。

## 十二、RabbitMq

### 1. 使用RabbitMQ

1. 引入amqp场景：RabbitAutoConfiguration就会自动生效

2. 给容器中自动配置了

   RabbitTemplate、AmqpAdmin、CachingConnetionFactory、RabbitMessagingTemplate；

   所有属性都是spring.rabbitmq

   @ConfigurationProperties(prefix = "spring.rabbitmq")

   public class RabbitProperties

3. 给配置文件中配置 spring.rabitmq信息

4. @EnableRabbit：开启功能

5. 监听消息：使用@RabbitListener；必须有@EnableRabbit

   @RabbitListener：类+方法上

   @RabbitHandler：标在方法上，比如当发送消息时有多种方式，发送的对象不同，就可以用多个标有@RabbitHandler注解的方法进行接收。

   * 发送消息序列化，消息转换，添加

     ```
     @Configuration
     public class MyRabbitConfig {
     
     	@Bean
     	public MessageConverter messageConverter() {
     		return new Jackson2JsonMessageConverter();
     	}
     
     }
     ```

     类

### ２．消息确认机制

Publier --ConfirmCallback-->  Broker[Exchange --ReturnCallback--> Queue] -> consumer

### 3. 延时队列

幂等性

解决重复消费问题。

死信队列（延迟队列）

![image-20201006105539082](C:\Users\lsy\AppData\Roaming\Typora\typora-user-images\image-20201006105539082.png)

```java
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.Exchange;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.context.annotation.Configuration;
import java.util.HashMap;
import java.util.Map;


@Configuration
public class MyMQConfig {

	@Bean
    public Queue orderDelayQueue() {

        Map<String, Object> arguments = new HashMap<>();
        arguments.put("x-dead-letter-exchange", "order-event-exchange");
        arguments.put("x-dead-letter-routing-key", "oder.create.order");
        arguments.put("x-message-ttl", 6000);

        //String name, boolean durable, boolean exclusive, boolean autoDelete, @Nullable Map<String, Object> arguments
        return new Queue("order-delay-queue", true, false, false, arguments);
    }

	@Bean
    public Queue orderReleaseOrderQueue() {
        // 队列名，持久化，排外，自动删除，参数
        //String name, boolean durable, boolean exclusive, boolean autoDelete, @Nullable Map<String, Object> arguments
        return new Queue("order-release-order-queue", true, false, false);
    }
	
	@Bean
    public Exchange orderEventExchange() {

        // String name, boolean durable, boolean autoDelete, Map<String, Object> arguments
        return new TopicExchange("order-event-exchange", true, false);
    }

	@Bean
    public Binding orderCreateOrderBinding() {
        // String destination, Binding.DestinationType destinationType, String exchange, String routingKey
        return new Binding("order-delay-queue",
                Binding.DestinationType.QUEUE,
                "order-event-exchange",
                "order.create.order",
                null);
    }

	@Bean
    public Binding orderReleaseOrderBinding() {
        // String destination, Binding.DestinationType destinationType, String exchange, String routingKey
        return new Binding("order-release-order-queue",
                Binding.DestinationType.QUEUE,
                "order-event-exchange",
                "order.release.order",
                null);
    }

}
```

## 十三、类

### 1. 接口和抽象类的区别

* 抽象类被子类继承，子类只能继承一个父类；接口被类实现，一个类可以实现多个接口，接口可以继承接口，但不能实现接口。
* 接口只能做方法声明，jdk1.8之后可以做实现，不过只能实现default和static关键字标注的方法；抽象类可以做方法声明，也可以做方法实现，不是abstract标注的方法需要实现，而abstract标注的方法子类需要重写。
* 接口中只能定义公共的静态常量；抽象类中的变量是普通变量。
* 都不能实例化

【父类和抽象类，抽象类可以定义抽象方法，自己不用实现，而父类定义的方法自己都需要实现】

【接口可以继承，也可以实现接口，类和抽象类都不能继承接口，只能实现接口】

### 2. 方法执行顺序

方法默认修饰是default

静态方法 > 构造块 > 构造方法

```java
class CodeBlock1{
    public CodeBlock1 () {
        System.out.println("CodeBlock1的构造方法");
    }
    {
        System.out.println("CodeBlock1的构造代码块");
    }
    static {
        System.out.println("CodeBlock1的静态方法");
    }
}

public class CodeTest {

    public CodeTest () {
        System.out.println("CodeTest的构造方法");
    }
    {
        System.out.println("CodeTest的构造代码块");
    }
    static {
        System.out.println("CodeTest的静态方法");
    }

    public static void main(String[] args) {
        // 因为main方法包含在CodeTest类中，static方法在类实例化（new时）的时候执行，所以先执行CodeTest类的static方法
        System.out.println("main方法");
        new CodeBlock1();
        System.out.println("---------");
        new CodeBlock1();
        System.out.println("------------");
        new CodeTest();
    }

}

console println:

CodeTest的静态方法
main方法
CodeBlock1的静态方法
CodeBlock1的构造代码块
CodeBlock1的构造方法
---------
CodeBlock1的构造代码块
CodeBlock1的构造方法
------------
CodeTest的构造代码块
CodeTest的构造方法
```

### 3. instance

分为3步：

```
memory = allocate(); // 分配对象内存空间
instance(memory); // 初始化对象
instance = memory; // 设置instance指向刚分配的内存地址，此时instance != null
```

### 4. 对象布局

* 查看对象布局

  ```xml
  <dependency>
  	<groupId>org.openjdk.jol</groupId>
  	<artifactId>jol-core</artifactId>
  	<version>0.9</version>
  </dependency>
  ```

  ```java
  Object o = new Object();
  
  System.out.println(ClassLayout.parseInstance(o).toPrintable());
  ```

  ```console
  java.lang.Object object internals:
  OFFSET  SIZE   TYPE DESCRIPTION            VALUE
        0     4        (object header)       01 00 00 00 (00000001 00000000 00000000 00000000) (1)
        4     4        (object header)       00 00 00 00 (00000000 00000000 00000000 00000000) (0)
        8     4        (object header)       e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
       12     4        (loss due to the next object alignment)
  Instance size: 16 bytes
  Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
  ```

* 对象头-----java对象的布局----java对象由什么组成----对象在堆上面要分配多大内存

答：因为64bit jvm byte，所以内存要是8的倍数

1. Java对象的实例数据（即属性等等）--不固定
2. 对象头--固定：(共12字节)
   * MarkWord（8字节）
   * klass pointer / Class Metadata Address：属于哪个类（4字节）
3. 数据对齐（将对象补齐到8的倍数大小）

对象头：

​	包含堆对象布局、类型、GC状态、同步状态和标识哈希码的基本信息，由两个词组成。

​	class指针、锁信息、hashCode

![image-20200927164343510](C:\Users\lsy\AppData\Roaming\Typora\typora-user-images\image-20200927164343510.png)

* 对象状态：

1. 无状态 new出来的时候   000
2. 偏向锁   101
3. 轻量锁    00
4. 重量锁    10
5. GC标记   11

## 十四、Linux

### 1. 常用命令

* top：查看cpu占用，显示的load average: 1.43, 1.02, 0.49 这个参数表示系统的负载均衡，3个值分别表示系统1分钟、5分钟、15分钟平均负载值，当3个值相加平均值除以100%超过60%，表示系统当前压力较大。uptime：是top命令的简化版。

* vmstat -n <采样间隔时间> <采样间隔次数>：采样间隔是2秒，采样次数是3次。显示的参数

  * procs：

    r 表示运行和等待CPU时间片的进程数，整个系统的运行队列不能超过总核数的2倍，否则表示系统压力过大。

    b 表示等待资源的进程数。

  * cpu：us + sy 大于80%，说明可能存在cpu不足。

    us：用户进程消耗cpu时间百分比。

    sy：内核进程消耗cpu时间百分比。

    id：空闲cpu的百分比。

* ps -ef|grep <进程包含的名字>：查看包含名字的进程情况

  * pidstat -u  <采样间隔时间> -p <PID>：每1秒查看对应PID进程的CPU占用情况。

* free -m：以M为单位，查看内存情况，空闲内存大于70%内存充足，20%到70%则内存够用，小于20%则说明内存不足。

  * pidstat -p <PID> -r <采样间隔时间>：查看内存占用情况

* df -h：查看硬盘剩余空间情况。

* iostat -xdk <采样间隔时间> <采样间隔次数>：查看IO情况。参数：

  * rkB/s：每秒读取数据量kB；
  * wkB/s：每秒写入数据量kB；
  * svctm ：I/O请求的平均服务时间，单位毫秒；
  * await：I/O请求的平均等待时间，单位毫秒；值越小性能越好。
  * util：一秒中有百分之几的时间用于I/O操作。接近100%时，表示磁盘带宽跑满，需要优化程序或者增加磁盘；

* netstat -ano：可以查看进程端口占用。

* tasklist |findstr <PID>：查看对应PID的进程。

* taskkill -f /PID <PID>：强制结束对应PID进程。

### 2. cpu占用过高，排查思路

* 先用top命令找出cpu占用最高的PID
* ps -ef 或者 jps -l 进一步定位，找到对应PID的程序
* 定位具体程序线程或代码
  * ps -mp <PID> -o THREAD,tid,time
    * -m：显示所有线程
    * -p：pid 进程使用cpu的时间
    * -o：该参数后是用户自定义格式
* 将需要的线程ID（TID）转换为16进制格式（英文小写格式）
* jstack <PID> | grep <线程ID16进制格式> -A60：显示该线程的问题代码行及其他情况。

## 十五、JUC

### 1. java.util.concurrent

* [Interface] BlockingQueue
  * ArrayBlockingQueue：由数组结构组成的有界阻塞队列
  * LinkedBlockingQueue：由链表结构组成的有界（但大小默认Integer.MAX_VALUE）阻塞队列
  * synchronousQueue：不存储元素的阻塞队列，也即单个元素队列
  
  | 方法类型 | 抛出异常  | 特殊值   | 阻塞   | 超时                 |
  | -------- | --------- | -------- | ------ | -------------------- |
  | 插入     | add()     | offer(e) | put(e) | offer(e, time, unit) |
  | 移除     | remove()  | poll()   | take() | poll(time, unit)     |
  | 检查     | element() | peek()   | 不可用 | 不可用               |
  
* [Interface] Callable

* [Interface] ConcurrentHashMap
  
  * ConcurrentHashMap
  
* [Interface] Executor
  * Executors
  * ThreadPoolExecutor
  
* [Interface] ExecutorService

* [Interface] Future
  
  * FutureTask
  
* [Interface] RunnableFuture

* [Interface] ThreadFactory

* CopyOnWriteArrayList：读写分别是两个不同的数组，底层拷贝数组Arrays.copyof()

* CopyOnWriteArraySet

* CountDownLatch：等待集齐再运行，每次减1

* CycliBarrier：等待集齐再运行，每次加1

* Semaphore：信号量，有资源就可以消耗，没资源不能消耗

### 2. java.util.concurrent.locks

* [interface] Condition
* [interface] Lock
  * ReentrantLock
* [interface] ReadWriteLock
  * ReentrantReadWriteLock

### 3. java.util.concurrent.atomic

* AtomicBoolean
* AtomicInteger
* AtomicIntegerArray
* AtomicLong
* AtomicArray
* AtomicReference
* AtomicReferenceArray
* AtomicStampedReference

## 十六、Java基础

### 1. 基本数据类型

* byte：1字节
* boolean
* char：2字节
* short
* int：4字节
* float
* long：8字节
* double

【小可转大，大转小会失去精度！！！】

【String类型占4个字节，是引用类型，但是他有一个常量池，虽然同样的值指向的引用一样，但是改变其中一个对象的值，只是改变该对象的指针指向别的引用，并不会改变之前引用的值。（String对象有24个字节，包括对象头12字节，char[] value4个字节，int hash4个字节，对齐4个字节）】

【StringBuffer线程安全，StringBuilder线程不安全】

### 2. 实例化对象的5种方法

* 通过new关键字实例化对象

* 使用Object类的clone()方法

  ```java
  User user1 = new User();
  User user2 = null;
  
  user2 = user1.clone();
  ```

* 通过反射对对象进行初始化

  ```java
  Class clazz = Class.forName("com.reece.admin.Member");
  
  // 无参构造器
  Constructor<Member> con = clazz.getConstructor();
  Member m = con.newInstance();
  
  // Member m = clazz.newInstance();
  ```

* IO流（反序列化）

* 工厂返回对象

### 3. 设计模式

* 6大原则：
  * 单一原则：一个类或一个方法只负责一项职责
  * 开闭原则：对扩展开放，对修改关闭
  * 依赖倒置原则：各类直接尽量不要相互依赖，尽量依赖于接口，实现解耦
  * 接口隔离原则：接口细化，单一
  * 里氏替换原则：所有引用父类（基类）的地方，必须能透明的使用子类的对象；父类出现的地方，都能替换为其子类
  * 迪米特原则：减少对象间的交互，或者交互时引入第三者来做中介，实现解耦

  
#### 3.1 代理模式

  * jdk动态代理（java默认），目标对象一定要实现接口，因为调用newInstance的参数中需要传入目标对象的接口。

  ```java
  public class JdkProxy implements InvocationHandler{
      private Object target;
      
      public Object getProxy(Object obj) {
          this.target = obj;
          // 参数：类加载器  接口 事件处理InvocationHandler
          return Proxy.newInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), this);
      }
      
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
          // 事务1...
          
          // 运用反射执行目标方法
          Object ret = method.invoke(target, args);
          
          //事务2...
          return ret;
      }
  }
  
  // 调用
  JdkProxy jdkProxy = new JdkProxy();
  Phone phone = (Phone)jdkProxy.getProxy(new Iphone);
  phone.call();
  ```

  * cglib代理，目标对象不需要实现接口；代理类实现MethodInterceptor，使用Enhancer实现代理。

  ```java
  public class CglibProxy implements MethodInterceptor {
      
      private Object target;
      
      public Object getProxy(Object obj) {
          this.target = obj;
          // 参数：类 事件处理MethodInterceptor
          return Enhancer.create(obj.getClass(), this);
      }
      
      @Override
      public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) {
          Object ret = method.invoke(target, args);
          return ret;
      }
  }
  
  // 调用
  CglibProxy cglibProxy = new CglibProxy();
  Phone phone = (Phone)cglibProxy.getProxy(new Iphone);
  phone.call();
  ```

#### 3.2 工厂模式

​	静态工厂模式，简单工厂模式，工厂方法模式，抽象工厂模式

#### 3.3 装饰者模式

在对象创建之后觉得不满意，调用装饰类进行装饰，采用的是聚合的方法。

#### 3.4 建造者模式

​	跟工厂模式类似，但是建造者模式注重过程，可以用链式调用方法，比如链式设置参数或者一些步骤，最后通过建造方法返回建造对象。

#### 3.5 单例模式

分为饿汉模式和懒汉模式，大致有5种方法，spring默认单例，饿汉模式，开启懒汉模式只需加注解@Lazy

```java
// 1.饿汉，常量对象
public class Singleton01 {

    public static final Singleton01 INSTANCE = new Singleton01();

    private Singleton01() {

    }

    public static Singleton01 getInstance() {
        return INSTANCE;
    }
}

// 2.饿汉，静态代码块
public class Singleton02 {

    private static Singleton02 INSTANCE;

    private Singleton02() {

    }

    static {
        INSTANCE = new Singleton02();
    }

    public static Singleton02 getInstance() {
        return INSTANCE;
    }
}

// 3.饿汉，静态内部类
public class Singleton04 {

    private Singleton04(){}

    public static class Singleton04Handler{
        private final static Singleton04 INSTANCE = new Singleton04();
    }

    public static Singleton04 getInstance(){
        return Singleton04Handler.INSTANCE;
    }
}

// 4.懒汉，双重检查
public class Singleton03 {

    private volatile static Singleton03 INSTANCE;

    private Singleton03(){

    }

    public static Singleton03 getInstance() {
        if (INSTANCE == null) {
            // 双重检查
            synchronized (Singleton03.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Singleton03();
                    return INSTANCE;
                }
            }
        }
        return INSTANCE;
    }
}

// 5.枚举，解决线程同步，防止序列化
public enum Singleton05 {
    INSTANCE;
}
```

#### 3.6 迭代器

Iterator：用于遍历容器

容器继承了Colletion接口，而Colletion接口有iterator方法，则容器需要实现Iterator方法，返回一个继承Iterator接口的内部类，内部类实现Iterator接口的hasNext方法和next方法。

```java
// 遍历hashMap
Iterator iterator = map.entrySet().iterator();
while(iterator.hasNext()){
	System.out.println(iterator.next());
}

// 遍历arraylist
Iterator iterator = list.iterator();
while(iterator.hasNext()){
	System.out.println(iterator.next());
}
```

#### 3.7 原型模式

要实现克隆的类需要继承Cloneable接口（空接口，作为标记使用），然后重写object的clone方法（因为Object的clone方法是protect，并且是native方法，只有他的子类才能调用，所以需要将他重写并定义为public方法）；只重写clone方法不继承Cloneable接口会报CloneNotSupportedException。

```java
public class Prototype {

    public static void main(String[] args) throws CloneNotSupportedException {
        Person p1 = new Person();
        // clone会将对象中的属性原封不动的clone到另一个对象中，包括对象中的引用对象
        Person p2 = (Person) p1.clone();
        System.out.println(p2.age + " " + p2.score);

        System.out.println(p1 == p2);
        System.out.println(p2.loc);

        // 此处因为Location没有进行深拷贝，所以引用的是同个对象，
        // 如果需要不同对象，则Location也要继承Cloneable接口并进行深拷贝
        System.out.println(p1.loc == p2.loc);
        p1.loc.stree = "sh";
        System.out.println(p2.loc);
    }

}

class Person implements Cloneable{
    int age = 8;
    int score = 100;

    Location loc = new Location("bj", 22);

    @Override
    protected Object clone() throws CloneNotSupportedException {
        
        return super.clone();
        
        /*
        // 深度拷贝
            Person p = (Person) super.clone();
            p.loc = (Location)loc.clone();
            return p;
        */
    }
}

class Location /*implements Cloneable*/{
    String stree;
    int roomNo;

    public Location(String stree, int roomNo) {
        this.stree = stree;
        this.roomNo = roomNo;
    }

    /*@Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }*/
    
    @Override
    public String toString() {
        return "Location{" +
                "stree='" + stree + '\'' +
                ", roomNo=" + roomNo +
                '}';
    }
}
```

#### 3.8 memento备份模式与序列化

```java
// save，类实现Serializable，之后进行文件备份
File f = new File("c:/memento.data");
ObjectOutputStream oos = null;
try {
    oos =  new ObjectOutputStream(new FileOutputStream(f));
    oos.writeObject(obj);
} catch(FileNotFoundException e) {
    e.printStackTrace();
} catch(IOException e) {
    e.printStackTrace();
} finally {
    if(oos != null) {
        oos.close();
    }
}


// load，读取备份文件，加载
File f = new File("c:/memento.data");
ObjectInputStream ois = null;
try {
    ois=  new ObjectInputStream(new FileInputStream(f));
    // 此处，先存的是什么对象，则先读的是什么对象
    Object obj = (Object)ois.readObject();
} catch(FileNotFoundException e) {
    e.printStackTrace();
} catch(IOException e) {
    e.printStackTrace();
} finally {
    if(ois != null) {
        ois.close();
    }
}
```

#### 3.9 TemplateMethod 模板方法（钩子函数）

```java
public class TemplateMethod {

    public static void main(String[] args) {
        F f = new C1();
        f.m();
    }

}

// 父类给子类提供方法，由子类去自己补充或重定义，调用的时候则覆盖方法
abstract class F {
    void m() {
        op1();
        op2();
    }

    abstract void op1();

    abstract void op2();
}

class C1 extends F{

    @Override
    void op1() {
        System.out.println("C1's op1");
    }

    @Override
    void op2() {
        System.out.println("C1's op2");
    }
}
```

#### 3.10 Adapter适配者模式

```java
FileInputStream fs = new FileInputStream("c:/data.txt");

// InputStreamReader此处充当适配器，一边是Inputstream字节流，一边是BufferedReader字符流，这里进行转接
InputStreamReader isr = new InputStreamReader(fs);

BufferedReader br = new BufferedReader(isr);

String line = br.readLine();

while (line != null && !line.equals("")){
	System.out.println(line);
}

br.close();
isr.close();
fs.close();
```

#### 3.11 Observer观察者模式

```java
// 被监听者抽象类，为了可扩展
public abstract class BeObserver {

    public abstract void addObserver(Observer observer);

    public abstract String toString();

}
```

```java
// 被监听者
public class Subject extends BeObserver{

    // 存放监听者的数组
    private List<Observer> observers = new ArrayList<>();

    private int state;
	
    // 添加监听者
    @Override
    public void addObserver(Observer observer) {
        observers.add(observer);
    }

    // 通知监听者
    public void notifyAllObserver(){
        for (Observer obs : observers) {
            obs.update();
        }
    }

    public int getState() {
        return state;
    }

    public void setState(int state) {
        this.state = state;
        notifyAllObserver();
    }
    
    @Override
    public String toString() {
        return "state " + getState();
    }
}
```

```java
// 监听者接口
public abstract class Observer {

    protected BeObServer subject;
	
    // 监听到之后的操作
    abstract void update();

}
```

```java
// 监听者实例对象
public class BinaryObserver extends Observer {

    // 构造方法声明监听者，设置监听对象，加入监听对象中的监听者数组
    public BinaryObserver(BeObServer subject){
        this.subject = subject;
        this.subject.addObserver(this);
    }

    @Override
    void update() {
        System.out.println("BinaryObserver "  + subject.toString());
    }
}
```

```java
// 调用
public static void main(String[] args) {
        Subject subject = new Subject();

        new BinaryObserver(subject);

        subject.setState(10);

        subject.setState(15);

}
```

#### 3.12 COR（Chain of Responsibility）责任链模式

一个请求接收后传递给下一个接收者。此处模拟Servlet发送请求和响应请求经过Filter的例子。

```java
// 抽象类
public abstract class Source {

    abstract String getSource();

    abstract void setSource(String str);

}
```

```java
// 模拟请求类...响应类基本一致，此处不详写
public class Request extends Source {

    private String msg;

    public Request(String msg) {
        this.msg = msg;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    @Override
    public String toString() {
        return "Request{" +
                "msg='" + msg + '\'' +
                '}';
    }

    @Override
    String getSource() {
        return getMsg();
    }

    @Override
    void setSource(String str) {
        setMsg(str);
    }
}

public class Response extends Source {
	...
}
```

```java
// 模拟Filter接口
public interface Filter {

    boolean doFilter(Request req, Response res, FilterChain chain);

}
```

```java
// 第一个过滤器，Servlet请求经过过滤器先正序，之后响应则是倒序
public class HtmlFilter implements Filter {

    @Override
    public boolean doFilter(Request req, Response res, FilterChain chain) {
        String str = req.getSource();
        req.setSource(str.replace("world", "man"));
        
        // 调用过滤
        chain.doFilter(req, res);

        // 等到上面请求调用过滤返回，则开始过滤响应
        String str2 = res.getSource();
        res.setSource(str2 + "Html");
        return true;
    }
}
```

```java
// 第二个过滤器，Servlet请求经过过滤器先正序，之后响应则是倒序
public class ServletFilter implements Filter {

    @Override
    public boolean doFilter(Request req, Response res, FilterChain chain) {
        String str = req.getSource();
        req.setSource(str.replace("Hello", "Hi"));

		// 调用过滤
        chain.doFilter(req, res);

		// 等到上面请求调用过滤返回，则开始过滤响应
        String str2 = res.getSource();
        res.setSource(str2 + "Servlet");

        return true;
    }
}
```

```java
// 过滤器链
public class FilterChain{
	
    // 存放过滤器的数组
    private List<Filter> filters = new ArrayList<>();
    // 执行到的过滤器下标
    private int index = 0;

    //添加过滤器
    public FilterChain addFilter(Filter filter) {
        filters.add(filter);
        return this;
    }

    public boolean doFilter(Request req, Response res) {
        // 当所有过滤器执行完，返回false
        if (index == filters.size()) return false;
        // 未执行完则拿到过滤器
        Filter f = filters.get(index);
        // 下标往后移
        index++;
        // 调用该过滤器进行过滤
        return f.doFilter(req, res, this);
    }
}
```

```java
public static void main(String[] args) {
    Request req = new Request("Hello world");
    Response res = new Response("Hi world");

    FilterChain filters = new FilterChain();
	
    // 添加过滤器
    filters.addFilter(new HtmlFilter())
    .addFilter(new ServletFilter());

    filters.doFilter(req, res);
	
    System.out.println(req.getMsg());
    System.out.println(res.getMsg());
}

// 打印结果
Hi man
Hi worldServletHtml
```

【过程：先添加过滤器，filter1和filter2，调用过滤器链的doFilter方法，开始遍历过滤器数组，拿到过滤器，按添加过滤器顺序，先是经过filter1，过滤请求完成之后，过滤器调用过滤器链的doFilter方法；继续获取下一个过滤器filter2，过滤完请求之后返回false，此时执行filter2的处理响应，完成，返回true，则跳回过滤器链doFilter方法，返回true；此时会跳到filter1调用过滤器链的doFilter的地方，返回true，继续执行响应过滤部分代码。完成处理请求正序经过过滤器，而处理响应则倒序经过过滤器。】

### 4. 条件

当&&条件时，前面的条件不成立，后面的条件不会执行。||则不管前面条件成不成立，后面条件都会执行。

```java
	@Test
	public void Test1()
	{
		String a = "111";
        
        // 不输出，前置条件不成立，后面条件不执行
		if(a.equals("11") && condition()){
			System.out.println("into");
		}
        
        // 输出
        // condition
        // into
        // 前置条件不成立，后面条件也执行
        if(a.equals("11") || condition()){
			System.out.println("into");
		}

	}


	public boolean condition(){
		System.out.println("condition");
		return true;
	}
```

## 十七、BIO/NIO/epoll

内存：分为用户空间和内核空间。

### 1. BIO（线程驱动模型）

socket连接（accept）和接收消息(read)会阻塞。

数据先到达网卡，应用程序执行accept方法，内核都是调用recvFrom方法，然后内核通过硬件驱动拿到数据，放到内核空间，之后应用程序执行read方法，将内核空间的数据拷贝到用户空间，之后读取用户空间的数据。accept和read都会阻塞，也就是说在执行过程中收不到其他客户端发送的连接或者数据，其他客户端数据都只存在网卡中，等到正在执行的客户端关闭后，其他客户端才能正常读取数据。

![image-20200926161120968](C:\Users\lsy\AppData\Roaming\Typora\typora-user-images\image-20200926161120968.png)

### 2. NIO（事件驱动模型）

开始的NIO是将recvFrom方法传参noblock，然后应用程序的线程中维护一个数组，存放连接，每次都循环查看是否连接和之前连接的是否有新的输入，这就不会阻塞了，可以同时接收多个客户端。但是如果在大量连接的情况下，会大量占用cpu资源。

现在采用多路复用，在线程中维护一个多路复用器select()，内核使用select/poll/epoll，在linux2.6之后都是采用epoll。

多个客户端连接，epoll给应用程序线程的多路复用器提示有客户端接入，将客户端数据全部放到内核空间，这个过程是非阻塞的，由epoll去维护一个链表，当有数据时，再调用recvFrom将数据拷贝到用户空间，这个过程是阻塞的，要一个一个拷贝，这里使用的是零拷贝（即拷贝的是数据的内存地址，减少空间消耗），然后线程对每个数据进行处理。

![image-20200927112855931](C:\Users\lsy\AppData\Roaming\Typora\typora-user-images\image-20200927112855931.png)

#### 2.1 IO多路复用

![image-20200926163044364](C:\Users\lsy\AppData\Roaming\Typora\typora-user-images\image-20200926163044364.png)

![image-20200926215826149](C:\Users\lsy\AppData\Roaming\Typora\typora-user-images\image-20200926215826149.png)

## 十八、Spring

### 1. Bean周期

* Bean实例化 --- 》Bean属性注入 --- 》 调用BeanNameAware的setBeanName方法 --- 》调用BeanFactoryAware的setBeanFactory()方法 --- 》调用ApplicationContextAware的setApplicationContext()方法 --- 》调用BeanPostProcessor的预初始化方法 --- 》调用InitializingBean的afterPropertiesSet()方法 --- 》调用自定义的初始化方法 --- 》调用BeanFactoryProcessor的初始化后方法 --- 》Bean可以使用 --- 》容器关闭 --- 》调用DisposableBean的destory()方法 --- 》调用自定义销毁方法 --- 》结束

* 狭义的Bean的周期：class - > new 对象 - > 填充属性（populateBean） - > Aware - > 初始化 - > AOP - > 放入单例池 （addSingleton）- > bean对象
  * 实例化
    * 实例化前：对象 --------InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation() ------ doCreateBean()
    * 实例化：推断构造方法，new 对象（newInstance()）--------------- createBeanInstance()
    * 实例化后：InstantiationAwareBeanPostProcessor.postProcessAfterInstantiation()  填充属性
  * 初始化
    * ----- 执行Aware，invokeAwareMethods，bean（对象 + beanName），可以拿到beanName，加载器和工厂
    * 初始化前：BeanPostProcess.postProcessBeforeInitialization()
    * 初始化：invokeInitMethods，InitializingBean.afterPropertiesSet，init-method
    * 初始化后：如果设置AOP代理，则将代理对象放入单例池；没有则将new出来的对象放入单例池。BeanPostProcessor.postProcessAfterInitialization

* BeanFactory是基于BeanDefinition生产bean的。
* class - > BeanDefinition - > BeanFactoryPostProcessor - > new 对象 - > 填充属性（populateBean） - > Aware - > 初始化 - > AOP - > 放入单例池 （addSingleton）- > bean对象

1. spring容器启动，查找并加载需要被spring管理的bean，会生成bean对应的BeanDefinition，重写BeanFactoryPostProcessor接口的postProcessBeanFactory方法，利用BeanDefinition定义bean，如单例多例、懒加载等。之后进行bean实例化和初始化，实例化分为实例化前、实例化和实例化后，初始化也分为初始化前、初始化和初始化后。

2. 在实例化前，如果bean实现InstantiationAwareBeanPostProcessor，可以自定义postProcessBeforeInstantiation方法对bean进行实例化前的设置。之后实例化bean，会推断构造方法，使用反射newInstance进行对象实例化。之后在实例化后阶段进行填充属性，如果重写postProcessAfterInstantiation方法，可以设置bean的一些属性。
3. 初始化会先执行Aware，invokeAwareMethods方法，这里可以拿到beanName、加载器和工厂等。初始化前，如果重写接口BeanPostProcess的postProcessBeforeInitialization方法，则调用，给我们机会对bean进行加工，如修改bean属性或者生成一个动态代理实例等。初始化的时候，执行afterPropertiesSet方法，配置bean。如果声明了init-method，则调用对应的方法。初始化之后，如果实现BeanPostProcessor接口，会调用postProcessAfterInitialization方法。初始化后，bean就可以使用了。
4. bean销毁，如果声明了destory-method，则调用对应方法。

【循环依赖：两个对象，一个对象的属性依赖另一个对象，而另一个对象的属性也依赖另一个对象，导致循环依赖。

AService生命周期

1. class --> BeanDefinition
2. aService = new AService() // 原始对象 --> 三级缓存{aServer: lambda(原始对象)}
3. 属性填充 --> 填充Bservice属性 --> 从单例池查找Bservice的bean对象 --> 找不到 --> 在二级缓存（HashMap）中查找 --> 找不到 --> 创建Bservice
4. 初始化
5. BeanPostProcessor --> 如果实现切面，aop
6. bean添加到单例池



3.------

BService生命周期：

1. class --> BeanDefinition
2. BService = new BService() // 原始对象
3. 属性填充 --> 填充Aservice属性 --> 从单例池查找Aservice的bean对象 --> 找不到  --> 出现循环依赖（AServer的第5步aop提前执行） --> 在二级缓存中查找 --> 找不到 --> 三级缓存 --> lambda(原始对象) --> 执行lambda --> aop --> 代理对象 --> 将生成的Aservice代理对象放入二级缓存 --> 删除三级缓存的lambda
4. 初始化
5. BeanPostProcessor
6. bean添加到单例池（concurrentHashMap）



一级缓存：singletonObjects --》concurrentHashMap

二级缓存：earlySingletonObjects --》HashMap

三级缓存：singletonFactories --》HashMap 

】

### 2. BeanFactory和ApplicationContext

BeanFactory不会在容器加载时就将bean实例化，从容器中拿bean的时候才实例化。

ApplicationContext在启动的时候就将所有bean都实例化。

### 3. BeanFactory和FactoryBean

BeanFactory是工厂，FactoryBean是一个可以生产和修饰对象生成的工厂bean，他们都是接口。当Bean实现Factory接口，通过getBean(beanName)获取的对象不是FactoryBean的实现类，而是这个实现类getObject返回的对象，要获得FactoryBean实现类，就要在getBean(&beanName)。

## 十九、ElasticSearch

### 1. 倒排索引

类似于Map，key值为数据值，value值为主键或者唯一字段的值。如数据列：

| id   | name     |
| ---- | -------- |
| 1    | 红海行动 |
| 2    | 红海     |
| 3    | 救援行动 |

建立倒排索引：

红海：1  2

行动：1  3

救援：3



IK中文分词器