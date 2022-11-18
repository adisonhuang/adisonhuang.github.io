## 用数组实现一个顺序队列
=== "java"

    ``` java
    public class ArrayQueue {
    private int[] items; // 数组
    private int count; // 队列中元素的个数
    private  int capacity; // 队列的容量

        public ArrayQueue(int capacity) {
            this.items = new int[capacity];
            this.capacity = capacity;
            this.count = 0;
        }

        public boolean enqueue(int item){
            if(count == capacity){
                return false;
            }
            items[count] = item;
            count++;
            return true;
        }

        public int dequeue(){
            if(count == 0){
                return -1;
            }
            int temp = items[0];
            for(int i = 0; i < count - 1; i++){
                items[i] = items[i+1];
            }
            count--;
            return temp;
        }

        public void printAll(){
            for(int i = 0; i < count; i++){
                System.out.print(items[i] + " ");
            }
            System.out.println();
        }


    }
   ```

=== "java"

    ``` java
    public class DynamicArrayQueue {
        // 数组：items，数组大小：n
        private String[] items;
        private int n = 0;
        // head表示队头下标，tail表示队尾下标
        private int head = 0;
        private int tail = 0;

        // 申请一个大小为capacity的数组
        public DynamicArrayQueue(int capacity) {
            items = new String[capacity];
            n = capacity;
        }

        // 入队操作，将item放入队尾
        public boolean enqueue(String item) {
            // tail == n表示队列末尾没有空间了
            if (tail == n) {
                // tail ==n && head==0，表示整个队列都占满了
                if (head == 0)
                    return false;
                // 数据搬移
                for (int i = head; i < tail; ++i) {
                    items[i - head] = items[i];
                }
                // 搬移完之后重新更新head和tail
                tail -= head;
                head = 0;
            }

            items[tail] = item;
            tail++;
            return true;
        }

        // 出队
        public String dequeue() {
            // 如果head == tail 表示队列为空
            if (head == tail)
                return null;
            String ret = items[head];
            ++head;
            return ret;
        }

        public void printAll() {
            for (int i = head; i < tail; ++i) {
                System.out.print(items[i] + " ");
            }
            System.out.println();
        }

    }
    ```   

=== "python"

    ``` python
    class ArrayQueue:
        def __init__(self, capacity):
            self._items = []
            self._capacity = capacity
            self._head = 0
            self._tail = 0
        def enqueue(self, item:str) -> bool:
            if self._tail == self._capacity:
                if self._head == 0:
                    return False
                for i in range(self._head, self._tail):
                    self._items[i - self._head] = self._items[i]
                self._tail -= self._head
                self._head = 0
            self._items.insert(self._tail, item)
            self._tail += 1
            return True
        def dequeue(self) -> str:
            if self._head == self._tail:
                return None
            ret = self._items[self._head]
            self._head += 1
            return ret

        def __repr__(self) -> str:
            return " ".join(item for item in self._items[self._head:self._tail]) 
                        
    ```

 === "JavaScript"

    ``` javascript  
    class ArrayQueue{
        constructor(){
            this.items = [];
        }
        enqueue(item){
            this.items.push(item);
        }
        dequeue(){
            return this.items.shift();
        }
        front(){
            return this.items[0];
        }
        isEmpty(){
            return this.items.length === 0;
        }
        size(){
            return this.items.length;
        }
        clear(){
            this.items = [];
        }
        print(){
            console.log(this.items.toString());
        }
    }

    ```    

## 用链表实现一个链式队列
=== "java"

    ``` java
    public class QueueBasedOnLinkedList {

        private static class Node {
            private int data;
            private Node next;

            public Node(int data, Node next) {
                this.data = data;
                this.next = next;
            }
        }

        private Node head = null;
        private Node tail = null;

        public void enqueue(int value) {
            if (tail == null) {
                Node newNode = new Node(value, null);
                head = newNode;
                tail = newNode;
            } else {
                tail.next = new Node(value, null);
                tail = tail.next;
            }
        }

        public int dequeue() {
            if (head == null) {
                return -1;
            }
            int value = head.data;
            head = head.next;
            if (head == null) {
                tail = null;
            }
            return value;
        }
    }
    ```

=== "python"

    ``` python
    class Node:
        def __init__(self, data):
            self.data = data
            self.next = None
    class LinkedQueue:
        def __init__(self) -> None:
            self._head = None
            self._tail = None

        def enqueue(self, value:str) -> bool:
            node = Node(value)
            if self._tail is None:
                self._head = node
            else:
                self._tail.next = node
            self._tail = node
            return True

        def dequeue(self) -> str:
            if self._head is Node:
                return None
            ret = self._head.data
            self._head = self._head.next
            return ret     

        def __repr__(self) -> str:
            if self._head is None:
                return "Empty queue"
            s = ""
            node = self._head
            while node is not None:
                s += str(node.data) + " "
                node = node.next
            return s 
    ```   
 === "JavaScript"

    ``` javascript  
    class Node {
        constructor(data){
            this.data = data;
            this.next = null;
        }
    }

    class LinkedQueue{
        constructor(){
            this.head = null;
            this.tail = null;
        }
        enqueue(item){
            if(this.tail == null){
                this.tail = new Node(item);
                this.head = this.tail;
            }else{
                this.tail.next = new Node(item);
                this.tail = this.tail.next;
            }
        }
        dequeue(){
            if(this.head !=null){
                const value = this.head.data
                this.head = this.head.next
                return value
            } else {
                return -1
            }
        }
        front(){
            if(this.head !=null){
                return this.head.data
            } else {
                return -1
            }
        }
        isEmpty(){
            return this.head == null;
        }
        size(){
            let p = this.head
            let count = 0
            while(p){
                count++
                p = p.next
            }
            return count
        }
        clear(){
            this.head = null
            this.tail = null
        }
        print(){
            let p = this.head
            while(p){
                console.log(p.data)
                p = p.next
            }
        }
    }
    ```     

## 实现一个循环队列
=== "java"

    ``` java
    public class CircularQueue {
        private int[] items;
        private int n = 0;
        private int head = 0;
        private int tail = 0;

        public CircularQueue(int capacity) {
            items = new int[capacity];
            n = capacity;
        }

        public boolean enqueue(int item){
            if((tail+1) % n == head) return false;
            items[tail] = item;
            tail = (tail+1)%n;
            return true;
        }

        public int dequeue(){
            // 如果head == tail 表示队列为空
            if (head == tail) return -1;
            int ret = items[head];
            head = (head +1)%n;
            return ret;
        }

        public void printAll(){
            if (head == tail) return;
            for (int i = head; i != tail; i = (i+1)%n) {
                System.out.print(items[i] + " ");
            }
            System.out.println();
        }

    }
    ```

=== "python"

    ``` python
    class CircularQueue:
        def __init__(self, capacity):
            self._items = []
            self._capacity = capacity + 1
            self._head = 0
            self._tail = 0

        def enqueue(self,item:str) -> bool:
            if (self._tail + 1) % self._capacity == self._head:
                return False
            self._items.insert(self._tail, item)
            self._tail = (self._tail +1) % self._capacity
            return True

        def dequeue(self) -> str:
            if self._head == self._tail:
                return None
            ret = self._items[self._head]
            self._head = (self._head + 1) % self._capacity
            return ret  

        def __repr__(self) -> str:
            if self._head < self._tail:
                return " ".join(item for item in self._items[self._head:self._tail])
            else:
                return " ".join(item for item in self._items[self._head:] + self._items[:self._tail]) 
                
         
    ```

 === "JavaScript"

    ``` javascript  
    class CicularQueue {
        constructor(size) {
            this.queue = new Array(size);
            this.head = 0;
            this.tail = 0;
            this.size = size;
        }

        enqueue(item) {
            if (this.isFull()) {
            return false;
            }
            this.queue[this.tail] = item;
            this.tail = (this.tail + 1) % this.size;
            return true;
        }

        dequeue() {
            if (this.isEmpty()) {
            return null;
            }
            const item = this.queue[this.head];
            this.head = (this.head + 1) % this.size;
            return item;
        }

        isFull() {
            return (this.tail + 1) % this.size === this.head;
        }

        isEmpty() {
            return this.head === this.tail;
        }
    }
    ```

## [设计实现双端队列]https://leetcode.cn/problems/design-circular-deque/)   

=== "python"

    ``` python 
    class CircularDeque:
        
            def __init__(self, k: int):
                self.size = k
                self.deque = []
        
            def insertFront(self, value: int) -> bool:
                if len(self.deque) == self.size:
                    return False
                self.deque.insert(0, value)
                return True
        
            def insertLast(self, value: int) -> bool:
                if len(self.deque) == self.size:
                    return False
                self.deque.append(value)
                return True
        
            def deleteFront(self) -> bool:
                if len(self.deque) == 0:
                    return False
                self.deque.pop(0)
                return True
        
            def deleteLast(self) -> bool:
                if len(self.deque) == 0:
                    return False
                self.deque.pop()
                return True
        
            def getFront(self) -> int:
                if len(self.deque) == 0:
                    return -1
                return self.deque[0]
        
            def getRear(self) -> int:
                if len(self.deque) == 0:
                    return -1
                return self.deque[-1]
        
            def isEmpty(self) -> bool:
                return len(self.deque) == 0
        
            def isFull(self) -> bool:
                return len(self.deque) == self.size   
    ```