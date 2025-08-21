---
title: 基本计算器II
date: 2022-06-05 13:06:36
tags:
  - leetcode
  - 算法
  - stack
---

[基本计算器 II](https://leetcode.cn/problems/basic-calculator-ii/)

给你一个字符串表达式 s ，请你实现一个基本计算器来计算并返回它的值。

整数除法仅保留整数部分。

你可以假设给定的表达式总是有效的。所有中间结果将在 [-231, 231 - 1] 的范围内。

注意：不允许使用任何将字符串作为数学表达式计算的内置函数，比如 eval() 。

<!--more-->

```
输入：s = "3+2*2"
输出：7
```

思路: 维护一个操作符栈和操作数栈以及操作符优先级字典. 流程如下: 遇到数字入栈,遇到操作符,如果优先级小于已入栈的操作符, 计算之前大于该操作符的操作,并将计算结果入栈操作数栈. 然后入栈新操作符,重复以上操作. 最终返回操作符栈剩下的结果即可.

代码:

```java
class Solution {
    public int calculate(String s) {
        Stack<Integer> nums = new Stack();
        Stack<Character> ops = new Stack();
        StringBuilder sb = new StringBuilder();
        Map<Character, Integer> pmap = new HashMap();
        pmap.put('+', 1);
        pmap.put('-', 1);
        pmap.put('*', 2);
        pmap.put('/', 2);
        for(int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if(c == ' ') {
                if(!sb.isEmpty()) {
                    nums.push(Integer.parseInt(sb.toString()));
                    sb = new StringBuilder();
                }
                continue;
            }
            if(pmap.containsKey(c)) {
                if(!sb.isEmpty()) {
                    nums.push(Integer.parseInt(sb.toString()));
                    sb = new StringBuilder();
                }
                if(ops.isEmpty()) {
                    ops.push(c);
                }else {
                    while(!ops.isEmpty() && pmap.get(ops.peek()) >= pmap.get(c)) {
                        int v2 = nums.pop();
                        int v1 = nums.pop();
                        nums.push(calc(v1, v2, ops.pop()));   
                    }
                    ops.push(c);
                }
            } else {
                sb.append(c);
            }
        }
        if(!sb.isEmpty()) {
            nums.push(Integer.parseInt(sb.toString()));
        }
        while(!ops.isEmpty()){
            int v2 = nums.pop();
            int v1 = nums.pop();
            nums.push(calc(v1, v2, ops.pop()));
        }
        return nums.pop();
    }

    public int calc(int v1, int v2, char op) {
        switch(op) {
            case '+':
                return v1 + v2;
            case '-':
                return v1 - v2;
            case '*':
                return v1 * v2;
            case '/':
                return v1 / v2;
            default:
                return 0;
        }
    }
}
```

