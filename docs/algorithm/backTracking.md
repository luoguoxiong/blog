# 回溯算法

**回溯法（back tracking）**（探索与回溯法）是一种选优搜索法，又称为试探法，按选优条件向前搜索，以达到目标。但当探索到某一步时，发现原先选择并不优或达不到目标，就退回一步重新选择，这种走不通就退回再走的技术为回溯法，而满足回溯条件的某个状态的点称为“回溯点”。

##  组合问题

![](https://camo.githubusercontent.com/6df2412d2f47ba76d0b0b6da77800a4ee248507aed2897e704940b4fc53f656e/68747470733a2f2f696d672d626c6f672e6373646e696d672e636e2f32303230313131383135323932383834342e706e67)

## 回溯模板

```js
void backtracking(参数) {
    if (终止条件) {
        存放结果;
        return;
    }

    for (选择：本层集合中元素（树中节点孩子的数量就是集合的大小）) {
        处理节点;
        backtracking(路径，选择列表); // 递归
        回溯，撤销处理结果
    }
}
```

```js
let result = []
let path = []
var combine = function(n, k) {
  result = []
  combineHelper(n, k, 1)
  return result
};
const combineHelper = (n, k, startIndex) => {
  // 终止条件
  if (path.length === k) {
    result.push([...path])
    return
  }
  // 横向遍历
  for (let i = startIndex; i <= n - (k - path.length) + 1; ++i) {
    path.push(i)
    // 纵向遍历
    combineHelper(n, k, i + 1)
    path.pop()
  }
}
```

