## 有序数组的二分查找算法

=== "java"

    ``` java
    public static int bsearch(int[] a, int value) {
        int low = 0;
        int high = a.length - 1;
        while (low <= high) {
            int mid = low + ((high - low) >> 1);
            if (a[mid] == value) {
                return mid;
            } else if (a[mid] < value) {
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
        return -1;
    }
    

        // 二分查找的递归实现
    public static int bsearchInternallyTest(int[] a, int val) {
        return bsearchInternally(a, 0, a.length - 1, val);
    }

    public static int bsearchInternally(int[] a, int low, int high, int value) {
        if (low > high)
            return -1;
        int mid = low + ((high - low) >> 1);
        if (a[mid] == value) {
            return mid;
        } else if (a[mid] < value) {
            return bsearchInternally(a, mid + 1, high, value);
        } else {
            return bsearchInternally(a, low, mid - 1, value);
        }
    }
    ```

=== "python"

    ``` python
    def bsearch(self, arr, target):
        low = 0
        high = len(arr) - 1
        while low <= high:
            mid = low + (high - low) // 2
            if arr[mid] == target:
                return mid
            elif arr[mid] < target:
                low = mid + 1
            else:
                high = mid - 1
        return -1
    ```

## 查找有序数组第一个值等于给定值的元素

=== "java"

    ``` java
    public static int bsearchFirst(int[] a, int value) {
        int low = 0;
        int high = a.length - 1;
        while (low <= high) {
            int mid = low + ((high - low) >> 1);
            if (a[mid] > value) {
                high = mid - 1;
            } else if (a[mid] < value) {
                low = mid + 1;
            } else {
                if ((mid == 0) || (a[mid - 1] != value))
                    return mid;
                else
                    high = mid - 1;
            }
        }
        return -1;

    }
    ```
=== "python"

    ``` python
    def bsearch_first(self, arr, target):
        low = 0
        high = len(arr) - 1
        while low <= high:
            mid = low + (high - low) // 2
            if arr[mid] > target:
                high = mid - 1
            elif arr[mid] < target:
                low = mid + 1
            else:
                if mid == 0 or arr[mid - 1] != target:
                    return mid
                high = mid - 1
        return -1
    ```    


## 查找最后一个值等于给定值的元素

=== "java"

    ``` java
    public static int bsearchLast(int[] a, int value) {
        int low = 0;
        int high = a.length - 1;
        while (low <= high) {
            int mid = low + ((high - low) >> 1);
            if (a[mid] > value) {
                high = mid - 1;
            } else if (a[mid] < value) {
                low = mid + 1;
            } else {
                if ((mid == a.length - 1) || (a[mid + 1] != value))
                    return mid;
                else
                    low = mid + 1;
            }
        }
        return -1;
    }

    ```
=== "python"

    ``` python
    def bsearch_last(self, arr, target):
        low = 0
        high = len(arr) - 1
        while low <= high:
            mid = low + (high - low) // 2
            if arr[mid] > target:
                high = mid - 1
            elif arr[mid] < target:
                low = mid + 1
            else:
                if mid == len(arr) - 1 or arr[mid + 1] != target:
                    return mid
                low = mid + 1
        return -1
    ```    

## 查找第一个大于等于给定值的元素

=== "java"

    ``` java
    public static int bsearchBigger(int[] a, int value) {
        int low = 0;
        int high = a.length - 1;
        while (low <= high) {
            int mid = low + ((high - low) >> 1);
            if (a[mid] >= value) {
                if ((mid == 0) || (a[mid - 1] < value))
                    return mid;
                else
                    high = mid - 1;
            } else {
                low = mid + 1;
            }
        }
        return -1;
    }
    ```
=== "python"

    ``` python
    def bsearch_first_ge(self, arr, target):
        low = 0
        high = len(arr) - 1
        while low <= high:
            mid = low + (high - low) // 2
            if arr[mid] >= target:
                if mid == 0 or arr[mid - 1] < target:
                    return mid
                high = mid - 1
            else:
                low = mid + 1
        return -1

    ```    

## 查找最后一个小于等于给定值的元素

=== "java"

    ``` java
    private int[] quickSort(int[] arr, int left, int right) {
        if (left < right) {
            int partitionIndex = partition(arr, left, right);
            quickSort(arr, left, partitionIndex - 1);
            quickSort(arr, partitionIndex + 1, right);
        }
        return arr;
    }

    private int partition(int[] arr, int left, int right) {
        // 设定基准值（pivot）
        int pivot = left;
        int index = pivot + 1;
        for (int i = index; i <= right; i++) {
            if (arr[i] < arr[pivot]) {
                swap(arr, i, index);
                index++;
            }
        }
        swap(arr, pivot, index - 1);
        return index - 1;
    }

    private void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
    ```
=== "python"

    ``` python
    public static int bsearchLitter(int[] a, int value) {
        int low = 0;
        int high = a.length - 1;
        while (low <= high) {
            int mid = low + ((high - low) >> 1);
            if (a[mid] <= value) {
                if ((mid == a.length - 1) || (a[mid + 1] > value))
                    return mid;
                else
                    low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
        return -1;
    }       
    ```    


## [Sqrt(x) （x 的平方根）](https://leetcode.cn/problems/sqrtx/)

=== "java"

    ``` java
    public static int mySqrt(int x) {
        // 特殊值判断
        if (x == 0) {
            return 0;
        }
        if (x == 1) {
            return 1;
        }

        int left = 1;
        int right = x / 2;
        // 在区间 [left..right] 查找目标元素
        while (left < right) {
            int mid = left + (right - left + 1) / 2;
            // 注意：这里为了避免乘法溢出，改用除法
            if (mid > x / mid) {
                // 下一轮搜索区间是 [left..mid - 1]
                right = mid - 1;
            } else {
                // 下一轮搜索区间是 [mid..right]
                left = mid;
            }
        }
        return left;
    }
    ```
=== "python"

    ``` python
    def mySqrt(self, x: int) -> int:
        if x == 0 or x == 1:
            return x
        low = 0
        high = x
        while low <= high:
            mid = low + (high - low) // 2
            if mid * mid > x:
                high = mid - 1
            else:
                low = mid + 1
        return high       
    ```    

## [搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/)

> 如果有序数组是一个循环有序数组，比如 4，5，6，1，2，3。针对这种情况，如何实现一个求“值等于给定值”的二分查找算法呢？

=== "java"

    ``` java
    public int search(int[] nums, int target) {
        int n = nums.length;
        if (n == 0) {
            return -1;
        }
        if (n == 1) {
            return nums[0] == target ? 0 : -1;
        }
        int l = 0, r = n - 1;
        while (l <= r) {
            int mid = (l + r) / 2;
            if (nums[mid] == target) {
                return mid;
            }
            if (nums[0] <= nums[mid]) {
                if (nums[0] <= target && target < nums[mid]) {
                    r = mid - 1;
                } else {
                    l = mid + 1;
                }
            } else {
                if (nums[mid] < target && target <= nums[n - 1]) {
                    l = mid + 1;
                } else {
                    r = mid - 1;
                }
            }
        }
        return -1;
    }
    ```
=== "python"

    ``` python
    def search(self, nums: List[int], target: int) -> int:
        if not nums:
            return -1
        if len(nums) == 1:
            return 0 if nums[0] == target else -1
        low = 0
        high = len(nums) - 1
        while low <= high:
            mid = low + (high - low) // 2
            if nums[mid] == target:
                return mid
            if nums[low] <= nums[mid]:
                if nums[low] <= target < nums[mid]:
                    high = mid - 1
                else:
                    low = mid + 1
            else:
                if nums[mid] < target <= nums[high]:
                    low = mid + 1
                else:
                    high = mid - 1
        return -1    
    ```        