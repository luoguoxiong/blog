# 二叉树遍历

## 一、深度优先遍历
![image-20210708113111640](../../img/binaryTree.png)
### 前序遍历为例
**递归**
```js
var preorderTraversal = function(root, res = []) {
    if (!root) return res;
    res.push(root.val); // 中
    preorderTraversal(root.left, res) // 左
    preorderTraversal(root.right, res) // 右
    return res;
};
```
**栈方法**
```js
var preorderTraversal = function(root, res = []) {
    const stack = [];
    if (root) stack.push(root);
    while(stack.length) {
        const node = stack.pop();
        if(!node) {
            res.push(stack.pop().val);
            continue;
        }
        if (node.right) stack.push(node.right); // 右
        if (node.left) stack.push(node.left); // 左
        stack.push(node); // 中
        stack.push(null);
    };
    return res;
};
```
## 二、广度优选遍历
![image-20210708113111640](../../img/level.gif)
```js
var levelOrder = function(root) {
    let res=[],queue=[];
    queue.push(root);
    if(root===null){
        return res;
    }
    while(queue.length!==0){
        let length=queue.length;
        let curLevel=[];
        for(let i=0;i<length;i++){
            let node=queue.shift();
            curLevel.push(node.val);
            node.left&&queue.push(node.left);
            node.right&&queue.push(node.right);
        }
        res.push(curLevel);
    }
    return res;
};
```