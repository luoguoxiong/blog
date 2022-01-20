# 二分法

### 二分查找

> 给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。
>
> 示例 1:
>
> 输入: nums = [-1,0,3,5,9,12], target = 9     
> 输出: 4       
> 解释: 9 出现在 nums 中并且下标为 4     

```js
var search = function(nums, target) {
  let left = 0;
  let right = nums.length;
  while(left < right){
    const middle = left + Math.floor((right - left) / 2);
    if(nums[middle] > target){
      right = middle;
    }else if(nums[middle] < target){
      left = middle + 1;
    }else{
      return middle;
    }
  }
  return -1;
};
```

### [搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

> 给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。
>
> 请必须使用时间复杂度为 O(log n) 的算法。
>
> 示例 1:
>
> 输入: nums = [1,3,5,6], target = 5
> 输出: 2

```js
var searchInsert = function(nums, target) {
  let begin = 0;
  let end = nums.length - 1;
  let index = nums.length;
  while(begin <= end){
    // const mid = ((end - begin) >> 1) + begin;
    const mid = Math.floor((end - begin) / 2) + begin;
    if(nums[mid] > target){
      end = mid - 1;
      index = mid;
    }else if(nums[mid] < target){
      begin = mid + 1;
    }else{
      index = mid;
      break;
    }
  }
  return index;
};
```
### 构造二叉搜索树

> 将一个按照升序排列的有序数组，转换为一棵高度平衡二叉搜索树。
>
> 一个高度平衡二叉树是指一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过 1。

```js
var sortedArrayToBST = function (nums) {
  const buildTree = (Arr, left, right) => {
        if (left > right)
            return null;

        let mid = Math.floor(left + (right - left) / 2);

        let root = new TreeNode(Arr[mid]);
        root.left = buildTree(Arr, left, mid - 1);
        root.right = buildTree(Arr, mid + 1, right);
        return root;
    }
    return buildTree(nums, 0, nums.length - 1);
}
```

