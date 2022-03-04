# javaScript标签加载机制
![preview](../img/jsload.png)

绿色代表`html`解析，灰色代表`html`解析停止，蓝色代表`script`下载，粉红色代表`script`执行。从上图很容易的看出来只要执行`script`,`html`就会停止渲染，除此之外也可以清晰的看出他们之间的加载关系。
