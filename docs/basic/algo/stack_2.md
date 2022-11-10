## 编程模拟实现一个浏览器的前进、后退功能

=== "java"

    ``` java
    public class SampleBrowser {
        private String currentPage;
        private LinkedListBasedStack backStack;
        private LinkedListBasedStack forwardStack;

        public static void main(String[] args) {
            SampleBrowser browser = new SampleBrowser();
            browser.open("http://www.baidu.com");
            browser.open("http://news.baidu.com/");
            browser.open("http://news.baidu.com/ent");
            browser.goBack();
            browser.goBack();
            browser.goForward();
            browser.open("http://www.qq.com");
            browser.goForward();
            browser.goBack();
            browser.goForward();
            browser.goBack();
            browser.goBack();
            browser.goBack();
            browser.goBack();
            browser.checkCurrentPage();
        }

        public SampleBrowser() {
            this.backStack = new LinkedListBasedStack();
            this.forwardStack = new LinkedListBasedStack();
        }

        public void open(String url) {
            if (this.currentPage != null) {
                this.backStack.push(this.currentPage);
                this.forwardStack.clear();
            }
            showUrl(url, "Open");
        }

        public boolean canGoBack() {
            return this.backStack.size() > 0;
        }

        public boolean canGoForward() {
            return this.forwardStack.size() > 0;
        }

        public String goBack() {
            if (this.canGoBack()) {
                this.forwardStack.push(this.currentPage);
                String backUrl = this.backStack.pop();
                showUrl(backUrl, "Back");
                return backUrl;
            }

            System.out.println("* Cannot go back, no pages behind.");
            return null;
        }


        public String goForward() {
            if (this.canGoForward()) {
                this.backStack.push(this.currentPage);
                String forwardUrl = this.forwardStack.pop();
                showUrl(forwardUrl, "Foward");
                return forwardUrl;
            }

            System.out.println("** Cannot go forward, no pages ahead.");
            return null;
        }


        public void showUrl(String url, String prefix) {
            this.currentPage = url;
            System.out.println(prefix + " page == " + url);
        }

        public void checkCurrentPage() {
            System.out.println("Current page is: " + this.currentPage);
        }





        public static class LinkedListBasedStack {

            private int size;
            private Node top;

            static Node createNode(String data, Node next) {
                return new Node(data, next);
            }

            public void clear() {
                this.top = null;
                this.size = 0;
            }

            public void push(String data) {
                Node node = createNode(data, this.top);
                this.top = node;
                this.size++;
            }

            public String pop() {
                Node popNode = this.top;
                if (popNode == null) {
                    System.out.println("Stack is empty.");
                    return null;
                }
                this.top = popNode.next;
                if (this.size > 0) {
                    this.size--;
                }
                return popNode.data;
            }

            public String getTopData() {
                if (this.top == null) {
                    return null;
                }
                return this.top.data;
            }

            public int size() {
                return this.size;
            }

            public void print() {
                System.out.println("Print stack:");
                Node currentNode = this.top;
                while (currentNode != null) {
                    String data = currentNode.getData();
                    System.out.print(data + "\t");
                    currentNode = currentNode.next;
                }
                System.out.println();
            }

            public static class Node {

                private String data;
                private Node next;

                public Node(String data) {
                    this(data, null);
                }

                public Node(String data, Node next) {
                    this.data = data;
                    this.next = next;
                }

                public void setData(String data) {
                    this.data = data;
                }

                public String getData() {
                    return this.data;
                }

                public void setNext(Node next) {
                    this.next = next;
                }

                public Node getNext() {
                    return this.next;
                }
            }

        }

    }

    
    ```

=== "python"

    ``` python
    from linked_stack import LinkedStack
    class Browser:
        def __init__(self):
            self.back_stack = LinkedStack()
            self.forward_stack = LinkedStack()
        def can_forward(self):
            return not self.forward_stack.is_empty()

        def can_back(self):
            return not self.back_stack.is_empty()
        def open(self, url):
            self.back_stack.push(url)

        def back(self):
            if self.can_back():
                self.forward_stack.push(self.back_stack.pop())
                return self.back_stack.peek()
            else:
                return None    
        def forward(self):
            if self.can_forward():
                self.back_stack.push(self.forward_stack.pop())
                return self.back_stack.peek()
            else:
                return None
        def __repr__(self) -> str:
            return f"Browser: {self.back_stack} {self.forward_stack}" 

    if __name__ == "__main__":
        browser = Browser()
        browser.open("google.com")
        browser.open("facebook.com")
        browser.open("youtube.com")
        print(browser)
        print(browser.back())
        print(browser.back())
        print(browser.back())
        print(browser.back())
        print(browser.forward())
        print(browser.forward())
        print(browser.forward())
        print(browser.forward())
        print(browser.forward())
        print(browser)  
                     
    ```

## [给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效](https://leetcode.cn/problems/valid-parentheses/)

!!! note ""

    有效字符串需满足：

    * 左括号必须用相同类型的右括号闭合。

    * 左括号必须以正确的顺序闭合。

    * 每个右括号都有一个对应的相同类型的左括号。

=== "python"

    ``` python
    def isValid(s):
        stack = []
        for c in s:
            if c == '(' or c == '[' or c == '{':
                stack.append(c)
            else:
                if len(stack) == 0:
                    return False
                top = stack.pop()
                if c == ')' and top != '(':
                    return False
                if c == ']' and top != '[':
                    return False
                if c == '}' and top != '{':
                    return False
        return len(stack) == 0
    ```    