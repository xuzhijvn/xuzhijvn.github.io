---
title: "剑指 Offer 09. 用两个栈实现队列"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"剑指offer"
]
series : [
"算法"
]
images : [

]
---

[comment]: <> (# 剑指 Offer 09. 用两个栈实现队列)



```java
class CQueue {
    Stack<Integer> stack1;
    Stack<Integer> stack2;

    public CQueue() {
        stack1 = new Stack();
        stack2 = new Stack();
    }
    
    public void appendTail(int value) {
        stack1.push(value);
    }
    
    public int deleteHead() {
        if(stack2.isEmpty()){
            while(stack1.isEmpty() ==  false){
                stack2.push(stack1.pop());
            }
        }
        if(stack2.isEmpty()){
            return -1;
        }else{
            return stack2.pop();
        }
    }
}

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.appendTail(value);
 * int param_2 = obj.deleteHead();
 */
```

[剑指 Offer 09. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)
