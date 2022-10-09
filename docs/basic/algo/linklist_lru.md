## 基于数组实现的LRU缓存

=== "java"

    ``` java
    
    import java.util.HashMap;
    import java.util.Map;
    /**
     *
     * 基于数组实现的LRU缓存
     * 1. 空间复杂度为O(n)
     * 2. 时间复杂度为O(n)
     * 3. 不支持null的缓存
     */
    public class LRUBasedArray<T> {
    
        private static final int DEFAULT_CAPACITY = (1 << 3);
    
        private int capacity;
    
        private int count;
    
        private T[] value;
    
        private Map<T, Integer> holder;
    
        public LRUBasedArray() {
            this(DEFAULT_CAPACITY);
        }
    
        public LRUBasedArray(int capacity) {
            this.capacity = capacity;
            value = (T[]) new Object[capacity];
            count = 0;
            holder = new HashMap<T, Integer>(capacity);
        }
    
        /**
         * 模拟访问某个值
         * @param object
         */
        public void offer(T object) {
            if (object == null) {
                throw new IllegalArgumentException("该缓存容器不支持null!");
            }
            Integer index = holder.get(object);
            if (index == null) {
                if (isFull()) {
                    removeAndCache(object);
                } else {
                    cache(object, count);
                }
            } else {
                update(index);
            }
        }
    
        /**
         * 若缓存中有指定的值，则更新位置
         * @param end
         */
        public void update(int end) {
            T target = value[end];
            rightShift(end);
            value[0] = target;
            holder.put(target, 0);
        }
    
        /**
         * 缓存数据到头部，但要先右移
         * @param object
         * @param end 数组右移的边界
         */
        public void cache(T object, int end) {
            rightShift(end);
            value[0] = object;
            holder.put(object, 0);
            count++;
        }
    
        /**
         * 缓存满的情况，踢出后，再缓存到数组头部
         * @param object
         */
        public void removeAndCache(T object) {
            T key = value[--count];
            holder.remove(key);
            cache(object, count);
        }
    
        /**
         * end左边的数据统一右移一位
         * @param end
         */
        private void rightShift(int end) {
            for (int i = end - 1; i >= 0; i--) {
                value[i + 1] = value[i];
                holder.put(value[i], i + 1);
            }
        }
    
        public boolean isContain(T object) {
            return holder.containsKey(object);
        }
    
        public boolean isEmpty() {
            return count == 0;
        }
    
        public boolean isFull() {
            return count == capacity;
        }
    
        @Override
        public String toString() {
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < count; i++) {
                sb.append(value[i]);
                sb.append(" ");
            }
            return sb.toString();
        }
    }
    
    ```

## 基于单链表LRU算法

=== "java"

    ``` java
    /**
     *  基于单链表LRU算法（java）
     */
    public class LRUBaseLinkedList {
    
           /**
         * 默认链表容量
         */
        private final static Integer DEFAULT_CAPACITY = 10;
      /**
         * 头结点
         */
        private SNode<T> headNode;
    
        /**
         * 链表长度
         */
        private Integer length;
    
        /**
         * 链表容量
         */
        private Integer capacity;
    
    
    
        public LRUBaseLinkedList() {
            this.headNode = new SNode<>();
            this.capacity = DEFAULT_CAPACITY;
            this.length = 0;
        }
    
        public LRUBaseLinkedList(Integer capacity) {
            this.headNode = new SNode<>();
            this.capacity = capacity;
            this.length = 0;
        }
    
    
        public void add(T data) {
            SNode preNode = findPreNode(data);
    
            // 链表中存在，删除原数据，再插入到链表的头部
            if (preNode != null) {
                deleteElemOptim(preNode);
                intsertElemAtBegin(data);
            } else {
                if (length >= this.capacity) {
                    //删除尾结点
                    deleteElemAtEnd();
                }
                intsertElemAtBegin(data);
            }
        }
    
    
          /**
         * 删除preNode结点下一个元素
         *
         * @param preNode
         */
        private void deleteElemOptim(SNode preNode) {
            SNode temp = preNode.getNext();
            preNode.setNext(temp.getNext());
            temp = null;
            length--;
        }
    
    
    
        /**
         * 链表头部插入节点
         *
         * @param data
         */
        private void intsertElemAtBegin(T data) {
            SNode next = headNode.getNext();
            headNode.setNext(new SNode(data, next));
            length++;
        }
    
          /**
         * 获取查找到元素的前一个结点
         *
         * @param data
         * @return
         */
        private SNode findPreNode(T data) {
            SNode node = headNode;
            while(node.getNext() !=null){
                if (data.equals(node.getNext().getElement())) {
                    return node;
                }
                node = node.getNext();
            }
            return null;
        }
    
    
        /**
         * 删除尾结点
         */
        private void deleteElemAtEnd() {
            SNode ptr = headNode;
             // 空链表直接返回
            if(ptr.getNext() == null){
                return;
            }
              // 倒数第二个结点
            while(ptr.getNext().getNext()!=null){
                    ptr = ptr.getNext();
            }
            SNode tmp = ptr.getNext();
            ptr.setNext(null);
            tmp = null;
            length --;
        }
    
        private void printAll() {
            SNode node = headNode.getNext();
            while (node != null) {
                System.out.print(node.getElement() + ",");
                node = node.getNext();
            }
            System.out.println();
        }
    
    
        public class SNode<T>{
            private T element ; 
            private SNode next;
    
            public SNode(T element) {
                this.element = element;
            }
    
            public SNode(T element, SNode next) {
                this.element = element;
                this.next = next;
            }
    
            public SNode() {
                this.next = null;
            }
    
            public T getElement() {
                return element;
            }
    
            public void setElement(T element) {
                this.element = element;
            }
    
            public SNode getNext() {
                return next;
            }
    
            public void setNext(SNode next) {
                this.next = next;
            }
        }
    }
    
    ```



## 哈希表+双向链表实现LRU

https://leetcode.cn/problems/lru-cache/?favorite=2cktkvj

=== "python"

    ``` python
    class DbListNode(object):
        def __init__(self, key, val):
            self.key = key
            self.val = val
            self.next = None
            self.prev = None
    
    class LRUCache:
        '''
            leet code: 146
            运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。
            它应该支持以下操作： 获取数据 get 和 写入数据 put 。
            获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
            写入数据 put(key, value) - 如果密钥不存在，则写入其数据值。
                当缓存容量达到上限时，它应该在写入新数据之前删除最近最少使用的数据值，从而为新的数据值留出空间
    
        哈希表+双向链表
        哈希表: 查询 O(1)
        双向链表: 有序, 增删操作 O(1)
        '''
    
        def __init__(self, capacity: int):
            self.capacity = capacity
            self.size = 0
            self.hash = {}
            # self.head和self.tail作为哨兵节点, 避免越界
            self.head = DbListNode(-1, -1)
            self.tail = DbListNode(-1, -1)
            self.head.next = self.tail
            self.tail.prev = self.head
    
        def get(self,key: int) -> int:
            if key not in self.hash:
                return -1
            node = self.hash[key]
            self._move_to_head(node)
            return node.val
        def put(self, key: int, value: int) -> None:
            if key in self.hash:
                node = self.hash[key]
                node.val = value
                self._move_to_head(node)
            else:
                node = DbListNode(key, value)
                self.hash[key] = node
                self._add_to_head(node)
                self.size += 1
                if self.size > self.capacity:
                    removed = self._remove_tail()
                    self.hash.pop(removed.key)
                    self.size -= 1
                    
        def _move_to_head(self, node):
            self._remove_node(node)
            self._add_to_head(node)
    
        def _remove_node(self, node):
            node.prev.next = node.next
            node.next.prev = node.prev
    
        def _add_to_head(self, node):
            node.next = self.head.next
            node.prev = self.head
            self.head.next.prev = node
            self.head.next = node
        def _remove_tail(self):
            node = self.tail.prev
            self._remove_node(node)
            return node
    
        def __repr__(self):
            res = []
            node = self.head.next
            while node != self.tail:
                res.append(f'{node.key}:{node.val}')
                node = node.next
            return '->'.join(res)
    
        def __str__(self):
            return self.__repr__()
    
        def __len__(self):
            return self.size
    
        def __iter__(self):
            node = self.head.next
            while node != self.tail:
                yield node
                node = node.next
        def __getitem__(self, key):
            return self.get(key)
    
        def __setitem__(self, key, value):
            self.put(key, value)
    
        def __contains__(self, key):
            return key in self.hash                                                       
    ```

