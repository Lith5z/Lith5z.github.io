---
layout: default
title: "算法 滑动窗口"
date: 2026-03-15
tags: [编程, 算法]
---

滑动窗口形式上是一种双指针法，思路是在**统计**（一般是求总和）某一变化的**连续区间的信息**（有些题目直接说就是个窗口）时，只有首末元素发生了变化，防止重新计算整个区间

## 定长滑动窗口
示例：
```
给你一个由 n 个元素组成的整数数组 nums 和一个整数 k 。
请你找出平均数最大且 长度为 k 的连续子数组，并输出该最大平均数。

示例 1：
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
右指针右移，尽量包含所有可能的元素
左指针右移，缩小包含范围以重新符合条件

### 越短越合法/求最长/最大
示例：
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
        int n = s.length(), ans = 0, left = 0;
        unordered_map<char, int> cnt;
        for (int right = 0; right < n; right++) {
            char c = s[right]; // 入
            cnt[c]++;
            while (cnt[c] > 1) { // 窗口内有重复字母
                cnt[s[left]]--; // 移除窗口左端点字母直到满足
                left++; // 缩小窗口
            }
            ans = max(ans, right - left + 1); // 更新
        }
        return ans;
    }
};
```

入-不满足条件就一直缩小-更新答案

## 一些非寻常滑窗的例子
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
  <img src="{{ '/assets/images/sliding_window_pic.png' | absolute_url }}" alt="example1" />
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