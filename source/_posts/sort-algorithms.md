---
title: 十大经典排序算法整理汇总（附代码）
top: false
cover: false
toc: true
mathjax: true
date: 2020-02-16 15:09:23
password:
summary: 本文整理并总结了十大经典的排序算法（冒泡排序、选择排序、插入排序、快速排序、归并排序、希尔排序、计数排序、基数排序、桶排序、堆排序）的时间复杂度、空间复杂度等性质。
tags:
- 算法
categories:
- 算法
---

## 前言

本文整理并总结了十大经典的排序算法（冒泡排序、选择排序、插入排序、快速排序、归并排序、希尔排序、计数排序、基数排序、桶排序、堆排序）的时间复杂度、空间复杂度等性质。

## 性质汇总

|   算法  |   最好  |  最坏   |  平均   |  空间   |  稳定性   | 是否基于比较
| --- | --- | --- | --- | --- | :---: | :---: |
|  冒泡排序   |  $O(n)$   |   $O(n^2)$  |  $O(n^2)$   |  $O(1)$   | $\checkmark$  | $\checkmark$ |
|   选择排序  |  $O(n^2)$  |   $O(n^2)$  |  $O(n^2)$   |  $O(1)$   | $\times$  | $\checkmark$ |
|   插入排序  |  $O(n)$   |   $O(n^2)$  |  $O(n^2)$   |  $O(1)$   | $\checkmark$  | $\checkmark$ |
|  快速排序   |  $O(n\log n)$   |  $O(n^2)$   |  $O(n\log n)$   |  $O(\log n)$~$O(n)$   |  $\times$   | $\checkmark$ |
|  归并排序   |  $O(n\log n)$   |   $O(n\log n)$  |  $O(n\log n)$   |   $O(n)$  |  $\checkmark$   | $\checkmark$ |
|   希尔排序  |  $O(n^{1.3})$   |   $O(n^2)$  |  $O(n\log n)$~$O(n^2)$   |  $O(1)$   | $\times$    | $\checkmark$ |
|  计数排序   |  $O(n+k)$   |   $O(n+k)$  |   $O(n+k)$  |  $O(n+k)$   |  $\checkmark$   | $\times$ |
|   基数排序  |   $O(nk)$  |  $O(nk)$   |   $O(nk)$  |   $O(n+k)$  |  $\checkmark$   | $\times$ |
|  桶排序   |   $O(n)$  |   $O(n)$  |   $O(n)$  |  $O(n+m)$   |  $\checkmark$   | $\times$ |
|  堆排序   |  $O(n\log n)$   |   $O(n\log n)$  |  $O(n\log n)$   |   $O(1)$  |  $\times$   | $\checkmark$ |



## 代码实现

### **主流排序算法**

#### 冒泡排序
比较前一个数和后一个数，如果前比后大，对换他们的位置；从左往右冒泡泡

```python
def bubble_sort(a):
    n = len(a)
    for i in range(n-1, 0, -1):
        for j in range(i):
            # j [0,n-1)... [0,2), [0,1)
            if a[j] > a[j+1]:
                a[j+1], a[j] = a[j], a[j+1]
```

#### 选择排序
在未排序序列中找到最小元素，存放到排序序列的起始位置。

```python
def sel_sort(a):
    n = len(a)
    for i in range(n):
        # [0, n), [1, n)...
        minindex = i
        for j in range(i+1, n):
            if a[j] < a[minindex]:
                minindex = j
        a[i], a[minindex] = a[minindex],a[i]
```

#### 插入排序
1. 从第一个元素开始，该元素可以认为已经被排序  
2. 取出下一个元素，在已经排序的元素序列中从后向前扫描  
3. 如果被扫描的元素（已排序）大于新元素，将该元素后移一位

```python
def insertSort(l):
    for i in range(1, len(l)):
        for j in range(i,0,-1):
            if l[j] < l[j-1]:
                l[j], l[j-1] = l[j-1], l[j]
```

#### 快排

- 从数列中挑出一个元素作为基准数。  
- 分区过程，将比基准数大的放到右边，小于或等于它的数都放到左边。  

**优化**

1、三数取中作基准

2、当待排序序列的长度分割到一定大小后，使用插入排序：对于很小和部分有序的数组，快排不如插排好。当待排序序列的长度分割到一定大小后，继续分割的效率比插入排序要差，此时可以使用插排而不是快排

3、在一次分割结束后，可以把与pivot相等的元素聚在一起，继续下次分割时，不用再对与pivot相等元素分割：实现的时候先放2边，然后调整到中间。

**复杂度分析**

https://zhuanlan.zhihu.com/p/341201904

**应用**

TOPK问题，可基于划分得到。

```python
class Solution:
    def randomized_partition(self, nums, l, r):
        pivot = random.randint(l, r)
        nums[pivot], nums[r] = nums[r], nums[pivot]
        i = l - 1
        for j in range(l, r):
            if nums[j] < nums[r]:
                i += 1
                nums[j], nums[i] = nums[i], nums[j]
        i += 1
        nums[i], nums[r] = nums[r], nums[i]
        return i

    def randomized_quicksort(self, nums, l, r):
        if r <= l:
            return
        mid = self.randomized_partition(nums, l, r)
        self.randomized_quicksort(nums, l, mid - 1)
        self.randomized_quicksort(nums, mid + 1, r)

    def sortArray(self, nums: List[int]) -> List[int]:
        self.randomized_quicksort(nums, 0, len(nums) - 1)
        return nums
```

#### 堆排序
1. 先将待排序的序列建成大根堆，使得每个父节点的元素大于等于它的子节点。
2. 此时整个序列最大值即为堆顶元素，我们将其与末尾元素交换，使末尾元素为最大值，然后再调整堆顶元素使得剩下的 n-1个元素仍为大根堆

```python
class Solution:
    def max_heapify(self, heap, root, heap_len):
        p = root
        while p * 2 + 1 < heap_len:
            l, r = p * 2 + 1, p * 2 + 2
            # 两个子节点取较大值，或者无右节点时取左节点
            if heap_len <= r or heap[r] < heap[l]:
                nex = l
            else:
                nex = r
            # 交换父节点和子节点较大值，并向下处理（一般下面已经完成有序处理）
            if heap[p] < heap[nex]:
                heap[p], heap[nex] = heap[nex], heap[p]
                p = nex
            else:
            # 不需要处理
                break

    def build_heap(self, heap):
    	# 倒序调整堆
        for i in range(len(heap) - 1, -1, -1):
            self.max_heapify(heap, i, len(heap))

    def heap_sort(self, nums):
    	# 先将待排序的序列建成大根堆，使得每个父节点的元素大于等于它的子节点
        self.build_heap(nums)
        for i in range(len(nums) - 1, -1, -1):
        # 将其与末尾元素交换，使末尾元素为最大值，然后再调整堆顶元素使得剩下的 n-1个元素仍为大根堆
            nums[i], nums[0] = nums[0], nums[i]
            self.max_heapify(nums, 0, i)

    def sortArray(self, nums: List[int]) -> List[int]:
        self.heap_sort(nums)
        return nums
```

#### 归并排序
1. 分治的思想来对序列进行排序。对一个长为 n 的待排序的序列，我们将其分解成两个长度为n/2的子序列。
2. 每次先递归调用函数使两个子序列有序，然后我们再线性合并两个有序的子序列使整个序列有序。

```python
class Solution:
    def merge_sort(self, nums, l, r):
        if l == r:
            return
        mid = (l + r) // 2
        self.merge_sort(nums, l, mid)
        self.merge_sort(nums, mid + 1, r)
        tmp = []
        i, j = l, mid + 1
        while i <= mid or j <= r:
            if i > mid or (j <= r and nums[j] < nums[i]):
                tmp.append(nums[j])
                j += 1
            else:
                tmp.append(nums[i])
                i += 1
        nums[l: r + 1] = tmp

    def sortArray(self, nums: List[int]) -> List[int]:
        self.merge_sort(nums, 0, len(nums) - 1)
        return nums
```

### **其他排序算法**

    // 希尔排序（40 ms）
    vector<int> shellSort(vector<int>& nums) {
        int n = nums.size();
        for (int gap = n/2; gap > 0; gap /= 2) {
            for (int i = gap; i < n; ++i) {
                for (int j = i; j-gap >= 0 && nums[j-gap] > nums[j]; j -= gap) {
                    swap(nums[j-gap], nums[j]);
                }
            }
        }
        return nums;
    }
    
    // 计数排序（32 ms）
    vector<int> countSort(vector<int>& nums) {
        int n = nums.size();
        if (!n) return {};
        int minv = *min_element(nums.begin(), nums.end());
        int maxv = *max_element(nums.begin(), nums.end());
        int m = maxv-minv+1;
        vector<int> count(m, 0);
        for (int i = 0; i < n; ++i) {
            count[nums[i]-minv]++;
        }
        vector<int> res;
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < count[i]; ++j) {
                res.push_back(i+minv);
            }
        }
        return res;
    }
    
    // 基数排序（不适用于负数）
    vector<int> radixSort(vector<int>& nums) {
        int n = nums.size();
        int maxv = *max_element(nums.begin(), nums.end());
        int maxd = 0;
        while (maxv > 0) {
            maxv /= 10;
            maxd++;
        }
        vector<int> count(10, 0), rank(n, 0);
        int base = 1;
        while (maxd > 0) {
            count.assign(10, 0);
            for (int i = 0; i < n; ++i) {
                count[(nums[i]/base)%10]++;
            }
            for (int i = 1; i < 10; ++i) {
                count[i] += count[i-1];
            }
            for (int i = n-1; i >= 0; --i) {
                rank[--count[(nums[i]/base)%10]] = nums[i];
            }
            for (int i = 0; i < n; ++i) {
                nums[i] = rank[i];
            }
            maxd--;
            base *= 10;
        }
        return nums;
    }
    
    // 桶排序 (20 ms)
    vector<int> bucketSort(vector<int>& nums) {
        int n = nums.size();
        int maxv = *max_element(nums.begin(), nums.end());
        int minv = *min_element(nums.begin(), nums.end());
        int bs = 1000;
        int m = (maxv-minv)/bs+1;
        vector<vector<int> > bucket(m);
        for (int i = 0; i < n; ++i) {
            bucket[(nums[i]-minv)/bs].push_back(nums[i]);
        }
        int idx = 0;
        for (int i = 0; i < m; ++i) {
            int sz = bucket[i].size();
            bucket[i] = quickSort(bucket[i]);
            for (int j = 0; j < sz; ++j) {
                nums[idx++] = bucket[i][j];
            }
        }
        return nums;
    }