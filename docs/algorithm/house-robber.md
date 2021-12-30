# 打家劫舍

### 一、非环形社区
```shell
你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。

给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。
输入：[1,2,3,1]
输出：4
解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
     偷窃到的最高金额 = 1 + 3 = 4 。
```

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
// db[i] 小偷到第i间房子的最大利润
// db[i] = db[i-2]+num[i] / db[i-1]
var rob = function(nums) {
    let db = []
    db[0]=nums[0]
    db[1] = Math.max(db[0],nums[1])
    for(let i = 2;i<nums.length;i++){
        db[i] = Math.max(db[i-1],db[i-2]+nums[i])
    }
    return db[nums.length - 1]
};
```

### 二、环形社区

```shell
你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 围成一圈 ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警 。
给定一个代表每个房屋存放金额的非负整数数组，计算你 在不触动警报装置的情况下 ，能够偷窃到的最高金额。

输入：nums = [2,3,2] 
输出：3 
解释：你不能先偷窃 1 号房屋（金额 = 2），然后偷窃 3 号房屋（金额 = 2）, 因为他们是相邻的。
```

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var rob = function(nums) {
  if(nums.length===1) return nums[0]
  const getRangeMax = (startIndex, end) => {
    const dp = [];
    dp[startIndex] = nums[startIndex];
    dp[startIndex + 1] = Math.max(nums[startIndex], nums[startIndex + 1]);
    for(let i = startIndex + 2;i < end;i++){
      dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
    }
    return dp[end - 1];
  };
  const m1 = getRangeMax(0, nums.length - 1);
  const m2 = getRangeMax(1, nums.length);
  return Math.max(m1, m2);
};
```

### 三、树形社区

```shell
在上次打劫完一条街道之后和一圈房屋后，小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为“根”。 除了“根”之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果两个直接相连的房子在同一天晚上被打劫，房屋将自动报警。

计算在不触动警报的情况下，小偷一晚能够盗取的最高金额。
```

```js
/**
 * Definition for a binary tree node.
 * function TreeNode(val, left, right) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.left = (left===undefined ? null : left)
 *     this.right = (right===undefined ? null : right)
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number}
 */
const memo = new Map();
var rob = function(root) {
  if(root===null) return 0
  if(root.left===null&&root.right===null) return root.val
  if(memo.has(root)) return memo.get(root)
  let val1 = root.val
  if(root.left){
      val1+=(rob(root.left.left)+rob(root.left.right))
  }
  if(root.right){
      val1+=(rob(root.right.left)+rob(root.right.right))
  }
  let val2  = rob(root.left)+rob(root.right)
  
  const retu  = Math.max(val1,val2)
  memo.set(root,retu)
  return retu
};
```

