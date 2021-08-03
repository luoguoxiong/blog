# 虚拟化列表

虚拟列表是指当成千上万条数据需要进行展示但是用户的“视窗”（一次性可见内容）又不大时，通过只渲染用户最大可见条数+“BufferSize”个元素并在用户进行滚动时动态更新每个元素中的内容从而达到一个和长list滚动一样的效果但花费非常少的资源。如下图：

![image-20210729154403961](../img/vList.jpg)

1. 布局

   ```html
   <div className="vListContainer">
     <div className="phantomContent" />
     <div className="actualContent">
       ...
       <!-- item-1 -->
       <!-- item-2 -->
       <!-- item-3 -->
       ....
     </div>
   </div>
   ```

   vListContainer 包裹容器，phantomContent 滚动内容 动态计算高度，actualContent渲染内容，通过transformY设置滚动距离。

2. 定义一个容器的height，预估每个item的estimateHeight，计算出首屏渲染的个数(total)为：

   ```js
    const limit = Math.ceil(height / estimatedRowHeight);
    Math.min(limit + bufferSize,data.length - 1) // 首屏渲染
   ```

3. 定义一个startIndex，渲染的第一项。初始值为：0

4. 定义一个endIndex，渲染的最后一项。初始值为：startIndex + total

5. 定义一个cacheSize，统计每个已渲染的item属性：

   ```js
   const cacheSize = useRef<any>((() => {
     const cachedPositions = [];
     for (let i = 0; i < data.length; ++i) {
       cachedPositions[i] = {
         index: i,
         height: estimatedRowHeight,
         top: i * estimatedRowHeight,
         bottom: (i + 1) * estimatedRowHeight,
         dValue: 0,
       };
     }
     return cachedPositions;
   })());
   ```

5. 通过监听vListContainer滚动距离计算出，phantomContent 高度和actualContent 的transformY距离及重新计算cacheSize，每个item的属性。

