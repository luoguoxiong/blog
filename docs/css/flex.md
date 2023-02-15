# flex的一些属性

## flex 属性计算


flex属性是flex-grow，flex-shrink和flex-basis的缩写

## flex-group 补充计算

fixWidth = totalWidth-leftBasis-rightBasis

width = fixWidth * leftGroup / (leftGroup + rightGroup)

## flex-shrik 溢出计算

overWidth = leftBasis + rightBasis - totalWidth

width = leftBasis - overWidth * (leftBasis * leftShrink / ( leftBasis * leftShrink + rightBasis * rightShrink)) */

`flex-basis 基础宽度，如果设置了width和flex-basis，那么flex-basis会覆盖width值`

## flex 缩写

1. `flex:initial`等同于设置`flex: 0 1 auto`，可以理解为flex属性的默认值。
 
 该dom内容不会扩大，但会缩小，内容宽度，在未发生缩小的情况下，dom宽度由内部元素决定。如果内容超过父容器，则会等比例缩小。

2. `flex: 0` 等同于设置 `flex: 0 1 0%`

表示flex-grow是0，flex-shrink是1，因此元素尺寸会收缩但不会扩展，在加上flex-basis:0%表示建议支持是0，因此，设置flex:0的元素的最终尺寸表现为最小内容宽度；

3. `flex: none` 等同于 `flex: 0 0 auto`

表示元素尺寸不会收缩也不会扩展，再加上flex-basis:auto表示固定尺寸由内容决定，由于元素不具有弹性，因此，元素内的内容不会换行，最终尺寸通常表现为最大内容宽度。

4. `flex:1` 等同于 `flex: 1 1 0%`

元素尺寸可以弹性增大，也可以弹性变小，具有十足的弹性，在尺寸不足时会优先最小化内容尺寸

5. `flex:auto`等同于 `flex: 1 1 auto`

元素尺寸可以弹性增大，也可以弹性变小，具有十足的弹性，在尺寸不足时会优先最大化内容尺寸。






