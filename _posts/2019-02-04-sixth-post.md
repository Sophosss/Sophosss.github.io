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

14. LinkedList的add方法，`list.add(0, temp)`表示每次插入都插在队首，也就是第一个位置，`list.add(temp)`表示正常插在队尾。

15. 一千万条数据取前20个大的数？（top k问题）
    - 使用最小堆，对20个数据建堆，依次遍历剩下的所有数，若比堆顶元素大，则替换并调整堆。
    - 使用分治法，快速排序中的partition思想。

16. `Arrays.sort(people, (o1,o2) -> o1[0] == o2[0] ? o1[1] - o2[1] : o2[0] - o1[0]);`

    该语句可以让数组按第一位降序，第二位升序排列。

17. a & (-a) 可以获得a最低的非0位，求解[260. 只出现一次的数字 III](https://leetcode-cn.com/problems/single-number-iii/)的关键。

18. n & (n-1)可以消除二进制的一个1，可用于判断2的方幂。

19. 使用位运算符注意括号！运算符有优先级。例如：`(array[j] & 1) == 0 && (array[j+1] & 1) != 0`

20. [172. 阶乘后的零](https://leetcode-cn.com/problems/factorial-trailing-zeroes/submissions/) 计算含有多少个5就可以了。

21. [231. 2的幂](https://leetcode-cn.com/problems/power-of-two/submissions/) 判断`n & n-1 == 0`就可以了。

20. 栈实现队列：A吃B，A吃，A吐B；队列实现栈：A吃，A吃B，互换；最后都是用B结构来取数。

21. 进行字符串按点(.)切分时，必须要注意转义！`String t[] = s.split("\\.");`

22. 海量日志数据，提取出某日访问百度次数最多的那个IP。

- 分而治之/hash映射：针对数据太大，内存受限，只能是：把大文件化成(取模映射)小文件，即16字方针：大而化小，各个击破，缩小规模，逐个解决
- hash统计：当大文件转化了小文件，那么我们便可以采用常规的Hashmap(ip，value)或trie树来进行频率统计。
- 堆/快速排序：统计完了之后，便进行排序(可采取最小堆)，得到次数最多的IP。

23. 链表的操作有半数情况需要使用 dummy 节点，比如删除倒数第N个节点，需要考虑删除头节点的情况。