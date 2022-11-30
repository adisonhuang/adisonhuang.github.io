## 冒泡排序

=== "java"

    ``` java
    public static void bubbleSort(int[] a, int n) {
        if (n <= 1)
            return;
        for (int i = 0; i < n; ++i) {
            boolean flag = false;
            for (int j = 0; j < n - i - 1; ++j) {
                if (a[j] > a[j + 1]) {
                    // swap
                    int tmp = a[j];
                    a[j] = a[j + 1];
                    a[j + 1] = tmp;
                    flag = true;
                }
            }
            if (!flag)
                break; // 没有数据交换，提前退出
        }
    }
    
    ```

=== "python"

    ``` python
    def bubbleSort(self, arr):
        for i in range(1, len(arr)):
            for j in range(0, len(arr)-i):
                if arr[j] > arr[j+1]:
                    arr[j], arr[j+1] = arr[j+1], arr[j]

        return arr
    ```

## 选择排序

=== "java"

    ``` java
    public static void selectionSort(int[] a, int n) {
        if (n <= 1)
            return;
        for (int i = 0; i < n - 1; ++i) {
            // 查找最小值
            int minIndex = i;
            for (int j = i + 1; j < n; ++j) {
                if (a[j] < a[minIndex]) {
                    minIndex = j;
                }
            }
            //swap
            int tmp = a[i];
            a[i] = a[minIndex];
            a[minIndex] = tmp;
        }
    }
    ```
=== "python"

    ``` python
    def selectionSort(self, arr):
        for i in range(0, len(arr)):
            # 记录最小数的索引
            minIndex = i
            for j in range(i+1, len(arr)):
                if arr[j] < arr[minIndex]:
                    minIndex = j
            # i 不是最小数时，将 i 和最小数进行交换
            if i != minIndex:
                arr[i], arr[minIndex] = arr[minIndex], arr[i]
        return arr
    ```    


## 插入排序

=== "java"

    ``` java
    public static void insertionSort(int[] a, int n) {
        if (n <= 1)
            return;
        for (int i = 1; i < n; ++i) {
            int value = a[i];
            int j = i - 1;
            // 查找要插入的位置并移动数据
            for (; j >= 0; --j) {
                if (a[j] > value) {
                    a[j + 1] = a[j];
                } else {
                    break;
                }
            }
            a[j + 1] = value;
        }
    }
    ```
=== "python"

    ``` python
    def insertionSort(self, arr):
        for i in range(len(arr)):
            preIndex = i - 1
            current = arr[i]
            while preIndex >= 0 and arr[preIndex] > current:
                arr[preIndex+1] = arr[preIndex]
                preIndex -= 1
            arr[preIndex + 1] = current
        return arr
    ```    

## 归并排序

=== "java"

    ``` java
    public static int[] mergeSort(int[] sourceArray){
        // 对 arr 进行拷贝，不改变参数内容
        int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);

        if (arr.length < 2) {
            return arr;
        }

        int middle = (int) Math.floor(arr.length / 2);
        int[] left = Arrays.copyOfRange(arr, 0, middle);
        int[] right = Arrays.copyOfRange(arr, middle, arr.length);

        return merge(mergeSort(left), mergeSort(right));
    }

    private static int[] merge(int[] left, int[] right) {
        int[] result = new int[left.length + right.length];
        int i = 0;
        while (left.length > 0 && right.length > 0) {
            if (left[0] <= right[0]) {
                result[i++] = left[0];
                left = Arrays.copyOfRange(left, 1, left.length);
            } else {
                result[i++] = right[0];
                right = Arrays.copyOfRange(right, 1, right.length);
            }
        }

        while (left.length > 0) {
            result[i++] = left[0];
            left = Arrays.copyOfRange(left, 1, left.length);
        }

        while (right.length > 0) {
            result[i++] = right[0];
            right = Arrays.copyOfRange(right, 1, right.length);
        }

        return result;
    }
    ```
=== "python"

    ``` python
    def mergeSort(arr):
        import math
        if(len(arr)<2):
            return arr
        middle = math.floor(len(arr)/2)
        left, right = arr[0:middle], arr[middle:]
        return merge(mergeSort(left), mergeSort(right))

    def merge(self, left, right):
        result = []
        while left and right:
            if left[0] <= right[0]:
                result.append(left.pop(0))
            else:
                result.append(right.pop(0))
        while left:
            result.append(left.pop(0))
        while right:
            result.append(right.pop(0))
        return result
    ```    

## 快速排序

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
    def quickSort(self, arr, left=None, right=None):
         left = 0 if not isinstance(left, (int, float)) else left
         right = len(arr)-1 if not isinstance(right, (int, float)) else right
         if left < right:
                partitionIndex = self.partition(arr, left, right)
                self.quickSort(arr, left, partitionIndex-1)
                self.quickSort(arr, partitionIndex+1, right)
         return arr

    def partition(self, arr, left, right):
        pivot = left
        index = pivot + 1
        i = index
        while i <= right:
            if arr[i] < arr[pivot]:
                self.swap(arr, i, index)
                index += 1
            i += 1
        self.swap(arr, pivot, index - 1)
        return index - 1

    def swap(arr, i, j):
        arr[i], arr[j] = arr[j], arr[i]           
    ```    