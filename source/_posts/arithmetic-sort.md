---
title: 算法之排序
date: 2018-02-06 22:37:41
description: 整理经典的排序算法
tags:
categories:
- Arithmetic
copyright: false
---


# 简述

常见算法分类：

1. 非线性时间比较类排序：
 交换类排序：**快速排序**  和 **冒泡排序**
 插入类排序：**简单插入排序** 和 **希尔排序**
 选择类排序：**简单选择排序** 和 **堆排序**
 归并排序：**二路归并排序** 和 **多路归并排序**
2. 线性时间非比较类排序：**计数排序**、**基数排序** 和 **桶排序**；

在比较类排序中，归并排序号称最快，其次是快速排序和堆排序，两者不相伯仲；但是有一点需要注意，数据初始排序状态对堆排序不会产生太大的影响，快速排序则相反；

线性时间非比较类排序一般要优于非线性时间比较类排序，但前者对待排序元素的要求比较严格，比如计数排序要求待排序数的最大值不能太大，桶排序要求元素按照 `hash` 分桶后桶内元素的数量要均匀。

线性时间非比较类元素特点是以空间换时间；

# 算法描述和实现

## 交换类排序

### 冒泡排序

**算法思想**

从数组中第一个数开始，依次遍历数组中的每一个数，通过相邻比较交换，每一轮循环下来找出剩余未排序数的中的最大数并“冒泡”至数列的顶端。

**算法步骤**

1. 从数组中第一个数开始，依次与下一个数比较并次交换比自己小的数，直到最后一个数。如果发生交换，则继续下面的步骤，如果未发生交换，则数组有序，排序结束，此时时间复杂度为 `O(n)`；
2. 每一轮“冒泡”结束后，最大的数将出现在乱序数列的最后一位。重复步骤（1）。

**稳定性**
稳定排序

**时间复杂度**

`O(n)` 至 `O(n^2)`，平均时间复杂度为 `O(n^2)`。

最好的情况：如果待排序数据序列为正序，则一趟冒泡就可完成排序，排序码的比较次数为 `n-1` 次，且没有移动，时间复杂度为 `O(n)`。

最坏的情况：如果待排序数据序列为逆序，则冒泡排序需要 `n-1` 次趟起泡，每趟进行 `n-i` 次排序码的比较和移动，即比较和移动次数均达到最大值： 


**实现代码 Python 版**

```
    def solve(self, nums):
        for i in range(len(nums) - 1):
            # 本趟冒泡是否存在交换
            has_swap = False
            for j in range(len(nums) - (i + 1)):
                if nums[j] > nums[j + 1]:
                    nums[j] = nums[j] + nums[j + 1]
                    nums[j + 1] = nums[j] - nums[j + 1]
                    nums[j] = nums[j] - nums[j + 1]

                    has_swap = True
            if not has_swap:
                break
```

### 鸡尾酒排序

**算法思想**

鸡尾酒排序，也叫定向冒泡排序，是冒泡排序的一种改进。此算法与冒泡排序的不同处在于**从低到高然后从高到低**，而冒泡排序则仅从低到高去比较序列里的每个元素。他可以得到比冒泡排序稍微好一点的效能。

### 快速排序

**算法思想**
冒泡排序是在相邻的两个记录进行比较和交换，每次交换只能上移或下移一个位置，导致总的比较与移动次数较多。快速排序又称分区交换排序，是对冒泡排序的改进，快速排序采用的思想是分治思想。

**算法步骤**

1. 从待排序的 `n` 个记录中任意选取一个记录（通常选取第一个记录）为分区标准;
2. 把所有小于该排序列的记录移动到左边，把所有大于该排序码的记录移动到右边，中间放所选记录，称之为第一趟排序；
3. 然后对前后两个子序列分别重复上述过程，直到所有记录都排好序。

**稳定性**
不稳定排序

**时间复杂度**
`O(nlog2 n)` 到 `O(n^2)` ，平均时间复杂度 `O(nlg n)`

最好的情况：是每趟结束后，每次划分使两个子文件的长度大致相等，时间复杂度为 `O（nlog2 n）`。

最坏的情况：是待排序记录已经排好序，第一趟经过 `n-1` 次比较后第一个记录保持位置不变，并得到一个 `n-1` 个元素的子记录；第二趟经过 `n-2` 次比较，将第二个记录定位在原来的位置上，并得到一个包括 `n-2` 个记录的子文件，依次类推，这样总的比较次数是：`O(n^2)`

**实现代码 Python 版**

```
    def solve(self, nums):
        # 数组长度小于等于 1 不用再排序
        if len(nums) <= 1:
            return nums
        # 当前值、左数组，右数组进行申明
        current_value = nums[0]
        left  = []
        right = []

        # 比当前值小的放 left  当前值大的放 right
        for index in range(1, len(nums)):
            if nums[index] <= current_value:
                left.append(nums[index])
            if nums[index] > current_value:
                right.append(nums[index])

        # 将有序的 left + 当前值 + 有序的 right 进行返回
        return_arr = []
        return_arr.extend(self.solve(left))
        return_arr.extend([current_value])
        return_arr.extend(self.solve(right))
        return return_arr
```

## 插入类排序

### 简单插入排序


**算法思想**
从待排序的 `n` 个记录中的第二个记录开始，依次与前面的记录比较并寻找插入的位置，每次外循环结束后，将当前的数插入到合适的位置。

**稳定性**
稳定排序

**时间复杂度**
`O(n)` 至 `O（n^2）`，平均时间复杂度是 `O（n^2）`。

最好情况：当待排序记录已经有序，这时需要比较的次数是 `Cmin = n−1 = O(n)`。

最坏情况：如果待排序记录为逆序，则最多的比较次数为 `O(n^2)`。

**实现代码 Python 版**

```
    def solve(self, nums):
        
        for index in range(1, len(nums)):
            # 要比较值的下标
            prev = index - 1
            # 当前值临时保存
            tmp = nums[index]
            # 从后向前 比 tmp 大的值依次向后挪位
            while prev >= 0 and nums[prev] > tmp:
                nums[prev + 1] = nums[prev]
                prev = prev - 1
            # 如果 prev 指针发生变动，则把 tmp 放到合适的位置上
            if prev != index - 1:
                nums[prev + 1] = tmp
```

### 希尔排序


**算法思想**
`Shell` 排序又称缩小增量排序, 由 `D. L. Shell` 在 1959 年提出，是对直接插入排序的改进。

步长为 1 ，就是直接插入排序。

**稳定性**
不稳定排序

**时间复杂度**
`O(n^1.3)` 到 `O(n^2)`。`Shell` 排序算法的时间复杂度分析比较复杂，实际所需的时间取决于各次排序时增量的个数和增量的取值。研究证明，若增量的取值比较合理，`Shell` 排序算法的时间复杂度约为 `O(n^1.3)`。

**实现代码 Python 版**

```
    def solve(self, nums):

        # 初始步长
        step = len(nums) / 2

        while step > 0:
            for index in range(step, len(nums)):
                # 要比较值的下标
                prev = index - step
                # 当前值临时保存
                tmp = nums[index]
                # 从后向前 比 tmp 大的值依次向后挪位
                while prev >= 0 and nums[prev] > tmp:
                    nums[prev + step] = nums[prev]
                    prev = prev - step
                # 如果 prev 指针发生变动，则把 tmp 放到合适的位置上
                if prev != index - step:
                    nums[prev + step] = tmp
            # 重置步长
            step = step / 2
```

## 选择类排序

### 简单选择排序

**算法思想**

从所有记录中选出最小的一个数据元素与第一个位置的记录交换；然后在剩下的记录当中再找最小的与第二个位置的记录交换，循环到只剩下最后一个数据元素为止。


**稳定性**

不稳定排序

**时间复杂度**
最坏、最好和平均复杂度均为 `O(n^2)`，因此，简单选择排序也是常见排序算法中性能最差的排序算法。简单选择排序的比较次数与文件的初始状态没有关系，在第i趟排序中选出最小排序码的记录，需要做 `n-i` 次比较；

**实现代码 Python 版**

```
    def solve(self, nums):

        for i in range(len(nums)):
            # 保存此趟循环的最小值的下标
            min_index = i
            for j in range(i + 1, len(nums)):
                # 存在更小值， 保存更小值的下标
                if nums[j] < nums[min_index]:
                    min_index = j
            # 交换
            if min_index != i:
                nums[i] = nums[i] + nums[min_index]
                nums[min_index] = nums[i] - nums[min_index]
                nums[i] = nums[i] - nums[min_index]
```

### 堆排序

**堆的特点**
一般都用数组来存储堆，`i` 结点的父结点下标就为 `(i–1)/2` 。它的左右子结点下标分别为 `2∗i+1` 和 `2∗i+2`。如第 0 个结点左右子结点下标分别为1和2。 

**算法思想**

建立： 以最小堆为例，如果以数组存储元素时，一个数组具有对应的树表示形式，但树并不满足堆的条件，需要重新排列元素，可以建立“堆化”的树。

![此处输入图片的描述][1]

插入：将一个新元素插入到表尾，即数组末尾时，如果新构成的二叉树不满足堆的性质，需要重新排列元素

![此处输入图片的描述][2]

删除：堆排序中，删除一个元素总是发生在堆顶，因为堆顶的元素是最小的（小顶堆中）。表中最后一个元素用来填补空缺位置，结果树被更新以满足堆条件。

![此处输入图片的描述][3]


**稳定性**
不稳定排序

**时间复杂度**



**实现代码 Python 版**

```
    def solve(self, nums):
        """
        堆排序入口
        """
        # 建立最小堆
        self.build(nums)
        # 从最后一个元素开始，与根结点换位置，重新调整堆
        for i in range(len(nums) - 1, 0, -1):
            # 与根结点换位置
            nums[0] = nums[0] + nums[i]
            nums[i] = nums[0] - nums[i]
            nums[0] = nums[0] - nums[i]
            # 重新调整堆
            self.heapify(nums, 0, i)

    def build(self, nums):
        """
        建堆
        """
        # 第一个非叶结点到首结点 依次调整堆
        for i in range(len(nums) / 2 - 1, -1, -1):
            self.heapify(nums, i, len(nums))

    def heapify(self, nums, heap_index, heap_size):
        """
        自 heap_index 下标开始向下调整堆
        """
        # 判断当前结点与左右子结点的大小，如果发生变化，则对相应的子结点作递归调整
        left_child_index  = 2 * heap_index + 1
        right_child_index = 2 * heap_index + 2
        min_index = heap_index
        # 比较左子结点与最小结点
        if left_child_index < heap_size and nums[left_child_index] < nums[min_index]:
            min_index = left_child_index
        # 比较右子结点与最小结点
        if right_child_index < heap_size and nums[right_child_index] < nums[min_index]:
            min_index = right_child_index
        # 如果最小结点非父节点
        if min_index != heap_index:
            # 交换最小子结点与父结点
            nums[heap_index] = nums[heap_index] + nums[min_index]
            nums[min_index] = nums[heap_index] - nums[min_index]
            nums[heap_index] = nums[heap_index] - nums[min_index]
            # 递归调整最小子结点所在的堆
            self.heapify(nums, min_index, heap_size)
```


## 归并排序

### 二路归并排序

**算法思想**
二路归并排序主要运用了“分治算法”，分治算法就是将一个大的问题划分为 `n` 个规模较小而结构相似的子问题。

这些子问题解决的方法都是类似的，解决掉这些小的问题之后，归并子问题的结果，就得到了“大”问题的解。

**算法步骤**

1. 将一个数组分成两个数组，分别对两个数组进行排序。
2. 循环第一步，直到划分出来的“小数组”只包含一个元素，只有一个元素的数组默认为已经排好序。
3. 将两个有序的数组合并到一个大的数组中。
4. 从最小的只包含一个元素的数组开始两两合并。此时，合并好的数组也是有序的。

![][4]

**稳定性**
稳定排序

**时间复杂度**
最坏，最好和平均时间复杂度都是 `O(nlgn)`。

**实现代码 Python 版**

```
class Solution(object):
    """
    二路归并排序，指定要排序的数组，及范围
    """
    def sort(self, data, low, high):
        if low < high:
            mid = (low + high) / 2
            # 分解
            self.sort(data, low, mid)
            self.sort(data, mid  + 1, high)
            # 归并
            self.merge(data, low, mid, high)

    """
    归并 data[low, mid] 和 data[mid, high]
    """
    def merge(self, data, low, mid, high):
        # left_index 指向第 1 有序区的第 1 个元素
        left_index = low
        # right_index 指向第 2 有序区的第 1 个元素，high 为第 2 有序区的最后一个元素
        right_index = mid + 1
        # temp数组暂存合并的有序序列
        temp = []
        # 顺序选取两个有序区的较小元素，存储到 temp 数组中，因为是递增排序
        while left_index <= mid and right_index <= high:
            if data[left_index] <= data[right_index]:
                temp.append(data[left_index])
                left_index = left_index + 1
            else:
                temp.append(data[right_index])
                right_index = right_index + 1
        # 比完之后，假如第 1 个有序区仍有剩余，则直接全部复制到 temp 数组
        if left_index <= mid:
            temp.extend(data[left_index:mid + 1])
        # 比完之后，假如第 2 个有序区仍有剩余，则直接全部复制到 temp 数组
        if right_index <= high:
            temp.extend(data[right_index:high + 1])
        # 将排好序的序列，重存回到 data 中 low 到 high 区间
        for i in range(low, high + 1):
            data[i] = temp.pop(0)
```


### 多路归并排序

**算法思想**
外部排序指的是大文件的排序，即待排序的记录存储在外存储器上，待排序的文件无法一次装入内存，需要在内存和外部存储器之间进行多次数据交换，以达到排序整个文件的目的。外部排序最常用的算法是多路归并排序，即将原文件分解成多个能够一次性装人内存的部分，分别把每一部分调入内存完成排序。然后，对已经排序的子文件进行归并排序。

多路归并排序算法在常见数据结构书中都有涉及。从 2 路到多路（k 路），增大 `k` 可以减少外存信息读写时间，但 `k` 个归并段中选取最小的记录需要比较 `k-1` 次， 为得到 `u` 个记录的一个有序段共需要 `(u-1)(k-1)` 次，若归并趟数为 `s` 次，那么对 `n` 个记录的文件进行外排时，内部归并过程中进行的总的比较次数为 `s(n-1)(k-1)`，也即 `(向上取整) (logkm)(k-1)(n-1)=(向上取整)(log2m/log2k)(k-1)(n-1)`，而 `(k- 1)/log2k` 随 `k` 增而增因此内部归并时间随 `k` 增长而增长了，抵消了外存读写减少的时间，这样做不行，由此引出了“败者树” `tree of loser` 的使用。在内部归并过程中利用败者树将 `k` 个归并段中选取最小记录比较的次数降为 `(向上取整)(log2k)` 次使总比较次数为`(向上取整) (log2m)(n-1)`，与 `k` 无关。

> 败者树是完全二叉树， 因此数据结构可以采用一维数组。其元素个数为 `k` 个叶子结点、`k-1` 个比较结点、1 个冠军结点共 `2k` 个。`ls[0]`为冠军结点，`ls[1]--ls[k - 1]` 为比较结点，`ls[k]--ls[2k-1]`为叶子结点（同时用另外一个指针索引 `b[0]--b[k-1]` 指向）。另外 `bk` 为一个附加的辅助空间，不属于败者树，初始化时存着 `MINKEY` 的值。
    
**算法步骤**

1. 首先将 `k` 个归并段中的首元素关键字依次存入 `b[0]--b[k-1]` 的叶子结点空间里，然后调用 `CreateLoserTree` 创建败者树，创建完毕之后最小的关键字下标（即所在归并段的序号）便被存入 `ls[0]` 中。

2. 然后不断循环：把 `ls[0]` 所存最小关键字来自于哪个归并段的序号得到为 `q`，将该归并段的首元素输出到有序归并段里，然后把下一个元素关键字放入上一个元素本来所 在的叶子结点 `b[q]` 中，调用 `Adjust` 顺着 `b[q]` 这个叶子结点往上调整败者树直到新的最小的关键字被选出来，其下标同样存在 `ls[0]` 中。循环这个 操作过程直至所有元素被写到有序归并段里。

**稳定性**
稳定

**时间复杂度**
最坏，最好和平均时间复杂度都是 `O(nlgn)`。

**实现代码 Python 版**

```
class Solution(object):
    """
    初始化
    """
    def __init__(self):
        # 文件数量
        self.file_num = 4
        # 存放每个文件排序好的 top 1 值
        self.b = [0 for i in range(0, self.file_num)]
        # 存放失败树的比较节点，ls[0] 是最小值
        self.ls = [-1 for i in range(0, self.file_num)]
        # 是否结束文件读取
        self.isend = [0 for i in range(0, self.file_num)]
        # 如果文件中的数值没了，则用 max 最大值 顶 
        self.max = 9999999

    """
    归并排序
    """
    def sort(self):
        # 保存每个文件对象
        file_objects = [0 for i in range(0, self.file_num)]
        for i in range(0, self.file_num):
            file_objects[i] = open(r'D:\vscode_workspace\python_flask\testdata%s.txt'%i)
            self.b[i] = int(file_objects[i].readline().replace('\n',''))

        self.createLoserTree()

        fileout_object = open(r'D:\vscode_workspace\python_flask\testdataout.txt','w')

        while self.b[self.ls[0]] != self.max:
            # 输出最小值
            fileout_object.write('%s\n'%self.b[self.ls[0]])
            # 读下一个值
            tmp = file_objects[self.ls[0]].readline().replace('\n', '')
            if tmp != '':
                self.b[self.ls[0]] = int(tmp)
            else:
                self.isend[self.ls[0]] = 1
                self.b[self.ls[0]] = self.max
            # 重新调整失败树
            self.adjust(self.ls[0])
                

        # 关闭文件
        for i in range(0, self.file_num):
            file_objects[i].close()
        fileout_object.close()

    """
    创建失败树
    """
    def createLoserTree(self):
        for i in range(self.file_num - 1, -1, -1):
            self.adjust(i)

    """
    调整失败树
    """
    def adjust(self, i):
            t = (i + self.file_num) // 2
            while t > 0:
                if (self.ls[t] == -1 or self.b[i] > self.b[self.ls[t]]) and i >= 0:  # b[i] 败，则更新父节点；
                    tmp = i
                    i = self.ls[t]
                    self.ls[t] = tmp
                t = t // 2

            self.ls[0] = i
```

参考 [python实现k路归并排序，败者树](http://blog.csdn.net/helina00/article/details/32712035)

## 计数排序

**算法思想**
由于用来计数的数组 `C` 的长度取决于待排序数组中数据的范围（等于待排序数组的最大值与最小值的差加上 1），这使得计数排序对于数据范围很大的数组，需要大量时间和内存。例如：计数排序是用来排序 0 到 100 之间的数字的最好的算法，但是它不适合按字母顺序排序人名。但是，计数排序可以用在基数排序中的算法来排序数据范围很大的数组。

**算法步骤**

1. 统计数组中每个值为 `i` 的元素出现的次数，存入数组 `C` 的第 `i` 项
2. 反向填充目标数组
    
**稳定性**
不稳定

**时间复杂度**
`O(n)`

**实现代码 Python 版**

```
class Solution(object):
    """
    排序
    """
    def sort(self, nums):
        # 字典 统计出现次数
        sort_dict = {}
        # 列表
        sort_data = []

        # 统计次数
        for i in range(len(nums)):
            if not sort_dict.has_key(nums[i]):
                sort_dict[nums[i]] = 0
            sort_dict[nums[i]] += 1
        
        # 填补
        sort_dict_keys = sort_dict.keys()
        for i in range(len(sort_dict_keys)):
            value = sort_dict[sort_dict_keys[i]]

            while value > 0:
                sort_data.append(sort_dict_keys[i])
                value = value - 1
        
        return sort_data
```

## 基数排序

**算法思想**
基数排序（`Radix sort`）是一种非比较型整数排序算法，其原理是将整数按位数切割成不同的数字，然后按每个位数分别比较。由于整数也可以表达字符串（比如名字或日期）和特定格式的浮点数，所以基数排序也不是只能使用于整数。

**算法步骤**

将所有待比较数值（正整数）统一为同样的数位长度，数位较短的数前面补零。

然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后，数列就变成一个有序序列。

**稳定性**
稳定

**时间复杂度**
基数排序的时间复杂度是 `O(k·n)`，其中 `n` 是排序元素个数，`k` 是数字位数。注意这不是说这个时间复杂度一定优于 `O(n·log(n))`，`k` 的大小取决于数字位的选择和待排序数据所属数据类型的全集的大小；`k` 决定了进行多少轮处理，而 `n` 是每轮处理的操作数目。

基数排序基本操作的代价较小，`k` 一般不大于 `logn` ，所以基数排序一般要快过基于比较的排序，比如快速排序。
　　
**实现代码 Python 版**

```
class Solution(object):
    """
    排序
    """
    def sort(self, nums):
        # 找到最大数，通过其长度判断要排序的趟数
        max_value = nums[0]
        for i in range(1, len(nums)):
            if max_value < nums[i]:
                max_value = nums[i]
        # 要排序的趟数
        times = len(str(max_value))

        # 初始化 10 个列表用于分配时暂存
        stash_list_array = []
        for i in range(10):
            stash_list_array.append([])
        
        for i in range(times):
            # 先分配
            for nums_i in range(len(nums)):
                index = nums[nums_i] / pow(10, i) % 10
                stash_list_array[index].append(nums[nums_i])
            # 再收集
            wait_sort_arr = []
            for i in range(10):
                wait_sort_arr.extend(stash_list_array[i])
                stash_list_array[i] = []
            nums = wait_sort_arr

        return nums
```

## 桶排序

**算法思想**

要对大小为 `[1..1000]` 范围内的 `n` 个整数 `A[1..n]` 排序，可以把桶设为大小为 10 的范围，具体而言，设集合  `B[1]` 存储 `[1..10]` 的整数，集合 `B[2]` 存储 `(10..20]` 的整数，……集合 `B[i]` 存储 `((i-1)*10, i*10]`的整数，`i = 1,2,..100`。总共有 100 个桶。然后对 `A[1..n]` 从头到尾扫描一遍，把每个 `A[i]` 放入对应的桶 `B[j]` 中。 然后再对这 100 个桶中每个桶里的数字排序，这时可用冒泡，选择，乃至快排，一般来说任何排序法都可以。最后依次输出每个桶里面的数字，且每个桶中的数字从小到大输出，这样就得到所有数字排好序的一个序列了。 

![此处输入图片的描述][5]

**稳定性**
不稳定

**时间复杂度**
`O(m+n)` ， 其中 `m` 为桶的个数， `n` 为元素的个数

**实现代码 Python 版**

```
    def sort(self, data):
        # 分桶
        sort_list = []
        for i in range(10):
            sort_list.append([])

        # 元素入桶
        for i in range(len(data)):
            index = int(data[i] * 10)
            sort_list[index].append(data[i])
        
        # 桶内元素排序 后合并
        sort_list_arr = []
        for i in range(10):
            if len(sort_list[i]) > 1:
                sort_list[i].sort()
            sort_list_arr.extend(sort_list[i])
        
        return sort_list_arr
```


  [1]: http://owk2q4gs5.bkt.clouddn.com/20160304170559093.png
  [2]: http://owk2q4gs5.bkt.clouddn.com/20160304170919536.png
  [3]: http://owk2q4gs5.bkt.clouddn.com/20160304171754671.png
  [4]: http://owk2q4gs5.bkt.clouddn.com/162133318062246.png
  [5]: http://owk2q4gs5.bkt.clouddn.com/25150004-a9e94d154e874a429ab27005f44a8cd9.jpg
