# babel 预设插件的选择

## babel 预设
在Babel6的时代，常见的preset有babel-preset-es2015、babel-preset-es2016、babel-preset-es2017、babel-preset-latest、babel-preset-stage-0、babel-preset-stage-1、babel-preset-stage-2等。

babel-preset-es2015、babel-preset-es2016、babel-preset-es2017分别是TC39每年发布的进入标准的ES语法的转换器预设，我们在这里称之为年代preset。

目前，Babel官方不再推出babel-preset-es2017以后的年代preset了。

babel-preset-stage-0、babel-preset-stage-1、babel-preset-stage-2、babel-preset-stage-3是TC39每年草案阶段的ES语法转换器预设

从Babel7版本开始，上述的预设都已经不推荐使用了，babel-preset-stage-X因为对开发造成了一些困扰，也不再更新。

babel-preset-latest，在Babel6的时候是你在使用它的时候所有年代preset的集合，在Babel6最后一个版本，它是babel-preset-es2015、babel-preset-es2016、babel-preset-es2017这三个的集合。因为Babel官方不再推出babel-preset-es2017以后的年代preset了，所以babel-preset-latest定义变成了TC39每年发布的进入标准的ES语法的转换器预设集合。其实，和Babel6时的内涵是一样的。

@babel/preset-env包含了babel-preset-latest的功能，并对其进行增强，现在@babel/preset-env完全可以替代babel-preset-latest。

经过一番梳理，可以总结为以前要用到的那么多preset预设，现在只需一个@babel/preset-env就可以了。

工程常用的预设

* @babel/preset-env
* @babel/preset-react
* @babel/preset-typescript
* @vue/babel-plugin-jsx

## babel plugin
虽然Babel7官方有90多个插件，不过大半已经整合在@babel/preset-env和@babel/preset-react等预设里了，我们在开发的时候直接使用预设就可以了。

目前比较常用的插件只有@babel/plugin-transform-runtime。目前我做过的几个项目，前端工程已经很少见到里使用其它的插件了。