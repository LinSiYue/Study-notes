# Java学习笔记

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
**乐观锁**则认为对于同一个数据的并发操作，是不会发生修改的。在更新数据的时候，会采用尝试更新，不断重新的方式更新数据。<u>乐观的认为，不加锁的并发操作是没有事情的。</u>

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
**轻量级锁**是指当锁是<u>偏向锁</u>的时候，<u>被另一个线程所访问</u>，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。
**重量级锁**是指当锁为<u>轻量级锁</u>的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当<u>自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁</u>。重量级锁会让其他申请的线程进入阻塞，性能降低。

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
    sign .compareAndSet(current, null);
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

## 二、十大经典排序算法

**术语说明**
* **稳定**：如果a原本在b前面，而a=b，排序之后a仍然在b的前面；

* **不稳定**：如果a原本在b的前面，而a=b，排序之后a可能会出现在b的后面；

* **内排序**：所有排序操作都在内存中完成；

* **外排序**：由于数据太大，因此把数据放在磁盘中，而排序通过磁盘和内存的数据传输才能进行；

* **时间复杂度：** 一个算法执行所耗费的时间。

* **空间复杂度**：运行完一个程序所需内存的大小。

  **算法总结**
  ![img](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image001.png?raw=true)

**图片名词解释：**
* n: 数据规模
* k: “桶”的个数
* In-place: 占用常数内存，不占用额外内存
* Out-place: 占用额外内存

**算法分类**

![img](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image003.png?raw=true)

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

**最佳情况：**T(n) = O(n)   **最差情况：**T(n) = O(n2)   **平均情况：**T(n) = O(n2)

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

最佳情况：T(n) = O(n2)  最差情况：T(n) = O(n2)  平均情况：T(n) = O(n2)

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
最佳情况：T(n) = O(n)   最坏情况：T(n) = O(n2)   平均情况：T(n) = O(n2)

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
最佳情况：T(n) = O(nlogn)   最差情况：T(n) = O(n2)   平均情况：T(n) = O(nlogn)　

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

![img](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image004.png?raw=true)

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

![clip_image007.jpg](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image007.jpg?raw=true)

![clip_image009.jpg](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image009.jpg?raw=true)

![img](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image011.jpg?raw=true)

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

![img](https://github.com/LinSiYue/Study-notes/blob/master/img/%E6%8E%92%E5%BA%8F/clip_image012.png?raw=true)

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

最佳情况：T(n) = O(n+k)   最差情况：T(n) = O(n+k)   平均情况：T(n) = O(n2)　

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