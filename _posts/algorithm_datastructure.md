# 快速排序、二分查找
``` javascript
def quick_sort(nums):                                        def bin_search(nums, num):
    if len(nums) < 2:                                            low, high = 0, len(nums) - 1
        return nums                                              while low <= high:
    mid = nums[len(nums) // 2]                                       mid = (low + high) // 2
    left , right = [], []                                            tmp = nums[mid]
    nums.remove(mid)                                                 if tmp == num:
    for x in nums:                                                       return mid
        if x < mid:                                                  elif tmp > num:
            left.append(x)                                               high = mid - 1
        else:                                                        else:
            right.append(x)                                              low = mid + 1
    return quick_sort(left) + [mid] + quick_sort(right)          return None

quick_sort([3, 44, 38, 5, 47, 15, 36, 26, 27])               bin_search([1,4,6,7,8,13,78,86,98], 6)
```

# Basics Core Data Structures, Algorithms, and Concepts
![image 图像站-20180311-01](https://github.com/Django-27/workspace/blob/master/pic/data-structures1.png)
![image](https://github.com/Django-27/workspace/blob/master/pic/data-structures2.png)

- (1)
- 冒泡排序的原理是将临近的数字两两进行比较，然后按照从小到大或者从大到小的顺序进行交换
- 优化方式：加上一个标志量用来表示某趟是否发生交换;如果遍历某一趟时，没有发生任何交换，说明此时排序已经完成
```javascript
def bubble_sort_better(nums):
    for i in range(len(nums) - 1):
        flag = False
        for j in range(len(nums) -i -1):
            if nums[j] > nums[j + 1]:
                nums[j], nums[j + 1] = nums[j + 1], nums[j]
                flag = True
        if not flag:
            return nums
    return nums
```
- (2)
- 插入排序：默认前面都是有序的，下一个记录，按其关键字大小插入到前面已经排好序的子序列中的适当位置
```
def insert_while(nums):
    for index in range(1, len(nums)):
        deal_num = nums[index]
        j = index - 1
        while j >= 0 and nums[j] > deal_num:
            nums[j + 1] = nums[j]
            j -= 1
        nums[j + 1] = deal_num
```
- (3)
- 希尔排序：也称递减增量排序算法，实质是分组插入排序
- 插入排序在对几乎已经排好序的数据操作时， 效率高， 即可以达到线性排序的效率（特征）
- 插入排序每次只能将数据移动一位; 希尔排序则以步长进行分组，每次排序一个分组
- 待整个序列基本有序时，结合插入排序的特征，进行一次插入排序
- 优化部分主要在于步长的选择上
```
def shell_sort(array, n):
    step = 2
    now_gap = n // step  # 初始步长
    while now_gap > 0:
        for i in range(now_gap, n):
             # 每个步长进行插入排序
            temp = array[i]
            j = i
            # 插入排序
            while j >= now_gap and array[j - now_gap] > temp:
                array[j] = array[j - now_gap]
                j = j - now_gap
            array[j] = temp
        # 新的步长
        now_gap //= step
    return array
```
- (4)
- 直接选择排序和直接插入排序类似，都将数据分为有序区和无序区，但是将无序区的第一个元素直接插入到有序区以形成一个更大的有序区，而直接选择排序是从无序区选一个最小的元素直接放到有序区的最后
```
def select_sort(array):
    for i in range(len(array)):
        min_index = i
        for j in range(i + 1, len(array)):
            if array[j] < array[min_index]:
                min_index = j
        array[min_index], array[i] = array[i], array[min_index]
    return array
```
- (5)
- 归并排序:采用分治法, 就是先递归分解数组，再合并数组
- 递归分解到只有一个数字，则该小组有序，然后合并相邻的两个分组
- 合并时也是相邻的分组进行合并，取小的，取了之后从原来的分组中删除，并将合并且有序的数组复制回去
```
def merge_array(array1, array2):
    left_index = right_index = 0
    result = []
    # 循环比较两个数组，知道某个数组为空
    while len(array1) > left_index and len(array2) > right_index:
        if array1[left_index] < array2[right_index]:
            result.append(array1[left_index])
            left_index += 1
        else:
            result.append(array2[right_index])
            right_index += 1
    # 将不为空数组剩下的数字依次加入到结果列表中。另一个是空列表，所以可以这样实现。
    result += array1[left_index:]
    result += array2[right_index:]
    return result


def divide_array(array):
    # 结束条件
    if len(array) <= 1:
        return array

    index = len(array) // 2

    left = divide_array(array[:index])  # 左半部分
    right = divide_array(array[index:])  # 右半部分

    return merge_array(left, right)
```
- (6)
- 快速排序：先从数列中取出一个数作为基准数
- 分区过程，将比这个数大的数全放到它的右边，小于或等于它的数全放到它的左边
- 在对左右区间重复第二步，直到只有一个数
- 也是采用了分治思想，要比同为O(nlgn)的算法更快
```
# 普通实现 (Simple Way)
def quick_sort_simple(array):
    if len(array) <= 1:
        return array
    key = array[0]
    less, greater = [], []
    for i in range(1, len(array)):
        if array[i] > key:
            greater.append(array[i])
        else:
            less.append(array[i])
    return quick_sort_simple(less) + [key] + quick_sort_simple(greater)


# 使用列表推导式实现
def quick_sort_nic(array):
    if len(array) <= 1:
        return array
    return quick_sort_nic([x for x in array if x < array[0]]) + [x for x in array if x == array[0]] + quick_sort_nic([x for x in array if x > array[0]])

# 不开辟空间实现
def quick_sort(ary):
    return qsort(ary,0,len(ary)-1)
def qsort(ary,left,right):
    #快排函数，ary为待排序数组，left为待排序的左边界，right为右边界
    if left >= right : return ary
    key = ary[left]     #取最左边的为基准数
    lp = left           #左指针
    rp = right          #右指针
    while lp < rp :
        while ary[rp] >= key and lp < rp :
            rp -= 1
        while ary[lp] <= key and lp < rp :
            lp += 1
        ary[lp],ary[rp] = ary[rp],ary[lp]
    ary[left],ary[lp] = ary[lp],ary[left]
    qsort(ary,left,lp-1)
    qsort(ary,rp+1,right)
    return ary
```
- (7)
- 堆排序Heap Sort:父节点的键值总是大于或者等于任何一个子节点的键值
- 每个节点的左右子树都是一个二叉树堆(都是最大堆或者是最小堆)
- 完全二叉树：叶节点只能出现在最下层和次下层，并且最下面一层的结点都集中在该层最左边的若干位置的二叉树
- 满二叉树：除最后一层无任何子节点外，每一层上的所有结点都有两个子结点(N=2**k - 1,  n=2**(i-1))
- 堆 是一种完全二叉树，堆排序是一种树形选择排序，利用了大顶堆堆顶元素最大的特点，不断取出最大元素，并调整使剩下的元素使之还是大顶堆，依次取出最大元素就实现了排序。O(NlogN)，不稳定
```
def heap_sort(ary):
    n = len(ary)
    first = int(n / 2 - 1)  # 最后一个非叶子节点
    for start in range(first, -1, -1):  # 构造大根堆
        max_heapify(ary, start, n - 1)
    for end in range(n - 1, 0, -1):  # 堆排，将大根堆转换成有序数组
        ary[end], ary[0] = ary[0], ary[end]
        max_heapify(ary, 0, end - 1)
    return ary


# 最大堆调整：将堆的末端子节点作调整，使得子节点永远小于父节点
# start为当前需要调整最大堆的位置，end为调整边界
def max_heapify(ary, start, end):
    root = start
    while True:
        child = root * 2 + 1  # 调整节点的子节点
        if child > end:
            break
        if child + 1 <= end and ary[child] < ary[child + 1]:
            child = child + 1  # 取较大的子节点
        if ary[root] < ary[child]:  # 较大的子节点成为父节点
            ary[root], ary[child] = ary[child], ary[root]  # 交换
            root = child
        else:
            break
```
- (8)
- 桶排序 bin-sort 或 bucket-sort，将数组分到有限数量的桶里；将项目放入对应的桶里面
- 对不是空的桶进行排序，从不是空的桶里取出元素放回原来的序列
```
def insertion_sort(a):
    for i in range(0, len(a)):
        cur_val = a[i]
        pos = i
        while pos > 0 and a[pos - 1] > cur_val:
            a[pos] = a[pos - 1]
            pos -= 1
            a[pos] = cur_val

def bin_sort(nums, bucket_size=16):
    if len(nums) == 0:
        return nums
        
    # get_buckets
    buckets = [[]]
    min_v, max_v = min(nums), max(nums)
    bucket_count = (max_v - min_v) // bucket_size + 1
    for x in nums:
        buckets[int((x - min_v) // bucket_size)].append(x)
    return buckets
    
    # sort each bucket and return a new array
    a = []
    for bucket in buckets:
        a += insertion_sort(bucket)
    return a
```
- (9)
- 二分搜索：也称折半搜索，是一种在有序数组中查找某一特定元素的搜索算法
```
def binary_search_recursive(nums, val, left=0, right=None):
    right = len(nums) - 1 if right is None else right
    if right >= left:
        mid = (right + left) // 2
        if nums[mid] == val:
            return mid
        elif nums[mid] > val:
            return binary_search_recursive(nums, val, left, mid - 1)
        else:
            return binary_search_recursive(nums, val, mid + 1, right)
    return None

def binary_search(nums, val):
    left, right = 0, len(nums) - 1
    whiel left <= right:
        mid = (right + left) // 2
        if nums[mid] == val:
            return mid
        elif nums[mid] > val:
            right = mid - 1
        else:
            left = mid + 1
    return None
```
补充：
- python 实现栈和队列还是比较容易的，使用列表就可以，栈到的就是pop，append
- 队列用到的是append，出队列时 tmp = s[0]; s.remove(tmp) 即可；但都要判断为空的情况 raise IndexError,'xx is empty'  