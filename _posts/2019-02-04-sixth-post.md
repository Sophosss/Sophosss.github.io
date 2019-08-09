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


6. `str.substring()`和`Arrays.sort()`都是左闭右开的区间。

7. 自己的回溯法模板，百试不爽（不行再动态规划呗…）

   ```java
   	List<List<Integer>> list = new ArrayList<>();
   	public List<List<Integer>> combinationSum(int[] candidates, int target) {
   		List<Integer> list1 = new ArrayList<> ();
   		Arrays.sort(candidates);
   		back(list1, 0, candidates, target);
   		return list;
   	}
   	public void back(List<Integer> list1, int start, int []arr, int target) {
   		if (target == 0){
   			list.add(new ArrayList<>(list1));
   		}else if (target > 0){
   			for (int i = start; i < arr.length; i++) {
   				list1.add(arr[i]);
   				back(list1, i, arr, target - arr [i]);
   				list1.remove(list1.size() - 1);
   			}
   		}
   	}
   ```


8. 二分查找模板：

   无脑写 `while left < right:`。

   中位数写成`int mid = (left + right) >>> 1;`，左边界排除中位数`left = mid + 1`，那么右边界不排除中位数`right = mid`，判断是否满足条件，满足返回左中位数`return left;`。

   中位数写成`int mid = (left + right + 1) >>> 1;`，左边界排除中位数`right = mid - 1;`，那么右边界不排除中位数`left = mid`，判断是否满足条件，满足返回右中位数`return right;`。

   `return left`或者`return right`是一样的，因为循环终止条件必然是left = right，关键在于你是取的左中位数还是右中位数，你所舍弃的区间是不是完全不包含需要的元素值。

9. 对于一个非负数n，它的平方根不会大于（n/2+1）。

10. 交换两个变量的值，例如 `a` 和 `b`，不使用第三个变量。如果`swap(int [] nums, int i, int j)`时，`i`和`j`相等时该方法失效。

    `a = a ^ b`
    `b = a ^ b`
    `a = a ^ b`

11. java.lang.IllegalArgumentException：

    JDK1.7的实现compare方法必须有一个返回0的情况，即必须判断两对象属性是否相等，相等必须返回0，不然就报这错，编译虽然不报错，但是运行会报错。

12. Integer 类 compareTo 方法内部直接调用 compare 方法

    ```java
    public int compareTo(Integer anotherInteger) {
             return compare(this.value, anotherInteger.value);
         }
    
    public static int compare(int x, int y) {
             return (x < y) ? -1 : ((x == y) ? 0 : 1);
         }
    ```

13. java.lang.ClassCastException：

    toArray有两个重载的方法：`list.toArray()`和`list.toArray(T[] a)`，第一个重载方法，是将list直接转为Object[] 数组，第二种方法是将list转化为你所需要类型的数组。

    java中的强制类型转换只是针对单个对象的，想要偷懒将整个数组转换成另外一种类型的数组是不行的，这和数组初始化时需要一个个来也是类似的。

    比如`return list.toArray(new int[0][]);`