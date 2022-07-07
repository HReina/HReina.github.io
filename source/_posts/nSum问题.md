---
title: nSum问题
date: 2022-03-26 17:13:48
tags: 
      - c++
      - 算法
categories: 
      - 算法
      - nSum问题
comments: true
---

> ​                       前一篇文章对2Sum问题进行了回顾，本文由浅入深对nSum问题进行分析总结。

<!-- more -->

#### 2Sum问题

> 先看一个基础的2Sum问题，一个数组nums和一个目标值，nums中可能有多对元素之和等于目标值target，请返回所有和为target的元素对，不能出现重复。

```c++
vector<vector<int>> towSumTarget(vector<int>& nums, int target){
    // 先对数组进行排序
    sort(nums.begin(), nums,end());
    vector<vector<int>> res;
    int left=0, right=nums.size()-1;
    while(lfet<right){
        int sum=nums[left]+nums[right];
        //记录索引left和right最初对应的值
        int temp_left=nums[left], temp_right=nums[right];
        if(sum<target) left++;
        else if(sum>target) right++;
        else{
            res.push_back({left,right});
            //跳过所有重复元素
            while(left<right && nums[left]==temp_left) left++;
            while(left<right && nums[right]==temp_right) right++;
        }
    }
    return res;
}
```

#### 3Sum问题

> 输入一个数组，判断其中是否存在三个元素a,b,c使得a+b+c=target，如果有请返回满足所有条件且不重复的三元组。

```c++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        //求和为0的三元组
        return threeSum(nums, 0);
    }
    
    //求目标值为target的三数之和
    vector<vector<int>> threeSum(vector<int>& nums, int target){
        //先对整个数组进行排序
        sort(nums.begin(), nums.end());
        int n=nums.size();
        vector<vector<int>> res;
        //穷举三个数的第一个数
        for(int i=0;i<n;i++){
            //对target-nums[i]计算twoSum
            vector<vector<int>> tuples=twoSum(nums, i+1, target-nums[i]);
            //如果存在满足条件的二元组，再加上nums[i]就是最终结果三元组
            for(vector<int>& tuple : tuples){
                tuple.push_back(nums[i]);
                res.push_back(tuple);
            }
            //跳过第一个数字重复的情况，否则会出现重复结果
            while(i<n-1 && nums[i]==nums[i+1]) i++;
        }
        return res;
    }
    vector<vector<int>> twoSum(vector<int>& nums, int start, int target){
        vector<vector<int>> res;
        int left=start, right=nums.size()-1;
        while(left<right){
            int sum=nums[left]+nums[right];
            //记录索引left和right最初对应的值
            int left_temp=nums[left], right_temp=nums[right];
            if(sum>target) right--;
            else if(sum<target) left++;
            else{
                res.push_back({left_temp, right_temp});
                //跳过所有重复元素
                while(left<right && nums[left]==left_temp) left++;
                while(left<right && nums[right]==right_temp) right--;
            }
        }
        return res;
    }
};
```

#### 4Sum问题

> 穷举第一个数字，然后调用3Sum函数计算剩下的三个数，最后进行合并得出结果。

> 思路：
>
> 调用 `nSum` 求解。

```c++
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        //先排序
        sort(nums.begin(), nums.end());
        //调用nSum函数
        return nSum(nums, 4, 0, target);
    }
    //求nSum通用递归解法
    vector<vector<int>> nSum(vector<int>& nums, int n, int start, int target){
        int sz=nums.size();
        vector<vector<int>> res;
        //至少是2Sum,且数组大小不能小于n
        if(n<2 || sz<n) return res;
        //base case 是2Sum
        if(n==2){
            int left=start, right=sz-1;
            while(left<right){
                int sum=nums[left]+nums[right];
                int l_temp=nums[left], r_temp=nums[right];
                if(sum<target) left++;
                else if(sum>target) right--;
                else{
                    res.push_back({l_temp, r_temp});
                    //消除重复
                    while(left<right && l_temp==nums[left]) left++;
                    while(left<right && r_temp==nums[right]) right--;
                }
            }
        }else{
            //当n>2时，递归计算(n-1)Sum的结果
            for(int i=start;i<sz;i++){
                //对target-nums[i]计算(n-1)Sum
                vector<vector<int>> subs=nSum(nums, n-1, i+1, target-nums[i]);
                for(vector<int>& sub : subs){
                    //(n-1)Sum加上nums[i]就是nSum
                    sub.push_back(nums[i]);
                    res.push_back(sub);
                }
                //消除重复元素
                while(i<sz-1 && nums[i]==nums[i+1]) i++;
            }
        }
        return res;
    }
};
```

#### nSum问题

```c++
class Solution {
public:
    vector<vector<int>> 100Sum(vector<int>& nums, int target) {
        //调用nSum之前需要将原数组进行排序
        sort(nums.begin(), nums.end());
        //调用nSum函数
        return nSum(nums, 100, 0, target);
    }
    
    //求nSum通用递归解法
    vector<vector<int>> nSum(vector<int>& nums, int n, int start, int target){
        int sz=nums.size();
        vector<vector<int>> res;
        //至少是2Sum,且数组大小不能小于n
        if(n<2 || sz<n) return res;
        //base case 是2Sum
        if(n==2){
            int left=start, right=sz-1;
            while(left<right){
                int sum=nums[left]+nums[right];
                int l_temp=nums[left], r_temp=nums[right];
                if(sum<target) left++;
                else if(sum>target) right--;
                else{
                    res.push_back({l_temp, r_temp});
                    //消除重复
                    while(left<right && l_temp==nums[left]) left++;
                    while(left<right && r_temp==nums[right]) right--;
                }
            }
        }else{
            //当n>2时，递归计算(n-1)Sum的结果
            for(int i=start;i<sz;i++){
                //对target-nums[i]计算(n-1)Sum
                vector<vector<int>> subs=nSum(nums, n-1, i+1, target-nums[i]);
                for(vector<int>& sub : subs){
                    //(n-1)Sum加上nums[i]就是nSum
                    sub.push_back(nums[i]);
                    res.push_back(sub);
                }
                //消除重复元素
                while(i<sz-1 && nums[i]==nums[i+1]) i++;
            }
        }
        return res;
    }
};
```
