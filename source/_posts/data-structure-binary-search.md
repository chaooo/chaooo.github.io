---
title: 「数据结构与算法」二分查找
date: 2022-06-02 18:02:00
tags: [算法, 数据结构]
categories: 数据结构
---

二分查找（Binary Search）又称折半查找、二分搜索、折半搜索等，查找思想有点类似**分治思想**，对应的时间复杂度为`O(logn)`。
二分查找算法仅适用于**有序**且使用**顺序存储结构**的序列（比如有序数组）。
核心思想是：不断地缩小搜索区域，降低查找目标元素的难度。（每次都通过跟区间的中间元素对比，将待查找的区间缩小为之前的一半，直到找到要查找的元素，或者区间被缩小为 0。）

- 以在升序序列中查找目标元素为例，二分查找算法的**实现思路**是：
    1. 初始状态下，将整个序列作为搜索区域（假设为 [B, E]）；
    2. 找到搜索区域内的中间元素（假设所在位置为 M），和目标元素进行比对。如果相等，则搜索成功；如果中间元素大于目标元素，将左侧 [B, M-1] 作为新的搜素区域；反之，若中间元素小于目标元素，将右侧 [M+1, E] 作为新的搜素区域；
    3. 重复执行第二步，直至找到目标元素。如果搜索区域无法再缩小，且区域内不包含任何元素，表明整个序列中没有目标元素，查找失败。
<!-- more -->


### 1. 标准二分查找
二分查找递归实现：
``` java
/**
 * 二分查找
 * 如果存在，返回其索引，否则返回 -1
 */
public int binarySearch(int[] nums, int target) {
    return searchRecursion(nums, 0, nums.length - 1, target);
}

/**
* 递归实现
*/
private int searchRecursion(int[] nums, int left, int right, int target) {
    if (left > right) return -1;
    // mid = left + (right - left) / 2;
    int mid =  left + ((right - left) >> 1);
    if (nums[mid] == target) {
        return mid;
    } else if (nums[mid] < target) {
        return searchRecursion(nums, mid+1, right, target);
    } else {
        return searchRecursion(nums, left, mid-1, target);
    }
}
```

二分查找循环实现：
``` java
/**
 * 二分查找循环实现
 */
public int binarySearch(int[] arr, int value) {
    int low = 0;
    int high = arr.length - 1;

    while (low <= high) {
        int mid =  low + ((high - low) >> 1);
        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }

    return -1;
}
```

二分查找虽然性能比较优秀，但应用场景也比较有限。
底层必须依赖**数组**，并且还要求数据是**有序**的。
对于较小规模的数据查找，我们直接使用顺序遍历就可以了，二分查找的优势并不明显。
二分查找更适合处理静态数据，也就是没有频繁的数据插入、删除操作。


### 2. 二分查找左边界
查找第一个值等于给定值的元素
``` java
public int leftSearch(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;

    while (left <= right) {
        int mid =  left + ((right - left) >> 1);
        if (nums[mid] == target) {
            if ((mid == 0) || (nums[mid - 1] != target)) return mid;
            // 即使我们找到了nums[mid] == target, 这个mid的位置也不一定就是最左侧的那个边界，
            // nums[mid - 1] == target，继续收缩右边界
            else right = mid - 1;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        }
    }
    return -1;
}
```


### 3. 二分查找右边界
查找最后一个值等于给定值的元素
``` java
public int rightSearch(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;

    while (left <= right) {
        int mid =  left + ((right - left) >> 1);
        if (nums[mid] == target) {
            if ((mid == nums.length - 1) || (nums[mid + 1] != target)) return mid;
            // 即使我们找到了nums[mid] == target, 这个mid的位置也不一定就是最右侧的那个边界，
            // nums[mid + 1] == target，继续收缩左边界
            else left = mid + 1;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        }
    }
    return -1;
}
```
