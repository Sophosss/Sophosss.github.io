---
title: Leetcode快乐源泉
author: Sophosss
layout: post
---
本页面不定期更新Leetcode小心得。

1. 通常来说链表涉及到需要删除某个结点，需要找到该结点的前驱结点，但由于在边界条件下，头节点没有前驱结点，所以定义哑结点`first.next = head;`。
2. 如果一个类被多次实例化且用的都是相同的对象名，这些对象所在的对象内存空间地址是不一样的，所以在递归方法开始处定义的如`List<String> now = new ArrayList();`，表示不同堆内存空间。定义`List<String> pre`接收下层递归返回的数据进一步处理。

3. 与众数（Boyer Moore投票法）

   ```java
   public int majorityElement(int[] nums) {
           int res = 0, count = 0;
           for(int num : nums){
               if(count == 0) res = num;
               count += num == res ? 1 : -1;
           }
           return res;
       }
   ```

4. 只出现一次的数字（异或运算）

   ```java
   public int singleNumber(int[] nums) {
           int temp = 0;
   		for (int num : nums){
   			temp ^= num;
   		}
   		return temp;
       }
   ```

5. 两个单链表相交的起始节点（链表拼接）

   ```java
   public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
   		ListNode l1 = headA, l2 = headB;
   		while (l1 != l2){
   			l1 = l1 == null ? headB : l1.next;
   			l2 = l2 == null ? headA : l2.next;
   		}
   		return l1;
   	}
   ```

   

