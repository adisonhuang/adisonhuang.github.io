## 常见链表操作

* 实现单链表反转

* 实现两个有序的链表合并为一个有序链表
  输入：1->2->4, 1->3->4
  输出：1->1->2->3->4->4

* 实现求链表的中间结点

* 链表中环的检测 : 如果链表中有某个节点，可以通过连续跟踪next 指针再次到达，则链表中存在环

* 删除链表倒数第n个结点

* 判断一个字符串是否为回文

  

=== "java"

    ``` java
    public class LinkedListAlgo {
    
      // 单链表反转
      public static Node reverse(Node list) {
        Node curr = list, pre = null;
        while (curr != null) {
          Node next = curr.next;
          curr.next = pre;
          pre = curr;
          curr = next;
        }
        return pre;
      }
    
      // 检测环
      public static boolean checkCircle(Node list) {
        if (list == null) return false;
    
        Node fast = list.next;
        Node slow = list;
    
        while (fast != null && fast.next != null) {
          fast = fast.next.next;
          slow = slow.next;
    
          if (slow == fast) return true;
        }
    
        return false;
      }
    
      // 有序链表合并
       public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
            ListNode soldier = new ListNode(0); //利用哨兵结点简化实现难度
            ListNode p = soldier;
            
            while ( l1 != null && l2 != null ){
                if ( l1.val < l2.val ){
                    p.next = l1;
                    l1 = l1.next;
                }
                else{
                    p.next = l2;
                    l2 = l2.next;
                }
                p = p.next;
            }
            
            if (l1 != null) { p.next = l1; }
            if (l2 != null) { p.next = l2; }
            return soldier.next;   
        }
    
    
      // 删除倒数第K个结点
      public static Node deleteLastKth(Node list, int k) {
        Node fast = list;
        int i = 1;
        while (fast != null && i < k) {
          fast = fast.next;
          ++i;
        }
    
        if (fast == null) return list;
    
        Node slow = list;
        Node prev = null;
        while (fast.next != null) {
          fast = fast.next;
          prev = slow;
          slow = slow.next;
        }
    
        if (prev == null) {
          list = list.next;
        } else {
          prev.next = prev.next.next;
        }
        return list;
      }
    
      // 求中间结点
      public static Node findMiddleNode(Node list) {
        if (list == null) return null;
    
        Node fast = list;
        Node slow = list;
    
        while (fast != null && fast.next != null) {
          fast = fast.next.next;
          slow = slow.next;
        }
    
        return slow;
      }
      // 是否是回文结构
      public static boolean palindrome(Node head){
        if(head == null || head.next == null) return true;
        Node fast = head;
        Node slow = head;
        while(fast.next != null && fast.next.next != null){
          fast = fast.next.next;
          slow = slow.next;
        }
        Node right = slow.next;
        Node pre = null;
        while(right != null){
          Node next = right.next;
          right.next = pre;
          pre = right;
          right = next;
        }
        Node left = head;
        while(pre != null){
          if(pre.data != left.data) return false;
          pre = pre.next;
          left = left.next;
        }
        return true;
    
      }
    
      public static void printAll(Node list) {
        Node p = list;
        while (p != null) {
          System.out.print(p.data + " ");
          p = p.next;
        }
        System.out.println();
      }
    
      public static Node createNode(int value) {
        return new Node(value, null);
      }
    
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
    
    }
    
    ```

=== "python"

    ``` python
    from math import fabs
    from typing import Optional
    
    import sys
    sys.path.append('singly_linked_list')
    from singly_linked_list import SinglyLinkedList,Node
    # class Node:
    #     def __init__(self, data: int, next=None):
    #         self.data = data
    #         self.next = next
    
    
    def reverse_linked_list(head: Node) -> Optional[Node]:
        """Reverse a linked list.
    
        Args:
            head (Node): The head of the linked list.
    
        Returns:
            Node: The head of the reversed linked list.
        """
        if not head or not head.next:
            return head
        prev = None
        current = head
        while current:
            next = current.next
            current.next = prev
            prev = current
            current = next
        return prev
    
    def is_palindrome(head: Node) -> bool:
        """Check if a linked list is a palindrome.
    
        Args:
            head (Node): The head of the linked list.
    
        Returns:
            bool: True if the linked list is a palindrome, False otherwise.
        """
        if not head or not head.next:
            return True
        print(head)    
        slow = head
        fast = head.next
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        # reverse the second half of the linked list
        prev = None
        current = slow.next
        while current:
            next = current.next
            current.next = prev
            prev = current
            current = next
        # compare the first half and the reversed second half
        while prev:
            if prev.data != head.data:
                return False
            prev = prev.next
            head = head.next
        return True
    
    
    def has_cycle(head: Node) -> bool:
        """Check if a linked list has a cycle.
        设置快、慢两种指针，快指针每次跨两步，慢指针每次跨一步，如果快指针没有与慢指针相遇而是顺利到达链表尾部
                说明没有环；否则，存在环
        Args:
            head (Node): The head of the linked list.
    
        Returns:
            bool: True if the linked list has a cycle, False otherwise.
        """
        if not head or not head.next:
            return False
        slow = head
        fast = head.next
        while slow != fast:
            if not fast or not fast.next:
                return False
            slow = slow.next
            fast = fast.next.next
        return True
    
    
    def find_middle_node(head: Node) -> Optional[Node]:
        """Find the middle node of a linked list.
               主体思想:
                设置快、慢两种指针，快指针每次跨两步，慢指针每次跨一步，则当快指针到达链表尾部的时候，慢指针指向链表的中间节点
    
        Args:
            head (Node): The head of the linked list.
    
        Returns:
            Node: The middle node of the linked list.
        """
        if not head or not head.next:
            return head
        slow = head
        fast = head.next
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
    
    
    def merge_sorted_list(l1: Node, l2: Node) -> Optional[Node]:
        """Merge two sorted linked list.
    
        Args:
            l1 (Node): The head of the first linked list.
            l2 (Node): The head of the second linked list.
    
        Returns:
            Node: The head of the merged linked list.
        """
        if not l1:
            return l2
        if not l2:
            return l1
        if l1.data < l2.data:
            l1.next = merge_sorted_list(l1.next, l2)
            return l1
        else:
            l2.next = merge_sorted_list(l1, l2.next)
            return l2
    
     def merge_sorted_list2(l1:Node,l2:Node) -> Optional[Node]:
        """Merge two sorted linked list.
    
        Args:
            l1 (Node): The head of the first linked list.
            l2 (Node): The head of the second linked list.
    
        Returns:
            Node: The head of the merged linked list.
        """
        if not l1:
            return l2
        if not l2:
            return l1
        dummy = Node(0)
        current = dummy
        while l1 and l2:
            if l1.data < l2.data:
                current.next = l1
                l1 = l1.next
            else:
                current.next = l2
                l2 = l2.next
            current = current.next
        if l1:
            current.next = l1
        if l2:
            current.next = l2
        return dummy.next
    
     def remove_nth_from_end(head:Node,n:int) -> Optional[Node]:
        """Remove the nth node from the end of a linked list.
                主体思路：
                设置快、慢两个指针，快指针先行，慢指针不动；当快指针跨了N步以后，快、慢指针同时往链表尾部移动，
                当快指针到达链表尾部的时候，慢指针所指向的就是链表的倒数第N个节点
        Args:
            head (Node): The head of the linked list.
            n (int): The nth node from the end.
    
        Returns:
            Node: The head of the linked list.
        """
        if not head or not head.next:
            return None
        dummy = Node(0)
        dummy.next = head
        first = dummy
        second = dummy
        for i in range(n):
            first = first.next
        while first.next:
            first = first.next
            second = second.next
        second.next = second.next.next
        return dummy.next           
    ```

