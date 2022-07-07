---
title: 2Sum问题的核心思想
date: 2022-03-26 17:11:52
tags: 
      - c++
      - 算法
categories: 
      - 算法
      - nSum问题
comments: true
---

> ​                                              本文简单回顾总结了一下2Sum系列问题的常见解法。

<!-- more -->

#### 2Sum I

> 描述：输入一个数组nums和一个整数target，保证数组中存在两个数的和为target，返回他们的索引。

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        //暴力穷举
        for(int i=0;i<nums.size();i++){
            for(int j=i+1;j<nums.size();j++){
                if(nums[i]+nums[j]==target){
                    return {i,j};
                }
            }
        }
        return {-1,-1};
    }
};
```

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        int n=nums.size();
        //构造哈希表，进行元素和索引的相关映射
        unordered_map<int,int> map;
        for(int i=0;i<n;i++){
            map[nums[i]]=i;
        }
        for(int i=0;i<n;i++){
            int temp=target-nums[i];
            //如果temp存在而且不是nums[i]本身
            if(map.count(temp) && map[temp]!=i){
                return {i,map[temp]};
            }
        }
        return{-1, -1};
    }
};
```

#### 2Sum II

> 描述：给一个下标从 **1** 开始的整数数组 nums，该数组已按 非递减顺序排列。

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size()-1;
        while(left<=right){
            int sum = nums[left] + nums[right];
            if(sum == target) return {left+1,right+1}; //注意小标从1开始
            else if(sum > target) right--;
            else if(sum < target) left++;
        }
        return {-1,-1};
    }
};
```