## 编程实现斐波那契数列求值 f(n)=f(n-1)+f(n-2)

=== "java"

    ``` java
    //斐波那契数列求值 1,1,2,3,5,8,13...
    public int fib(int n){
        if(n ==1 || n==2)return 1;
        return fib(n-1)+fib(n-2);
    }
    
    ```

=== "python"

    ``` python
    """
    斐波那契数列求值 1,1,2,3,5,8,13...
    """
    def fib(n):
        if n == 1 or n == 2:
            return 1
        return fib(n-1) + fib(n-2)
                        
    ```

## 编程实现求阶乘 n!

=== "java"

    ``` java
    //求阶乘 n!
    public int factorial(int n){
        if(n==1)return 1;
        return n* factorial(n-1);
    }
    ```

=== "python"

    ``` python
    """
    求阶乘 n!
    """
    def factorial(n):
        if n == 1:
            return 1
        return n * factorial(n-1)
                        
    ```

## 编程实现一组数据集合的全排列

=== "java"

    ``` java
    //全排列
    public void permute(char[] chars, int start, int end){
        if(start == end){
            System.out.println(chars);
        }else{
            for(int i=start;i<=end;i++){
                swap(chars, start,i);
                permute(chars,start+1,end);
                swap(chars, start,i);
            }
        }
    }

    public void swap(char[] chars, int i, int j){
        char temp = chars[i];
        chars[i] = chars[j];
        chars[j] = temp;
    }
    ```

=== "python"

    ``` python
    """
    对于给定的集合A{a1,a2,...,an},其中的n个元素互不相同，如何输出这n个元素的所有排列
    """
    def permute(A, start, end):
        if start == end:
            print(A)
        else:
            for i in range(start, end):
                # 交换A[start]和A[i]
                A[start], A[i] = A[i], A[start]
                permute(A, start+1, end)
                A[start], A[i] = A[i], A[start]         
    ```    

## [爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

=== "java"

    ``` java
    //爬楼梯
    public int climbStairs(int n){
        if(n==1)return 1;
        if(n==2)return 2;
        return climbStairs(n-1)+climbStairs(n-2);
    }
    ```

=== "python"

    ``` python
    """
    爬楼梯
    """
    def climbStairs(n):
        if n == 1:
            return 1
        if n == 2:
            return 2
        return climbStairs(n-1) + climbStairs(n-2)
    ```        