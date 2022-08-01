# 动态规划


### 最长递增子序列
给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。

子序列是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。

示例 1： 输入：nums = [10,9,2,5,3,7,101,18] 输出：4 解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。

示例 2： 输入：nums = [0,1,0,3,2,3] 输出：4

示例 3： 输入：nums = [7,7,7,7,7,7,7] 输出：1

提示：

1 <= nums.length <= 2500
-10^4 <= nums[i] <= 104

```js
var lengthOfLIS = function(nums) {
  let max = 1;
  // db[i] 表示前i项最长递增子序列
  const dp = new Array(nums.length).fill(1);
  for(let i = 1;i < nums.length;i++){
    for(let j = 0;j < i;j++){
      if(nums[i] > nums[j]){
        dp[i] = Math.max(db[i], db[j] + 1);
      }
    }
    max = Math.max(max, db[i]);
  }
  return max;
};
```

### 最长连续递增子序列
给定一个未经排序的整数数组，找到最长且 连续递增的子序列，并返回该序列的长度。

连续递增的子序列 可以由两个下标 l 和 r（l < r）确定，如果对于每个 l <= i < r，都有 nums[i] < nums[i + 1] ，那么子序列 [nums[l], nums[l + 1], ..., nums[r - 1], nums[r]] 就是连续递增子序列。

示例 1： 输入：nums = [1,3,5,4,7] 输出：3 解释：最长连续递增序列是 [1,3,5], 长度为3。 尽管 [1,3,5,7] 也是升序的子序列, 但它不是连续的，因为 5 和 7 在原数组里被 4 隔开。

示例 2： 输入：nums = [2,2,2,2,2] 输出：1 解释：最长连续递增序列是 [2], 长度为1。

提示：

0 <= nums.length <= 10^4
-10^9 <= nums[i] <= 10^9

```js
var findLengthOfLCIS = function(nums) {
  const db = new Array(nums.length).fill(1)
  for(let i = 1;i<nums.length;i++){
     if(nums[i]>nums[i-1]){
         db[i] = db[i-1]+1
     }
  }
  return Math.max(...db)
};
```

### 最长重复（连续）子数组

给两个整数数组 A 和 B ，返回两个数组中公共的、长度最长的子数组的长度。

示例：

输入： A: [1,2,3,2,1] B: [3,2,1,4,7] 输出：3 解释： 长度最长的公共子数组是 [3, 2, 1] 。

提示：

1 <= len(A), len(B) <= 1000
0 <= A[i], B[i] < 100

```js
const findLength = (A, B) => {
    // A、B数组的长度
    const [m, n] = [A.length, B.length];
    // dp数组初始化，都初始化为0
    const dp = new Array(m + 1).fill(0).map(x => new Array(n + 1).fill(0));
    // 初始化最大长度为0
    let res = 0;
    for (let i = 1; i <= m; i++) {
        for (let j = 1; j <= n; j++) {
            // 遇到A[i - 1] === B[j - 1]，则更新dp数组
            if (A[i - 1] === B[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            }
            // 更新res
            res = dp[i][j] > res ? dp[i][j] : res;
        }
    }
    // 遍历完成，返回res
    return res;
};
```