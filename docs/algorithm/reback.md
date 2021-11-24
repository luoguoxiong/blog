# 回溯算法组合问题

leetCode原地址
https://leetcode-cn.com/problems/combination-sum-ii/submissions/

给定一个数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的每个数字在每个组合中只能使用一次。

注意：解集不能包含重复的组合。 

```shell
输入: candidates = [10,1,2,7,6,1,5], target = 8,
输出: [[1,1,6],[1,2,5],[1,7],[2,6]]
```
解题思路
1. 组合问题是典型的回溯解法
2. 因为candidates 中的每个数字在每个组合中只能使用一次,所以candidates需要进行排序，为了剪枝优化
3. 剪枝问题：
    1. 当前总数（sum+item>target）直接跳过
    2. 如果当前for循环的第i项且和第i-1项的值相同，也跳过。

    为什么i 和 i-1项对比？因为：如果i和i+1想对比，则上一项会丢失，造成组合枚举丢失。
        
```js
    var combinationSum2 = function(candidates, target) {
            const res = [];
            const path = [];
            let sum = 0;
            candidates = candidates.sort();
            const compins = (startIndex) => {
            if(sum > target)return;
            if(sum === target){
                res.push([...path]);
                return;
            }
            for(let i = startIndex;i < candidates.length;i++){
                const item = candidates[i];
                if(sum + item > target || i > startIndex && candidates[i] === candidates[i - 1])continue;
                sum += item;
                path.push(item);
                compins(i + 1);
                sum -= item;
                path.pop();
            }
        };
        compins(0);
        return res;
    };
```