---
layout: post
title:  "LeetSilinceCode"
date:   2020-07-20 13:20:06 +0800--
categories: [数据结构]
tags: [LeetCode,数据结构 ]  

---

# PLAN

> [labuladong 的算法小抄 技巧模版总结](https://labuladong.gitbook.io/algo/)   [repo](https://github.com/labuladong/fucking-algorithm)    [公众号完整文章](https://mp.weixin.qq.com/s/AWsL7G89RtaHyHjRPNJENA)
>
> now: 回溯算法解题套路框架
>
> [刷题目录](https://github.com/CyC2018/CS-Notes/blob/master/notes/Leetcode%20%E9%A2%98%E8%A7%A3%20-%20%E7%9B%AE%E5%BD%95.md)
>
> [如何科学的刷 LeetCode ](https://zhuanlan.zhihu.com/p/96883783)



# 算法思想

## 双指针 

| 题目                                                         | 算法思想        | 正确率 |
| ------------------------------------------------------------ | --------------- | ------ |
| [\#11 盛最多水的容器](http://www.silince.cn/2020/07/20/LeetSilinceCode/#11-盛最多水的容器) | 双指针          | 0%     |
| [\#167 有序数组的 Two Sum](http://www.silince.cn/2020/07/20/LeetSilinceCode/#167-%E6%9C%89%E5%BA%8F%E6%95%B0%E7%BB%84%E7%9A%84-two-sum) | 双指针/二分查找 | 50%    |
| [\#633 两数平方和](http://www.silince.cn/2020/07/20/LeetSilinceCode/#633-%E4%B8%A4%E6%95%B0%E5%B9%B3%E6%96%B9%E5%92%8C) | 双指针          | 50%    |
| [\#345 反转字符串中的元音字符](http://www.silince.cn/2020/07/20/LeetSilinceCode/#345-%E5%8F%8D%E8%BD%AC%E5%AD%97%E7%AC%A6%E4%B8%B2%E4%B8%AD%E7%9A%84%E5%85%83%E9%9F%B3%E5%AD%97%E7%AC%A6) | 双指针          | 50%    |
| [\#680 回文字符串](http://www.silince.cn/2020/07/20/LeetSilinceCode/#680-%E5%9B%9E%E6%96%87%E5%AD%97%E7%AC%A6%E4%B8%B2) | 双指针          | 50%    |
| [\#88 合并两个有序数组](http://www.silince.cn/2020/07/20/LeetSilinceCode/#88-%E5%90%88%E5%B9%B6%E4%B8%A4%E4%B8%AA%E6%9C%89%E5%BA%8F%E6%95%B0%E7%BB%84) | 双指针          | 0%     |
| [\#141 判断链表是否存在环](http://www.silince.cn/2020/07/20/LeetSilinceCode/#141-判断链表是否存在环) | 双指针          | 0%     |
| [\#524 最长子序列](http://www.silince.cn/2020/07/20/LeetSilinceCode/#542-最长子序列) | 双指针          | 0%     |



## 排序算法

| 题目                                                         | 算法思想 | 正确率 |
| ------------------------------------------------------------ | -------- | ------ |
| [\#215 数组中的第K个最大元素](http://www.silince.cn/2020/07/20/LeetSilinceCode/#215-数组中的第k个最大元素) | 快速排序 | 0%     |
| [\#347 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/) | 桶排序   | 0%     |
| [\#451 根据字符出现频率排序](https://leetcode-cn.com/problems/sort-characters-by-frequency/) | 桶排序   |        |
| [\#75 颜色分类 ]([http://www.silince.cn/2020/07/20/LeetSilinceCode/#75-%E9%A2%9C%E8%89%B2%E5%88%86%E7%B1%BB](http://www.silince.cn/2020/07/20/LeetSilinceCode/#75-颜色分类)) |          |        |
|                                                              |          |        |
|                                                              |          |        |
|                                                              |          |        |



![image-20200720125956366](/assets/imgs/image-20200720125956366-9188642.png)

[十大排序算法](https://mp.weixin.qq.com/s/Qf416rfT4pwURpW3aDHuCg)

[排序算法的复杂度、实现和稳定性](https://www.jianshu.com/p/916b15eae350)

## 贪心思想

## 二分查找

## 分治

## 搜索

## 动态规划

> [动态规划解题套路框架](https://labuladong.gitbook.io/algo/di-ling-zhang-bi-du-xi-lie/dong-tai-gui-hua-xiang-jie-jin-jie)

| 题目                                                         | 算法思想 | 正确率 |
| ------------------------------------------------------------ | -------- | ------ |
| [\#509 斐波那契数](http://www.silince.cn/2020/07/20/LeetSilinceCode/#509-%E6%96%90%E6%B3%A2%E9%82%A3%E5%A5%91%E6%95%B0) | 动态规划 | 0%     |
| [\#322 零钱兑换](http://www.silince.cn/2020/07/20/LeetSilinceCode/#322-零钱兑换) | 动态规划 |        |

**首先，动态规划问题的一般形式就是求最值**。动态规划其实是运筹学的一种最优化方法，只不过在计算机问题上应用比较多，比如说让你求**最长**递增子序列呀，**最小**编辑距离呀等等。

既然是要求最值，核心问题是什么呢？**求解动态规划的核心问题是穷举**。因为要求最值，肯定要把所有可行的答案穷举出来，然后在其中找最值呗。

动态规划这么简单，就是穷举就完事了？我看到的动态规划问题都很难啊！

首先，动态规划的穷举有点特别，因为这类问题**存在「重叠子问题」**，如果暴力穷举的话效率会极其低下，所以需要「备忘录」或者「DP table」来优化穷举过程，避免不必要的计算。

而且，动态规划问题一定会**具备「最优子结构」**，才能通过子问题的最值得到原问题的最值。

另外，虽然动态规划的核心思想就是穷举求最值，但是问题可以千变万化，穷举所有可行解其实并不是一件容易的事，只有列出**正确的「状态转移方程」**才能正确地穷举。

以上提到的重叠子问题、最优子结构、状态转移方程就是动态规划三要素。具体什么意思等会会举例详解，但是在实际的算法问题中，**写出状态转移方程是最困难的**，这也就是为什么很多朋友觉得动态规划问题困难的原因，我来提供我研究出来的一个思维框架，辅助你思考状态转移方程：

**明确 base case -> 明确「状态」-> 明确「选择」 -> 定义 dp 数组/函数的含义**。

按上面的套路走，最后的结果就可以套这个框架：

```java
# 初始化 base case
dp[0][0][...] = base
# 进行状态转移
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            dp[状态1][状态2][...] = 求最值(选择1，选择2...)
```

第一个斐波那契数列的问题，解释了如何通过「备忘录」或者「dp table」的方法来优化递归树，并且明确了这两种方法本质上是一样的，只是自顶向下和自底向上的不同而已。

第二个凑零钱的问题，展示了如何流程化确定「状态转移方程」，只要通过状态转移方程写出暴力递归解，剩下的也就是优化递归树，消除重叠子问题而已。

**计算机解决问题其实没有任何奇技淫巧，它唯一的解决办法就是穷举**，穷举所有可能性。算法设计无非就是先思考“如何穷举”，然后再追求“如何聪明地穷举”。

列出动态转移方程，就是在解决“如何穷举”的问题。之所以说它难，一是因为很多穷举需要递归实现，二是因为有的问题本身的解空间复杂，不那么容易穷举完整。

备忘录、DP table 就是在追求“如何聪明地穷举”。用空间换时间的思路。



## 回溯算法

> [回溯算法解题套路框架](https://labuladong.gitbook.io/algo/di-ling-zhang-bi-du-xi-lie/hui-su-suan-fa-xiang-jie-xiu-ding-ban)

| 题目                                                         | 算法思想 | 正确率 |
| ------------------------------------------------------------ | -------- | ------ |
| [\#46. 全排列](http://www.silince.cn/2020/07/20/LeetSilinceCode/#46-全排列) | 回溯算法 | 0%     |
| [\#51. N 皇后](http://www.silince.cn/2020/07/20/LeetSilinceCode/#51-n-皇后) | 回溯算法 | 0%     |

**解决一个回溯问题，实际上就是一个决策树的遍历过程**。你只需要思考 3 个问题：

1、路径：也就是已经做出的选择。

2、选择列表：也就是你当前可以做的选择。

3、结束条件：也就是到达决策树底层，无法再做选择的条件。

代码方面，回溯算法的框架：

- **写** **`backtrack`** **函数时，需要维护走过的「路径」和当前可以做的「选择列表」，当触发「结束条件」时，将「路径」记入结果集**。
- **其核心就是 for 循环里面的递归，在递归调用之前「做选择」，在递归调用之后「撤销选择」**，特别简单。

```
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return


    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```

其实想想看，回溯算法和动态规划是不是有点像呢？我们在动态规划系列文章中多次强调，动态规划的三个需要明确的点就是「状态」「选择」和「base case」，是不是就对应着走过的「路径」，当前的「选择列表」和「结束条件」？

某种程度上说，动态规划的暴力求解阶段就是回溯算法。只是有的问题具有重叠子问题性质，可以用 dp table 或者备忘录优化，将递归树大幅剪枝，这就变成了动态规划。而今天的两个问题，都没有重叠子问题，也就是回溯算法问题了，复杂度非常高是不可避免的。



## BFS算法

> [BFS算法解题套路框架](https://labuladong.gitbook.io/algo/di-ling-zhang-bi-du-xi-lie/bfs-kuang-jia)

| 题目                        | 算法思想 | 正确率 |
| --------------------------- | -------- | ------ |
| [\#111. 二叉树的最小深度]() | BFS算法  | 0%     |
| [\#752. 打开转盘锁]()       | BFS算法  | 0%     |

首先，你要说 labuladong 没写过 BFS 框架，这话没错，今天写个框架你背住就完事儿了。但要是说没写过 DFS 框架，那你还真是说错了，**其实 DFS 算法就是回溯算法**。

BFS 的核心思想应该不难理解的，就是把一些问题抽象成图，从一个点开始，向四周开始扩散。一般来说，我们写 BFS 算法都是用「队列」这种数据结构，每次将一个节点周围的所有节点加入队列。

BFS 相对 DFS 的最主要的区别是：**BFS 找到的路径一定是最短的，但代价就是空间复杂度比 DFS 大很多**，至于为什么，我们后面介绍了框架就很容易看出来了。

***算法框架：***

要说框架的话，我们先举例一下 BFS 出现的常见场景好吧，**问题的本质就是让你在一幅「图」中找到从起点** **`start`** **到终点** **`target`** **的最近距离，这个例子听起来很枯燥，但是 BFS 算法问题其实都是在干这个事儿**，把枯燥的本质搞清楚了，再去欣赏各种问题的包装才能胸有成竹嘛。

这个广义的描述可以有各种变体，比如走迷宫，有的格子是围墙不能走，从起点到终点的最短距离是多少？如果这个迷宫带「传送门」可以瞬间传送呢？

再比如说两个单词，要求你通过某些替换，把其中一个变成另一个，每次只能替换一个字符，最少要替换几次？

再比如说连连看游戏，两个方块消除的条件不仅仅是图案相同，还得保证两个方块之间的最短连线不能多于两个拐点。你玩连连看，点击两个坐标，游戏是如何判断它俩的最短连线有几个拐点的？

再比如……

净整些花里胡哨的，这些问题都没啥奇技淫巧，本质上就是一幅「图」，让你从一个起点，走到终点，问最短路径。这就是 BFS 的本质，框架搞清楚了直接默写就好：

**队列 `q` 就不说了，BFS 的核心数据结构；`cur.adj()` 泛指 `cur` 相邻的节点，比如说二维数组中，`cur` 上下左右四面的位置就是相邻节点；`visited` 的主要作用是防止走回头路，大部分时候都是必须的，但是像一般的二叉树结构，没有子节点到父节点的指针，不会走回头路就不需要 `visited`。**

```java
// 计算从起点 start 到终点 target 的最近距离
int BFS(Node start, Node target) {
    LinkedList<TreeNode> queue = new LinkedList<>(); // 核心数据结构
    Set<Node> visited; // 避免走回头路

    queue.offer(start); // offer是queue的插入方法，将起点加入队列
    visited.add(start);
    int step = 0; // 记录扩散的步数

    while (!queue.isEmpty()) {
        int size = queue.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < size; i++) {
            Node cur = queue.poll(); // poll() 检索并删除此列表的头部（第一个元素）
            /* 划重点：这里判断是否到达终点 */
            if (cur is target)
                return step;
            /* 将 cur 的相邻节点加入队列 */
            for (Node x : cur.adj())
                if (x not in visited) {
                    q.offer(x);
                    visited.add(x);
                }
        }
        /* 划重点：更新步数在这里 */
        step++;
    }
}
```






## 数学







# 数据结构相关

正确率 > 80% 可移除 👩🏻‍💻

## 数组

> https://leetcode-cn.com/tag/array/

| 题目                                                         | 算法思想                  | 正确率 |
| ------------------------------------------------------------ | ------------------------- | ------ |
| [\#26 删除排序数组中的重复项](http://www.silince.cn/2020/07/20/LeetSilinceCode/#26-%E5%88%A0%E9%99%A4%E6%8E%92%E5%BA%8F%E6%95%B0%E7%BB%84%E4%B8%AD%E7%9A%84%E9%87%8D%E5%A4%8D%E9%A1%B9) | 双指针                    | 0%     |
| [\#88 合并两个有序数组](http://www.silince.cn/2020/07/20/LeetSilinceCode/#88-合并两个有序数组) | 双指针                    | 0%     |
| [\#169 多数元素](http://www.silince.cn/2020/07/20/LeetSilinceCode/#169-多数元素) | 哈希表/排序/随机化/投票法 | 0%     |
| [\#674 最长连续递增序列](http://www.silince.cn/2020/07/20/LeetSilinceCode/#674-最长连续递增序列) | 动态规划                  | 0%     |
| [\#1051 高度检查器](http://www.silince.cn/2020/07/20/LeetSilinceCode/#1051-高度检查器) | 桶排序                    | 50%    |
| [\#1160 拼写单词](http://www.silince.cn/2020/07/20/LeetSilinceCode/#1160-拼写单词) | counter方法/HashMap       | 0%     |





## 链表

## 二叉树

[目录](https://github.com/CyC2018/CS-Notes/blob/master/notes/Leetcode%20%E9%A2%98%E8%A7%A3%20-%20%E6%A0%91.md#1-%E6%A0%91%E7%9A%84%E9%AB%98%E5%BA%A6)

| 题目                                                         | 算法思想      | 正确率 |
| ------------------------------------------------------------ | ------------- | ------ |
| [\#104 树的高度](http://www.silince.cn/2020/07/20/LeetSilinceCode/#104-二叉树的最大深度) | 递归/广度优先 | 50%    |
| [\#110 平衡二叉树](http://www.silince.cn/2020/07/20/LeetSilinceCode/#110-平衡二叉树) | 递归          | 50%    |
| [\#543 两节点的最长路径](http://www.silince.cn/2020/07/20/LeetSilinceCode/#543-二叉树的直径) | 递归          | 0%     |
| [\#226 翻转树 ⭐️](http://www.silince.cn/2020/07/20/LeetSilinceCode/#226-%E7%BF%BB%E8%BD%AC%E4%BA%8C%E5%8F%89%E6%A0%91) | 前序遍历/递归 | 100%   |
| [\#116 填充每个节点的下一个右侧节点指针 ⭐️](http://www.silince.cn/2020/07/20/LeetSilinceCode/#116-填充每个节点的下一个右侧节点指针) | 前序遍历/递归 | 0%     |
| [\#114. 二叉树展开为链表 ⭐️](http://www.silince.cn/2020/07/20/LeetSilinceCode/#114-二叉树展开为链表) | 后序遍历/递归 | 0%     |
| [\#617 归并两棵树](http://www.silince.cn/2020/07/20/LeetSilinceCode/#617-%E5%90%88%E5%B9%B6%E4%BA%8C%E5%8F%89%E6%A0%91) | 递归          | 0%     |
| [\#654. 最大二叉树 ⭐️](http://www.silince.cn/2020/07/20/LeetSilinceCode/#654-最大二叉树) | 递归          | 50%    |
| [\#105. 从前序与中序遍历序列构造二叉树 ⭐️](http://www.silince.cn/2020/07/20/LeetSilinceCode/#105-从前序与中序遍历序列构造二叉树) | 递归          | 0%     |
| [\#106. 从中序与后序遍历序列构造二叉树 ⭐️](http://www.silince.cn/2020/07/20/LeetSilinceCode/#106-从中序与后序遍历序列构造二叉树) | 递归          | 100%   |
| [\#652 寻找重复的子树 ⭐️](http://www.silince.cn/2020/07/20/LeetSilinceCode/#652-寻找重复的子树) | 递归          | 0%     |
| [\#112 判断路径和是否等于一个数](http://www.silince.cn/2020/07/20/LeetSilinceCode/#112-%E8%B7%AF%E5%BE%84%E6%80%BB%E5%92%8C) | 递归          |        |
| [\#437 统计路径和等于一个数的路径数量](http://www.silince.cn/2020/07/20/LeetSilinceCode/#437-%E8%B7%AF%E5%BE%84%E6%80%BB%E5%92%8C-iii) | 递归          |        |
| [\#572 子树](http://www.silince.cn/2020/07/20/LeetSilinceCode/#572-%E5%8F%A6%E4%B8%80%E4%B8%AA%E6%A0%91%E7%9A%84%E5%AD%90%E6%A0%91) | 递归          |        |
| [\#101 树的对称](http://www.silince.cn/2020/07/20/LeetSilinceCode/#101-%E5%AF%B9%E7%A7%B0%E4%BA%8C%E5%8F%89%E6%A0%91) | 递归          |        |
| [\#111 最小路径](http://www.silince.cn/2020/07/20/LeetSilinceCode/#111-%E4%BA%8C%E5%8F%89%E6%A0%91%E7%9A%84%E6%9C%80%E5%B0%8F%E6%B7%B1%E5%BA%A6) | 递归          |        |
| [\#404 统计左叶子结点的和](http://www.silince.cn/2020/07/20/LeetSilinceCode/#404-%E5%B7%A6%E5%8F%B6%E5%AD%90%E4%B9%8B%E5%92%8C) | 递归          |        |
| [\#687 相同节点值的最大路径长度](http://www.silince.cn/2020/07/20/LeetSilinceCode/#687-%E6%9C%80%E9%95%BF%E5%90%8C%E5%80%BC%E8%B7%AF%E5%BE%84) | 递归          |        |
| [\#337 间隔遍历](http://www.silince.cn/2020/07/20/LeetSilinceCode/#337-%E6%89%93%E5%AE%B6%E5%8A%AB%E8%88%8D-iii) | 递归          |        |
| [\#671 找出二叉树中第二小的节点](http://www.silince.cn/2020/07/20/LeetSilinceCode/#671-%E4%BA%8C%E5%8F%89%E6%A0%91%E4%B8%AD%E7%AC%AC%E4%BA%8C%E5%B0%8F%E7%9A%84%E8%8A%82%E7%82%B9) | 递归          |        |
| [\#637 一棵树每层节点的平均数](http://www.silince.cn/2020/07/20/LeetSilinceCode/#637-%E4%BA%8C%E5%8F%89%E6%A0%91%E7%9A%84%E5%B1%82%E5%B9%B3%E5%9D%87%E5%80%BC) | 层次遍历      |        |
| [\#513 得到左下角的节点](http://www.silince.cn/2020/07/20/LeetSilinceCode/#513-%E6%89%BE%E6%A0%91%E5%B7%A6%E4%B8%8B%E8%A7%92%E7%9A%84%E5%80%BC) | 层次遍历      |        |
| [\#144 非递归实现二叉树的前序遍历](http://www.silince.cn/2020/07/20/LeetSilinceCode/#144-%E4%BA%8C%E5%8F%89%E6%A0%91%E7%9A%84%E5%89%8D%E5%BA%8F%E9%81%8D%E5%8E%86) | 前序遍历      |        |
| [\#145 非递归实现二叉树的后序遍历](http://www.silince.cn/2020/07/20/LeetSilinceCode/#145-%E4%BA%8C%E5%8F%89%E6%A0%91%E7%9A%84%E5%90%8E%E5%BA%8F%E9%81%8D%E5%8E%86) | 后序遍历      |        |
| [\#94 非递归实现二叉树的中序遍历](http://www.silince.cn/2020/07/20/LeetSilinceCode/#94-%E4%BA%8C%E5%8F%89%E6%A0%91%E7%9A%84%E4%B8%AD%E5%BA%8F%E9%81%8D%E5%8E%86) | 中序遍历      |        |
| [\#699 修剪二叉查找树](http://www.silince.cn/2020/07/20/LeetSilinceCode/#669-%E4%BF%AE%E5%89%AA%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91) | BST           |        |
| [\#230 寻找二叉查找树的第 k 个元素](http://www.silince.cn/2020/07/20/LeetSilinceCode/#230-%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91%E4%B8%AD%E7%AC%ACk%E5%B0%8F%E7%9A%84%E5%85%83%E7%B4%A0) | BST           |        |
| [\#538 把二叉查找树每个节点的值都加上比它大的节点的值](http://www.silince.cn/2020/07/20/LeetSilinceCode/#538-%E6%8A%8A%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91%E8%BD%AC%E6%8D%A2%E4%B8%BA%E7%B4%AF%E5%8A%A0%E6%A0%91) | BST           |        |
| [\#235 二叉查找树的最近公共祖先](http://www.silince.cn/2020/07/20/LeetSilinceCode/#235-%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91%E7%9A%84%E6%9C%80%E8%BF%91%E5%85%AC%E5%85%B1%E7%A5%96%E5%85%88) | BST           |        |
| [\#236 二叉树的最近公共祖先](http://www.silince.cn/2020/07/20/LeetSilinceCode/#236-%E4%BA%8C%E5%8F%89%E6%A0%91%E7%9A%84%E6%9C%80%E8%BF%91%E5%85%AC%E5%85%B1%E7%A5%96%E5%85%88) | BST           |        |
| [\#108 从有序数组中构造二叉查找树](http://www.silince.cn/2020/07/20/LeetSilinceCode/#108-%E5%B0%86%E6%9C%89%E5%BA%8F%E6%95%B0%E7%BB%84%E8%BD%AC%E6%8D%A2%E4%B8%BA%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91) | BST           |        |
| [\#109 根据有序链表构造平衡的二叉查找树](http://www.silince.cn/2020/07/20/LeetSilinceCode/#109-%E6%9C%89%E5%BA%8F%E9%93%BE%E8%A1%A8%E8%BD%AC%E6%8D%A2%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91) | BST           |        |
| [\#653 在二叉查找树中寻找两个节点，使它们的和为一个给定值](http://www.silince.cn/2020/07/20/LeetSilinceCode/#653-%E4%B8%A4%E6%95%B0%E4%B9%8B%E5%92%8C-iv---%E8%BE%93%E5%85%A5-bst) | BST           |        |
| [\#530 在二叉查找树中查找两个节点之差的最小绝对值](http://www.silince.cn/2020/07/20/LeetSilinceCode/#530-%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91%E7%9A%84%E6%9C%80%E5%B0%8F%E7%BB%9D%E5%AF%B9%E5%B7%AE) | BST           |        |
| [\#501 寻找二叉查找树中出现次数最多的值](http://www.silince.cn/2020/07/20/LeetSilinceCode/#501-%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91%E4%B8%AD%E7%9A%84%E4%BC%97%E6%95%B0) | BST           |        |
| [\# 208 实现-trie-前缀树](http://www.silince.cn/2020/07/20/LeetSilinceCode/#208-%E5%AE%9E%E7%8E%B0-trie-%E5%89%8D%E7%BC%80%E6%A0%91) | Trie          |        |
| [\# 677 键值映射](http://www.silince.cn/2020/07/20/LeetSilinceCode/#677-%E9%94%AE%E5%80%BC%E6%98%A0%E5%B0%84) | Trie          |        |





## 栈和队列

## 哈希表

## 字符串

## 图



# 题

## [\#11. 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

- medium
- 2019.09.13：😭 
- 2019.09.15：😭  写成height[i++]<height[j--]了，呕

题目：

```xml
给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0)。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。
说明：你不能倾斜容器，且 n 的值至少为 2。

示例：
输入：[1,8,6,2,5,4,8,3,7]
输出：49
```

![image-20200913205928929](/assets/imgs/image-20200913205928929.png)

分析：双指针法

```xml
算法流程： 设置双指针 i,j 分别位于容器壁两端，根据规则移动指针（后续说明），并且更新面积最大值 res，直到 i == j 时返回 res。

“若向内移动短板，水槽的短板 min(h[i], h[j]) 可能变大，因此水槽面积 S(i, j)可能增大。若向内移动长板，水槽的短板 min(h[i], h[j]) 不变或变小，下个水槽的面积一定小于当前水槽面积。“其实可以加一句，无论是移动短板或者长板，我们都只关注移动后的新短板会不会变长，而每次移动的木板都只有三种情况，比原短板短，比原短板长，与原短板相等；如向内移动长板，对于新的木板：1.比原短板短，则新短板更短。2.与原短板相等或者比原短板长，则新短板不变。所以，向内移动长板，一定不能使新短板变长。
```

代码：

```java
class Solution {
    public int maxArea(int[] height) {
      	// 设置双指针 i,j 分别位于容器壁两端
        int i = 0, j = height.length - 1, res = 0;
      	// 每次向内移动短板，并且更新面积最大值 res，直到 i == j 时返回 res。
        while(i < j){
            res = height[i] < height[j] ? 
                Math.max(res, (j - i) * height[i++]): 
                Math.max(res, (j - i) * height[j--]); 
        }
        return res;
    }
}
```

---







## [\#46. 全排列](https://leetcode-cn.com/problems/permutations/)

- 中等
- 2020.11.17：😭  

题目：

```xml
给定一个 没有重复 数字的序列，返回其所有可能的全排列。

示例:

输入: [1,2,3]
输出:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]
```

分析：

我们在高中的时候就做过排列组合的数学题，我们也知道 `n` 个不重复的数，全排列共有 n! 个。

PS：**为了简单清晰起见，我们这次讨论的全排列问题不包含重复的数字**。

那么我们当时是怎么穷举全排列的呢？比方说给三个数 `[1,2,3]`，你肯定不会无规律地乱穷举，一般是这样：

先固定第一位为 1，然后第二位可以是 2，那么第三位只能是 3；然后可以把第二位变成 3，第三位就只能是 2 了；然后就只能变化第一位，变成 2，然后再穷举后两位……

其实这就是回溯算法，我们高中无师自通就会用，或者有的同学直接画出如下这棵回溯树：

![image-20201117203224104](/assets/imgs/image-20201117203224104.png)

只要从根遍历这棵树，记录路径上的数字，其实就是所有的全排列。**我们不妨把这棵树称为回溯算法的「决策树」**。

**为啥说这是决策树呢，因为你在每个节点上其实都在做决策**。比如说你站在下图的红色节点上：

![image-20201117203253335](/assets/imgs/image-20201117203253335.png)

你现在就在做决策，可以选择 1 那条树枝，也可以选择 3 那条树枝。为啥只能在 1 和 3 之中选择呢？因为 2 这个树枝在你身后，这个选择你之前做过了，而全排列是不允许重复使用数字的。

***现在可以解答开头的几个名词：`[2]` 就是「路径」，记录你已经做过的选择；`[1,3]` 就是「选择列表」，表示你当前可以做出的选择；「结束条件」就是遍历到树的底层，在这里就是选择列表为空的时候。***

如果明白了这几个名词，**可以把「路径」和「选择」列表作为决策树上每个节点的属性**，比如下图列出了几个节点的属性：

![image-20201117203349079](/assets/imgs/image-20201117203349079.png)

**我们定义的** **`backtrack`** **函数其实就像一个指针，在这棵树上游走，同时要正确维护每个节点的属性，每当走到树的底层，其「路径」就是一个全排列**。

再进一步，如何遍历一棵树？这个应该不难吧。回忆一下之前「学习数据结构的框架思维」写过，各种搜索问题其实都是树的遍历问题，而多叉树的遍历框架就是这样：

```java
void traverse(TreeNode root) {
    for (TreeNode child : root.childern)
        // 前序遍历需要的操作
        traverse(child);
        // 后序遍历需要的操作
}
```

而所谓的前序遍历和后序遍历，他们只是两个很有用的时间点，我给你画张图你就明白了：

![image-20201117203430533](/assets/imgs/image-20201117203430533.png)

**前序遍历的代码在进入某一个节点之前的那个时间点执行，后序遍历代码在离开某个节点之后的那个时间点执行**。

回想我们刚才说的，「路径」和「选择」是每个节点的属性，函数在树上游走要正确维护节点的属性，那么就要在这两个特殊时间点搞点动作：

![image-20201117203447638](/assets/imgs/image-20201117203447638.png)

现在，你是否理解了回溯算法的这段核心框架？

```
for 选择 in 选择列表:
    # 做选择
    将该选择从选择列表移除
    路径.add(选择)
    backtrack(路径, 选择列表)
    # 撤销选择
    路径.remove(选择)
    将该选择再加入选择列表
```

**我们只要在递归之前做出选择，在递归之后撤销刚才的选择**，就能正确得到每个节点的选择列表和路径。

下面，直接看全排列代码：

```java
class Solution {
  
  private List<List<Integer>> res = new LinkedList<>();

  /* 主函数，输入一组不重复的数字，返回它们的全排列 */
  public List<List<Integer>> permute(int[] nums) {
    // 记录「路径」
    LinkedList<Integer> track = new LinkedList<>();
    backtrack(nums, track);
    return res;
  }

  // 路径：记录在 track 中
  // 选择列表：nums 中不存在于 track 的那些元素
  // 结束条件：nums 中的元素全都在 track 中出现
  public void backtrack(int[] nums, LinkedList<Integer> track) {
    // 触发结束条件
    if (track.size() == nums.length) {
      res.add(new LinkedList(track));
      return;
    }

    for (int i = 0; i < nums.length; i++) {
      // 判断何时才能前进，排除不合法的选择
      if (track.contains(nums[i])){
        // 做选择
        track.add(nums[i]);
        // 进入下一层决策树
        backtrack(nums, track);
        // 取消选择
        track.removeLast();
      }
    }
  }
}
```

我们这里稍微做了些变通，没有显式记录「选择列表」，而是通过 `nums` 和 `track` 推导出当前的选择列表：

![image-20201117203531227](/assets/imgs/image-20201117203531227.png)

至此，我们就通过全排列问题详解了回溯算法的底层原理。当然，这个算法解决全排列不是很高效，因为对链表使用 `contains` 方法需要 O(N) 的时间复杂度。有更好的方法通过交换元素达到目的，但是难理解一些，这里就不写了，有兴趣可以自行搜索一下。

但是必须说明的是，不管怎么优化，都符合回溯框架，而且时间复杂度都不可能低于 O(N!)，因为穷举整棵决策树是无法避免的。**这也是回溯算法的一个特点，不像动态规划存在重叠子问题可以优化，回溯算法就是纯暴力穷举，复杂度一般都很高**。



---



## [\#51. N 皇后](https://leetcode-cn.com/problems/n-queens/)

- 困难
- 2020.11.17：😭  

题目：

```xml
n 皇后问题研究的是如何将 n 个皇后放置在 n×n 的棋盘上，并且使皇后彼此之间不能相互攻击。
给定一个整数 n，返回所有不同的 n 皇后问题的解决方案。
每一种解法包含一个明确的 n 皇后问题的棋子放置方案，该方案中 'Q' 和 '.' 分别代表了皇后和空位。

示例：
输入：4
输出：[
 [".Q..",  // 解法 1
  "...Q",
  "Q...",
  "..Q."],

 ["..Q.",  // 解法 2
  "Q...",
  "...Q",
  ".Q.."]
]
解释: 4 皇后问题存在两个不同的解法。
 
提示：
皇后彼此不能相互攻击，也就是说：任何两个皇后都不能处于同一条横行、纵行或斜线上。
```

![image-20201117202557542](/assets/imgs/image-20201117202557542.png)



分析：

这个问题本质上跟全排列问题差不多，决策树的每一层表示棋盘上的每一行；每个节点可以做出的选择是，在该行的任意一列放置一个皇后。

直接套用框架:

```java
private List<List<String>> res = new LinkedList<>();

/* 输入棋盘边长 n，返回所有合法的放置 */
public List<List<String>> solveNQueens(int n) {
    // '.' 表示空，'Q' 表示皇后，初始化空棋盘。
    char[][] board = new char[n][n];
    //初始化数组
    for (int i = 0; i < n; i++)
      for (int j = 0; j < n; j++)
        board[i][j] = '.';

    backtrack(board, 0);
    return res;
}

// 路径：board 中小于 row 的那些行都已经成功放置了皇后
// 选择列表：第 row 行的所有列都是放置皇后的选择
// 结束条件：row 超过 board 的最后一行
public void backtrack(char[][] board, int row) {
    // 触发结束条件,最后一行都走完了，说明找到了一组，把它加入到集合res中 
    if (row == board.length) {
        res.add(construct(board));
        return;
    }

    // 遍历选择列表
    for (int col = 0; col < board.length; col++) {
        // 判断何时前进,排除不合法选择
        if (isValid(board, row, col)) {
          // 做选择
          board[row][col] = 'Q';
          // backtrack(路径，选择列表)
          backtrack(board, row + 1);
          // 撤销选择
          board[row][col] = '.';
      }
    }
}

//把数组转为list
private List<String> construct(char[][] board) {
    List<String> path = new ArrayList<>();
    for (int i = 0; i < board.length; i++) {
      path.add(new String(board[i]));
    }
    return path;
}
```

这部分主要代码，其实跟全排列问题差不多，`isValid` 函数的实现也很简单：

```java
/* 是否可以在 board[row][col] 放置皇后？ */
private boolean isValid(char[][] board, int row, int col) {
  //判断当前列有没有皇后,因为他是一行一行往下走的，
  //我们只需要检查走过的行数即可，通俗一点就是判断当前
  //坐标位置的上面有没有皇后
  for (int i = 0; i < row; i++) {
    if (board[i][col] == 'Q') {
      return false;
    }
  }
  //判断当前坐标的右上角有没有皇后
  for (int i = row - 1, j = col + 1; i >= 0 && j < board.length; i--, j++) {
    if (board[i][j] == 'Q') {
      return false;
    }
  }
  //判断当前坐标的左上角有没有皇后
  for (int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
    if (board[i][j] == 'Q') {
      return false;
    }
  }
  return true;
}
```

函数 `backtrack` 依然像个在决策树上游走的指针，通过 `row` 和 `col` 就可以表示函数遍历到的位置，通过 `isValid` 函数可以将不符合条件的情况剪枝：

![image-20201117214315687](/assets/imgs/image-20201117214315687.png)

如果直接给你这么一大段解法代码，可能是懵逼的。但是现在明白了回溯算法的框架套路，还有啥难理解的呢？无非是改改做选择的方式，排除不合法选择的方式而已，只要框架存于心，你面对的只剩下小问题了。

当 `N = 8` 时，就是八皇后问题，数学大佬高斯穷尽一生都没有数清楚八皇后问题到底有几种可能的放置方法，但是我们的算法只需要一秒就可以算出来所有可能的结果。

不过真的不怪高斯。这个问题的复杂度确实非常高，看看我们的决策树，虽然有 `isValid` 函数剪枝，但是最坏时间复杂度仍然是 O(N^(N+1))，而且无法优化。如果 `N = 10` 的时候，计算就已经很耗时了。

**有的时候，我们并不想得到所有合法的答案，只想要一个答案，怎么办呢**？比如解数独的算法，找所有解法复杂度太高，只要找到一种解法就可以。

其实特别简单，只要稍微修改一下回溯算法的代码即可：

```java
// 函数找到一个答案后就返回 true
bool backtrack(vector<string>& board, int row) {
    // 触发结束条件
    if (row == board.size()) {
        res.push_back(board);
        return true;
    }
    ...
    for (int col = 0; col < n; col++) {
        ...
        board[row][col] = 'Q';

        if (backtrack(board, row + 1))
            return true;

        board[row][col] = '.';
    }

    return false;
}
```

这样修改后，只要找到一个答案，for 循环的后续递归穷举都会被阻断。











代码：

```java

```

---





## [#26. 删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/) 

- Easy
- 2019.08.30：😭 

题目：

```xml
给定一个排序数组，你需要在 原地 删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。
不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。
```

分析：双指针法

```xml
数组完成排序后，我们可以放置两个指针 i 和 j，其中 i 是慢指针，而 j 是快指针。慢指针 i 用于记录最后一次出现的数字，快指针 j 用于遍历数组的每一个元素，并把未出现过的数赋值给第 i+1 个元素。

只要 nums[i] = nums[j]，我们就增加 j 以跳过重复项。当我们遇到 nums[j] 不等于 nums[i]，跳过重复项的运行已经结束，因此我们必须把它（nums[j]）的值复制到 nums[i + 1]。然后递增 i，接着我们将再次重复相同的过程，直到 j 到达数组的末尾为止。

复杂度分析
时间复杂度：O(n)O(n)，假设数组的长度是 n，那么 i 和 j 分别最多遍历 n 步。
空间复杂度：O(1)O(1)。
```

代码：

```java
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int i = 0;
    for (int j = 1; j < nums.length; j++) {
        if (nums[j] != nums[i]) {
            nums[++i]=nums[j];
        }
    }
    return i + 1;
}
```

---



## [\#75. 颜色分类](https://leetcode-cn.com/problems/sort-colors/)

- easy
- 2019.08.28：😭  

题目：

```xml

```

分析：

```xml

```

代码：

```java

```





## [#88. 合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/)  

- Easy
- 2019.08.30：😭 

题目：

```xml
给你两个有序整数数组 nums1 和 nums2，请你将 nums2 合并到 nums1 中，使 nums1 成为一个有序数组。

说明:
初始化 nums1 和 nums2 的元素数量分别为 m 和 n 。
你可以假设 nums1 有足够的空间（空间大小大于或等于 m + n）来保存 nums2 中的元素。 
```

分析：双指针

```xml
方法一 : 合并后排序
直觉
最朴素的解法就是将两个数组合并之后再排序。该算法只需要一行(Java是2行)，时间复杂度较差，为O((n + m)log(n + m)。这是由于这种方法没有利用两个数组本身已经有序这一点。
复杂度分析:
时间复杂度 : O((n+m)log(n+m))
空间复杂度 : O(1)

方法二 : 双指针 / 从前往后
一般而言，对于有序数组可以通过 双指针法达到O(n + m)的时间复杂度。
最直接的算法实现是将指针p1 置为 nums1的开头， p2为 nums2的开头，在每一步将最小值放入输出数组中。
由于 nums1 是用于输出的数组，需要将nums1中的前m个元素放在其他地方，也就需要 O(m) 的空间复杂度。
复杂度分析:
时间复杂度 : O(n+m)
空间复杂度 : O(m)

⭐️方法三 : 双指针 / 从后往前
方法二已经取得了最优的时间复杂度O(n + m)，但需要使用额外空间。这是由于在从头改变nums1的值时，需要把nums1中的元素存放在其他位置。
如果我们从结尾开始改写 nums1 的值又会如何呢？这里没有信息，因此不需要额外空间。
这里的指针 p 用于追踪添加元素的位置。
复杂度分析:
时间复杂度 : O(n + m)
空间复杂度 : O(1)
```

代码：

```java
// 方法一
class Solution {
  public void merge(int[] nums1, int m, int[] nums2, int n) {
    System.arraycopy(nums2, 0, nums1, m, n);
    Arrays.sort(nums1);
  }
}

// 方法二
class Solution {
  public void merge(int[] nums1, int m, int[] nums2, int n) {
    // 由于 nums1 是用于输出的数组，需要将nums1中的前m个元素放在其他地方.
    int [] nums1_copy = new int[m];
    System.arraycopy(nums1, 0, nums1_copy, 0, m);

    //  设置 nums1_copy 和 nums2 的指针
    int p1 = 0;
    int p2 = 0;

    // 设置 nums1 的指针
    int p = 0;

    // 比较 nums1_copy 和 nums2 中的元素，并把小的元素放入 nums1
    while ((p1 < m) && (p2 < n))
      nums1[p++] = (nums1_copy[p1] < nums2[p2]) ? nums1_copy[p1++] : nums2[p2++];
    // 如果还有元素未被放入 nums1
    if (p1 < m)
      System.arraycopy(nums1_copy, p1, nums1, p1 + p2, m + n - p1 - p2);
    if (p2 < n)
      System.arraycopy(nums2, p2, nums1, p1 + p2, m + n - p1 - p2);
  }
}

// ⭐️方法三
class Solution {
  public void merge(int[] nums1, int m, int[] nums2, int n) {
    // 设置 nums1 和 nums2 的指针
    int p1 = m - 1;
    int p2 = n - 1;
    // 设置 合并后nums1 的指针
    int p = m + n - 1;

    // 比较 nums1 和 nums2 中的元素，并把大的元素放入 nums1 [从尾部放入]
    while ((p1 >= 0) && (p2 >= 0))
      nums1[p--] = (nums1[p1] < nums2[p2]) ? nums2[p2--] : nums1[p1--];

    // 如果 nums2 还有剩余，全部放入 nums1 的顶端 (如果剩余的是nums1则p2=-1,等于无变化)
    System.arraycopy(nums2, 0, nums1, 0, p2 + 1);
  }
}
```

---



## [\#94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

- Medium
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#101. 对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

- easy
- 2020.08.28：😭  
- 2020.10.10: 😎(递归)

题目：

```xml
给定一个二叉树，找出其最大深度。
二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。
说明: 叶子节点是指没有子节点的节点。


示例：
给定二叉树 [3,9,20,null,null,15,7]，
    3
   / \
  9  20
    /  \
   15   7
返回它的最大深度 3 。
```

分析：

***方法一***：递归 ⭐️

如果我们知道了左子树和右子树的最大深度 l 和 r，那么该二叉树的最大深度即为.  $max(l,r)+1$

而左子树和右子树的最大深度又可以以同样的方式进行计算。***因此我们在计算当前二叉树的最大深度时，可以先递归计算出其左子树和右子树的最大深度，然后在 O(1) 时间内计算出当前二叉树的最大深度。***递归在访问到空节点时退出。

- 时间复杂度：O(n),其中 *n* 为二叉树节点的个数。每个节点在递归中只被遍历一次。
- 空间复杂度：O(height),其中*height* 表示二叉树的高度。递归函数需要栈空间，而栈空间取决于递归的深度，因此空间复杂度等价于二叉树的高度。



***方法二***：广度优先搜索

我们也可以用「广度优先搜索」的方法来解决这道题目，但我们需要对其进行一些修改，此时我们广度优先搜索的队列里存放的是「当前层的所有节点」。每次拓展下一层的时候，不同于广度优先搜索的每次只从队列里拿出一个节点，我们需要将队列里的所有节点都拿出来进行拓展，这样能保证每次拓展完的时候队列里存放的是当前层的所有节点，即我们是一层一层地进行拓展，最后我们用一个变量 ans 来维护拓展的次数，该二叉树的最大深度即为 ans。

- 时间复杂度：O(n)
- 空间复杂度：此方法空间的消耗取决于队列存储的元素数量，其在最坏情况下会达到O(n)



代码：

```java
// ⭐️方法一 递归/深度优先
class Solution {
    public int maxDepth(TreeNode root) {
        // 因此我们在计算当前二叉树的最大深度时，
			  // 可以先递归计算出其左子树和右子树的最大深度，然后在 O(1) 时间内计算出当前二叉树的最大深度
        if (root == null) return 0;
        int leftHeight = maxDepth(root.left);
        int rightHeight = maxDepth(root.right);
        return Math.max(leftHeight, rightHeight) + 1;
       
    }
}

// 方法二 广度优先搜索
class Solution {
    public int maxDepth(TreeNode root) {
        if (root==null ) return 0;
      	// 队列里存放的是「当前层的所有节点」
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
      /**
       * add是list的
       * offer是queue的
       * api里说：
       * add：Inserts the specified element at the specified position in this list
       * 将指定的元素插入到list中指定的的位置。
       * offer：
       * 如果在不违反容量限制的情况下，尽可能快的将指定的元素插入到queue中去
       * */
        queue.offer(root);
        int ans = 0;
        while (!queue.isEmpty()) {
            // 每层节点的数量
            int size = queue.size();
            // 每次拓展下一层的时候，不同于广度优先搜索的每次只从队列里拿出一个节点，我们需要将队列里的所有节点都拿出来进行拓展             
            for (int i = 0; i < size; i++) {
              // poll() 检索并删除此列表的头部（第一个元素）。
              TreeNode node = queue.poll();
              if (node.left!=null) queue.offer(node.left);
              if (node.right!=null) queue.offer(node.right);
            }
            ans++;
        }
        return ans;
    }
}
```

---



## [\#105. 从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

- Medium
- 2020.10.01：😭  
- 2020.11.01：😎

题目：

```xml
根据一棵树的前序遍历与中序遍历构造二叉树。

注意:
你可以假设树中没有重复的元素。

例如，给出

前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
返回如下的二叉树：

    3
   / \
  9  20
    /  \
   15   7
```

分析：

**我们肯定要想办法确定根节点的值，把根节点做出来，然后递归构造左右子树即可**。

我们先来回顾一下，前序遍历和中序遍历的结果有什么特点？

```java
void traverse(TreeNode root) {
    // 前序遍历
    preorder.add(root.val);
    traverse(root.left);
    traverse(root.right);
}

void traverse(TreeNode root) {
    traverse(root.left);
    // 中序遍历
    inorder.add(root.val);
    traverse(root.right);
}
```

这样的遍历顺序差异，导致了`preorder`和`inorder`数组中的元素分布有如下特点：

![image-20201021122425297](/assets/imgs/image-20201021122425297.png)

***找到根节点是很简单的，前序遍历的第一个值`preorder[0]`就是根节点的值，关键在于如何通过根节点的值，将`preorder`和`postorder`数组划分成两半，构造根节点的左右子树？***

换句话说，对于以下代码中的`?`部分应该填入什么：

```java
/* 主函数 */
TreeNode buildTree(int[] preorder, int[] inorder) {
    return build(preorder, 0, preorder.length - 1,
                 inorder, 0, inorder.length - 1);
}

/* 
   若前序遍历数组为 preorder[preStart..preEnd]，
   后续遍历数组为 postorder[postStart..postEnd]，
   构造二叉树，返回该二叉树的根节点 
*/
TreeNode build(int[] preorder, int preStart, int preEnd, 
               int[] inorder, int inStart, int inEnd) {
    // root 节点对应的值就是前序遍历数组的第一个元素
    int rootVal = preorder[preStart];
    // rootVal 在中序遍历数组中的索引
    int index = 0;
    for (int i = inStart; i <= inEnd; i++) {
        if (inorder[i] == rootVal) {
            index = i;
            break;
        }
    }

    TreeNode root = new TreeNode(rootVal);
    // 递归构造左右子树
    root.left = build(preorder, ?, ?,
                      inorder, ?, ?);

    root.right = build(preorder, ?, ?,
                       inorder, ?, ?);
    return root;
}
```

对于代码中的`rootVal`和`index`变量，就是下图这种情况：

![image-20201021122515752](/assets/imgs/image-20201021122515752.png)

现在我们来看图做填空题，下面这几个问号处应该填什么：

```java
root.left = build(preorder, ?, ?,
                  inorder, ?, ?);

root.right = build(preorder, ?, ?,
                   inorder, ?, ?);
```

对于左右子树对应的`inorder`数组的起始索引和终止索引比较容易确定：

![image-20201021122535222](/assets/imgs/image-20201021122535222.png)

```java
root.left = build(preorder, ?, ?,
                  inorder, inStart, index - 1);

root.right = build(preorder, ?, ?,
                   inorder, index + 1, inEnd);
```

对于`preorder`数组呢？如何确定左右数组对应的起始索引和终止索引？

这个可以通过左子树的节点数推导出来，假设左子树的节点数为`leftSize`，那么`preorder`数组上的索引情况是这样的：

![image-20201021122555818](/assets/imgs/image-20201021122555818.png)

看着这个图就可以把`preorder`对应的索引写进去了：

```java
int leftSize = index - inStart;

root.left = build(preorder, preStart + 1, preStart + leftSize,
                  inorder, inStart, index - 1);

root.right = build(preorder, preStart + leftSize + 1, preEnd,
                   inorder, index + 1, inEnd);
```

至此，整个算法思路就完成了，我们再补一补 base case 即可写出解法代码：



代码：

```java
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
		return build(preorder,0,preorder.length-1,
				inorder,0,inorder.length-1);
    }

   // 若前序遍历数组为 preorder[preStart..preEnd]，
   // 中序遍历数组为 inorder[inStart..inEnd]，
   // 构造二叉树，返回该二叉树的根节点
    public TreeNode build(int[] preorder,int preStart,int preEnd,int[] inorder,int inStart,int inEnd){
    // 递归出口
		if (preStart>preEnd){
			return null;
		}

    // 先构建根节点 再递归生成左右子树
		// root 节点对应的值就是前序遍历数组的第一个元素
		int rootVal = preorder[preStart];
		// rootVal 再中序数组中的索引
		int index = 0;
		for (int i = inStart; i <= inEnd ; i++) {
			if (inorder[i]==rootVal){
				index=i;
				break;
			}
		}

		TreeNode root = new TreeNode(rootVal);

		// 递归构造左右子树
		int leftSize = index-inStart;
		root.left = build(preorder,preStart+1,preStart+leftSize,inorder,inStart,index-1);
		root.right = build(preorder,preStart+leftSize+1,preEnd,inorder,index+1,inEnd);

		return root;
	}
}
```

---



## [\#106. 从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

- Medium
- 2020.10.01：😎  

题目：

```xml
根据一棵树的中序遍历与后序遍历构造二叉树。

注意:
你可以假设树中没有重复的元素。

例如，给出
中序遍历 inorder = [9,3,15,20,7]
后序遍历 postorder = [9,15,7,20,3]
返回如下的二叉树：
    3
   / \
  9  20
    /  \
   15   7

```

分析：

与105类似，现在`postoder`和`inorder`对应的状态如下：

![image-20201022101616643](/assets/imgs/image-20201022101616643.png)





- 时间复杂度：O()
- 空间复杂度：O()

代码：

```java
class Solution {
    public TreeNode buildTree(int[] inorder, int[] postorder) {
			return build(inorder,0,inorder.length-1,
					postorder,0,postorder.length-1);
    }

    public TreeNode build(int[] inorder,int inStart,int inEnd, int[] postorder,int postStart,int postEnd){
    	// 递归出口
		if (inStart>inEnd) return null;

    	// 找到根节点
		int rootVal = postorder[postEnd]; // 别用postorder.length-1 太浪费时间
		int index = -1;
		for (int i = inStart; i <=inEnd; i++) {
			if (inorder[i]==rootVal) {
				index = i;
				break;
			}
		}
		// 迭代生成左右子树
		TreeNode root = new TreeNode(rootVal);
		int leftSize = index-inStart; // 左子树节点个数

		root.left = build(inorder,inStart,index-1,postorder,postStart,postStart+leftSize-1);
		root.right = build(inorder,index+1,inEnd,postorder,postStart+leftSize,postEnd-1);

		return root;

	}
}
```

---





## [\#108. 将有序数组转换为二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/)

- Medium
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()

代码：

```java

```

---





## [\#109. 有序链表转换二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree/)

- Medium
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()

代码：

```java

```

---





## [\#110. 平衡二叉树](https://leetcode-cn.com/problems/balanced-binary-tree/)

- easy
- 2020.08.28：😭 
- 2020.10.14：😎 

题目：

```xml
给定一个二叉树，判断它是否是高度平衡的二叉树。
本题中，一棵高度平衡二叉树定义为：一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过1。

示例 1:
给定二叉树 [3,9,20,null,null,15,7]

    3
   / \
  9  20
    /  \
   15   7
返回 true 。

示例 2:
给定二叉树 [1,2,2,3,3,null,null,4,4]

       1
      / \
     2   2
    / \
   3   3
  / \
 4   4
```

分析：

***方法一***：从底至顶（提前阻断） ⭐️

> 此方法为本题的最优解法，但“从底至顶”的思路不易第一时间想到。

思路是对二叉树做先序遍历，从底至顶返回子树最大高度，若判定某子树不是平衡树则 “剪枝” ，直接向上返回。

算法流程：

`recur(root)`:

- 递归返回值：
  1. 当节点`root` 左 / 右子树的高度差 <=1 ：则返回以节点`root`为根节点的子树的最大高度，即节点 `root` 的左右子树中最大高度加 1 （` max(left, right) + 1` ）；
  2. 当节点`root` 左 / 右子树的高度差  `≥2 `：则返回 −1 ，代表 此子树不是平衡树 。
- 递归终止条件：
  1. 当越过叶子节点时，返回高度 0 ；
  2. 当左（右）子树高度` left== -1 `时，代表此子树的 左（右）子树 不是平衡树，因此直接返回 −1 ；

`isBalanced(root)` ：

返回值： 若` recur(root) != -1` ，则说明此树平衡，返回 true ； 否则返回 false 

复杂度分析：
时间复杂度 O(N)： N 为树的节点数；最差情况下，需要递归遍历树的所有节点。
空间复杂度 O(N)： 最差情况下（树退化为链表时），系统递归需要使用 O(N) 的栈空间。

***方法二：从顶至底（暴力法）***:

> 此方法容易想到，但会产生大量重复计算，时间复杂度较高。

构造一个获取当前节点最大深度的方法 depth(root) ，通过比较此子树的左右子树的最大高度差abs(depth(root.left) - depth(root.right))，来判断此子树是否是二叉平衡树。若树的所有子树都平衡时，此树才平衡。

**算法流程：**
**isBalanced(root) ：判断树 root 是否平衡**

- **特例处理**： 若树根节点 root 为空，则直接返回 true ；
- **返回值**： 所有子树都需要满足平衡树性质，因此以下三者使用与逻辑 && 连接；
  1. abs(self.depth(root.left) - self.depth(root.right)) <= 1 ：判断 当前子树 是否是平衡树；
  2. self.isBalanced(root.left) ： 先序遍历递归，判断 当前子树的左子树 是否是平衡树；
  3. self.isBalanced(root.right) ： 先序遍历递归，判断 当前子树的右子树 是否是平衡树；

**depth(root) ： 计算树 root 的最大高度**

- **终止条件**： 当 root 为空，即越过叶子节点，则返回高度 0 ；
- **返回值**： 返回左 / 右子树的最大高度加 1 。

**复杂度分析：**
时间复杂度 O(Nlog2N)： 最差情况下， isBalanced(root) 遍历树所有节点，占用 O(N) ；判断每个节点的最大高度 depth(root) 需要遍历 各子树的所有节点 ，子树的节点数的复杂度为 O(log 2N) 
空间复杂度O(N)： 最差情况下（树退化为链表时），系统递归需要使用 O(N) 的栈空间。

代码：

```java
// ⭐️ 方法一
class Solution {
	// 对二叉树做先序遍历，从底至顶返回子树最大高度，若判定某子树不是平衡树则 “剪枝” ，直接向上返回
	public boolean isBalanced(TreeNode root) {
		return recur(root) != -1;
	}

	private int recur(TreeNode root) {
		if (root == null) return 0;
		int left = recur(root.left);
		// 当左（右）子树高度 left== -1 时，代表此子树的 左（右）子树 不是平衡树，因此直接返回 -1 ；
		if(left == -1) return -1;
		int right = recur(root.right);
		if(right == -1) return -1;
		// 当 左/右 子树深度差大于 1 时，返回 -1 ；否则，返回 左/右子树深度最大值 + 1
		// max(left, right) + 1 为当前子树的深度;以此作为返回值，才能判断树是否是"平衡二叉树"，即 abs(left - right) <=1 是否成立
		return Math.abs(left - right) <=1 ? Math.max(left, right) + 1 : -1;
	}
}

// 方法二
class Solution {
    public boolean isBalanced(TreeNode root) {
		if(root==null) return true;
		// 比较此子树的左右子树的最大高度差
		return (Math.abs(depth(root.left)-depth(root.right))<=1)
				&& isBalanced(root.left)
				&& isBalanced(root.right);
    }

    // 获取当前节点最大深度
	  // 终止条件： 当 root 为空，即越过叶子节点，则返回高度 0
    private int depth(TreeNode root){
    	if (root ==null) return 0;
    	return Math.max(depth(root.left),depth(root.right))+1;
	}
}
```



## [\#111. 二叉树的最小深度](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/)

- easy
- 2020.10.01：😭  

题目：

```xml
给定一个二叉树，找出其最小深度。
最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

说明：叶子节点是指没有子节点的节点。
示例：
输入：root = [3,9,20,null,null,15,7]
输出：2
```

![image-20201125124341863](/assets/imgs/image-20201125124341863.png)

分析：

***方法一：***BFS

怎么套到 BFS 的框架里呢？首先明确一下起点 `start` 和终点 `target` 是什么，怎么判断到达了终点？

**显然起点就是** **`root`** **根节点，终点就是最靠近根节点的那个「叶子节点」嘛**，叶子节点就是两个子节点都是 `null` 的节点：

```java
if (cur.left == null && cur.right == null) 
    // 到达叶子节点
```

那么，按照我们上述的框架稍加改造来写解法即可：

```java
class Solution {
  public int minDepth(TreeNode root) {
    if (root == null) return 0;
    LinkedList<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    // root 本身就是一层，depth 初始化为1
    int depth = 1;

    while (!queue.isEmpty()) {
      int size = queue.size();
      // 将当前队列中的所有节点向四周扩散
      for (int i = 0; i < size; i++) {
        TreeNode cur = queue.poll(); // poll() 检索并删除此列表的头部（第一个元素）
        // 判断是否达到终点
        if (cur.left == null && cur.right == null) return depth;
        if (cur.left != null) queue.offer(cur.left);
        if (cur.right != null) queue.offer(cur.right);
      }
      // 增加步数
      depth++;
    }
    return depth;
  }
}
```

二叉树是很简单的数据结构，我想上述代码你应该可以理解的吧，其实其他复杂问题都是这个框架的变形，再探讨复杂问题之前，我们解答两个问题：

**1、为什么 BFS 可以找到最短距离，DFS 不行吗**？

首先，你看 BFS 的逻辑，`depth` 每增加一次，队列中的所有节点都向前迈一步，这保证了第一次到达终点的时候，走的步数是最少的。

DFS 不能找最短路径吗？其实也是可以的，但是时间复杂度相对高很多。你想啊，DFS 实际上是靠递归的堆栈记录走过的路径，你要找到最短路径，肯定得把二叉树中所有树杈都探索完才能对比出最短的路径有多长对不对？而 BFS 借助队列做到一次一步「齐头并进」，是可以在不遍历完整棵树的条件下找到最短距离的。

形象点说，DFS 是线，BFS 是面；DFS 是单打独斗，BFS 是集体行动。这个应该比较容易理解吧。

**2、既然 BFS 那么好，为啥 DFS 还要存在**？

BFS 可以找到最短距离，但是空间复杂度高，而 DFS 的空间复杂度较低。

还是拿刚才我们处理二叉树问题的例子，假设给你的这个二叉树是满二叉树，节点数为 `N`，对于 DFS 算法来说，空间复杂度无非就是递归堆栈，最坏情况下顶多就是树的高度，也就是 `O(logN)`。

但是你想想 BFS 算法，队列中每次都会储存着二叉树一层的节点，这样的话最坏情况下空间复杂度应该是树的最底层节点的数量，也就是 `N/2`，用 Big O 表示的话也就是 `O(N)`。

由此观之，BFS 还是有代价的，一般来说在找最短路径的时候使用 BFS，其他时候还是 DFS 使用得多一些（主要是递归代码好写）。



---





## [\#112. 路径总和](https://leetcode-cn.com/problems/path-sum/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---



## [\#114. 二叉树展开为链表](https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/)

- 中等
- 2020.10.01：😭  
- 2020.10.27：😭   while写成了if
- 2020.11.01：😭  忘记把左子树置空了，只差一点点距离！

题目：

```xml
给定一个二叉树，原地将它展开为一个单链表。

例如，给定二叉树

    1
   / \
  2   5
 / \   \
3   4   6
将其展开为：

1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6
```

分析：

我们尝试给出这个函数的定义：

**给`flatten`函数输入一个节点`root`，那么以`root`为根的二叉树就会被拉平为一条链表**。

我们再梳理一下，如何按题目要求把一棵树拉平成一条链表？很简单，以下流程：

1、将左子树作为右子树

2、将原先的右子树接到当前右子树的末端

![image-20201019091037744](/assets/imgs/image-20201019091037744.png)

上面三步看起来最难的应该是第一步对吧，如何把`root`的左右子树拉平？其实很简单，按照`flatten`函数的定义，对`root`的左右子树递归调用`flatten`函数即可：

⚠️：***另外注意递归框架是后序遍历，因为我们要先拉平左右子树才能进行后续操作。***

代码：

```java
class Solution {
  public void flatten(TreeNode root) {
    // base case
    if (root == null) return;

    flatten(root.left);
    flatten(root.right);

      /**** 后序遍历位置 ****/
      // 1、左右子树已经被拉平成一条链表
      TreeNode left = root.left;
      TreeNode right = root.right;

      // 2、将左子树作为右子树
      root.left = null;
      root.right = left;

      // 3、将原先的右子树接到当前右子树的末端
      while (root.right != null) { // ⚠️ while非常重要，需要循环找到最后当前右子树的末端
        root = root.right;
      }
      root.right = right;
}
}
```

---







## [\#116. 填充每个节点的下一个右侧节点指针](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/)

- 中等
- 2020.10.01：😭  
- 2020.10.27：😭    无法处理跨父节点连接
- 2020.11.01：😭    奇怪的前序遍历

题目：

```xml
给定一个完美二叉树，其所有叶子节点都在同一层，每个父节点都有两个子节点。
填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 NULL。
初始状态下，所有 next 指针都被设置为 NULL。

提示：
你只能使用常量级额外空间。
使用递归解题也符合要求，本题中递归程序占用的栈空间不算做额外的空间复杂度。
```

分析：

题目的意思就是把二叉树的每一层节点都用`next`指针连接起来：

![image-20201018223328790](/assets/imgs/image-20201018223328790.png)

而且题目说了，输入是一棵「完美二叉树」，形象地说整棵二叉树是一个正三角形，除了最右侧的节点`next`指针会指向`null`，其他节点的右侧一定有相邻的节点。

⚠️ **二叉树的问题难点在于，如何把题目的要求细化成每个节点需要做的事情**，但是如果只依赖一个节点的话，肯定是没办法连接「跨父节点」的两个相邻节点的。

那么，我们的做法就是增加函数参数，一个节点做不到，我们就给他安排两个节点，「将每一层二叉树节点连接起来」可以细化成「将每两个相邻节点都连接起来」：

这样，`connectTwoNode`函数不断递归，可以无死角覆盖整棵二叉树，将所有相邻节点都连接起来，也就避免了我们之前出现的问题，这道题就解决了。



代码：

```java
class Solution {
	public Node connect(Node root) {
		if (root == null) return null;
		connectTwoNode(root.left, root.right);
		return root;
	}

	// 定义：输入两个节点，将它俩连接起来
	public void connectTwoNode(Node node1, Node node2) {
		if (node1 == null || node2 == null) {
			return;
		}
		/**** 前序遍历位置 ****/
		// 将传入的两个节点连接
		node1.next = node2;

		// 连接相同父节点的两个子节点
		connectTwoNode(node1.left, node1.right);
		connectTwoNode(node2.left, node2.right);
		// 连接跨越父节点的两个子节点
		connectTwoNode(node1.right, node2.left);
	}
}
```

---











## [\#141. 判断链表是否存在环](https://leetcode-cn.com/problems/linked-list-cycle/description/) 

- Easy
- 2019.09.13：😭 

题目：

```xml
给定一个链表，判断链表中是否有环。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。

示例 1：
输入：head = [3,2,0,-4], pos = 1
输出：true
解释：链表中有一个环，其尾部连接到第二个节点。
```

实例1:![image-20200913094053928](/assets/imgs/image-20200913094053928.png)

分析：

```xml
方法一：哈希表
我们遍历所有结点并在哈希表中存储每个结点的引用（或内存地址）。如果当前结点为空结点 null（即已检测到链表尾部的下一个结点），那么我们已经遍历完整个链表，并且该链表不是环形链表。如果当前结点的引用已经存在于哈希表中，那么返回 true（即该链表为环形链表）。
复杂度分析:
时间复杂度：O(n),对于含有 n 个元素的链表，我们访问每个元素最多一次。添加一个结点到哈希表中只需要花费 O(1)的时间。
空间复杂度：O(n)，空间取决于添加到哈希表中的元素数目，最多可以添加 n 个元素。

方法二：双指针 ⭐️
通过使用具有 不同速度 的快、慢两个指针遍历链表，空间复杂度可以被降低至 O(1)。慢指针每次移动一步，而快指针每次移动两步。
如果列表中不存在环，最终快指针将会最先到达尾部，此时我们可以返回 false。

现在考虑一个环形链表，把慢指针和快指针想象成两个在环形赛道上跑步的运动员（分别称之为慢跑者与快跑者）。而快跑者最终一定会追上慢跑者。这是为什么呢？考虑下面这种情况（记作情况 A）- 假如快跑者只落后慢跑者一步，在下一次迭代中，它们就会分别跑了一步或两步并相遇。
其他情况又会怎样呢？例如，我们没有考虑快跑者在慢跑者之后两步或三步的情况。但其实不难想到，因为在下一次或者下下次迭代后，又会变成上面提到的情况 A。
```

代码：

```java
// 方法一
public boolean hasCycle(ListNode head) {
    Set<ListNode> nodesSeen = new HashSet<>();
    while (head != null) {
        if (nodesSeen.contains(head)) {
            return true;
        } else {
            nodesSeen.add(head);
        }
        head = head.next;
    }
    return false;
}

// 方法二 ⭐️
public boolean hasCycle(ListNode head) {
    if (head == null || head.next == null) {
        return false;
    }
    ListNode slow = head; // 慢指针
    ListNode fast = head.next; // 快指针
  	// 直到两个指针相遇都没有指向null则返回true
    while (slow != fast) {
        if (fast == null || fast.next == null) { //fast.next == null 防止fast.next.next空指针异常
            return false;
        }
        slow = slow.next;
        fast = fast.next.next;
    }
    return true;
}
```



## [\#144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

- Medium
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

- Medium
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#167. 有序数组的 Two Sum](https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/description/)

- easy
- 2019.09.15：😭  
- 2019.09.16：😎

题目：

```xml
给定一个已按照升序排列的有序数组，找到两个数使得它们相加之和等于目标数。
函数应该返回这两个下标值 index1 和 index2，其中 index1 必须小于 index2。

说明:
返回的下标值（index1 和 index2）不是从零开始的。
你可以假设每个输入只对应唯一的答案，而且你不可以重复使用相同的元素。

示例:
输入: numbers = [2, 7, 11, 15], target = 9
输出: [1,2]
解释: 2 与 7 之和等于目标数 9 。因此 index1 = 1, index2 = 2 。
```

分析：

```
方法一：二分查找
在数组中找到两个数，使得它们的和等于目标值，可以首先固定第一个数，然后寻找第二个数，第二个数等于目标值减去第一个数的差。利用数组的有序性质，可以通过二分查找的方法寻找第二个数。为了避免重复寻找，在寻找第二个数时，只在第一个数的右侧寻找。
时间复杂度：O(nlogn)，其中 n 是数组的长度。需要遍历数组一次确定第一个数，时间复杂度是 O(n)，寻找第二个数使用二分查找，时间复杂度是 O(logn)，因此总时间复杂度是 O(nlogn)。
空间复杂度：O(1)

方法二：双指针
初始时两个指针分别指向第一个元素位置和最后一个元素的位置。每次计算两个指针指向的两个元素之和，并和目标值比较。如果两个元素之和等于目标值，则发现了唯一解。如果两个元素之和小于目标值，则将左侧指针右移一位。如果两个元素之和大于目标值，则将右侧指针左移一位。移动指针之后，重复上述操作，直到找到答案。

使用双指针的实质是缩小查找范围。那么会不会把可能的解过滤掉？
如果左指针先到达下标 i 的位置，此时右指针还在下标 j 的右侧，sum>target，因此一定是右指针左移，左指针不可能移到 i 的右侧。
如果右指针先到达下标 j 的位置，此时左指针还在下标 i 的左侧，sum<target，因此一定是左指针右移，右指针不可能移到 j 的左侧。
由此可见，在整个移动过程中，左指针不可能移到 i 的右侧，右指针不可能移到 j 的左侧，因此不会把可能的解过滤掉。由于题目确保有唯一的答案，因此使用双指针一定可以找到答案。
时间复杂度：O(n)，其中 n 是数组的长度。两个指针移动的总次数最多为 n 次。
空间复杂度：O(1)
```

代码：

```java
// 方法一 二分查找
class Solution {
    // 可以首先固定第一个数，然后寻找第二个数，第二个数等于目标值减去第一个数的差。
    // 利用数组的有序性质，可以通过二分查找的方法寻找第二个数。
    public int[] twoSum(int[] numbers, int target) {
        for (int i = 0; i < numbers.length; i++) {
            int low = i + 1; // 为了避免重复寻找，在寻找第二个数时，只在第一个数的右侧寻找(low=i+1)。
          	int high = numbers.length - 1;
            while (low <= high) {
                int mid = (high - low) / 2 + low; // 如果用mid=(left+right)/2，在运行二分查找程序时可能溢出超时。因为如果left和right相加超过int表示的最大范围时就会溢出变为负数。所以如果想避免溢出，不能使用mid=(left+right)/2，应该使用mid=left+(right-left)/2。
                if (numbers[mid] == target - numbers[i]) {
                    return new int[]{i + 1, mid + 1};
                } else if (numbers[mid] > target - numbers[i]) {
                    high = mid - 1; // mid+i > target high指针移到mid左边
                } else {
                    low = mid + 1;
                }
            }
        }
        return new int[]{-1, -1};
    }
}

// 方法二 双指针
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int low = 0, high = numbers.length - 1; // 定义指针
        while (low < high) {
            int sum = numbers[low] + numbers[high];
            if (sum == target) {
                return new int[]{low + 1, high + 1};
            } else if (sum < target) { // 因为是有序数组
                ++low;  // 小于target的话头指针++
            } else {
                --high; // 大于target的话尾指针++
            }
        }
        return new int[]{-1, -1};
    }
}
```





## [#169. 多数元素](https://leetcode-cn.com/problems/majority-element/) 

- Easy
- 2019.08.30：😭 哈希表，naive！ 
- 2019.08.30：😎 哈希表

题目：

```xml
给定一个大小为 n 的数组，找到其中的多数元素。多数元素是指在数组中出现次数大于 ⌊ n/2 ⌋ 的元素。
你可以假设数组是非空的，并且给定的数组总是存在多数元素。
示例 1:
输入: [3,2,3]
输出: 3

示例 2:
输入: [2,2,1,1,1,2,2]
输出: 2
```

分析：

```xml
方法一：哈希表
思路：
遍历整个数组，对记录每个数值出现的次数(利用HashMap，其中key为数值，value为出现次数)；
接着遍历HashMap中的每个Entry，寻找value值> nums.length/2 的key即可。
复杂度分析：
时间复杂度：O(n)。
空间复杂度：O(n)。哈希表最多包含 n - [n/2] 个键值对，所以占用的空间为 O(n)。

方法二：排序  众数
思路：
如果将数组 nums 中的所有元素按照单调递增或单调递减的顺序排序，那么下标为 [n/2] 的元素（下标从 0 开始）一定是众数。
算法：
对于这种算法，我们先将 nums 数组排序，然后返回上文所说的下标对应的元素。下面的图中解释了为什么这种策略是有效的。在下图中，第一个例子是 n 为奇数的情况，第二个例子是 n 为偶数的情况。
对于每种情况，数组下面的线表示如果众数是数组中的最小值时覆盖的下标，数组下面的线表示如果众数是数组中的最大值时覆盖的下标。对于其他的情况，这条线会在这两种极端情况的中间。对于这两种极端情况，它们会在下标为 [n/2] 的地方有重叠。因此，无论众数是多少，返回 [n/2]下标对应的值都是正确的。
复杂度分析：
时间复杂度：O(nlogn)。将数组排序的时间复杂度为 O(nlogn)。
空间复杂度：O(logn)。如果使用语言自带的排序算法，需要使用 O(logn) 的栈空间。如果自己编写堆排序，则只需要使用 O(1) 的额外空间。


方法三：摩尔投票法
摩尔投票法，遇到相同的数，就投一票，遇到不同的数，就减一票，最后还存在票的数就是众数
候选人(cand_num)初始化为nums[0]，票数count初始化为1。
当遇到与cand_num相同的数，则票数count = count + 1，否则票数count = count - 1。
当票数count为0时，更换候选人，并将票数count重置为1。
遍历完数组后，cand_num即为最终答案。

为何这行得通呢？
投票法是遇到相同的则票数 + 1，遇到不同的则票数 - 1。
且“多数元素”的个数> ⌊ n/2 ⌋，其余元素的个数总和<= ⌊ n/2 ⌋。
因此“多数元素”的个数 - 其余元素的个数总和 的结果 肯定 >= 1。
这就相当于每个“多数元素”和其他元素 两两相互抵消，抵消到最后肯定还剩余至少1个“多数元素”。
无论数组是1 2 1 2 1，亦或是1 2 2 1 1，总能得到正确的候选人。
```

方法二说明：

![image-20200831201456212](/assets/imgs/image-20200831201456212-9188642.png)

代码：

```java
// 方法一：哈希表计数法 ☑️
class Solution {
    public int majorityElement(int[] nums) {
       int limit = nums.length/2;
    	// 1 遍历整个数组放入HashMap key为数值，value为次数
		Map<Integer,Integer> map = new HashMap<>(); //构造一个具有指定初始容量和默认负载因子（0.75）的空HashMap。
		for (int num : nums) {
			map.merge(num,1,(o_val,n_val)->{return o_val+n_val;}); //它将新的值赋值给到key中（如果不存在）或更新具有给定值的现有key（UPSERT）
		}
		// 2 遍历HashMap中的每个Entry 寻找value大于半数的值
		for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
			if (entry.getValue()>limit) return entry.getKey();
		}
    	return -1
    }
}

// 方法二: 排序 ☑️
class Solution {
    public int majorityElement(int[] nums) {
        Arrays.sort(nums);
        return nums[nums.length/2];
    }
}


// 方法三：摩尔投票法
// 摩尔投票法，遇到相同的数，就投一票，遇到不同的数，就减一票，最后还存在票的数就是众数
class Solution {
    public int majorityElement(int[] nums) {
        int cand_num = nums[0], count = 1;
        for (int i = 1; i < nums.length; ++i) {
            if (cand_num == nums[i])
                ++count;
            else if (--count == 0) {
                cand_num = nums[i];
                count = 1;
            }
        }
        return cand_num;
    }
}
```

---



## [\#208. 实现 Trie (前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---







## [\#215. 数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

- easy
- 2019.08.28：😭  

题目：

```xml
在未排序的数组中找到第 k 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

示例 1:
输入: [3,2,1,5,6,4] 和 k = 2
输出: 5
示例 2:
输入: [3,2,3,1,2,4,5,5,6] 和 k = 4
输出: 4

说明:
你可以假设 k 总是有效的，且 1 ≤ k ≤ 数组的长度。
```

分析：

***方法一：基于快速排序的选择方法***

> 适用于确定数据量的情况

我们可以用***快速排序***来解决这个问题，先对原数组排序，再返回倒数第 k 个位置，这样平均时间复杂度是O(nlogn)，但其实我们可以做的更快。

![image-20201006132933073](/assets/imgs/image-20201006132933073.png)

因此我们可以改进快速排序算法来解决这个问题：在分解的过程当中，我们会对子数组进行划分，如果划分得到的 q 正好就是我们需要的下标，就直接返回 a[q]；否则，***如果 q 比目标下标小，就递归右子区间，否则递归左子区间。这样就可以把原来递归两个区间变成只递归一个区间，提高了时间效率。这就是「快速选择」算法。***

我们知道快速排序的性能和「划分」出的子数组的长度密切相关。直观地理解如果每次规模为 n 的问题我们都划分成 1 和 n - 1，每次递归的时候又向 n−1 的集合中递归，这种情况是最坏的，时间代价是 $O(n^2)$。我们可以引入随机化来加速这个过程，它的时间代价的期望是 O(n)。

- 时间复杂度：O(n)
- 空间复杂度：O(nlogn)

代码：

```java
// 方法一：快速选择
class Solution {
    Random random = new Random();

    public int findKthLargest(int[] nums, int k) {
        // 要找到的元素所在索引:  前K大，即倒数索引第K个
        int index = nums.length - k;
        int right = nums.length - 1;
        int left = 0;
        return quickSelect(nums, left, right, index);
    }
    
    public int quickSelect(int[] nums, int left, int right, int index) {
        // 随机生成轴值，得到分区值后的轴值索引
        int q = randomPartition(nums, left, right);

        // 递归的出口
        if (q == index) {
            // 如果刚好索引q就是想要的索引，则直接返回
            return nums[q];

        } else {
            // 如果不是，比较q 与 index ,确定下次要检索的区间, 要么是[q+1, right], 要么就是[left, q-1]
            return q < index ? quickSelect(nums, q + 1, right, index) : quickSelect(nums, left, q - 1, index);
        }
    }

  	// 随机选择轴值
    public int randomPartition(int[] nums, int l, int r) {
        // 1. 随机数范围: [0, r-l+1) 同时加l, 则是 [l, r+1).即在这个[l,r] 中随机选一个索引出来
        int i = random.nextInt(r - l + 1) + l;

        // 2. 交换nums[i]， nums[r], 也就是将随机数先放在最右边nums[r]上
        swap(nums, i, r);
        return partition(nums, l, r);
    }

    // 该函数负责把比轴值大的元素放到右边，小的元素放到左边，最后返回新数组中轴值的下标
    // 为了方便比较，把轴值反倒数组的最右边，并且初始化第一个比轴值小的位置(数组的最左边)
    public int partition(int[] nums, int l, int r) {
        // 3. 在调用当前方法的randomPartition方法中，已经确定了随机数是nums[r]
        int x = nums[r];
        int i = l - 1; // i+1 为当前大于轴值的元素下标

        // 首先比较区间在[l， r)之间
        // 这个for循环操作就是将小于 x 的数都往[i, j]的左边区间设置，从而实现存在[l, i]区间,使得对应数值都 小于 x
        for (int j = l; j < r; j++) {
            // 4. nums[j] 跟随机数 x 比较
            if (nums[j] <= x) {
                i++; // 有小于x的元素，这轴值坐标+1
                swap(nums, i, j);
            }
        }

        // 换回轴值
        //5. 既然已经将<x的值都放在一边了，现在将x也就是nums[r] 跟nums[i+1]交换，从而分成两个区间[l.i+1]左, [i+2, r]右，左边区间的值都小于x
        swap(nums, i + 1, r);

        // 然后返回这个分区值(新的初始索引left)
        return i + 1;
    }

    public void swap(int[] a, int i, int j) {
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
}
```



## [\#226. 翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

- easy
- 2020.10.01：😎
- 2020.10.27：😎   前序遍历写成了后序，也可以实现

题目：

```xml
翻转一棵二叉树。

示例：
输入：
     4
   /   \
  2     7
 / \   / \
1   3 6   9
输出：
     4
   /   \
  7     2
 / \   / \
9   6 3   1
```

分析：

通过观察，**我们发现只要把二叉树上的每一个节点的左右子节点进行交换，最后的结果就是完全翻转之后的二叉树**。

代码：

```java
class Solution {
    public TreeNode invertTree(TreeNode root) {
    	if (root==null) return null;

		/**** 前序遍历位置 ****/
		// root 节点需要交换它的左右子节点
    	TreeNode temp = root.left;
    	root.left=root.right;
    	root.right=temp;

		// 让左右子节点继续翻转它们的子节点
    	invertTree(root.left);
    	invertTree(root.right);
      
    	return root;
    }
}
```

---

## [\#230. 二叉搜索树中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/)

- Medium
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#235. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#236. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---



## [\#322. 零钱兑换](https://leetcode-cn.com/problems/coin-change/)

- 中等
- 2020.11.16：😭  
- 2020.11.17：😭   忘记考虑了子问题无解！

题目：

```xml
给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。

你可以认为每种硬币的数量是无限的。

示例 1：
输入：coins = [1, 2, 5], amount = 11
输出：3 
解释：11 = 5 + 5 + 1

示例 2：
输入：coins = [2], amount = 3
输出：-1
```

分析：

***方法一：***暴力递归

首先，这个问题是动态规划问题，因为它具有「最优子结构」的。**要符合「最优子结构」，子问题间必须互相独立**。

凑零钱问题，为什么说它符合最优子结构呢？比如你想求 `amount = 11` 时的最少硬币数（原问题），如果你知道凑出 `amount = 10` 的最少硬币数（子问题），你只需要把子问题的答案加一（再选一枚面值为 1 的硬币）就是原问题的答案。因为硬币的数量是没有限制的，所以子问题之间没有相互制，是互相独立的。

那么，既然知道了这是个动态规划问题，就要思考**如何列出正确的状态转移方程**？

1、**确定 base case**，这个很简单，显然目标金额 `amount` 为 0 时算法返回 0，因为不需要任何硬币就已经凑出目标金额了。

2、**确定「状态」，也就是原问题和子问题中会变化的变量**。由于硬币数量无限，硬币的面额也是题目给定的，只有目标金额会不断地向 base case 靠近，所以唯一的「状态」就是目标金额 `amount`。

3、**确定「选择」，也就是导致「状态」产生变化的行为**。目标金额为什么变化呢，因为你在选择硬币，你每选择一枚硬币，就相当于减少了目标金额。所以说所有硬币的面值，就是你的「选择」。

4、**明确** **`dp`** **函数/数组的定义**。我们这里讲的是自顶向下的解法，所以会有一个递归的 `dp` 函数，一般来说函数的参数就是状态转移中会变化的量，也就是上面说到的「状态」；函数的返回值就是题目要求我们计算的量。就本题来说，状态只有一个，即「目标金额」，题目要求我们计算凑出目标金额所需的最少硬币数量。所以我们可以这样定义 `dp` 函数：

`dp(n)` 的定义：输入一个目标金额 `n`，返回凑出目标金额 `n` 的最少硬币数量。

搞清楚上面这几个关键点，解法的伪码就可以写出来了：

```java
class Solution {
    // 伪代码框架 	
    public int coinChange(int[] coins, int amount) {
        return dp(coins,amount);
    }
    
    // 定义：要凑出金额n,至少需要dp(n)个硬币
    public int dp(int[] coins,int amount){
    	int res = Integer.MAX_VALUE;
      // 做选择，选择需要硬币最少的那个结果
      for (int coin : coins) {
          res=Math.min(res,1+dp(coins,amount-coin));
       }
       return res;
    }
}
```

根据伪码，我们加上 base case 即可得到最终的答案。显然目标金额为 0 时，所需硬币数量为 0；当目标金额小于 0 时，无解，返回 -1：

```java
class Solution {
    // 伪代码框架
    public int coinChange(int[] coins, int amount) {
       return dp(coins,amount);
    }

    // 定义：要凑出金额n,至少需要dp(n)个硬币
    public int dp(int[] coins,int amount){
        // base case
        if (amount==0) return 0;
        if (amount<0) return -1;

        // 求最小值，使用初始化为Integer的最大值
        int res = Integer.MAX_VALUE;
        for (int coin : coins) {
            int subProblem = dp(coins,amount-coin);
            // 子问题无解 跳过
            if (subProblem==-1) continue;
            res=Math.min(res,1+subProblem);
        }
        return res!=Integer.MAX_VALUE?res:-1;
   }
}
```

至此，状态转移方程其实已经完成了，以上算法已经是暴力解法了，以上代码的数学形式就是状态转移方程：

![image-20201116205314827](/assets/imgs/image-20201116205314827.png)

至此，这个问题其实就解决了，只不过需要消除一下重叠子问题，比如 `amount = 11, coins = {1,2,5}` 时画出递归树看看：

![image-20201116205333883](/assets/imgs/image-20201116205333883.png)

**递归算法的时间复杂度分析：子问题总数 x 每个子问题的时间**。

子问题总数为递归树节点个数，这个比较难看出来，是 O(n^k)，总之是指数级别的。每个子问题中含有一个 for 循环，复杂度为 O(k)。所以总时间复杂度为 O(k * n^k)，指数级别。



方法二：带备忘录的递归

类似之前斐波那契数列的例子，只需要稍加修改，就可以通过备忘录消除子问题.

很显然「备忘录」大大减小了子问题数目，完全消除了子问题的冗余，所以子问题总数不会超过金额数 `n`，即子问题数目为 O(n)。处理一个子问题的时间不变，仍是 O(k)，所以总的时间复杂度是 O(kn)。

```java
class Solution {

    // 伪代码框架
    public int coinChange(int[] coins, int amount) {

    // 初始化备忘录
		HashMap<Integer, Integer> memo = new HashMap<>();
       return dp(memo,coins,amount);
    }

    // 定义：要凑出金额n,至少需要dp(n)个硬币
    public int dp(HashMap<Integer, Integer> memo,int[] coins,int amount){

        // 查备忘录，避免重复计算
        Integer integer = memo.get(amount);
        if (integer !=null) return integer;
        // base case
        if (amount==0) return 0;
        if (amount<0) return -1;

        // 求最小值，使用初始化为Integer的最大值
        int res = Integer.MAX_VALUE;
        for (int coin : coins) {
            int subProblem = dp(memo,coins,amount-coin);
            // 子问题无解 跳过
            if (subProblem==-1) continue;
              res=Math.min(res,1+subProblem);
         }

        // 记录备忘录
        memo.put(amount,res!=Integer.MAX_VALUE?res:-1);
        return memo.get(amount);
   }
}
```



方法三：dp数组的迭代解法

当然，我们也可以自底向上使用 dp table 来消除重叠子问题，关于「状态」「选择」和 base case 与之前没有区别，`dp` 数组的定义和刚才 `dp` 函数类似，也是把「状态」，也就是目标金额作为变量。不过 `dp` 函数体现在函数参数，而 `dp` 数组体现在数组索引：

**`dp`** **数组的定义：当目标金额为** **`i`** **时，至少需要** **`dp[i]`** **枚硬币凑出**。

根据我们文章开头给出的动态规划代码框架可以写出如下解法：

PS：为啥 `dp` 数组初始化为 `amount + 1` 呢，因为凑成 `amount` 金额的硬币数最多只可能等于 `amount`（全用 1 元面值的硬币），所以初始化为 `amount + 1` 就相当于初始化为正无穷，便于后续取最小值。

```java
class Solution {

  public int coinChange(int[] coins, int amount) {

    // 数组大小为 amount+1，初始值也为amount+1
    int[] dp = new int[amount + 1];
    for (int i = 0; i <= amount; i++) {
      dp[i]=amount+1;
    }

    // base case
    dp[0]=0;
    // 外层 for 循环遍历所有状态的所有值
    for (int i = 0; i < dp.length; i++) {
      // 内层 for 循环在求所有选择的最小值
      for (int coin : coins) {
        // 子问题无解，跳过
        if (i-coin<0) continue;
        dp[i]=Math.min(dp[i],1+dp[i-coin]);
      }
    }
    return (dp[amount]==amount+1)?-1:dp[amount];
  }
}
```

![image-20201116221952265](/assets/imgs/image-20201116221952265.png)



---





## [\#337. 打家劫舍 III](https://leetcode-cn.com/problems/house-robber-iii/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#345. 反转字符串中的元音字符](https://leetcode-cn.com/problems/reverse-vowels-of-a-string/description/)

- Easy
- 2019.09.11：😎  
- 2019.09.12：😭 temp变量没定义，判断写成了`chars.indexOf(s[i]!=-1);`

题目：

```xml
编写一个函数，以字符串作为输入，反转该字符串中的元音字母。

示例 1：
输入："hello"
输出："holle"
```

分析：

```xml
双指针法交换前后元音元素
转换成数组后，分别定义前后两个索引指针用 while 依次遍历数组，同时遇到元音则交换。
最后扫描完数组后，一定要在返回的时候再转成字符串 String 输出
```

代码：

```java
class Solution {
  public String reverseVowels(String s) {
    String str = "aeuioAEUIO";
    int i = 0;
    int j = s.length() - 1;
    char[] chars = s.toCharArray();

    while (i < j) {
      if (str.indexOf(chars[i]) != -1) { // 判断元音 str.indexOf(ch)!=-1
        if (str.indexOf(chars[j]) != -1) {
          char temp = chars[i]; // 变量忘记定义，python写习惯了
          chars[i] = chars[j];
          chars[j] = temp;
          i++;
          j--;
        }else {
          j--; //j 不是元音 ++
        }
      } else {i++;} // i 不是元音 ++
    }
    return new String(chars);
  }
}
```



## [\#347. 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

- medium
- 2019.09.23：😭  

题目：

```xml
给定一个非空的整数数组，返回其中出现频率前 k 高的元素。

示例 1:
输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]
示例 2:
输入: nums = [1], k = 1
输出: [1]
 
提示：
你可以假设给定的 k 总是合理的，且 1 ≤ k ≤ 数组中不相同的元素的个数。
你的算法的时间复杂度必须优于 O(n log n) , n 是数组的大小。
题目数据保证答案唯一，换句话说，数组中前 k 个高频元素的集合是唯一的。
你可以按任意顺序返回答案。
```

分析：

***方法一：粗暴排序法***
使用排序算法对元素按照频率由高到低进行排序，然后再取前 k 个元素。使用常规的诸如 冒泡、选择、甚至快速排序都是不满足题目要求，它们的时间复杂度都是大于或者等于 O(nlog⁡n)，而题目要求算法的时间复杂度必须优于 O(nlogn)。

***方法二：最小堆***
题目最终需要返回的是前 k 个频率最大的元素，可以想到借助堆这种数据结构，对于 k 频率之后的元素不用再去处理，进一步优化时间复杂度。

![image-20200923141746579](/assets/imgs/image-20200923141746579.png)具体操作为：

- 借助 哈希表 来建立数字和其出现次数的映射，遍历一遍数组统计元素的频率
- 维护一个元素数目为 k 的最小堆
- 每次都将新的元素与堆顶元素（堆中频率最小的元素）进行比较
- 如果新的元素的频率比堆顶端的元素大，则弹出堆顶端的元素，将新的元素添加进堆中
- 最终，堆中的 k 个元素即为前 k 个高频元素

![b548a3796066fa7072baa2b1e06e0d54641a7913d87c88c61d73b6b9ad0e90db-file_1561712388100](/assets/imgs/b548a3796066fa7072baa2b1e06e0d54641a7913d87c88c61d73b6b9ad0e90db-file_1561712388100-0841953.gif)

***方法三：桶排序法***
首先依旧使用哈希表统计频率，统计完成后，创建一个数组，将频率作为数组下标，对于出现频率不同的数字集合，存入对应的数组下标即可。

![image-20200923142420110](/assets/imgs/image-20200923142420110.png)

代码：

```java
// 方法二
class Solution {
    public List<Integer> topKFrequent(int[] nums, int k) {
        // 使用字典，统计每个元素出现的次数，元素为键，元素出现的次数为值
        HashMap<Integer,Integer> map = new HashMap();
        for(int num : nums){
            if (map.containsKey(num)) {
               map.put(num, map.get(num) + 1);
             } else {
                map.put(num, 1);
             }
        }
        // 遍历map，用最小堆保存频率最大的k个元素
        PriorityQueue<Integer> pq = new PriorityQueue<>(new Comparator<Integer>() {
            @Override
            public int compare(Integer a, Integer b) {
                return map.get(a) - map.get(b);
            }
        });
        for (Integer key : map.keySet()) {
            if (pq.size() < k) {
                pq.add(key);
            } else if (map.get(key) > map.get(pq.peek())) {
                pq.remove();
                pq.add(key);
            }
        }
        // 取出最小堆中的元素
        List<Integer> res = new ArrayList<>();
        while (!pq.isEmpty()) {
            res.add(pq.remove());
        }
        return res;
    }
}

// 方法三
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
		List<Integer> res = new ArrayList();
		// 使用字典，统计每个元素出现的次数，元素为键，元素出现的次数为值
		HashMap<Integer, Integer> map = new HashMap<>();
		for (int num : nums) {
			if (map.containsKey(num)){
				map.put(num,map.get(num)+1);
			}else{
				map.put(num,1);
			}
		}

		// 桶排序 将频率作为数组下标，对于出现频率不同的数字集合，存入对应的数组下标
		// 把频率作为桶/数组下标，再存入对应的数
		List<Integer>[] list = new List[nums.length+1]; // 最大频率为nums.length
		for (Integer key : map.keySet()) {
			// 获取出现的次数作为下标
			int i = map.get(key);
			if (list[i]==null){
				list[i]=new ArrayList<>();
			}
			list[i].add(key);
		}

		// 倒序遍历数组获取出现顺序从大到小的排列
		for(int i = list.length - 1;i >= 0 && res.size() < k;i--){
			if(list[i] == null) continue;
			res.addAll(list[i]);
		}

		// list转数组
		int[] arr = new int[res.size()];
		for (int i = 0; i < res.size(); i++) {
			arr[i]=res.get(i);
		}

    	return arr;
    }
}
```



## [\#404. 左叶子之和](https://leetcode-cn.com/problems/sum-of-left-leaves/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#437. 路径总和 III](https://leetcode-cn.com/problems/path-sum-iii/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#451. 根据字符出现频率排序](https://leetcode-cn.com/problems/sort-characters-by-frequency/)

- easy
- 2019.08.28：😭  

题目：

```xml

```

分析：

```xml

```

代码：

```java

```





## [\#501. 二叉搜索树中的众数](https://leetcode-cn.com/problems/find-mode-in-binary-search-tree/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---



## [\#509. 斐波那契数](https://leetcode-cn.com/problems/fibonacci-number/)

- easy
- 2020.11.01：😭  

题目：

```xml
斐波那契数，通常用 F(n) 表示，形成的序列称为斐波那契数列。该数列由 0 和 1 开始，后面的每一项数字都是前面两项数字的和。也就是：
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
给定 N，计算 F(N)。

示例 1：
输入：2
输出：1
解释：F(2) = F(1) + F(0) = 1 + 0 = 1.

示例 2：
输入：3
输出：2
解释：F(3) = F(2) + F(1) = 1 + 1 = 2.
```

分析：

***方法一：暴力递归***

这个不用多说了，学校老师讲递归的时候似乎都是拿这个举例。我们也知道这样写代码虽然简洁易懂，但是十分低效，低效在哪里？假设 n = 20，请画出递归树：

![image-20201101220116962](/assets/imgs/image-20201101220116962.png)

**递归算法的时间复杂度怎么计算？就是用子问题个数乘以解决一个子问题需要的时间。**

首先计算子问题个数，即递归树中节点的总数。显然二叉树节点总数为指数级别，所以子问题个数为 O(2^n)。

然后计算解决一个子问题的时间，在本算法中，没有循环，只有 `f(n - 1) + f(n - 2)` 一个加法操作，时间为 O(1)。

所以，这个算法的时间复杂度为二者相乘，即 O(2^n)，指数级别，爆炸。

观察递归树，很明显发现了算法低效的原因：存在大量重复计算，比如 `f(18)` 被计算了两次，而且你可以看到，以 `f(18)` 为根的这个递归树体量巨大，多算一遍，会耗费巨大的时间。更何况，还不止 `f(18)` 这一个节点被重复计算，所以这个算法及其低效。

这就是动态规划问题的第一个性质：**重叠子问题**。下面，我们想办法解决这个问题。

***方法二：带备忘录的递归解法***

明确了问题，其实就已经把问题解决了一半。即然耗时的原因是重复计算，那么我们可以造一个「备忘录」，每次算出某个子问题的答案后别急着返回，先记到「备忘录」里再返回；每次遇到一个子问题先去「备忘录」里查一查，如果发现之前已经解决过这个问题了，直接把答案拿出来用，不要再耗时去计算了。

一般使用一个数组充当这个「备忘录」，当然你也可以使用哈希表（字典），思想都是一样的。

![image-20201101220735568](/assets/imgs/image-20201101220735568.png)

实际上，带「备忘录」的递归算法，把一棵存在巨量冗余的递归树通过「剪枝」，改造成了一幅不存在冗余的递归图，极大减少了子问题（即递归图中节点）的个数。

![image-20201101220937465](/assets/imgs/image-20201101220937465.png)

复杂度计算：

子问题个数，即图中节点的总数，由于本算法不存在冗余计算，子问题就是 `f(1)`, `f(2)`, `f(3)` ... `f(20)`，数量和输入规模 n = 20 成正比，所以子问题个数为 O(n)。

解决一个子问题的时间，同上，没有什么循环，时间为 O(1)。

所以，本算法的时间复杂度是 O(n)。比起暴力算法，是降维打击。

至此，带备忘录的递归解法的效率已经和迭代的动态规划解法一样了。实际上，***这种解法和迭代的动态规划已经差不多了，只不过这种方法叫做「自顶向下」，动态规划叫做「自底向上」。***

啥叫「自顶向下」？注意我们刚才画的递归树（或者说图），是从上向下延伸，都是从一个规模较大的原问题比如说 `f(20)`，向下逐渐分解规模，直到 `f(1)` 和 `f(2)` 这两个 base case，然后逐层返回答案，这就叫「自顶向下」。

啥叫「自底向上」？反过来，我们直接从最底下，最简单，问题规模最小的 `f(1)` 和 `f(2)` 开始往上推，直到推到我们想要的答案 `f(20)`，这就是动态规划的思路，这也是为什么动态规划一般都脱离了递归，而是由循环迭代完成计算。



***⭐️ 方法三：dp 数组的迭代解法***

有了上一步「备忘录」的启发，我们可以把这个「备忘录」独立出来成为一张表，就叫做 DP table 吧，在这张表上完成「自底向上」的推算岂不美哉！

![image-20201101222001359](/assets/imgs/image-20201101222001359.png)

画个图就很好理解了，而且你发现这个 DP table 特别像之前那个「剪枝」后的结果，只是反过来算而已。实际上，带备忘录的递归解法中的「备忘录」，最终完成后就是这个 DP table，所以说这两种解法其实是差不多的，大部分情况下，效率也基本相同。

这里，引出「状态转移方程」这个名词，实际上就是描述问题结构的数学形式：

![image-20201101222521144](/assets/imgs/image-20201101222521144.png)

为啥叫「状态转移方程」？其实就是为了听起来高端。你把 `f(n)` 想做一个状态 `n`，这个状态 `n` 是由状态 `n - 1` 和状态 `n - 2` 相加转移而来，这就叫状态转移，仅此而已。

你会发现，上面的几种解法中的所有操作，例如 `return f(n - 1) + f(n - 2)`，`dp[i] = dp[i - 1] + dp[i - 2]`，以及对备忘录或 DP table 的初始化操作，都是围绕这个方程式的不同表现形式。可见列出「状态转移方程」的重要性，它是解决问题的核心。而且很容易发现，其实状态转移方程直接代表着暴力解法。

**千万不要看不起暴力解，动态规划问题最困难的就是写出这个暴力解，即状态转移方程**。只要写出暴力解，优化方法无非是用备忘录或者 DP table，再无奥妙可言。

⚠️ 这个例子的最后，讲一个细节优化。细心的读者会发现，根据斐波那契数列的状态转移方程，当前状态只和之前的两个状态有关，其实并不需要那么长的一个 DP table 来存储所有的状态，只要想办法存储之前的两个状态就行了。所以，可以进一步优化，把空间复杂度降为 O(1)：

这个技巧就是所谓的「**状态压缩**」，如果我们发现每次状态转移只需要 DP table 中的一部分，那么可以尝试用状态压缩来缩小 DP table 的大小，只记录必要的数据，上述例子就相当于把DP table 的大小从 `n` 缩小到 2。后续的动态规划章节中我们还会看到这样的例子，一般来说是把一个二维的 DP table 压缩成一维，即把空间复杂度从 O(n^2) 压缩到 O(n)。



代码：

```java
// 方法一
class Solution {
    public int fib(int N) {
		if (N==1||N==2) return 1;
		return fib(N-1)+fib(N-2);
    }
}

// 方法二
class Solution {
    public int fib(int N) {
		if (N<1) return 0;
		// 备忘录全初始化为0
		int[] memo = new int[N+1];
		// 进行带备忘录的回归
		return helper(memo,N);
    }
  
    public int helper(int[] memo,int n){
    	// base case
		if (n==1||n==2) return 1;
		// 已经计算过
		if (memo[n] !=0) return memo[n];
		memo[n] = helper(memo,n-1)+helper(memo,n-2);
		return memo[n];
	}
}

// 方法三
class Solution {
    public int fib(int N) {
		int[] dp = new int[N+1];
		// base case
		dp[1] = dp[2]=1;
		for (int i = 3; i <=N ; i++) {
			dp[i]=dp[i-1]+dp[i-2];
		}
		return dp[N];
    }
}
// 再优化
// 当前状态只和之前的两个状态有关，其实并不需要那么长的一个 DP table 来存储所有的状态
class Solution {
  public int fib(int N) {
    if (N==0) return 0;
    if (N == 2 || N == 1) return 1;
    int prev = 1;
    int curr = 1;
    for (int i = 3; i <= N; i++) {
      int sum = prev + curr;
      prev = curr;
      curr = sum;
    }
    return curr;
  }
}
```

---





## [\#513. 找树左下角的值](https://leetcode-cn.com/problems/find-bottom-left-tree-value/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#524. 最长子序列](https://leetcode-cn.com/problems/longest-word-in-dictionary-through-deleting/description/)

- easy
- 2019.09.16：😭  

题目：

```xml
给定一个字符串和一个字符串字典，找到字典里面最长的字符串，该字符串可以通过删除给定字符串的某些字符来得到。如果答案不止一个，返回长度最长且字典顺序最小的字符串。如果答案不存在，则返回空字符串。

示例 1:
输入:
s = "abpcplea", d = ["ale","apple","monkey","plea"]
输出: 
"apple"

说明:
所有输入的字符串只包含小写字母。
字典的大小不会超过 1000。
所有输入的字符串长度不会超过 1000。
```

分析：

```xml
这题的关键就是怎么在字符串字典中找到那个对应的字符串。其实很简单。
只要利用两个指针i,j，一个指向s字符串，一个指向s1字符串，每一次查找过程中,i依次后移，若i,j对应的两个字符相等，则j后移，如果j可以移到s1.length()，那么说明s1中对应的字符s中都有，即s中删除一些字符后，可以得到s1字符串，最后一步就是比较当前的结果字符与找到的s1字符，按照题目的需求来决定是否改变结果字符，是不是还挺简单的呀。
时间复杂度：O(n)
空间复杂度：O(1)
```

代码：

```java
class Solution {
	// s = "abpcplea", d = ["ale","apple","monkey","plea"]
    public String findLongestWord(String s, List<String> d) {
    	// 定义连个指针，一个指向s字符串，一个指向s1
		String str = "";
		for (String s1 : d) {
			// 每一次查找过程中,i依次后移，若i,j对应的两个字符相等，则j后移，如果j可以移到s1.length()，
			// 那么说明s1中对应的字符s中都有，即s中删除一些字符后，可以得到s1字符串，
			for (int i=0,j=0;i<s.length()&&j<s1.length();i++){
				if (s.charAt(i)==s1.charAt(j)) j++;
				if (j==s1.length()){
					// 比较当前的结果字符与找到的s1字符，按照题目的需求来决定是否改变结果字符
					// 找到字典里面最长的字符串;如果答案不止一个，返回长度最长且字典顺序最小的字符串
					if (s1.length()>str.length()||(s1.length()==str.length()&&str.compareTo(s1)>0)){
						str=s1;
					}
				}
			}
		}
    	return str;
    }
}
```



## [\#530. 二叉搜索树的最小绝对差](https://leetcode-cn.com/problems/minimum-absolute-difference-in-bst/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()

代码：

```java

```

---



## [\#538. 把二叉搜索树转换为累加树](https://leetcode-cn.com/problems/convert-bst-to-greater-tree/)

- Medium
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#543. 二叉树的直径](https://leetcode-cn.com/problems/diameter-of-binary-tree/)

- easy
- 2020.10.14：😭  

题目：

```xml
给定一棵二叉树，你需要计算它的直径长度。一棵二叉树的直径长度是任意两个结点路径长度中的最大值。这条路径可能穿过也可能不穿过根结点。

示例 :
给定二叉树

          1
         / \
        2   3
       / \     
      4   5    
返回 3, 它的长度是路径 [4,2,1,3] 或者 [5,2,1,3]。

注意：两结点之间的路径长度是以它们之间边的数目表示。
```

分析：

***方法一：***深度优先搜索

首先我们知道一条路径的长度为该路径经过的节点数减一，所以求直径（即求路径长度的最大值）等效于求路径经过节点数的最大值减一。

***而任意一条路径均可以被看作由某个节点为起点，从其左儿子和右儿子向下遍历的路径拼接得到。***

如图我们可以知道路径` [9, 4, 2, 5, 7, 8]` 可以被看作以 `2` 为起点，从其左儿子向下遍历的路径 `[2, 4, 9]` 和从其右儿子向下遍历的路径` [2, 5, 7, 8] `拼接得到。

***返回直径5(两结点之间的路径长度是以它们之间边的数目表示),也可以理解为2(左子树深度) +3(右子树深度)。***

![image-20201014104354737](/assets/imgs/image-20201014104354737.png)

- 时间复杂度：O(N)，其中 N 为二叉树的节点数，即遍历一棵二叉树的时间复杂度，每个结点只被访问一次。

- 空间复杂度：O(Height)，其中 Height 为二叉树的高度。由于递归函数在递归过程中需要为每一层递归函数分 配栈空间，所以这里需要额外的空间且该空间取决于递归的深度，而递归的深度显然为二叉树的高度，并且每次递归调用的函数里又只用了常数个变量，所以所需空间复杂度为 O(Height) 。

  

代码：

```java
class Solution {
    int max=0;
    public int diameterOfBinaryTree(TreeNode root) {
        depth(root);
        return max;
    }
    public int depth(TreeNode node){
        if(node==null){
            return 0;
        }
        int Left = depth(node.left);
        int Right = depth(node.right);
        max=Math.max(Left+Right,maxd);//将每个节点最大直径(左子树深度+右子树深度)当前最大值比较并取大者
        return Math.max(Left,Right)+1;//返回节点深度
    }
}
```

---



## [\#572. 另一个树的子树](https://leetcode-cn.com/problems/subtree-of-another-tree/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#617. 合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#633. 两数平方和 ](https://leetcode-cn.com/problems/sum-of-square-numbers/description/)

- Easy
- 2019.09.10：😭 
- 2019.09.11：😎 

题目：

```xml
给定一个非负整数 c ，你要判断是否存在两个整数 a 和 b，使得 a2 + b2 = c。

示例1:

输入: 5
输出: True
解释: 1 * 1 + 2 * 2 = 5
```

分析：

```xml
方法一：双指针
判断c是否为非负整数，若是，则直接返回false
利用Math包中sqrt()方法求出小于c的平方根的最大整数作为右指针，同时设置左指针从0开始；
开始循环，若左指针小于右指针，判断两指针之和与c的大小；
若和等于c，返回false；
若和小于c，左指针加1；
若和大于c，右指针减1；
默认返回false
复杂度分析：
时间复杂度O(sqrt(c))
空间复杂度O(1)
```

代码：

```java
// 双指针
class Solution {
    public boolean judgeSquareSum(int c) {
    	if (c<0) return false;
    	int i = 0; //双指针的左指针
    	int j = (int) Math.sqrt(c); //双指针的右指针
    	while (i<=j){
    		long sum = i*i+j*j; // 防止 sum变量溢出
    		if (sum==c) return true;
    		else if (sum<c){
    			i++;
			}else{j--;}
		}
		return false;
    }
}
```

---



## [\#637. 二叉树的层平均值](https://leetcode-cn.com/problems/average-of-levels-in-binary-tree/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---



## [\#652. 寻找重复的子树](https://leetcode-cn.com/problems/find-duplicate-subtrees/)

- 中等
- 2020.10.01：😭  

题目：

```xml
给定一棵二叉树，返回所有重复的子树。对于同一类的重复子树，你只需要返回其中任意一棵的根结点即可。

两棵树重复是指它们具有相同的结构以及相同的结点值。

示例 1：

        1
       / \
      2   3
     /   / \
    4   2   4
       /
      4
下面是两个重复的子树：

      2
     /
    4
和

    4
因此，你需要以列表的形式返回上述重复子树的根结点。
```

分析：

这题咋做呢？**还是老套路，先思考，对于某一个节点，它应该做什么**。

比如说，你站在图中这个节点 2 上：

如果你想知道以自己为根的子树是不是重复的，是否应该被加入结果列表中，你需要知道什么信息？

**你需要知道以下两点**：

**1、以我为根的这棵二叉树（子树）长啥样**？

**2、以其他节点为根的子树都长啥样**？

这就叫知己知彼嘛，我得知道自己长啥样，还得知道别人长啥样，然后才能知道有没有人跟我重复，对不对？

好，那我们一个一个来看，先来思考，**我如何才能知道以自己为根的二叉树长啥样**？

其实看到这个问题，就可以判断本题要使用「后序遍历」框架来解决：

```java
void traverse(TreeNode root) {
    traverse(root.left);
    traverse(root.right);
    /* 解法代码的位置 */
}
```

***为什么？很简单呀，我要知道以自己为根的子树长啥样，是不是得先知道我的左右子树长啥样，再加上自己，就构成了整棵子树的样子？***

所以，我们可以通过拼接字符串的方式把二叉树序列化，看下代码：

```java
String traverse(TreeNode root) {
    // 对于空节点，可以用一个特殊字符表示
    if (root == null) {
        return "#";
    }
    // 将左右子树序列化成字符串
    String left = traverse(root.left);
    String right = traverse(root.right);
    /* 后序遍历代码位置 */
    // 左右子树加上自己，就是以自己为根的二叉树序列化结果
    String subTree = left + "," + right + "," + root.val;
    return subTree;
}
```

我们用非数字的特殊符 `#` 表示空指针，并且用字符 `,` 分隔每个二叉树节点值，这属于序列化二叉树的套路了，不多说。

注意我们 `subTree` 是按照左子树、右子树、根节点这样的顺序拼接字符串，也就是后序遍历顺序。你完全可以按照前序或者中序的顺序拼接字符串，因为这里只是为了描述一棵二叉树的样子，什么顺序不重要。

**这样，我们第一个问题就解决了，对于每个节点，递归函数中的** **`subTree`** **变量就可以描述以该节点为根的二叉树**。

**现在我们解决第二个问题，我知道了自己长啥样，怎么知道别人长啥样**？这样我才能知道有没有其他子树跟我重复对吧。

这很简单呀，我们借助一个外部数据结构，让每个节点把自己子树的序列化结果存进去，这样，对于每个节点，不就可以知道有没有其他节点的子树和自己重复了么？

初步思路可以使用 `HashMap`，额外记录每棵子树的出现次数，代码如下：

```java
// 记录所有子树以及出现的次数
HashMap<String, Integer> memo = new HashMap<>();
// 记录重复的子树根节点
LinkedList<TreeNode> res = new LinkedList<>();

/* 主函数 */
List<TreeNode> findDuplicateSubtrees(TreeNode root) {
    traverse(root);
    return res;
}

/* 辅助函数 */
String traverse(TreeNode root) {
    if (root == null) {
        return "#";
    }

    String left = traverse(root.left);
    String right = traverse(root.right);
  
    String subTree = left + "," + right+ "," + root.val;

    int freq = memo.getOrDefault(subTree, 0);
    // 多次重复也只会被加入结果集一次
    if (freq == 1) {
        res.add(root);
    }
    // 给子树对应的出现次数加一
    memo.put(subTree, freq + 1);
    return subTree;
}
```



代码：

```java
class Solution {

	// 记录所有子树以及出现的次数
	HashMap<String, Integer> memo = new HashMap<>();
	// 记录重复的子树根节点
	LinkedList<TreeNode> res = new LinkedList<>();

  public List<TreeNode> findDuplicateSubtrees(TreeNode root) {
    traverse(root);
    return res;
  }

		/* 辅助函数 */
    public String traverse(TreeNode root){
		// 对于空节点，可以用一个特殊字符表示
		if (root==null) return "#";
		// 将左右子树序列化成字符串
		String left = traverse(root.left);
		String right = traverse(root.right);
		/* 后序遍历代码位置 */
		// 左右子树加上自己，就是以自己为根的二叉树序列化结果
		String subTree=left + "," + right + "," + root.val; // 描述以该节点为根的二叉树。

		int freq = memo.getOrDefault(subTree,0);
		// 多次重复也只会被加入结果集一次
		if (freq==1) res.add(root);

		// 对子树对应的出现次数加1
		memo.put(subTree,freq+1);
		return subTree;
	}
}
```

---





## [\#653. 两数之和 IV - 输入 BST](https://leetcode-cn.com/problems/two-sum-iv-input-is-a-bst/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()

代码：

```java

```

---



## [\#654. 最大二叉树](https://leetcode-cn.com/problems/maximum-binary-tree/)

- 中等
- 2020.10.01：😭  
- 2020.10.21：😎  
- 2020.11.01：😎  

题目：

```xml
给定一个不含重复元素的整数数组。一个以此数组构建的最大二叉树定义如下：
二叉树的根是数组中的最大元素。
左子树是通过数组中最大值左边部分构造出的最大二叉树。
右子树是通过数组中最大值右边部分构造出的最大二叉树。
通过给定的数组构建最大二叉树，并且输出这个树的根节点。

示例 ：
输入：[3,2,1,6,0,5]
输出：返回下面这棵树的根节点：

      6
    /   \
   3     5
    \    / 
     2  0   
       \
        1
```

分析：

按照我们刚才说的，先明确根节点做什么？**对于构造二叉树的问题，根节点要做的就是把想办法把自己构造出来**。

我们肯定要遍历数组把找到最大值`maxVal`，把根节点`root`做出来，然后对`maxVal`左边的数组和右边的数组进行递归调用，作为`root`的左右子树。

按照题目给出的例子，输入的数组为`[3,2,1,6,0,5]`，对于整棵树的根节点来说，其实在做这件事：

```java
TreeNode constructMaximumBinaryTree([3,2,1,6,0,5]) {
    // 找到数组中的最大值
    TreeNode root = new TreeNode(6);
    // 递归调用构造左右子树
    root.left = constructMaximumBinaryTree([3,2,1]);
    root.right = constructMaximumBinaryTree([0,5]);
    return root;
}
```

再详细一点，就是如下伪码：

```java
TreeNode constructMaximumBinaryTree(int[] nums) {
    if (nums is empty) return null;
    // 找到数组中的最大值
    int maxVal = Integer.MIN_VALUE;
    int index = 0;
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] > maxVal) {
            maxVal = nums[i];
            index = i;
        }
    }

    TreeNode root = new TreeNode(maxVal);
    // 递归调用构造左右子树
    root.left = constructMaximumBinaryTree(nums[0..index-1]);
    root.right = constructMaximumBinaryTree(nums[index+1..nums.length-1]);
    return root;
}
```

看懂了吗？**对于每个根节点，只需要找到当前`nums`中的最大值和对应的索引，然后递归调用左右数组构造左右子树即可**。



代码：

```java
class Solution {
    public TreeNode constructMaximumBinaryTree(int[] nums) {
    	return build(nums,0,nums.length-1);
    }

    // 将 nums[lo..hi] 构造成符合条件的树，返回根节点
    public TreeNode build(int[] nums,int low,int high){
    	if (low>high) return null;

		// 找到数组中的最大值和对应的索引
		int index = -1;
		int maxVal = Integer.MIN_VALUE;
		for (int i = low; i <=high; i++) { // ⚠️ 只能用<= 因为是要得到索引
			if (maxVal<nums[i]){
				index=i;
				maxVal = nums[i];
			}
		}

		TreeNode root = new TreeNode(maxVal);
		// 递归调用构造左右子树
		root.left = build(nums,low,index-1);
		root.right = build(nums,index+1,high);

		return root;
	}
```

---





## [\#669. 修剪二叉搜索树](https://leetcode-cn.com/problems/trim-a-binary-search-tree/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [#674. 最长连续递增序列](https://leetcode-cn.com/problems/longest-continuous-increasing-subsequence/)  

- Easy
- 2019.08.30：😭 

题目：

```xml
给定一个未经排序的整数数组，找到最长且连续的的递增序列，并返回该序列的长度。

示例 1:
输入: [1,3,5,4,7]
输出: 3
解释: 最长连续递增序列是 [1,3,5], 长度为3。
尽管 [1,3,5,7] 也是升序的子序列, 但它不是连续的，因为5和7在原数组里被4隔开。

注意：数组长度不会超过10000。
```

分析：动态规划

```xml
算法：
每个（连续）增加的子序列是不相交的，并且每当 nums[i-1]>=nums[i] 时，每个此类子序列的边界都会出现。当它这样做时，它标志着在 nums[i] 处开始一个新的递增子序列，我们将这样的 i 存储在变量 anchor 中。
例如，如果 nums=[7，8，9，1，2，3]，那么 anchor 从 0 开始（nums[anchor]=7），并再次设置为 anchor=3（nums[anchor]=1）。无论 anchor 的值如何，我们都会记录 i-anchor+1 的候选答案、子数组 nums[anchor]、nums[anchor+1]、…、nums[i] 的长度，并且我们的答案会得到适当的更新。

复杂度分析：
时间复杂度：O(N)O(N)，其中 NN 是 nums 的长度。我们通过 nums 执行一个循环。
空间复杂度：O(1)O(1)，anchor 和 ans 使用了常数级空间。
```

代码：

```java
class Solution {
    public int findLengthOfLCIS(int[] nums) {
        // nums[anchor] 处开始一个新的递增子序列
        // ans 当前子序列长度
        int ans = 0, anchor = 0;
        for (int i = 0; i < nums.length; i++) {
            // i=0时候，i>0为false就不会继续操作&&后的代码了
            if (i > 0 && nums[i-1] >= nums[i]) anchor = i;
            ans = Math.max(ans, i - anchor + 1);
        }
        return ans;
    }
}
```

---



## [\#671. 二叉树中第二小的节点](https://leetcode-cn.com/problems/second-minimum-node-in-a-binary-tree/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#677. 键值映射](https://leetcode-cn.com/problems/map-sum-pairs/)

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---







## [\#680. 回文字符串](https://leetcode-cn.com/problems/valid-palindrome-ii/description/) 

- Easy
- 2019.09.12：😭  
- 2019.10.13：😎

题目：

```xml
给定一个非空字符串 s，最多删除一个字符。判断是否能成为回文字符串。

示例 1:
输入: "aba"
输出: True

示例 2:
输入: "abca"
输出: True
解释: 你可以删除c字符。
```

分析：

```xml
方法一：双指针（简单易懂）
所谓的回文字符串，是指具有左右对称特点的字符串，例如 "abcba" 就是一个回文字符串。
使用双指针可以很容易判断一个字符串是否是回文字符串：令一个指针从左到右遍历，一个指针从右到左遍历，这两个指针同时移动一个位置，每次都判断两个指针指向的字符是否相同，如果都相同，字符串才是具有左右对称性质的回文字符串。
本题的关键是处理删除一个字符。在使用双指针遍历字符串时，如果出现两个指针指向的字符不相等的情况，我们就试着删除一个字符，再判断删除完之后的字符串是否是回文字符串。
在判断是否为回文字符串时，我们不需要判断整个字符串，因为左指针左边和右指针右边的字符之前已经判断过具有对称性质，所以只需要判断中间的子字符串即可。
在试着删除字符时，我们既可以删除左指针指向的字符，也可以删除右指针指向的字符。
复杂度分析：
时间复杂度O(n)，空间复杂度如果忽略递归的压栈信息，O(1)

方法二：贪心算法

```

代码：

```java
// 方法一：
class Solution {
  public boolean validPalindrome(String s) {
    // 最多删除一个字符
    // 在遇到不同的时候,移除一个元素（左/右）
    int i = 0;
    int j = s.length()-1;
    while(i<j){
      if (s.charAt(i)!=s.charAt(j)){
        // 在试着删除字符时，我们既可以删除左指针指向的字符(左边加一)，也可以删除右指针指向的字符(右边减一)
        return isPalindrome(s,i,j-1)||isPalindrome(s,i+1,j);
      }
      i++;
      j--;
    }
    return true;
  }
  
  // 判断是否是回文字符串
  // 在判断是否为回文字符串时，我们不需要判断整个字符串，因为左指针左边和右指针右边的字符之前已经判断过具有对称性质，所以只需要判断中间的子字符串即可
  public boolean isPalindrome(String s, int i, int j) {
    while (i < j) {
      if (s.charAt(i++) != s.charAt(j--)) {
        return false;
      }
    }
    return true;
  }
}
```



## [\#687. 最长同值路径](https://leetcode-cn.com/problems/longest-univalue-path/) 

- easy
- 2020.10.01：😭  

题目：

```xml

```

分析：

***方法一：***递归



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---





## [\#752. 打开转盘锁](https://leetcode-cn.com/problems/open-the-lock/)

- 中等
- 2020.11.25：😭  

题目：

```xml
你有一个带有四个圆形拨轮的转盘锁。每个拨轮都有10个数字： '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' 。每个拨轮可以自由旋转：例如把 '9' 变为  '0'，'0' 变为 '9' 。每次旋转都只能旋转一个拨轮的一位数字。
锁的初始数字为 '0000' ，一个代表四个拨轮的数字的字符串。
列表 deadends 包含了一组死亡数字，一旦拨轮的数字和列表里的任何一个元素相同，这个锁将会被永久锁定，无法再被旋转。
字符串 target 代表可以解锁的数字，你需要给出最小的旋转次数，如果无论如何不能解锁，返回 -1。

示例 1:
输入：deadends = ["0201","0101","0102","1212","2002"], target = "0202"
输出：6
解释：
可能的移动序列为 "0000" -> "1000" -> "1100" -> "1200" -> "1201" -> "1202" -> "0202"。
注意 "0000" -> "0001" -> "0002" -> "0102" -> "0202" 这样的序列是不能解锁的，
因为当拨动到 "0102" 时这个锁就会被锁定。
```

分析：

***方法一：***BFS

题目中描述的就是我们生活中常见的那种密码锁，若果没有任何约束，最少的拨动次数很好算，就像我们平时开密码锁那样直奔密码拨就行了。

但现在的难点就在于，不能出现 `deadends`，应该如何计算出最少的转动次数呢？

**第一步，我们不管所有的限制条件，不管** **`deadends`** **和** **`target`** **的限制，就思考一个问题：如果让你设计一个算法，穷举所有可能的密码组合，你怎么做**？

穷举呗，再简单一点，如果你只转一下锁，有几种可能？总共有 4 个位置，每个位置可以向上转，也可以向下转，也就是有 8 种可能对吧。

比如说从 `"0000"` 开始，转一次，可以穷举出 `"1000", "9000", "0100", "0900"...` 共 8 种密码。然后，再以这 8 种密码作为基础，对每个密码再转一下，穷举出所有可能...

**⭐️ 仔细想想，这就可以抽象成一幅图，每个节点有 8 个相邻的节点**，又让你求最短距离，这不就是典型的 BFS 嘛，框架就可以派上用场了，先写出一个「简陋」的 BFS 框架代码再说别的：

```java
	// BFS 框架，打印出所有可能的密码
    public int openLock(String[] deadends, String target) {
		LinkedList<String> q = new LinkedList<>();
		q.offer("0000");
		
		// 步数
		int step = 0;
		
		while(!q.isEmpty()){
			int size = q.size();
			for (int i = 0; i < size; i++) {
				String cur = q.poll();
				// 判断是否到终点
				System.out.println(cur);

				// 将一个节点的相邻节点加入队列 
				for (int j = 0; j < 4; j++) {
					String up=plusOne(cur,j);
					String down=minusOne(cur,j);
					q.offer(up);
					q.offer(down);
				}
			}
			// 增加步数
			step++;
		}
		return step;
	}

	// 将 s[j] 向上拨动一次
	private String plusOne(String s, int j) {
		char[] chars = s.toCharArray();
		if (chars[j]=='9'){ chars[j]='0';}else {chars[j]+=1;}
		return new String(chars);
	}

	// 将 s[i] 向下拨动一次
	private String minusOne(String s, int j) {
		char[] chars = s.toCharArray();
		if (chars[j]=='0'){ chars[j]='9';}else {chars[j]-=1;}
		return new String(chars);
	}
```

PS：这段代码当然有很多问题，但是我们做算法题肯定不是一蹴而就的，而是从简陋到完美的。不要完美主义，咱要慢慢来，好不。

**这段 BFS 代码已经能够穷举所有可能的密码组合了，但是显然不能完成题目，有如下问题需要解决**：

1、会走回头路。比如说我们从 `"0000"` 拨到 `"1000"`，但是等从队列拿出 `"1000"` 时，还会拨出一个 `"0000"`，这样的话会产生死循环。

2、没有终止条件，按照题目要求，我们找到 `target` 就应该结束并返回拨动的次数。

3、没有对 `deadends` 的处理，按道理这些「死亡密码」是不能出现的，也就是说你遇到这些密码的时候需要跳过。

如果你能够看懂上面那段代码，真得给你鼓掌，只要按照 BFS 框架在对应的位置稍作修改即可修复这些问题：

```java
class Solution {
	// BFS 框架，打印出所有可能的密码
    public int openLock(String[] deadends, String target) {

		// 记录需要跳过的死亡密码
		HashSet<String> deads = new HashSet<>();
		for (String deadend : deadends) {
			deads.add(deadend);
		}

		// 记录已经穷举过的密码，防止走回头路
		HashSet<String> visited = new HashSet<>();

		// 从起点开始启动广度优先搜索
		LinkedList<String> q = new LinkedList<>();
		q.offer("0000");
		visited.add("0000");

		// 步数
		int step = 0;
		
		while(!q.isEmpty()){
			int size = q.size();
			for (int i = 0; i < size; i++) {
				String cur = q.poll();
				// 判断是否到终点
				if (deads.contains(cur)) continue;
				if (cur.equals(target)) return step;

				// 将一个节点的相邻节点加入队列 
				for (int j = 0; j < 4; j++) {
					String up=plusOne(cur,j);
					if (!visited.contains(up)){
						q.offer(up);
						visited.add(up);
					}
					String down=minusOne(cur,j);
					if (!visited.contains(down)){
						q.offer(down);
						visited.add(down);
					}
				}
			}
			// 增加步数
			step++;
		}
		// 如果穷举完都没找到目标密码，那就是找不到了
		return -1;
	}

	// 将 s[j] 向上拨动一次
	private String plusOne(String s, int j) {
		char[] chars = s.toCharArray();
		if (chars[j]=='9'){ chars[j]='0';}else {chars[j]+=1;}
		return new String(chars);
	}

	// 将 s[i] 向下拨动一次
	private String minusOne(String s, int j) {
		char[] chars = s.toCharArray();
		if (chars[j]=='0'){ chars[j]='9';}else {chars[j]-=1;}
		return new String(chars);
	}


}
```

至此，我们就解决这道题目了。有一个比较小的优化：可以不需要 `dead` 这个哈希集合，可以直接将这些元素初始化到 `visited` 集合中，效果是一样的，可能更加优雅一些。



***方法二：双向 BFS 优化***

你以为到这里 BFS 算法就结束了？恰恰相反。BFS 算法还有一种稍微高级一点的优化思路：**双向 BFS**，可以进一步提高算法的效率。

篇幅所限，这里就提一下区别：**传统的 BFS 框架就是从起点开始向四周扩散，遇到终点时停止；而双向 BFS 则是从起点和终点同时开始扩散，当两边有交集的时候停止**。

为什么这样能够能够提升效率呢？其实从 Big O 表示法分析算法复杂度的话，它俩的最坏复杂度都是 `O(N)`，但是实际上双向 BFS 确实会快一些，我给你画两张图看一眼就明白了：

![image-20201125134747671](/assets/imgs/image-20201125134747671.png)

![image-20201125134757728](/assets/imgs/image-20201125134757728.png)

图示中的树形结构，如果终点在最底部，按照传统 BFS 算法的策略，会把整棵树的节点都搜索一遍，最后找到 `target`；而双向 BFS 其实只遍历了半棵树就出现了交集，也就是找到了最短距离。从这个例子可以直观地感受到，双向 BFS 是要比传统 BFS 高效的。

**不过，双向 BFS 也有局限，因为你必须知道终点在哪里**。比如我们刚才讨论的二叉树最小高度的问题，你一开始根本就不知道终点在哪里，也就无法使用双向 BFS；但是第二个密码锁的问题，是可以使用双向 BFS 算法来提高效率的，代码稍加修改即可：

```java
public int openLock(String[] deadends, String target) {

  HashSet<String> deads = new HashSet<>();
  for (String s : deadends) {
    deads.add(s);
  }
  // 用集合不用队列，可以快速判断元素是否存在
  HashSet<String> q1 = new HashSet<>();
  HashSet<String> q2 = new HashSet<>();
  HashSet<String> visited = new HashSet<>();

  int step = 0;
  q1.add("0000");
  q2.add(target);

  while (!q1.isEmpty()&&!q2.isEmpty()){
    // 哈希集合在遍历的过程中不能修改，用 temp 存储扩散结果
    HashSet<String> temp = new HashSet<>();

    /* 将 q1 中的所有节点向周围扩散 */
    for (String cur : q1) {
      /* 判断是否到达终点 */
      if (deads.contains(cur))
        continue;
      if (q2.contains(cur))
        return step;
      visited.add(cur);

      /* 将一个节点的未遍历相邻节点加入集合 */
      for (int j = 0; j < 4; j++) {
        String up = plusOne(cur, j);
        if (!visited.contains(up))
          temp.add(up);
        String down = minusOne(cur, j);
        if (!visited.contains(down))
          temp.add(down);
      }
    }
    /* 在这里增加步数 */
    step++;
    // temp 相当于 q1
    // 这里交换 q1 q2，下一轮 while 就是扩散 q2
    q1 = q2;
    q2 = temp;
  }
  return -1;
}
```

双向 BFS 还是遵循 BFS 算法框架的，只是**不再使用队列，而是使用 HashSet 方便快速判断两个集合是否有交集**。

另外的一个技巧点就是 **while 循环的最后交换** **`q1`** **和** **`q2`** **的内容**，所以只要默认扩散 `q1` 就相当于轮流扩散 `q1` 和 `q2`。

其实双向 BFS 还有一个优化，就是在 while 循环开始时做一个判断：

```java
// ...
while (!q1.isEmpty() && !q2.isEmpty()) {
    if (q1.size() > q2.size()) {
        // 交换 q1 和 q2
        temp = q1;
        q1 = q2;
        q2 = temp;
    }
    // ...
```

为什么这是一个优化呢？

因为按照 BFS 的逻辑，队列（集合）中的元素越多，扩散之后新的队列（集合）中的元素就越多；在双向 BFS 算法中，如果我们每次都选择一个较小的集合进行扩散，那么占用的空间增长速度就会慢一些，效率就会高一些。

不过话说回来，**无论传统 BFS 还是双向 BFS，无论做不做优化，从 Big O 衡量标准来看，时间复杂度都是一样的**，只能说双向 BFS 是一种 trick，算法运行的速度会相对快一点，掌握不掌握其实都无所谓。最关键的是把 BFS 通用框架记下来，反正所有 BFS 算法都可以用它套出解法。

---







## [#1051. 高度检查器](https://leetcode-cn.com/problems/height-checker/)  

- Easy
- 2019.08.28：😭  在对比一次后，忘记加arr[j]--；
- 2019.08.29：😎

题目：

```xml
学校在拍年度纪念照时，一般要求学生按照 非递减 的高度顺序排列。
请你返回能让所有学生以 非递减 高度排列的最小必要移动人数。
注意，当一组学生被选中时，他们之间可以以任何可能的方式重新排序，而未被选中的学生应该保持不动。
提示：1 <= heights.length <= 100
		 1 <= heights[i] <= 100
```

分析：桶排序

```xml
非递减 排序也就是升序排列，最直观的一种解法就是排序后对比计数每个位置的不同数量。
但是涉及到比较排序，时间复杂度最低也有 O(NlogN)O(NlogN)。

我们真的需要排序吗？
首先我们其实并不关心排序后得到的结果，我们想知道的只是在该位置上，与最小的值是否一致
题目中已经明确了值的范围 1 <= heights[i] <= 100
这是一个在固定范围内的输入，比如输入： [1,1,4,2,1,3]
输入中有 3 个 1,1 个 2，1 个 3 和 1 个 4，  3 个 1 肯定会在前面，依次类推
所以，我们需要的仅仅只是计数而已
```

代码：

```java
public int heightChecker(int[] heights) {
  			//1 设置100个桶，用于放入身高
        // 值的范围是1 <= heights[i] <= 100，因此需要1,2,3,...,99,100，共101个桶,0号桶不用
        int[] arr = new int[101];
        // 遍历数组heights，计算每个桶中有多少个元素，也就是数组heights中有多少个1，多少个2，...
        for (int i = 0; i < heights.length; i++) {
            arr[heights[i]]++;
        }

        //2 排序按桶的顺序依次取出(将这101个桶中的元素顺序桶地取出来，元素就是有序的)，与原数组做对比
        int j = 1;
        int count = 0;
        // 每个height必须比到一个非0值
        for (int height : heights) {
            while (arr[j] <= 0) {
                j++;
            }
            if (height != j) count++;
          	// 2019.08.28：😭 匹配到非0值后需要移除一个桶元素
            arr[j]--;
        }
        return count;
    }
```

---

## [#1160. 拼写单词](https://leetcode-cn.com/problems/find-words-that-can-be-formed-by-characters/) 

- Easy
- 2019.08.30：😭 

题目：

```xml
给你一份『词汇表』（字符串数组） words 和一张『字母表』（字符串） chars。
假如你可以用 chars 中的『字母』（字符）拼写出 words 中的某个『单词』（字符串），那么我们就认为你掌握了这个单词。
注意：每次拼写（指拼写词汇表中的一个单词）时，chars 中的每个字母都只能用一次。
返回词汇表 words 中你掌握的所有单词的 长度之和。

示例 1：
输入：words = ["cat","bt","hat","tree"], chars = "atach"
输出：6
解释： 
可以形成字符串 "cat" 和 "hat"，所以答案是 3 + 3 = 6。
```

分析：

```xml
这是一类经典的题型。凡是和“变位词”、“字母顺序打乱”相关的题目，都考虑统计字母出现的次数。这种方法我叫做 “counter 方法”。
我们既统计“字母表”中字母出现的次数，也统计单词中字母出现的次数。如果单词中每种字母出现的次数都小于等于字母表中字母出现的次数，那么这个单词就可以由字母表拼出来。
如何实现计数结构呢？一般的方法是用 Java 的 HashMap。但是我们注意到题目有一个额外的条件：所有字符串中都仅包含小写英文字母。这意味着我们可以用一个长度为 26 的数组来进行计数。
```

![5cda24dc2d60dba242e50c7c9c2e008e3856d19d28ec14b6ed48292aaf0ee526](/assets/imgs/5cda24dc2d60dba242e50c7c9c2e008e3856d19d28ec14b6ed48292aaf0ee526.gif)

代码：

```java
public int countCharacters(String[] words, String chars) {
    int[] chars_count = count(chars); // 统计字母表的字母出现次数
    int res = 0; // 所有可用单词长度之和
    for (String word : words) {
        int[] word_count = count(word); // 统计单词的字母出现次数
        if (contains(chars_count, word_count)) {
            res += word.length();
        }
    }
    return res;
}

// 检查字母表的字母出现次数是否覆盖单词的字母出现次数
boolean contains(int[] chars_count, int[] word_count) {
    for (int i = 0; i < 26; i++) {
        if (chars_count[i] < word_count[i]) {
            return false;
        }
    }
    return true;
}

// 统计 26 个字母出现的次数,用一个26位的数组表示
int[] count(String word) {
    int[] counter = new int[26];
    for (int i = 0; i < word.length(); i++) {
        char c = word.charAt(i);
        counter[c-'a']++;
    }
    return counter;
}
```



# 模版.

### 题号 

- 中等
- 2020.11.25：😭  

题目：

```xml

```

分析：

***方法一：***



- 时间复杂度：O()
- 空间复杂度：O()



代码：

```java

```

---

