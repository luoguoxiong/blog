# requestIdleCallback 实现

```js
const requestIdleCallback = (cb) => {
  const start = Date.now();
  return setTimeout(function() {
    cb({
      timeRemaining: function() {
        return Math.max(0, 60 - (Date.now() - start));
      },
    });
  }, 0);
};

const cancelIdleCallback = function(id) {
  return clearTimeout(id);
};

let taskIndex = 0;

const task = [{
  name: '任务1',
  total: 0,
},
{
  name: '任务2',
  total: 0,
},
{
  name: '任务3',
  total: 0,
},
];

const target = 2000000;

function myNonEssentialWork(deadline) {
  const obj = task[taskIndex];
  console.log(`${obj.name}任务执行中...`);
  // 当前任务有剩余时间，则继续执行任务
  while(obj.total < target && deadline.timeRemaining() > 0 ){
    obj.total++;
  }
  // 任务执行完，准备下一个任务
  if(obj.total === target){
    taskIndex++;
  }
  // 任务未执行完，继续调度
  if(taskIndex < task.length){
    requestIdleCallback(myNonEssentialWork);
  }
}
requestIdleCallback(myNonEssentialWork);
// 任务1任务执行中...
// 任务1任务执行中...
// 任务2任务执行中...
// 任务2任务执行中...
// 任务3任务执行中...
// 任务3任务执行中...
```