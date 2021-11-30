# 贪心算法一

### [分发饼干](https://leetcode-cn.com/problems/assign-cookies/)

> 对每个孩子 i，都有一个胃口值 g[i]，这是能让孩子们满足胃口的饼干的最小尺寸；并且每块饼干 j，都有一个尺寸 s[j] 。如果 s[j] >= g[i]，我们可以将这个饼干 j 分配给孩子 i ，这个孩子会得到满足。你的目标是尽可能满足越多数量的孩子，并输出这个最大数值。
>
> 示例 1:
>
> - 输入: g = [1,2,3], s = [1,1]
> - 输出: 1 
> - 解释:你有三个孩子和两块小饼干，3个孩子的胃口值分别是：1,2,3。虽然你有两块小饼干，由于他们的尺寸都是1，你只能让胃口值是1的孩子满足。所以你应该输出1。

```js
var findContentChildren = function(g, s) {
  // 先把饼干和胃口从小到大排列
  g.sort((a, b) => a - b);
  s.sort((a, b) => a - b);
  let index = 0;
  // 饼干先分给胃口最小的孩子。最少分配原则
  for(let i = 0;i < s.length;i++){
    if(s[i] >= g[index]){
      index++;
    }
  }
  return index;
};
```

### [摆动序列](https://leetcode-cn.com/problems/wiggle-subsequence/)

>如果连续数字之间的差严格地在正数和负数之间交替，则数字序列称为摆动序列。第一个差（如果存在的话）可能是正数或负数。少于两个元素的序列也是摆动序列。
>
>- 输入: [1,7,4,9,2,5]
>- 输出: 6
>- 解释: 整个序列均为摆动序列。

```js
var wiggleMaxLength = function(nums) {
  let result = 1;
  let curVal = 0;
  let preVal = 0;
  for(let i = 0;i < nums.length - 1;i++){
    // 统计curVal差值
    curVal = nums[i + 1] - nums[i];
    // 如果差值为相反，则满足摆动系列
    if(curVal > 0 && preVal <= 0 || preVal >= 0 && curVal < 0){
      // 交换
      preVal = curVal;
      result++;
    }
  }
  return result;
};
```

### [最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)

> 给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
>
> * 输入: [-2,1,-3,4,-1,2,1,-5,4] 
>
> * 输出: 6 
>
> * 解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。

```js
var maxSubArray = function(nums) {
    let result = -Infinity
    let count = 0
    for(let i = 0; i < nums.length; i++) {
        // 求和
        count += nums[i]
      	// 求最大值
        if(count > result) {
            result = count
        }
        // 如果小于0则重置 
        if(count < 0) {
            count = 0
        }
    }
    return result
};
```

### [买卖股票最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

> 给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。
>
> 设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。
>
> 注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
>
> - 输入: [7,1,5,3,6,4]
> - 输出: 7
> - 解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4。随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。

```js
var maxProfit = function(prices) {
  let result = 0;
  for(let i = 0;i < prices.length - 1;i++){
    // 统计每天收益为正
    result += Math.max(prices[i + 1] - prices[i], 0);
  }
  return result;
};
```

### [跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

> 给定一个非负整数数组，你最初位于数组的第一个位置。
>
> 数组中的每个元素代表你在该位置可以跳跃的最大长度。
>
> 判断你是否能够到达最后一个位置。
>
> - 输入: [2,3,1,1,4]
> - 输出: true
> - 解释: 我们可以先跳 1 步，从位置 0 到达 位置 1, 然后再从位置 1 跳 3 步到达最后一个位置。

```js
var canJump = function(nums) {
  let cover = 0;
  if(nums.length === 0)return true;
  for(let i = 0;i <= cover;i++){
    // 第i测能走的最远距离
    cover = Math.max(i + nums[i], cover);
    // 能到大终点
    if(cover >= nums.length - 1)return true;
  }
  return false;
};
```

### [跳跃游戏二](https://leetcode-cn.com/problems/jump-game-ii/)

> 给定一个非负整数数组，你最初位于数组的第一个位置。
>
> 数组中的每个元素代表你在该位置可以跳跃的最大长度。
>
> 你的目标是使用最少的跳跃次数到达数组的最后一个位置。
>
> - 输入: [2,3,1,1,4]
> - 输出: 2
> - 解释: 跳到最后一个位置的最小跳跃数是 2。从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
>
> 说明: 假设你总是可以到达数组的最后一个位置。

```js
var jump = function(nums) {
  let cur = 0;
  let next = 0;
  let res = 0;
  for(let i = 0;i < nums.length-1;i++){
    // 能到达最远的距离
    next = Math.max(i + nums[i], next);
	 // 如果next 能到达终点，则再走一步即可    
    if(next>=nums.length-1){
        return res+1
    }
    // 已经走到上次跳转的最远距离
    if(i === cur){
      // 更新下次最远距离
      cur = next;
      res++;
    }
  }
  return res;
};
```

### [K次取反后最大化的数组和](https://leetcode-cn.com/problems/maximize-sum-of-array-after-k-negations/)

> 给定一个整数数组 A，我们只能用以下方法修改该数组：我们选择某个索引 i 并将 A[i] 替换为 -A[i]，然后总共重复这个过程 K 次。（我们可以多次选择同一个索引 i。）
>
> 以这种方式修改数组后，返回数组可能的最大和。
>
> - 输入：A = [4,2,3], K = 1
> - 输出：5
> - 解释：选择索引 (1,) ，然后 A 变为 [4,-2,3]。

```js
var largestSumAfterKNegations = function(nums, k) {
  // 按绝对值从大到小排列
  nums.sort((a, b) => Math.abs(b) - Math.abs(a));
  // 把负数的值进行k>0次转换
  for(let i = 0;i < nums.length;i++){
    if(nums[i] < 0 && k > 0){
      nums[i] = -nums[i];
      k--;
    }
  }
  // 如果K>0而且为奇数，则转换最小的那个数
  if(k>0&&k % 2 === 1){
    nums[nums.length-1] = -nums[nums.length-1];
  }
  // 累加
  const sum = nums.reduce((a, b) => a + b, 0);
  return sum;
};
```

