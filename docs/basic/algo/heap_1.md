## 实现堆排序
 === "java"

    ``` java
    public class HeapSort {

        public static void main(String[] args) {
            int[] arr = { 1, 3, 2, 6, 5, 7, 8, 9, 10, 0 };
            heapSort(arr);
            for (int i : arr) {
                System.out.print(i + " ");
            }
        }

        public static void heapSort(int[] arr) {
            if (arr == null || arr.length < 2) {
                return;
            }
            buildHeap(arr, arr.length);
            sort(arr, arr.length);
        }

        private static void buildHeap(int[] a, int n) {
            for (int i = n / 2; i >= 0; --i) {
                heapify(a, n, i);
            }
        }

        private static void heapify(int[] a, int n, int i) {
            while (true) {
                int maxPos = i;
                if (i * 2 +1 < n && a[i] < a[i * 2 + 1])
                    maxPos = i * 2+1;
                if (i * 2 + 2 < n && a[maxPos] < a[i * 2 + 2 ])
                    maxPos = i * 2 + 2;
                if (maxPos == i)
                    break;
                swap(a, i, maxPos);
                i = maxPos;
            }
        }

        // n表示数据的个数，数组a中的数据从下标1到n的位置。
        public static void sort(int[] a, int n) {
            buildHeap(a, n);
            int k = n-1;
            while (k > 0) {
                swap(a, 0, k);
                --k;
                heapify(a, k, 0);
            }
        }

        public static void swap(int[] arr, int i, int j) {
            int tmp = arr[i];
            arr[i] = arr[j];
            arr[j] = tmp;
        }

    }

    ```

## 利用优先级队列合并 K 个有序数组

 === "java"

    ``` java
    public static int[] mergeKSortedArrays(int[][] arr) {
        int k = arr.length;
        int n = arr[0].length;
        int[] result = new int[k * n];
        PriorityQueue<Integer> queue = new PriorityQueue<Integer>();
        for (int i = 0; i < k; i++) {
            for (int j = 0; j < n; j++) {
                queue.add(arr[i][j]);
            }
        }
        for (int i = 0; i < k * n; i++) {
            result[i] = queue.poll();
        }
        return result;
    }

    ```   

 ## 求一组动态数据集合的最大 Top K

 === "java"

    ``` java
    public static int[] topK(int[] arr, int k) {
        int[] result = new int[k];
        PriorityQueue<Integer> queue = new PriorityQueue<Integer>();
        for (int i = 0; i < arr.length; i++) {
            if (queue.size() < k) {
                queue.add(arr[i]);
            } else {
                if (queue.peek() < arr[i]) {
                    queue.poll();
                    queue.add(arr[i]);
                }
            }
        }
        for (int i = 0; i < k; i++) {
            result[i] = queue.poll();
        }
        return result;
    }
    ```   