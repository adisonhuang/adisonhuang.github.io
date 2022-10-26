## 用数组实现一个顺序栈

=== "java"

    ``` java
    public class ArrayStack {
        private String[] items; // 数组
        private int count; // 栈中元素个数
        private int n; // 栈的大小

        /**
         * 初始化数组，申请一个大小为n的数组空间
         * 
         * @param n
         */
        public ArrayStack(int n) {
            this.items = new String[n];
            this.n = n;
            this.count = 0;
        }

        public boolean push(String item) {
            // 数组空间不够了，直接返回false，入栈失败。
            if (count == n)
                return false;
            // 将item放到下标为count的位置，并且count加一
            items[count] = item;
            ++count;
            return true;
        }

        /**
         * 出栈操作
         * 
         * @return
         */
        public String pop() {
            // 栈为空，则直接返回null
            if (count == 0)
                return null;
            // 返回下标为count-1的数组元素，并且栈中元素个数count减一
            String tmp = items[count - 1];
            --count;
            return tmp;

        }

    }
    
    ```

=== "python"

    ``` python
    class ArrayStack:
        def __init__(self):
            self._data = []

        def __len__(self):
            return len(self._data)

        def is_empty(self):
            return len(self._data) == 0

        def push(self, e):
            self._data.append(e)

        def top(self):
            if self.is_empty():
                raise Empty('Stack is empty')
            return self._data[-1]

        def pop(self):
            if self.is_empty():
                raise Empty('Stack is empty')
            return self._data.pop()
    class Empty(Exception):
        pass    
    ```

## 用链表实现一个链式栈

=== "java"

    ``` java
    public class StackBasedOnLinkedList {
        public static class Node {
            private int data;
            private Node next;

            public Node(int data, Node next) {
                this.data = data;
                this.next = next;
            }

            public int getData() {
                return data;
            }
        }

        private Node top = null;

        public void push(int value) {
            Node newNode = new Node(value, null);
            if (top == null) {
                top = newNode;
            } else {
                newNode.next = top;
                top = newNode;
            }

        }

        public int pop() {
            if (top == null)
                return -1;
            int value = top.data;
            top = top.next;
            return value;
        }

        public void printAll() {
            Node p = top;
            while (p != null) {
                System.out.print(p.data + " ");
                p = p.next;
            }
            System.out.println();
        }

    }
    ```
=== "python"

    ``` python
    class Node:
        def __init__(self, data):
            self.data = data
            self.next = None

    class LinkedStack:
        def __init__(self):
            self.head = None

        def push(self, data):
            node = Node(data)
            node.next = self.head
            self.head = node

        def pop(self):
            if self.head is None:
                return None
            data = self.head.data
            self.head = self.head.next
            return data

        def peek(self):
            if self.head is None:
                return None
            return self.head.data

        def is_empty(self):
            return self.head is None

        def __str__(self):
            if self.head is None:
                return "Empty stack"
            s = ""
            node = self.head
            while node is not None:
                s += str(node.data) + " "
                node = node.next
            return s
    ```    

=== "javascript"

    ``` javascript
    class Node {
        constructor(data, next) {
            this.data = data;
            this.next = next;
        }
    }

    class StackBasedLinkedList{
        constructor() {
            this.top = null;
        }
        push(value){
            const node = new Node(value, null)
            if(this.top === null){
                this.top = node
            } else {
                node.next = this.top
                this.top = node
            }

        }

        pop(){
            if(this.top === null) return -1
            const value = this.top.data
            this.top = this.top.next
            return value
        }

        clear(){
            this.top = null
        }

        printAll(){
            let p = this.top
            while(p){
                console.log(p.data)
                p = p.next
            }
        }
    }
    ```



