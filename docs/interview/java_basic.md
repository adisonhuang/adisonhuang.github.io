# JAVA基础问题
!!! question "==与equals区别"
??? noet "回答"

!!! question "说一下hashcode的作用"
??? noet "回答"

!!! question "请手写equal方法【String类】，讲讲具体的原理？Object类的equla方法是怎样的？"
??? noet "回答"
    * 代码如下所示
    ```java
    public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String) anObject;
        int n = count;
        if (n == anotherString.count) {
            int i = 0;
            while (n-- != 0) {
                if (charAt(i) != anotherString.charAt(i))
                        return false;
                i++;
            }
            return true;
        }
    }
    return false;
    }
    ```
    * Object类的equla方法是怎样的
    ```java
    public boolean equals(Object obj) {
        return (this == obj);
    }
    ```
!!! question "请说下String与StringBuffer区别，StringBuffer底部如何实现"
??? noet "回答"

!!! question "为什么 Java 中的 String 是不可变的（Immutable）？字符串设计和实现考量？String不可变的好处？"
??? noet "回答"


!!! question "如何实现对象克隆？克隆有哪些方式？深克隆和浅克隆有何区别？深克隆和浅克隆分别说的是什么意思？"
??? noet "回答"

!!! question "new Integer(123) 与 Integer.valueOf(123)有何区别，请从底层实现分析两者区别？"
??? noet "回答"

!!! question "instanceof它的作用是什么？在使用过程中注意事项有哪些？它底层原理是如何实现的，说说你的理解？"
??? noet "回答"

!!! question "为什么Java中只有值传递"
??? noet "回答"

!!! question "说一下对final的理解"
??? noet "回答"