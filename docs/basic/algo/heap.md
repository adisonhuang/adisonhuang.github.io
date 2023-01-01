## 实现一个小顶堆
=== "java"

    ``` java
    public class MinHeap {

        private int[] a; // 数组，从下标1开始存储数据
        private int n; // 堆可以存储的最大数据个数
        private int count; // 堆中已经存储的数据个数

        public MinHeap(int capacity) {
            a = new int[capacity + 1];
            n = capacity;
            count = 0;
        }

        // 插入元素
        public void insert(int data) {
            if (count >= n) return; // 堆满了
            ++count;
            a[count] = data;
            int i = count;
            while (i / 2 > 0 && a[i] < a[i / 2]) { // 自下往上堆化
                swap(a, i, i / 2); // swap()函数作用：交换下标为i和i/2的两个元素
                i = i / 2;
            }
        }

        // 删除堆顶元素
        public void removeMin() {
            if (count == 0) return; // 堆中没有数据
            a[1] = a[count];
            --count;
            heapify(a, count, 1);
        }

        // 堆化
        private void heapify(int[] a, int n, int i) { // 自上往下堆化
            while (true) {
                int minPos = i;
                if (i * 2 <= n && a[i] > a[i * 2]) minPos = i * 2;
                if (i * 2 + 1 <= n && a[i] > a[i * 2 + 1]) minPos = i * 2 + 1;
                if (minPos == i) break;
                swap(a, i, minPos);
                i = minPos;
            }
        }

        // 交换元素位置
        private void swap(int[] a, int i, int j) {
            int tmp = a[i];
            a[i] = a[j];
            a[j] = tmp;
        }

        public static void main(String[] args) {
            MinHeap minHeap = new MinHeap(10);
            minHeap.insert(3);
        
    }

    ```

## 实现一个大顶堆

=== "java"

    ``` java
    public class MaxHeap {

        private int[] a; // 数组，从下标1开始存储数据
        private int n; // 堆可以存储的最大数据个数
        private int count; // 堆中已经存储的数据个数

        public MaxHeap(int capacity) {
            //这里加1是指原来能装的元素个数，那去掉0位，只能装capacity个元素
            a = new int[capacity + 1];
            n = capacity;
            count = 0;
        }

        // 插入元素
        public void insert(int data) {
            if (count >= n) return; // 堆满了
            ++count;
            a[count] = data;
            shiftUp(count);
        }

        private void shiftUp(int k){
            while (k / 2 > 0 && a[k] > a[k / 2]) { // 自下往上堆化
                swap(a, k, k / 2); // swap()函数作用：交换下标为i和i/2的两个元素
                k = k / 2;
            }
        }


        // 删除堆顶元素
        public void removeMax() {
            if (count == 0) return; // 堆中没有数据
            a[1] = a[count];
            --count;
            heapify(a, count, 1);
        }

        // 堆化
        private void heapify(int[] a, int n, int i) { // 自上往下堆化
            while (true) {
                int maxPos = i;
                if (i * 2 <= n && a[i] < a[i * 2]) maxPos = i * 2;
                if (i * 2 + 1 <= n && a[i] < a[i * 2 + 1]) maxPos = i * 2 + 1;
                if (maxPos == i) break;
                swap(a, i, maxPos);
                i = maxPos;
            }
        }

        // 交换元素位置
        private void swap(int[] a, int i, int j) {
            int tmp = a[i];
            a[i] = a[j];
            a[j] = tmp;
        }

        public static void main(String[] args) {
                // MaxHeap<Integer> hMaxHeap = new MaxHeap<>(100);
            // int N = 50; // 堆中元素个数
            // int M = 100; // 堆中元素取值范围[0, M)
            // for( int i = 0 ; i < N ; i ++ )
            //     hMaxHeap.insert( new Integer((int)(Math.random() * M)) );
            // System.out.println(hMaxHeap.size());


            MaxHeap<Integer> heapShiftDown = new MaxHeap<Integer>(100);
            // 堆中元素个数
            int N = 100;
            // 堆中元素取值范围[0, M)
            int M = 100;
            for( int i = 0 ; i < N ; i ++ )
                heapShiftDown.insert( new Integer((int)(Math.random() * M)) );
            Integer[] arr = new Integer[N];
            // 将最大堆中的数据逐渐使用extractMax取出来
            // 取出来的顺序应该是按照从大到小的顺序取出来的
            for( int i = 0 ; i < N ; i ++ ){
                arr[i] = heapShiftDown.extractMax();
                System.out.print(arr[i] + " ");
            }
            // 确保arr数组是从大到小排列的
            for( int i = 1 ; i < N ; i ++ )
                assert arr[i-1] >= arr[i];
        }
        
    }

    ```   


## 实现一个优先级队列

=== "java"

    ``` java
    public class PrioritQueue {
        
        private int[] arr;
        private int size;
        
        public PrioritQueue(int initSize) {
            if (initSize < 0) {
                throw new IllegalArgumentException("The init size is less than 0");
            }
            this.arr = new int[initSize];
            this.size = 0;
        }
        
        // ------------------------------ Public methods
        
        public int max() {
            return this.arr[0];
        }
        
        public int maxAndRemove() {
            int t = max();
            
            this.arr[0] = this.arr[--size];
            sink(0, this.arr[0]);
            return t;
        }
        
        public void add(int data) {
            resize(1);
            this.arr[size++] = data;
            pop(size - 1, data);
        }
        
        // ------------------------------ Private methods
        
        /**
         * key下沉方法
         */
        private void sink(int i, int key) {
            while (2 * i <= this.size - 1) {
                int child = 2 * i;
                if (child < this.size - 1 && this.arr[child] < this.arr[child + 1]) {
                    child++;
                }
                if (this.arr[i] >= this.arr[child]) {
                    break;
                }
                swap(i, child);
                i = child;
            }
        }
        
        /**
         * key上浮方法
         */
        private void pop(int i, int key) {
            while (i > 0) {
                int parent = i / 2;
                if (this.arr[i] <= this.arr[parent]) {
                    break;
                }
                swap(i, parent);
                i = parent;
            }
        }

        public int peek() {
            return this.arr[0];
        }

        public int poll() {
            int t = peek();
            this.arr[0] = this.arr[--size];
            sink(0, this.arr[0]);
            return t;
        }
        
        /**
         * 重新调整数组大小
         */
        private void resize(int size) {
            int[] newArr = new int[this.arr.length + size];
            for (int i = 0; i < this.arr.length; i++) {
                newArr[i] = this.arr[i];
            }
            this.arr = newArr;
        }
        
        /**
         * 交换数组中两个元素的位置
         */
        private void swap(int i, int j) {
            int tmp = this.arr[i];
            this.arr[i] = this.arr[j];
            this.arr[j] = tmp;
        }
    }

    ```