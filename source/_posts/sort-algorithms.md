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

> 如果发现表中有错误，请留言告知。

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

我觉得还是英文维基百科讲的比较详细、严谨。如果大家看的比较累的话，可以自己百度搜索相应的教程。

**冒泡排序**  
[https://en.wikipedia.org/wiki/Bubble_sort](https://en.wikipedia.org/wiki/Bubble_sort)

**选择排序**  
[https://en.wikipedia.org/wiki/Selection_sort](https://en.wikipedia.org/wiki/Selection_sort)

**插入排序**  
[https://en.wikipedia.org/wiki/Insertion_sort](https://en.wikipedia.org/wiki/Insertion_sort)

**快速排序**  
[https://en.wikipedia.org/wiki/Quicksort](https://en.wikipedia.org/wiki/Quicksort)

**归并排序**  
[https://en.wikipedia.org/wiki/Merge_sort](https://en.wikipedia.org/wiki/Merge_sort)

**希尔排序**  
[https://en.wikipedia.org/wiki/Shellsort](https://en.wikipedia.org/wiki/Shellsort)

**计数排序**  
[https://en.wikipedia.org/wiki/Counting_sort](https://en.wikipedia.org/wiki/Counting_sort)

**基数排序**  
[https://en.wikipedia.org/wiki/Radix_sort](https://en.wikipedia.org/wiki/Radix_sort)

**桶排序**  
[https://en.wikipedia.org/wiki/Bucket_sort](https://en.wikipedia.org/wiki/Bucket_sort)

**堆排序**  
[https://en.wikipedia.org/wiki/Heapsort](https://en.wikipedia.org/wiki/Heapsort)

## 代码实现

**主流排序算法**

```python
# 冒泡排序
# 比较前一个数和后一个数，如果前比后大，对换他们的位置；从左往右冒泡泡
def bubbleSort(l):
    for i in range(1, len(l)):
        for j in range(len(l) - i):
            if l[j] > l[j + 1]:
                l[j], l[j + 1] = l[j + 1], l[j]

# 选择排序
# 在未排序序列中找到最小（大）元素，存放到排序序列的起始位置。
def select_sort(l):  
    for i in range(len(l)):  
        small_index = i  
        for j in range(i, len(l)):  
            if l[small_index] > l[j]:  
                small_index = j  
        l[i], l[small_index] = l[small_index], l[i]  


# 插入排序（超时）
# 从第一个元素开始，该元素可以认为已经被排序  
# 取出下一个元素，在已经排序的元素序列中从后向前扫描  
# 如果被扫描的元素（已排序）大于新元素，将该元素后移一位
def insertSort(l):
    for i in range(1, len(l)):
        for j in range(i,0,-1):
            if l[j] < l[j-1]:
                l[j], l[j-1] = l[j-1], l[j]
                

# 从数列中挑出一个元素作为基准数。  
#分区过程，将比基准数大的放到右边，小于或等于它的数都放到左边。  
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

    // 堆排序（32 ms）
    void adjust(vector<int>& nums, int p, int s) {
        while (2*p+1 < s) {
            int c1 = 2*p+1;
            int c2 = 2*p+2;
            int c = (c2<s && nums[c2]>nums[c1]) ? c2 : c1;
            if (nums[c] > nums[p]) swap(nums[c], nums[p]);
            else break;
            p = c;
        }
    }

    vector<int> heapSort(vector<int>& nums) {
        int n = nums.size();
        for (int i = n/2-1; i >= 0; --i) {
            adjust(nums, i, n);
        }
        for (int i = n-1; i > 0; --i) {
            swap(nums[0], nums[i]);
            adjust(nums, 0, i);
        }
        return nums;
    }
    
    // 归并排序（192 ms）
    vector<int> mSort(vector<int>& nums, int l, int r) {
        if (l >= r) return {nums[l]};
        int m = l+(r-l)/2;
        vector<int> lnums = mSort(nums, l, m);
        vector<int> rnums = mSort(nums, m+1, r);
        vector<int> res;
        int i = 0, j = 0;
        while (i <= m-l && j <= r-m-1) {
            if (lnums[i] < rnums[j]) {
                res.push_back(lnums[i++]);
            } else {
                res.push_back(rnums[j++]);
            }
        }
        while (i <= m-l) {
            res.push_back(lnums[i++]);
        }
        while (j <= r-m-1) {
            res.push_back(rnums[j++]);
        }
        return res;
    }

    vector<int> mergeSort(vector<int>& nums) {
        int n = nums.size();
        nums = mSort(nums, 0, n-1);
        return nums;
    }

def merge_sort(arr):
    # divide to two
    if len(arr) < 2:
        return arr
    mid = int(len(arr)/2)
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

def merge(left, right):
    result = []
    j = 0
    i = 0
    while i < len(left) and j < len(right):
        if left[i] < right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    # add the larger part both left and right
    result += left[i:]
    result += right[j:]
    return result

```

**其他排序算法**

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