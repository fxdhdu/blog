[130. 被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions/)

[463. 岛屿的周长](https://leetcode-cn.com/problems/island-perimeter/)

[500. 键盘行](https://leetcode-cn.com/problems/keyboard-row/) 小写字符转大写字符 ch = (char)(ch - 32)。两个括号都不能少



bfs：广度优先用队列，visited记录已遍历节点，step定义允许步长。节点入队时visited = true; 常用于需要遍历所有特殊节点的场景。

dfs：深度优先用递归。dfs(s,t) = s -> s1 + dis(s1,t)。常用于找终点。

