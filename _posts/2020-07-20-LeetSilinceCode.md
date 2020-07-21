---
layout: post
title:  "LeetSilinceCode"
date:   2020-07-20 13:20:06 +0800--
categories: [面试]
tags: [LeetCode, ]  

---

# PLAN

> leetcode: https://leetcode-cn.com/tag/array/
>
> 刷题模版： https://greyireland.gitbook.io/algorithm-pattern/	

```
我的方法只适合连数据结构都不扎实的菜鸡选手～不要完全按tag！头一次刷，先把这五个tag做了：array，string，tree，linkedlist，math，其它的千万别按tag刷。这样不存在前面答案说的思维暗示问题，反而帮助巩固数据结构，还可以自己归纳某种数据结构的全部技巧～ 每个tag内部就按照easy-medium-hard的顺序做，这样最开始一天刷10道easy，后面熟了这个数据结构一天也能刷5道难题，不会一开始就卡壳，搞得自己很郁闷。这时候已经100多道了，之后从hard往easy刷！前面虐虐虐，后面一天20道easy爽歪歪，很快就刷完啦！赶快买个会员开始第二遍吧！
```

## **刷题策略**：

**时间紧迫**：

- 在 [https://leetcode-cn.com/problemset/all/](https://link.zhihu.com/?target=https%3A//leetcode-cn.com/problemset/all/) 页面的右侧。先刷热题 HOT 100，再刷精选 TOP 面试题，之后刷其他的题。

**如果你时间比较充裕，那我建议你：**

- 按从低到高的难度分组刷
- 按 tag 分类刷
- 定期复习，重做之前刷过的题

掌握 LeetCode 刷题方法再开始刷题，属于磨刀不误砍柴工。掌握正确方法是非常重要的。

如果你在刷题的时候发现怎么也写不出来，别担心，这是正常的。

如果你还发现，之前明明刷过的题，过段时间再做的时候，自己还是不会。别担心，这也是正常的。

## **刷题方法：**

- 第一遍：可以先思考，之后看参考答案刷，结合其他人的题解刷。思考、总结并掌握本题的类型，思考方式，最优题解。
- 第二遍：先思考，回忆最优解法，并与之前自己写过的解答作比对，总结问题和方法。
- 第三遍：提升刷题速度，拿出一个题，就能够知道其考察重点，解题方法，在短时间内写出解答。

## **定期总结：**

- 按照题目类型进行总结：针对一类问题，总结有哪些解题方法，哪种方法是最优的，为什么。
- 总结重点：有些题你刷了好多遍都还是不会，那就要重点关注，多思考解决方法，不断练习强



# TAG

## 1. Array

> https://leetcode-cn.com/tag/array/

### #1051 高度检查器

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
        // 值的范围是1 <= heights[i] <= 100，因此需要1,2,3,...,99,100，共101个桶
        int[] arr = new int[101];
        // 遍历数组heights，计算每个桶中有多少个元素，也就是数组heights中有多少个1，多少个2，...，多少个100
        // 将这101个桶中的元素，一个一个桶地取出来，元素就是有序的
        for (int height : heights) {
            arr[height]++;
        }

        int count = 0;
        for (int i = 1, j = 0; i < arr.length; i++) {
            // arr[i]，i就是桶中存放的元素的值，arr[i]是元素的个数
            // arr[i]-- 就是每次取出一个，一直取到没有元素，成为空桶
            while (arr[i]-- > 0) {
                // 从桶中取出元素时，元素的排列顺序就是非递减的，然后与heights中的元素比较，如果不同，计算器就加1
                if (heights[j++] != i) count++;
            }
        }
        return count;
    }
```

### #674 最长连续递增序列

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
        int ans = 0, anchor = 0;
        for (int i = 0; i < nums.length; ++i) {
            if (i > 0 && nums[i-1] >= nums[i]) anchor = i;
            ans = Math.max(ans, i - anchor + 1);
        }
        return ans;
    }
}
```





## 2. String

## 3. Tree

## 4. LinkedList

## 5. Math



# 排序算法

![image-20200720125956366](/assets/imgs/image-20200720125956366.png)

[十大排序算法](https://mp.weixin.qq.com/s/Qf416rfT4pwURpW3aDHuCg)

[排序算法的复杂度、实现和稳定性](https://www.jianshu.com/p/916b15eae350)

