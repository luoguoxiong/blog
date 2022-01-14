# 双指针

### 有序数组的平方

> 给你一个按 非递减顺序 排序的整数数组 nums，返回 每个数字的平方 组成的新数组，要求也按 非递减顺序 排序。
>
> 示例 1： 输入：nums = [-4,-1,0,3,10] 输出：[0,1,9,16,100] 解释：平方后，数组变为 [16,1,0,9,100]，排序后，数组变为 [0,1,9,16,100]
>
> 示例 2： 输入：nums = [-7,-3,2,3,11] 输出：[4,9,9,49,121]

```js
var sortedSquares = function(nums) {
    let n = nums.length;
    let res = new Array(n).fill(0);
    let i = 0, j = n - 1, k = n - 1;
    while (i <= j) {
        let left = nums[i] * nums[i],
            right = nums[j] * nums[j];
        if (left < right) {
            res[k--] = right;
            j--;
        } else {
            res[k--] = left;
            i++;
        }
    }
    return res;
};
```

### 移除数组元素

> 给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。
>
> 不要使用额外的数组空间，你必须仅使用 $O(1)$ 额外空间并**原地**修改输入数组。
>
> 元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。
>
> 示例 1: 给定 nums = [3,2,2,3], val = 3, 函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。 你不需要考虑数组中超出新长度后面的元素。
>
> 示例 2: 给定 nums = [0,1,2,2,3,0,4,2], val = 2, 函数应该返回新的长度 5, 并且 nums 中的前五个元素为 0, 1, 3, 0, 4。

```js
var removeElement = function(nums, val) {
  let slowIndex = 0;
  let fastIndex = 0;
  while(fastIndex < nums.length){
    if(nums[fastIndex] !== val){
      nums[slowIndex] = nums[fastIndex];
      slowIndex++;
    }
    fastIndex++;
  }
  return slowIndex;
};
```

### 三数之和

> 给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。
>
> **注意：** 答案中不可以包含重复的三元组。
>
> 示例：
>
> 给定数组 nums = [-1, 0, 1, 2, -1, -4]，
>
> 满足要求的三元组集合为： [ [-1, 0, 1], [-1, -1, 2] ]

```js
var threeSum = function(nums) {
  const len = nums.length;
  if(len < 3) return [];
  nums.sort((a, b) => a - b);
  const resSet = new Set();
  for(let i = 0; i < len - 2; i++) {
    if(nums[i] > 0) break;
    let l = i + 1,
      r = len - 1;
    while(l < r) {
      const sum = nums[i] + nums[l] + nums[r];
      if(sum < 0) { l++; continue; };
      if(sum > 0) { r--; continue; };
      resSet.add(`${nums[i]},${nums[l]},${nums[r]}`);
      l++;
      r--;
    }
  }
  return Array.from(resSet).map((i) => i.split(','));
};
```

### 数组倒序

```js
var reverseString = function(s) {
   let h = 0;
   let m = s.length - 1;
   while(h < m){
   [s[h], s[m]] = [s[m], s[h]];
    h++;
    m--;
   }
   return s;
};
```

### 替换空格

> 请实现一个函数，把字符串 s 中的每个空格替换成"%20"。
>
> 示例 1： 输入：s = "We are happy."
> 输出："We%20are%20happy."

```js
const replaceSpace = (s)=>{
  const count = s.split(',').map(item=>item === '').length
  let left = s.length;
  let right = left + 2 * count - 1
  whilt(left>0){
    if(s[left]===''){
      s[right --] = 0
      s[right--] = 2
      s[right--] = '%'
    }
    s[right--] = s[left--]
  }
  return s
}
```

