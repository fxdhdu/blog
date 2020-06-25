先刷题 20

晚上思考学习



|                                                              |                                                              |                                |                                                |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------ | ---------------------------------------------- |
|                                                              |                                                              |                                |                                                |
| [剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/) | 1）先序遍历树A。找到值相同的节点时，再先序遍历树B。只有当B完全遍历完时，才能确认是否B为A的子树，而不是遍历到某个叶子节点时就返回true。可以搞个全局变量记录结果。 | 重点：思路和实现都错了好几次。 | 树：遍历函数不带返回值，用全局遍历整合遍历结果 |
| [剑指 Offer 28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/) | 递归判断左右子树是否互为镜像。                               |                                | 树：遍历函数带返回值                           |
| [剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/) | 1)使用一个Set。空间复杂度O(n)，时间复杂度O(n)<br>2)先计算两个链表各自的长度，长的链表的指针先走长度的差值，然后两个指针一起走。空间复杂度O(1)，时间复杂度O(n) |                                | 链表                                           |
| [剑指 Offer 53 - I. 在排序数组中查找数字 I](https://leetcode-cn.com/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/) | 1)一次遍历。空间复杂度O(1)，时间复杂度O(n)<br/>2）二分查找查找第一个和最后一个等于给定值的元素。空间复杂度O(1)，时间复杂度O(logn)<br/> |                                | 数组                                           |
| [剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/) | 1)二分查找，2种退出条件。特殊情况：第一个数字不存在，最后一个数字不存在。 |                                | 数组                                           |
| [剑指 Offer 54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/) |                                                              |                                | 树：遍历函数不带返回值，用全局变量记录遍历过程 |



二分查找的时间复杂度怎么算

不要忘记循环步长





- 树的遍历
  - 遍历函数带返回值
  - 遍历函数不带返回值，用全局变量。（遍历过程需要传递到上层递归调用）