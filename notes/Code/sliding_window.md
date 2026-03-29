---
layout: default
title: "算法 滑动窗口"
date: 2026-03-29
tags: [编程, 算法]
---

# 滑动窗口
滑动窗口形式上是一种双指针法，思路是在**统计**（一般是求总和）某一变化的**连续区间的信息**（有些题目直接说就是个窗口）时，利用只有首末元素有变化，从而防止重新计算整个区间

关键词： 
- 连续区间（不连续可能要排序变为连续的）
- 子字符串、子区间（暗含连续）

## 定长滑动窗口
示例：
```
给你一个由 n 个元素组成的整数数组 nums 和一个整数 k 。
请你找出平均数最大且 长度为 k 的连续子数组，并输出该最大平均数。

示例 ：
输入：nums = [1,12,-5,-6,50,3], k = 4
输出：12.75
解释：最大平均数 (12-5-6+50)/4 = 51/4 = 12.75

提示：
n == nums.length
1 <= k <= n <= 10^5
-10^4 <= nums[i] <= 10^4
```
```cpp
class Solution {
public:
    double findMaxAverage(vector<int>& nums, int k) {
        int res = INT_MIN, tmp = 0;
        //因为有负数，res要设置为最小可能结果

        for (int l = 0, r = 0; r < nums.size(); r++) {
            tmp += nums[r];
            //入

            if (r - l + 1 < k) continue;
            //定长窗口是否形成？

            if (tmp > res) res = tmp;
            //更新结果

            tmp -= nums[l++];
            //出
        }
        return (double)res / k;
    }
};
```

入-更新答案-出
对于不同题目可能有一些额外处理
1. sum含负数，res设置为`INT_MIN`（如果res = 0，对于全是负数等情况就不行了）
2. 如果不形成窗口，结果数组值为-1 -> 先全填充-1，再单独考虑可行的情况（特殊情况特殊考虑）
3. 更新条件，可能还要满足其他条件，不只是sum更大即可

## 不定长滑动窗口
框架：
- 右指针右移，尽量包含所有可能的元素（要遍历整个数组）
- 左指针右移，缩小包含范围以尽可能重新符合条件
- 满足约束时，更新答案

### 越短越合法/求最长/最大
越短越合法的意思是，子序列越短，越容易符合题目的约束（无重复字符、最多有一个0、最多有k个例外情况等等）
在这个约束条件下，要求出最长的子序列

关键就是**找约束和求什么**

#### 示例1
力扣3
```
给定一个字符串 s ，请你找出其中不含有重复字符的 最长 子串 的长度。

示例 1:
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。注意 "bca" 和 "cab" 也是正确答案。
```
```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int ans = 0, left = 0;
        unordered_map<char, int> cnt;
        for (int right = 0; right < s.length(); right++) {
            char c = s[right]; // 右指针右移，扩大
            cnt[c]++;
            while (cnt[c] > 1) { // 不满足约束
                cnt[s[left]]--; // 左指针右移，移除窗口左端点字母直到满足
                left++;
            }
            ans = max(ans, right - left + 1); // 更新最大的答案
        }
        return ans;
    }
};
```
**本题约束条件：无重复字母**
子串越短越容易满足该条件，要找最长的

#### 示例 2
力扣1208
```
给你两个长度相同的字符串，s 和 t。
将 s 中的第 i 个字符变到 t 中的第 i 个字符需要 |s[i] - t[i]| 的开销（开销可能为 0），也就是两个字符的 ASCII 码值的差的绝对值。
用于变更字符串的最大预算是 maxCost。在转化字符串时，总开销应当小于等于该预算，这也意味着字符串的转化可能是不完全的。
如果你可以将 s 的子字符串转化为它在 t 中对应的子字符串，则返回可以转化的最大长度。
如果 s 中没有子字符串可以转化成 t 中对应的子字符串，则返回 0。

示例 1：
输入：s = "abcd", t = "bcdf", maxCost = 3
输出：3
解释：s 中的 "abc" 可以变为 "bcd"。开销为 3，所以最大长度为 3。

示例 2：
输入：s = "abcd", t = "cdef", maxCost = 3
输出：1
解释：s 中的任一字符要想变成 t 中对应的字符，其开销都是 2。因此，最大长度为 1。

示例 3：
输入：s = "abcd", t = "acde", maxCost = 0
输出：1
解释：a -> a, cost = 0，字符串未发生变化，所以最大长度为 1。
```
```cpp
class Solution {
public:
    int equalSubstring(string s, string t, int maxCost) {
        int max = 0, cost = 0;
        for (int l = 0, r = 0; r < s.length(); r++) {
            cost += abs(t[r] - s[r]); //右指针右移，扩大
            while (cost > maxCost) { //如果不满足，缩小
                cost -= abs(t[l] - s[l]);
                l++;
            }
            if (r - l + 1 > max) max = r - l + 1; //更新最大的答案
        }
        return max;
    }
};
```
**约束条件：花费的总cost不能超过上限maxCost**
子字符串是**连续**的，且越短花费的cost越少，要找最长的

#### 几个特殊情况的优化
力扣1695
```
给你一个正整数数组 nums ，请你从中删除一个含有 若干不同元素 的子数组。删除子数组的 得分 就是子数组各元素之 和 。
返回 只删除一个 子数组可获得的 最大得分 。
如果数组 b 是数组 a 的一个连续子序列，即如果它等于 a[l],a[l+1],...,a[r] ，那么它就是 a 的一个子数组。

示例 1：
输入：nums = [4,2,4,5,6]
输出：17
解释：最优子数组是 [2,4,5,6]
示例 2：
输入：nums = [5,2,1,2,5,2,1,2,5]
输出：8
解释：最优子数组是 [5,2,1] 或 [1,2,5]
```
```cpp
class Solution {
public:
    int maximumUniqueSubarray(vector<int>& nums) {
        unordered_set<int> st;
        int ans = 0, s = 0, left = 0;
        for (int x : nums) { //why no r ptr?
            while (st.contains(x)) { //why this first instead of insert(~ r ptr move)
                st.erase(nums[left]);
                s -= nums[left];
                left++;
            }
            st.insert(x);
            s += x;
            ans = max(ans, s);
        }
        return ans;
    }
};
```
题解：[灵茶山艾府](https://leetcode.cn/problems/maximum-erasure-value/solutions/3720052/hua-dong-chuang-kou-bu-er-shu-zu-ha-xi-j-rsvy/)

1. 为什么不用r指针？因为这一题不需要知道子数组的**长度**，只要维护sum，既然r指针的功能只剩下一个个遍历，和for-each是一样的
2. 为什么先判断满足条件/l指针右移 再r指针右移？因为这题使用了set，如果不判断新进入元素在不在集合中就直接添加，后续左指针右移删除元素的时候会误判（这样就只能用哈希表）

### 越长越合法/求最短/最小
本质上和越长越合法的题目时是一样的，只不过约束条件变了，更新答案方式也会改变

力扣209：
```
给定一个含有 n 个正整数的数组和一个正整数 target 。
找出该数组中满足其总和大于等于 target 的长度最小的 子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。

示例 1：
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。
示例 2：
输入：target = 4, nums = [1,4,4]
输出：1
示例 3：
输入：target = 11, nums = [1,1,1,1,1,1,1,1]
输出：0
```
这题要找总和大于target的最短子数组，**约束是总和大于target**，子数组越长显然越满足约束条件
```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int sum = 0, res = INT_MAX; //res也可以初始化设置为nums.size()+1
        for (int l = 0, r = 0; r < nums.size(); r++) {
            sum += nums[r]; //右指针，扩大

            //扩大的过程中肯定是更满足约束了，所以左指针右移的目的是优化答案找最短的（类似之前右指针右移，目的是找到最长的子串）
            while (sum - nums[l] >= target) { //如果可以，尽可能缩小
                sum -= nums[l];
                l++;
            }

            if (sum >= target) res = min(res,r - l + 1); //满足条件的话，更新答案
        }
        return res == INT_MAX ? 0 : res;
    }
};

//上面这种做法比较符合求最长的框架
//也可以把更新答案写在while里面，而且，大部分求最短的题目只能这样写
class Solution2 {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int sum = 0, ans = INT_MAX;
        for (int left = 0, right = 0; right < nums.size(); right++) {
            sum += nums[right]; //入

            while (sum >= target) {
                ans = min(ans, right - left + 1); //while已经确保满足，立刻更新答案
                sum -= nums[left];
                left++; // 左端点右移
            }
        }
        return ans == INT_MAX ? 0 : ans;
    }
};
```

## 一些非寻常滑窗的例子

### 示例1/2
力扣1423：
```
几张卡牌 排成一行，每张卡牌都有一个对应的点数。点数由整数数组 cardPoints 给出。
每次行动，你可以从行的开头或者末尾拿一张卡牌，最终你必须正好拿 k 张卡牌。
你的点数就是你拿到手中的所有卡牌的点数之和。
给你一个整数数组 cardPoints 和整数 k，请你返回可以获得的最大点数。

示例 1：
输入：cardPoints = [1,2,3,4,5,6,1], k = 3
输出：12
解释：第一次行动，不管拿哪张牌，你的点数总是 1 。但是，先拿最右边的卡牌将会最大化你的可获得点数。最优策略是拿右边的三张牌，最终点数为 1 + 6 + 5 = 12
```
这题可以枚举可能的首尾拿牌情况，由于参与计算**总分**的是一个**连续区间**，可以利用滑窗思想快速得到区间变化时，总分的变化



和刚刚那题类似，也是首尾取数字，但是刚刚那题是已知拿数字的总数，相当于定长滑窗，而下面这题是知道总和
力扣1658：
```
给你一个整数数组 nums 和一个整数 x 。每一次操作时，你应当移除数组 nums 最左边或最右边的元素，然后从 x 中减去该元素的值。请注意，需要 修改 数组以供接下来的操作使用。
如果可以将 x 恰好 减到 0 ，返回 最小操作数 ；否则，返回 -1 。

示例 1：
输入：nums = [1,1,4,2,3], x = 5
输出：2
解释：最佳解决方案是移除后两个元素，将 x 减到 0 。
示例 2：
输入：nums = [5,6,7,8,9], x = 4
输出：-1
示例 3：
输入：nums = [3,2,20,1,1,3], x = 10
输出：5
解释：最佳解决方案是移除后三个元素和前两个元素（总共 5 次操作），将 x 减到 0 。
```
[灵神题解 滑动窗口/双指针](https://leetcode.cn/problems/minimum-operations-to-reduce-x-to-zero/)
这题当然可以直接做，不过情况就比较麻烦，**正难则反**，与其判断首位的总和，不如考虑删掉之后剩余的数字总和，剩余的部分还是**连续**的
- 首位部分目标总和是x，数组总和固定sum，所以剩余部分的目标和就是sum - x
- 最佳解决方案是最小操作数，对于剩余部分来说就是**找最长**
- 约束条件是满足总和，越长越不容易满足，这是个不定长滑窗

```cpp
//根据灵神的代码自己写的
class Solution {
public:
    int minOperations(vector<int>& nums, int x) {
        int target = accumulate(nums.begin(),nums.end(),0) - x;
        if (target < 0) return -1;
        if (target == 0) return nums.size();

        int sum = 0, res = 0;
        for (int l = 0, r = 0; r < nums.size(); r++) {
            sum += nums[r];
            while (sum > target) {
                sum -= nums[l];
                l++;
            }
            if (sum == target) res = max(r - l + 1,res); //注意，只有满足约束的情况才能更新答案
            //因为之前这样的约束都是不等式，在while处已经保证处理好了，到更新答案时肯定是满足的；而这里是等式

            //不要写if (sum == target) break; 滑窗要让右指针遍历整个数组才能找到最优解
        }
        return res == 0 ? -1 : nums.size() - res;
        //如果res一直是0，说明没有满足过约束条件，返回-1（其实把res初始化为-1更好）
    }
};
```

### 示例3
很多时候原问题不是简单的滑窗，而是需要加工一下：
力扣3439：
```
给你一个整数 eventTime 表示一个活动的总时长，这个活动开始于 t = 0 ，结束于 t = eventTime 。
同时给你两个长度为 n 的整数数组 startTime 和 endTime 。它们表示这次活动中 n 个时间 没有重叠 的会议，其中第 i 个会议的时间为 [startTime[i], endTime[i]] 。
你可以重新安排 至多 k 个会议，安排的规则是将会议时间平移，且保持原来的 会议时长 ，你的目的是移动会议后 最大化 相邻两个会议之间的 最长 连续空余时间。
移动前后所有会议之间的 相对 顺序需要保持不变，而且会议时间也需要保持互不重叠。
请你返回重新安排会议以后，可以得到的 最大 空余时间。
注意，会议 不能 安排到整个活动的时间以外。

示例 2：
输入：eventTime = 10, k = 1, startTime = [0,2,9], endTime = [1,4,10]
输出：6
解释：
将 [2, 4] 的会议安排到 [1, 3] ，得到空余时间 [3, 9] 。
```
<figure>
  <img src="{{ '/assets/images/slidingWindow_pic.webp' | absolute_url }}" alt="example1" />
</figure>
能移动k次，每次移动都可以合并两处**相邻（隔着一个会议）**的空闲时间，也就是可以把k+1个空闲时间段**计入总数**
这题要先把给的会议时间数据加工成一个所有空闲时间的数组，这个例子里，就是[1,5]，k=1，故纳入计算的长度为2，返回1+5
转换之后，就是只需要对空闲时间数组用滑窗即可

参考答案，我自己写的，非最佳：
```cpp
class Solution {
public:
    static int maxFreeTime(int eventTime, int k, vector<int>& startTime, vector<int>& endTime) {
        vector<int> gap;
        gap.push_back(startTime.front() - 0);
        for (int i = 1; i < startTime.size(); i++) {
            gap.push_back(startTime[i] - endTime[i-1]);
        }
        gap.push_back(eventTime - endTime.back());
        print(gap);

        int tmp = 0, max = gap.front();
        for (int l = 0, r = 0; r < gap.size(); r++) {
            tmp += gap[r];
            if (r - l < k) continue;
            if (tmp > max) max = tmp;
            tmp -= gap[l++];
        }
        return max;
    }
};
```