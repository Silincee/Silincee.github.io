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
>
> repo：https://github.com/labuladong/fucking-algorithm

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

### [#1051 高度检查器](https://leetcode-cn.com/problems/height-checker/)  $1/2$

> 2019.08.28：😭  在对比一次后，忘记加上arr[j]--
>
> 2019.08.29：😎

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

### [#674 最长连续递增序列](https://leetcode-cn.com/problems/longest-continuous-increasing-subsequence/) 

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

### [#26 删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

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



### [#88 合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/)

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



### [#1160 拼写单词](https://leetcode-cn.com/problems/find-words-that-can-be-formed-by-characters/)

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





## 2. String

## 3. Tree

## 4. LinkedList

## 5. Math



# 排序算法

![image-20200720125956366](/assets/imgs/image-20200720125956366.png)

[十大排序算法](https://mp.weixin.qq.com/s/Qf416rfT4pwURpW3aDHuCg)

[排序算法的复杂度、实现和稳定性](https://www.jianshu.com/p/916b15eae350)



# Ex.

题目：

```xml

```

分析：

```xml

```

代码：

```java

```

