## 实现一个基于链表法解决冲突问题的散列表
=== "java"

    ``` java
    public class HashTable<K, V> {

        /**
         * 散列表默认长度
         */
        private static final int DEFAULT_INITAL_CAPACITY = 8;

        /**
         * 装载因子
         */
        private static final float LOAD_FACTOR = 0.75f;

        /**
         * 初始化散列表数组
         */
        private Entry<K, V>[] table;

        /**
         * 实际元素数量
         */
        private int size = 0;

        /**
         * 散列表索引数量
         */
        private int use = 0;

        public HashTable() {
            table = (Entry<K, V>[]) new Entry[DEFAULT_INITAL_CAPACITY];
        }

        static class Entry<K, V> {
            K key;
            V value;
            Entry<K, V> next;

            public Entry(K key, V value, HashTable.Entry<K, V> next) {
                this.key = key;
                this.value = value;
                this.next = next;
            }

        }

        /**
         * 新增
         *
         * @param key
         * @param value
         */
        public void put(K key, V value) {
            int index = hash(key);
            if (table[index] == null) {
                table[index] = new Entry(null, null, null);
            }
            Entry<K, V> tmp = table[index];
            // 新增节点
            if (tmp.next == null) {
                tmp.next = new Entry<>(key, value, null);
                size++;
                use++;
                if (use >= table.length * LOAD_FACTOR) {
                    resize();
                }
            } else {
                // 解决散列冲突，使用链表法
                do {
                    tmp = tmp.next;
                    if (tmp.key == key) {
                        tmp.value = value;
                        return;
                    }
                } while (tmp.next != null);
                Entry<K, V> temp = table[index].next;
                table[index].next = new Entry<>(key, value, temp);
                size++;
            }
        }

        /**
         * 散列函数
         * <p>
         * 参考hashmap散列函数
         *
         * @param key
         * @return
         */
        private int hash(Object key) {
            int h;
            return (key == null) ? 0 : ((h = key.hashCode()) ^ (h >>> 16)) % table.length;
        }

        /**
         * 扩容
         */
        private void resize() {
            Entry<K, V> oldTable = table;
            table = (Entry<K, V>[]) new Entry[table.length * 2];
            use = 0;
            for (int i = 0; i < oldTable.length; i++) {
                if (oldTable[i] == null || oldTable[i].next == null) {
                    continue;
                }
                Entry<K, V> e = oldTable[i];
                while (e.next != null) {
                    e = e.next;
                    int index = hash(e.key);
                    if (table[index] == null) {
                        use++;
                        // 创建哨兵节点
                        table[index] = new Entry<>(null, null, null);
                    }
                    table[index].next = new Entry<>(e.key, e.value, table[index].next);
                }
            }
        }

        /**
         * 删除
         *
         * @param key
         */
        public void remove(K key) {
            int index = hash(key);
            Entry e = table[index];
            if (e == null || e.next == null) {
                return;
            }

            Entry pre;
            Entry<K, V> headNode = table[index];
            do {
                pre = e;
                e = e.next;
                if (key == e.key) {
                    pre.next = e.next;
                    size--;
                    if (headNode.next == null)
                        use--;
                    return;
                }
            } while (e.next != null);
        }

        /**
         * 获取
         *
         * @param key
         * @return
         */
        public V get(K key) {
            int index = hash(key);
            Entry<K, V> e = table[index];
            if (e == null || e.next == null) {
                return null;
            }
            while (e.next != null) {
                e = e.next;
                if (key == e.key) {
                    return e.value;
                }
            }
            return null;
        }
    }
    ```

## 实现一个 LRU 缓存淘汰算法


=== "java"

    ``` java
    public class LRUBaseHashTable<K, V> {
        /**
         * 默认链表容量
         */
        private final static Integer DEFAULT_CAPACITY = 10;

        /**
         * 头结点
         */
        private DNode<K, V> headNode;

        /**
         * 尾节点
         */
        private DNode<K, V> tailNode;

        /**
         * 链表长度
         */
        private Integer length;

        /**
         * 链表容量
         */
        private Integer capacity;

        /**
         * 散列表存储key
         */
        private HashMap<K, DNode<K, V>> table;

        /**
         * 双向链表
         */
        static class DNode<K, V> {

            private K key;

            /**
             * 数据
             */
            private V value;

            /**
             * 前驱指针
             */
            private DNode<K, V> prev;

            /**
             * 后继指针
             */
            private DNode<K, V> next;

            DNode() {
            }

            DNode(K key, V value) {
                this.key = key;
                this.value = value;
            }

        }

        public LRUBaseHashTable(int capacity) {
            this.length = 0;
            this.capacity = capacity;

            headNode = new DNode<>();

            tailNode = new DNode<>();

            headNode.next = tailNode;
            tailNode.prev = headNode;

            table = new HashMap<>();
        }

        public LRUBaseHashTable() {
            this(DEFAULT_CAPACITY);
        }

        /**
         * 新增
         *
         * @param key
         * @param value
         */
        public void add(K key, V value) {
            DNode<K, V> node = table.get(key);
            if (node == null) {
                DNode<K, V> newNode = new DNode<>(key, value);
                table.put(key, newNode);
                addNode(newNode);

                if (++length > capacity) {
                    DNode<K, V> tail = popTail();
                    table.remove(tail.key);
                    length--;
                }
            } else {
                node.value = value;
                moveToHead(node);
            }
        }

        /**
         * 将新节点加到头部
         *
         * @param newNode
         */
        private void addNode(DNode<K, V> newNode) {
            newNode.next = headNode.next;
            newNode.prev = headNode;

            headNode.next.prev = newNode;
            headNode.next = newNode;
        }

        /**
         * 弹出尾部数据节点
         */
        private DNode<K, V> popTail() {
            DNode<K, V> node = tailNode.prev;
            removeNode(node);
            return node;
        }

        /**
         * 移除节点
         *
         * @param node
         */
        private void removeNode(DNode<K, V> node) {
            node.prev.next = node.next;
            node.next.prev = node.prev;
        }

        /**
         * 将节点移动到头部
         *
         * @param node
         */
        private void moveToHead(DNode<K, V> node) {
            removeNode(node);
            addNode(node);
        }

        /**
         * 获取节点数据
         *
         * @param key
         * @return
         */
        public V get(K key) {
            DNode<K, V> node = table.get(key);
            if (node == null) {
                return null;
            }
            moveToHead(node);
            return node.value;
        }

        /**
         * 移除节点数据
         *
         * @param key
         */
        public void remove(K key) {
            DNode<K, V> node = table.get(key);
            if (node == null) {
                return;
            }
            removeNode(node);
            length--;
            table.remove(node.key);
        }

        private void printAll() {
            DNode<K, V> node = headNode.next;
            while (node.next != null) {
                System.out.print(node.value + ",");
                node = node.next;
            }
            System.out.println();
        }

    }

    ```   