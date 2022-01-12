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

