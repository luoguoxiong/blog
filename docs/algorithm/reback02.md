# 回溯算法全排列问题

### 一、 没有重复数字的数组全排列
https://leetcode-cn.com/problems/permutations/submissions/

```shell
给定一个不含重复数字的数组 nums ，返回其 所有可能的全排列 。你可以 按任意顺序 返回答案。

输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

```js
var permute = function(nums) {
  const path = [];
  const res = [];
  const indexs = []; // 已经使用过的数组小标，下次使用则跳过
  const comps = () => {
    if(path.length === nums.length){
      res.push([...path]);
      return;
    }
    for(let i = 0;i < nums.length;i++){
      if(indexs.includes(i)) continue;
      path.push(nums[i]);
      indexs.push(i);
      comps(i + 1);
      indexs.pop();
      path.pop();
    }
  };
  comps();
  return res;
};
```

### 一、 有重复数字的数组全排列
https://leetcode-cn.com/problems/permutations-ii/submissions/

```shell
给定一个可包含重复数字的序列 nums ，按任意顺序 返回所有不重复的全排列。

输入：nums = [1,1,2]
输出：[[1,1,2],[1,2,1],[2,1,1]]
 ```

 ```js
 var permuteUnique = function(nums) {
  const path = [];
  const res = [];
  const isUs = [] // 已经使用过的数组小标，下次使用则跳过
  const comps = () => {
    if(path.length === nums.length){
      res.push([...path]);
      return;
    }
    const obj = {} // 判断当前循环是否有重复的数字
    for(let i = 0;i < nums.length;i++){
      // 过滤已经使用过的数值下标或者当前循环有重复的数字
      if(isUs.includes(i)|| nums[i] in obj) continue
      path.push(nums[i]);
      item = nums[i]
      obj[nums[i]] = i
      isUs.push(i)
      comps(i + 1);
      path.pop();
      isUs.pop()
    }
  };
  comps();
  return res;
};
```