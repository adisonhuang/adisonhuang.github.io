## 实现一个支持动态扩容的数组

=== "java"

    ``` java
    public class GenericArray<T> {
        private T[] data;
        private int size;
    
        // 根据传入容量，构造Array
        public GenericArray(int capacity) {
            data = (T[]) new Object[capacity];
            size = 0;
        }
    
        // 无参构造方法，默认数组容量为10
        public GenericArray() {
            this(10);
        }
    
        // 获取数组容量
        public int getCapacity() {
            return data.length;
        }
    
        // 获取当前元素个数
        public int count() {
            return size;
        }
    
        // 判断数组是否为空
        public boolean isEmpty() {
            return size == 0;
        }
    
        // 修改 index 位置的元素
        public void set(int index, T e) {
            checkIndex(index);
            data[index] = e;
        }
    
        // 获取对应 index 位置的元素
        public T get(int index) {
            checkIndex(index);
            return data[index];
        }
    
        // 查看数组是否包含元素e
        public boolean contains(T e) {
            for (int i = 0; i < size; i++) {
                if (data[i].equals(e)) {
                    return true;
                }
            }
            return false;
        }
    
        // 获取对应元素的下标, 未找到，返回 -1
        public int find(T e) {
            for ( int i = 0; i < size; i++) {
                if (data[i].equals(e)) {
                    return i;
                }
            }
            return -1;
        }
    
    
        // 在 index 位置，插入元素e, 时间复杂度 O(m+n)
        public void add(int index, T e) {
            checkIndexForAdd(index);
            // 如果当前元素个数等于数组容量，则将数组扩容为原来的2倍
            if (size == data.length) {
                resize(2 * data.length);
            }
    
            for (int i = size - 1; i >= index; i--) {
                data[i + 1] = data[i];
            }
            data[index] = e;
            size++;
        }
    
        // 向数组头插入元素
        public void addFirst(T e) {
            add(0, e);
        }
    
        // 向数组尾插入元素
        public void addLast(T e) {
            add(size, e);
        }
    
        // 删除 index 位置的元素，并返回
        public T remove(int index) {
            checkIndex(index);
    
            T ret = data[index];
            for (int i = index + 1; i < size; i++) {
                data[i - 1] = data[i];
            }
            size --;
            data[size] = null;
    
            // 缩容
            if (size == data.length / 4 && data.length / 2 != 0) {
                resize(data.length / 2);
            }
    
            return ret;
        }
    
        // 删除第一个元素
        public T removeFirst() {
            return remove(0);
        }
    
        // 删除末尾元素
        public T removeLast() {
            return remove(size - 1);
        }
    
        // 从数组中删除指定元素
        public void removeElement(T e) {
            int index = find(e);
            if (index != -1) {
                remove(index);
            }
        }
    
        @Override
        public String toString() {
            StringBuilder builder = new StringBuilder();
            builder.append(String.format("Array size = %d, capacity = %d \n", size, data.length));
            builder.append('[');
            for (int i = 0; i < size; i++) {
                builder.append(data[i]);
                if (i != size - 1) {
                    builder.append(", ");
                }
            }
            builder.append(']');
            return builder.toString();
        }
    
    
        // 扩容方法，时间复杂度 O(n)
        private void resize(int capacity) {
            T[] newData = (T[]) new Object[capacity];
    
            for (int i = 0; i < size; i++) {
                newData[i] = data[i];
            }
            data = newData;
        }
    
        private void checkIndex(int index) {
            if (index < 0 || index >= size) {
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
            }
        }
    
        private void checkIndexForAdd(int index) {
            if(index < 0 || index > size) {
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
            }
        }
    
         private String outOfBoundsMsg(int index) {
            return "Index: "+index+", Size: "+size;
        }
    }
    
    ```

=== "Kotlin"

    ``` kotlin
    /**
     * 动态扩容的数组
     */
    class DynamicArray {
        companion object {
            // 默认容量
            const val DEFAULT_CAPACITY = 10
    
            // 最大容量
            const val MAX_CAPACITY = Int.MAX_VALUE
        }
    
        // 当前已使用大小
        private var usedSize = 0
    
        // 当前容量大小
        private var capacity = 0
    
        // 数组容器
        private var data: Array<Int>
    
        init {
            this.capacity = DEFAULT_CAPACITY
            this.data = Array(this.capacity) { 0 }
        }
    
        /**
         * 增加元素
         */
        fun add(value: Int) {
            if (this.usedSize == this.capacity - 1) {
                this.doubleCapacity()
            }
            this.data[this.usedSize] = value
            ++this.usedSize
        }
    
        /**
         * 移除元素
         */
        fun remove(value: Int) {
            if (this.usedSize >= 0) {
                var target = -1
    
                // 查找目标所在位置
                for (i in 0 until this.usedSize) {
                    if (this.data[i] == value) {
                        target = i
                        break
                    }
                }
    
                // 找到了
                if (target >= 0) {
                    val size = this.usedSize - 1
    
                    // 把后续元素往前搬
                    for (i in target until size) {
                        this.data[i] = this.data[i + 1]
                    }
    
                    // 最后一个元素位置置为空
                    this.data[size] = 0
    
                    // 更新已使用大小
                    this.usedSize = size
                }
            }
        }
    
        /**
         * 通过索引设置元素的值
         */
        fun set(index: Int, value: Int) {
            if (this.checkIndex(index)) {
                this.data[index] = value
                return
            }
    
            throw IllegalArgumentException("index must be in rang of 0..${this.usedSize}")
        }
    
        /**
         * 获取元素
         */
        fun get(index: Int): Int? {
            if (this.checkIndex(index)) {
                return this.data[index]
            }
    
            throw IllegalArgumentException("index must be in rang of 0..${this.usedSize}")
        }
    
        /**
         * 获取当前数组的大小
         */
        fun getSize(): Int = this.usedSize
    
        private fun checkIndex(index: Int): Boolean {
            return index >= 0 && index < this.usedSize
        }
    
        /**
         * 按原容量的两倍进行扩容
         */
        private fun doubleCapacity() {
            if (this.capacity < MAX_CAPACITY) {
                this.capacity = Math.min(this.capacity * 2, MAX_CAPACITY)
                val newArray = Array(this.capacity) { 0 }
    
                for (i in 0 until this.usedSize) {
                    newArray[i] = this.data[i]
                }
    
                this.data = newArray
            }
        }
    }
    ```
