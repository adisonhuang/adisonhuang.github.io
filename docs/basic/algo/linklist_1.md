## 实现单链表

=== "java"

    ``` java
    /**
     * 1）单链表的插入、删除、查找操作；
     * 2）链表中存储的是int类型的数据；
     */
    public class SinglyLinkedList {
    
        public static class Node{
            private int data;
            private Node next;
            public Node(int data, Node next){
                this.data = data;
                this.next = next;
            }
    
            public int getData() {
                return data;
            }
        }
    
        private Node head =null;
        public Node findByValue(int value){
            Node p = head;
            while(p!=null && p.data!=value){
                p = p.next;
            }
            return p;
        }
    
        public Node findByIndex(int index){
            Node p = head;
            int pos = 0;
            while(p!=null && pos!=index){
                pos++;
                p = p.next;
            }
            return p;
        }
          //无头结点
        //表头部插入
        //这种操作将于输入的顺序相反，逆序
        public void insertToHead(int value){
            Node newNode = new Node(value,null);
            insertToHead(newNode);
        }
    
        public void insertToHead(Node newNode){
            if(head == null){
                head = newNode;
            }else{
                newNode.next = head;
                head = newNode;
            }
        }
       //顺序插入
        //链表尾部插入
        public void insertTail(int value){
            Node newNode = new Node(value,null);
            //空链表，可以插入新节点作为head，也可以不操作
            if(head == null){
                head = newNode;
            } else {
                Node q = head;
                while(q.next!=null){
                    q = q.next;
                }
                newNode.next = q.next;
                q.next = newNode;
            }
        }
    
        public void insertAfter(Node p, int value){
            Node newNode = new Node(value,null);
            insertAfter(p,newNode);
        }
    
        public void insertAfter(Node p, Node newNode){
            if(p == null)return;
            newNode.next = p.next;
            p.next = newNode;
        }
    
        public void insertBefore(Node p , int value){
            Node newNode = new Node(value, null);
            insertBefore(p,newNode);
        }
    
        public void insertBefore(Node p , Node newNode){
            if(p == null) return;
            if(head == p){
                insertToHead(newNode);
                return;
            }
            Node q = head;
            while(q.next !=p && q != null){ 
                q= q.next;
            }
            if (q == null) {
                return;
            }
            q.next = newNode;
            newNode.next = p;
    
        }
    
        public void deleteByNode(Node p){
            if(p == null || head == null) return;
            if(p == head){
                head = head.next;
                return;
            }
            Node q = head;
            while(q.next !=p && q != null){ 
                q= q.next;
            }
            if(q == null){
                return;
            }
            q.next =q.next.next;
        }
    
        public void deleteByValue(int value){
            if(head == null) return;
            if(head.data == value){
                head = head.next;
                return;
            }
            Node q = head;
            if(q.data != value && q!=null){
                q = q.next;
            }
            if(q == null){
                return;
            }
            q.next =q.next.next;
        }
    
        public void printAll() {
            Node p = head;
            while(p!=null ){
                System.out.print(p.data+" ");
                p= p.next;
            }
            System.out.println();
        }
    }
    
    ```



=== "kotlin"

    ``` kotlin
    class SinglyLikedList {
        class Node (var data:Int,var next:Node?)
        private var head: Node? = null;
    
        companion object{
          @JvmStatic
            fun main(args: Array<String>) {
    
                val link = SinglyLinkedList()
                println("hello")
                val data = intArrayOf(1, 2, 5, 3, 1)
    
                for (i in data.indices) {
                    //link.insertToHead(data[i]);
                    link.insertTail(data[i])
                }
    
                println("打印原始:")
                link.printAll()
                // if (link.palindrome()) {
                //     println("回文")
                // } else {
                //     println("不是回文")
                // }
            }
        }
    
        fun findByValue(value:Int):Node?{
            var p = head
            while (p != null && p.data != value){
                p = p.next
            }
            return p
        }
    
        fun findByIndex(index:Int):Node?{
            var p = head
            var pos = 0
            while (p != null && pos != index){
                p = p.next
                ++pos
            }
            return p
        }
    
        fun insertToHead(value:Int){
            var newNode = Node(value,null)
            insertToHead(newNode)
        }
    
        fun insertToHead(newNode:Node){
          if(head == null){
            head = newNode
          } else {
            newNode.next = head
            head = newNode
          }
        }
    
        fun insertTail(value:Int){
            val newNode = Node(value,null)
            if(head == null){
                head = newNode
            } else {
               var q = head
               while(q?.next !=null){
                  q =q.next
               }
               newNode.next = q?.next
               q?.next = newNode
        }
        }
    
        fun insertAfter(p:Node?,value:Int){
            val newNode = Node(value,null)
            insertAfter(p,newNode)
        }
    
        fun insertAfter(p:Node?,newNode:Node){
            if(p == null) return
            newNode.next = p.next
            p.next = newNode
        }
    
        fun insertBerfore(p:Node?,value:Int){
            val newNode = Node(value,null)
            insertBerfore(p,newNode)
        }
    
        fun insertBerfore(p:Node?,newNode:Node){
            if(p == null) return
            if(head == p){
                insertToHead(newNode)
                return
            }
            var q = head
            while(q!=null && q.next !=p){
                q = q.next
            }
    
            if(q == null) return
            newNode.next = q.next
            q.next = newNode
        }
    
    
        fun deleteByNode(p:Node?){
            if(p == null || head == null) return
            if(p == head){
                head = head?.next
                return
            }
            var q = head
            while(q!=null && q.next !=p){
                q = q.next
            }
            if(q == null) return
            q.next = q.next?.next
        }
    
        fun deleteByValue(value:Int){
          if(head == null) return
          var p = head
          var q:Node? = null
          while(p!=null && p.data != value){
            q = p
            p = p.next
          }
          if(p == null) return
            if(q == null){
                head = head?.next
            } else {
                q.next = q.next?.next
            }
        }
    
        fun printAll() {
            var p = head
            while (p != null) {
                print("${p.data} ")
                p = p.next
            }
            println()
        }
    
        
    
    }
    ```



=== "js"

    ``` javascript
    class Node{
        constructor(data){
            this.data = data;
            this.next = null;
        }
    }
    
    class LinkedList {
        constructor(){
            this.head =  new Node("head");
        }
    
        findByValue(item){
            let currentNode = this.head;
            while(currentNode !== null && currentNode.data !== item){
                currentNode = currentNode.next;
            }
            return currentNode === null ? -1 : currentNode;
        }
    
        findByIndex(index){
            let currentNode = this.head.next;
            let pos = 0;
            while(currentNode !== null && pos !== index){
                currentNode = currentNode.next;
                pos++;
            }
            console.log(currentNode);
            return currentNode === null ? -1 : currentNode;
        }
    
        append(newElement){
            const newNode = new Node(newElement);
            let currentNode = this.head;
            while(currentNode.next){
                currentNode = currentNode.next;
            }
            currentNode.next = newNode;
        }
    
        insert(newElement, element){
            const currentNode = this.findByValue(element);
            if(currentNode === -1){
                console.log("未找到插入位置");
                return;
            }
            const newNode = new Node(newElement);
            newNode.next = currentNode.next;
            currentNode.next = newNode;
        }
    
        findPrev(item){
            let currentNode = this.head;
            while(currentNode.next !== null && currentNode.next.data !== item){
                currentNode = currentNode.next;
            }
            if(currentNode.next === null){
                return -1;
            }
            return currentNode;
        }
    
        remove(item){
            const prevNode = this.findPrev(item);
            if(prevNode === -1){
                console.log("未找到元素");
                return;
            }
            prevNode.next = prevNode.next.next;
        }
    
        display(){
            let currentNode = this.head.next;
            while(currentNode !== null){
                console.log(currentNode.data);
                currentNode = currentNode.next;
            }
        }
    }
    ```

=== "python"

    ``` python
    class Node:
        def __init__(self, data, next=None):
            self.data = data
            self.next = next
    
    class SinglyLinkedList:
        def __init__(self):
            self.head = None
    
        def find_by_value(self, value):
            p = self.head
            while p and p.data != value:
                p = p.next
            return p
    
        def find_by_index(self, index):
            p = self.head
            pos = 0
            while p and pos != index:
                p = p.next
                pos += 1
            return p
    
        def insert_value_to_head(self,value):
            new_node = Node(value)
            self.insert_node_to_head(new_node)
    
        def insert_node_to_head(self, new_node):
            if self.head is None:
                self.head = new_node
            else:
                new_node.next = self.head
                self.head = new_node     
        def insert_value_after(self,node,value):
            if node is None:
                return
            new_node = Node(value)
            self.insert_node_after(node,new_node)
    
        def insert_node_after(self, node, new_node):
            if not node or not new_node:
                return
            new_node.next = node.next     
            node.next = new_node
    
        def insert_value_before(self, node, value):
            new_node = Node(value)
            self.insert_node_before(node, new_node)
    
        def insert_node_before(self, node, new_node):
            if not self.head or not node or not new_node:
               return
            if self.head == node:
                self.insert_node_to_head(new_node)
                return
            current = self.head
            while current and current.next != node:
                current = current.next
            if not current:
                return
            new_node.next = current.next
            current.next = new_node    
    
        def delete_by_node(self, node):
            if not self.head or not node:
                return
            if node == self.head:
                self.head = node.next
                return
            current = self.head
            while current and current.next != node:
                current = current.next
            if not current:
                return
            current.next = current.next.next
    
        def delete_by_value(self, value):
            if not self.head:
                return
            current = self.head
            while current and current.data != value:
                current = current.next
            if not current:
                return
            self.delete_by_node(current)
    
    
        def __repr__(self):
            nums = []
            current = self.head
            while current:
                nums.append(current.data)
                current = current.next
            return "->".join(str(num) for num in nums)
    
        def __iter__(self):
            p = self.head
            while p:
                yield p.data
                p = p.next 
    ```

## 实现循环链表

=== "java"

    ``` java
    public class LoopLinkedList<T> {
        public static class Node<T> {
            // 数据
            public T data;
            // 下一个节点
            public Node next;
    
            public Node(T data) {
                this.data = data;
            }
        }
    
        public int size;
        public Node head;
    
        /**
         * 添加元素
         * 
         * @param obj
         * @return
         */
        public Node add(T obj) {
            Node newNode = new Node(obj);
            if (size == 0) {
                head = newNode;
                head.next = head;
            } else {
                Node target = head;
                while (target.next != head) {
                    target = target.next;
                }
                target.next = newNode;
                newNode.next = head;
            }
            size++;
            return newNode;
        }
    
        /**
         * 在指定位置插入元素
         * 
         * @return
         */
        public Node insert(int index, T obj) {
            if (index >= size) {
                return null;
            }
            Node newNode = new Node(obj);
            if (index == 0) {
                newNode.next = head;
                head = newNode;
            } else {
                Node target = head;
                Node previous = head;
                int pos = 0;
                while (pos != index) {
                    previous = target;
                    target = target.next;
                    pos++;
                }
                previous.next = newNode;
                newNode.next = target;
            }
            size++;
            return newNode;
        }
    
        /**
         * 删除链表头部元素
         * 
         * @return
         */
        public Node removeHead() {
            if (size > 0) {
                Node node = head;
                Node target = head;
                while (target.next != head) {
                    target = target.next;
                }
                head = head.next;
                target.next = head;
                size--;
                return node;
            } else {
                return null;
            }
        }
    
        /**
         * 删除指定位置元素
         * 
         * @return
         */
        public Node remove(int index) {
            if (index >= size) {
                return null;
            }
            Node result = head;
            if (index == 0) {
                head = head.next;
            } else {
                Node target = head;
                Node previous = head;
                int pos = 0;
                while (pos != index) {
                    previous = target;
                    target = target.next;
                    pos++;
                }
                previous.next = target.next;
                result = target;
            }
            size--;
            return result;
        }
    
        /**
         * 删除指定元素
         * 
         * @return
         */
        public Node removeNode(T obj) {
            Node target = head;
            Node previoust = head;
            if (obj.equals(target.data)) {
                head = head.next;
                size--;
            } else {
                while (target.next != null) {
                    if (obj.equals(target.next.data)) {
                        previoust = target;
                        target = target.next;
                        size--;
                        break;
                    } else {
                        target = target.next;
                        previoust = previoust.next;
                    }
                }
                previoust.next = target.next;
            }
            return target;
        }
    
        /**
         * 返回指定元素
         * 
         * @return
         */
        public Node findNode(T obj) {
            Node target = head;
            while (target.next != null) {
                if (obj.equals(target.data)) {
                    return target;
                } else {
                    target = target.next;
                }
            }
            return null;
        }
    
        /**
         * 输出链表元素
         */
        public void show() {
            if (size > 0) {
                Node node = head;
                int length = size;
                System.out.print("[");
                while (length > 0) {
                    if (length == 1) {
                        System.out.print(node.data);
                    } else {
                        System.out.print(node.data + ",");
                    }
                    node = node.next;
                    length--;
                }
                System.out.println("]");
            } else {
                System.out.println("[]");
            }
        }
    
    }
    ```

## 实现双向链表

=== "java"

    ``` java
    public class DoubleLinkedList<T> {
    
        public static class SNode<T> {
            public T data;
            public SNode<T> pre;
            public SNode<T> next;
    
            public SNode() {
    
            }
    
            public SNode(T data) {
                this.data = data;
    
            }
    
        }
    
        /**
         * 双向链表的头结点，存储第一个有效结点的基地址
         */
        private SNode<T> head;
    
        private int size;
    
        public DoubleLinkedList() {
            head = null;
            size = 0;
        }
    
        public int getSize() {
            return size;
        }
    
        public boolean isEmpty() {
            return size == 0;
        }
        /**
         * 插入结点到双向链表末尾
         * @param newNode
         */
        public void addLast(SNode<T> newNode) {
            if (isEmpty()) {
                head = newNode;
                head.next = null;
                head.pre = null;
                size++;
            } else {
                SNode<T> temp = head;
                while (temp.next != null) {
                    temp = temp.next;
                }
                // 将新结点加入双向链表
                add(temp, newNode);
            }
        }
    
        /**
         * 将新的节点插入到指定节点后
         *
         * @param preNode 指定节点
         * @param newNode 新的节点
         */
        public void add(SNode<T> preNode, SNode<T> newNode) {
            // 要插入到链表末尾时，不需要维护下一个结点的前驱指针
            if (preNode.next != null) {
                preNode.next.pre = newNode;
            }
            newNode.next = preNode.next;
            newNode.pre = preNode;
            preNode.next = newNode;
            size++;
        }
    
        /**
         * 删除数据域为指定值的元素
         *
         * @param e
         */
        public void del(T e) {
            SNode<T> temp = head;
            while (temp != null) {
                if (temp.data.equals(e)) {
                    // 维护 head，head 永远指向双向链表第一个有效结点
                    temp.next.pre = temp.pre;
                    if (temp == head) {
                        head = head.next;
                        head.pre = null;
                    } else {
                        temp.pre.next = temp.next;
                    }
                    return;
                }
                // temp 向后移
                temp = temp.next;
            }
        }
    
        /**
         * 删除指定位置的结点
         *
         * @param k
         */
        public void del(int k) {
            SNode<T> delNode = find(k);
            delNode.next.pre = delNode.pre;
            if (delNode == head) {
                head = head.next;
                head.pre = null;
            } else {
                delNode.pre.next = delNode.next.next;
            }
        }
    
        /**
         * 找到双向链表第 k 个结点
         *
         * @param k k 从 0 开始
         * @return
         */
        public SNode<T> find(int k) {
            if (k > size || k < 0) {
                throw new RuntimeException("传入的参数 k 必须大于等于零并且小于等于链表的长度 size");
            }
            int cnt = 0;
            SNode<T> t = head;
            while (cnt != k) {
                cnt++;
                t = t.next;
            }
            return t;
        }
    
         /**
         * 打印单链表所有有效节点
         *
         * @return
         */
        public String printAll() {
            StringBuilder sb = new StringBuilder();
            SNode<T> temp = head;
            while (temp != null) {
                sb.append(temp.data);
                sb.append(" ");
                temp = temp.next;
            }
            return sb.toString();
        }
    }
    ```



