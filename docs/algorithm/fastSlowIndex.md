# 快慢指针

### [删除链表的倒数第 N 个结点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

> 输入：head = [1,2,3,4,5], n = 2
> 输出：[1,2,3,5]

```js
var removeNthFromEnd = function(head, n) {
  let slow = head;
  let fast = head;
  while(n > 0){
    fast = fast.next;
    n--;
  }
  if(fast === null){
    return slow.next;
  }
  while(fast.next){
    slow = slow.next;
    fast = fast.next;
  }
  slow.next = slow.next.next;
  return head;
};
```

### 链表是否有环

> 输入：head = [3,2,0,-4], pos = 1
> 输出：返回索引为 1 的链表节点
> 解释：链表中有一个环，其尾部连接到第二个节点。

```js
var detectCycle = function(head) {
  let fast = head;
  let slow = head;
  while(fast && fast.next && fast.next.next){
    fast = fast.next.next;
    slow = slow.next;
    if(fast === slow){
      let cur = head;
      while(cur){
        if(cur === slow){
          return cur;
        }
        cur = cur.next;
        slow = slow.next;
      }
    }
  }
  return null;
};
```

