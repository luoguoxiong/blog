# 动态划窗

#### [长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

> 给定一个含有 n 个正整数的数组和一个正整数 target 。
>
> 找出该数组中满足其和 ≥ target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。
>
> 示例 1：
>
> 输入：target = 7, nums = [2,3,1,2,4,3]
> 输出：2
> 解释：子数组 [4,3] 是该条件下的长度最小的子数组。

```js
var minSubArrayLen = function(target, nums) {
  let res = Number.MAX_VALUE
  let left = 0;
  let sums = 0;
  for(let startIndex = 0;startIndex < nums.length;startIndex++){
    sums += nums[startIndex];
    while(sums >= target){
      res = Math.min(res, startIndex - left + 1);
      sums -= nums[left++];
    }
  }
  return res > nums.length ? 0 : res;
};
```

