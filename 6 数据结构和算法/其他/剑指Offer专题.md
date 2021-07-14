|                                                              |                                                              |                                |                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------ | -------------------------------------------------- |
| [剑指 Offer 09. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/) | 插入元素直接放入stack1，复杂度为O(1)。取出元素时，直接从stack2取，能取到则直接获取栈顶元素，否则从stack1把元素压入stack2，然后再获取stack2的栈顶元素。 |                                | 栈、队列                                           |
| [剑指 Offer 10- I. 斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/) | 求余规则：（x + y）% n = (x % n + y % n) % n                 |                                | 动态规划                                           |
| [剑指 Offer 15. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/) | 1)n为负数时，导致无限循环的解法。n右移法。<br>2)常规做法，1左移法。<br>3)n&(n-1) 可以将n的最右边的一个1置为0 |                                |                                                    |
| [剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/) | 1）先序遍历树A。找到值相同的节点时，再先序遍历树B。只有当B完全遍历完时，才能确认是否B为A的子树，而不是遍历到某个叶子节点时就返回true。可以搞个全局变量记录结果。 | 重点：思路和实现都错了好几次。 | 树：遍历函数不带返回值，用全局遍历整合遍历结果     |
| [剑指 Offer 28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/) | 递归判断左右子树是否互为镜像。                               |                                | 树：遍历函数带返回值                               |
| [剑指 Offer 34. 二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/) |                                                              |                                | 树：深度优先遍历（递归） + 回溯 + 确认递归终止条件 |
| [剑指 Offer 40. 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/) | 灵活使用快排的partition函数                                  |                                |                                                    |
| [剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/) | 1)使用一个Set。空间复杂度O(n)，时间复杂度O(n)<br>2)先计算两个链表各自的长度，长的链表的指针先走长度的差值，然后两个指针一起走。空间复杂度O(1)，时间复杂度O(n) |                                | 链表                                               |
| [剑指 Offer 53 - I. 在排序数组中查找数字 I](https://leetcode-cn.com/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/) | 1)一次遍历。空间复杂度O(1)，时间复杂度O(n)<br/>2）二分查找查找第一个和最后一个等于给定值的元素。空间复杂度O(1)，时间复杂度O(logn)<br/> |                                | 数组                                               |
| [剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/) | 1)二分查找，2种退出条件。特殊情况：第一个数字不存在，最后一个数字不存在。 |                                | 数组                                               |
| [剑指 Offer 54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/) |                                                              |                                | 树：遍历函数不带返回值，用全局变量记录遍历过程     |
| [剑指 Offer 59 - II. 队列的最大值](https://leetcode-cn.com/problems/dui-lie-de-zui-da-zhi-lcof/) | 使用两个双端队列实现。<br>其中一个为普通先进先出队列，另一个为单调递减双端队列。（队列中只保留对结果有影响的值） |                                |                                                    |
| [剑指 Offer 68 - I. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/) | 最近公共祖先在左子树，在右子树，或则为当前节点。             |                                |                                                    |



二分查找的时间复杂度怎么算

不要忘记循环步长





- 树的遍历
  - 遍历函数带返回值
  - 遍历函数不带返回值，用全局变量。（遍历过程需要传递到上层递归调用）



双栈实现队列

<img src="./asset/image-20200704075221371.png" alt="image-20200704075221371" style="zoom:67%;" />