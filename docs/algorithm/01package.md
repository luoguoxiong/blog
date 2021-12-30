# 01背包

### 一、背包最大价值

```js
//  容量   0    1   2   3  4
// 1 15 [ 0 , 15, 15, 15, 15 ]
// 3 20 [ 0 , 15, 15, 20, 35 ]
// 4 30 [ 0 , 15, 15, 20, 35 ]
// db[i][j] 表示 取前i个物品在容量j的背包中的最大价值
function testWeightBagProblem(wight, value, size) {
  const len = wight.length;

  //   初始化 i * j 的二维数组 每个坑表示质量J,0-I组合物品内承载的最大质量
  const dp = Array.from({ length: len + 1 }).map(
    () => Array(size + 1).fill(0)
  );

  for(let i = 1;i <= len;i++){
    for(let j = 1;j <= size;j++){
      if(wight[i - 1] <= j){
        // 如果当前物品还能放入背包，则判断当前物品的价值，和出去当前物品的质量能存放的价值和与不放当前物品时的价值对比
        dp[i][j] = Math.max(dp[i - 1][j], value[i - 1] + dp[i - 1][j - wight[i - 1]]);
      }else{
        // 如果放不进去了，则取没有当前物品背包的价值
        dp[i][j] = dp[i - 1][j];
      }
    }
  }
  return dp[len][size];
}
```

### 二、 分割等和子集

```shell
给定一个只包含正整数的非空数组。是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

注意: 每个数组中的元素不会超过 100 数组的大小不会超过 200

示例 1: 输入: [1, 5, 11, 5] 输出: true 解释: 数组可以分割成 [1, 5, 5] 和 [11].

示例 2: 输入: [1, 2, 3, 5] 输出: false 解释: 数组不能分割成两个元素和相等的子集.
```

```js
// 是否存在背包的价值刚好为target的情况，其中价值和物品质量相等
// 在数组中找到累加和为总和一般的情况。
var canPartition = function(nums) {
  const sum = nums.reduce((prev, cur) => prev + cur, 0);
  if(sum % 2 === 1) return false;
  const target = sum / 2;

  const db = (new Array(nums.length + 1).fill(0)).map(() => {
    return new Array(target + 1).fill(0);
  });
  for(let i = 1;i <= nums.length;i++){
    for(let j = 1;j <= target;j++){
      if(nums[i] <= j){
        db[i][j] = Math.max(db[i - 1][j], db[i - 1][j - nums[i]] + nums[i]);
        if(db[i][j] === target){
          return true;
        }
      }else{
        db[i][j] = db[i - 1][j];
      }
    }
  }
  return db[nums.length][target] === target;
};
```

### 三、1049. 最后一块石头的重量 II

```shell
有一堆石头，每块石头的重量都是正整数。

每一回合，从中选出任意两块石头，然后将它们一起粉碎。假设石头的重量分别为 x 和 y，且 x <= y。那么粉碎的可能结果如下：

如果 x == y，那么两块石头都会被完全粉碎； 如果 x != y，那么重量为 x 的石头将会完全粉碎，而重量为 y 的石头新重量为 y-x。 最后，最多只会剩下一块石头。返回此石头最小的可能重量。如果没有石头剩下，就返回 0。

示例： 输入：[2,7,4,1,8,1] 输出：1 解释： 组合 2 和 4，得到 2，所以数组转化为 [2,7,1,8,1]， 组合 7 和 8，得到 1，所以数组转化为 [2,1,1,1]， 组合 2 和 1，得到 1，所以数组转化为 [1,1,1]， 组合 1 和 1，得到 0，所以数组转化为 [1]，这就是最优值。
```

```js
// 在数组中找到累加和为 总和一般的价值
var lastStoneWeightII = function (stones) {
    let sum = stones.reduce((s, n) => s + n);

    let dpLen = Math.floor(sum / 2);
    let dp = new Array(dpLen + 1).fill(0);

    for (let i = 0; i < stones.length; ++i) {
        for (let j = dpLen; j >= stones[i]; --j) {
            dp[j] = Math.max(dp[j], dp[j - stones[i]] + stones[i]);
        }
    }

    return sum - 2 * dp[dpLen];
};
```

