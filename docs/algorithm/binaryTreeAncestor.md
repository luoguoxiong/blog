# 二叉树公共祖先问题

## 一、 二叉树的公共祖先

leetCode原地址
https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/

实现思路：
1. 由于需要自底向上查找就可以找到公共祖先（后序遍历）
2. 回溯，二叉树回溯的过程就是从低到上。
3. 当某个节点的值等于p或者q,或者该节点的子树存在某个节点值等于p或者q，则向上返回true
4. 当某个节点同时含有两个子树的值等于p、q,或者当前的值有等于p、q,而且子树存在一个节点，则当前节点就是公共祖先。
```js
var lowestCommonAncestor = function(root, p, q) {
    let res = null
    const dfs = (root)=>{
        if(root===null) return false
        const leftIsHas = dfs(root.left)
        const rightIsHas = dfs(root.right)
        // 如果子节点存在两个值等于p、q或者当前的值等于p或q并且，子节点已存在等于p或q的节点
        if(leftIsHas&&rightIsHas||(root.val===p.val||root.val===q.val&&(leftIsHas||rightIsHas))){
            res = root
        }
        // 如果子节点存在等于p、q或者当前值等p、q
        return leftIsHas||rightIsHas||root.val===p.val||root.val===q.val
    }
    dfs(root)
    return res
};
```


## 二、二叉搜索树的最近公共祖先

leetCode原地址
https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/

解题思路
1. 二叉搜索的左子树的值都小于该节点的值，右子树的值都大于该节点的值。
2. 二叉搜索数的中序遍历的值从小到大排列。（常用于判断是否是二叉搜索树）
3. 利用特点，只要从上往下走，存在某个节点大于p小于q或某个节点小于p大于q,则该节点就是公共祖先。

```js
var lowestCommonAncestor = function(root, p, q) {
    while(root){
        if(root.val>p.val&&root.val>q.val){
            root = root.left
        }
        else if(root.val<p.val&&root.val<q.val){
            root = root.right
        } else  return root
    }
    return null
};
```