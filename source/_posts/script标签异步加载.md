---
title: script标签加载
date: 2020/09/11
tags: ["script","defer","async"]
---



## 默认情况

默认情况，浏览器解析html文件，遇到script标签，将停止html文件的解析，下载并执行js脚本后，（如果是内联脚本，则执行js脚本后）html文件才能继续向下执行

- 如果下载、执行js文件过长，页面将显示空白页，造成不好的用户体验

- 因此html5中提出defer、async两个属性，实现异步下载、执行js脚本

## async defer type="module"

async和defer都仅适用于外联脚本
**async**

- 异步下载完成后，立即执行js脚本后，在解析HTML文档
- 如果声明多个async脚本，不能确保彼此的先后顺序
- async会在load事件之前执行，并不能确保与DOMContentLoaded的执行先后顺序
- 因为是下载完立即执行，不能保证多个加载时的先后顺序，因此确保异步脚本之间互不依赖
**defer**

- 异步下载完成后，待HTML文档解析完毕，在执行该脚本
- 如果声明多个defer脚本，会按照顺序下载和执行
- defer脚本在DOMContentLoaded和load事件之前执行
**type="module"**
script标签设置type=module，则也是也不加载，延迟执行的，等同于添加了defer属性

{% asset_img script执行.png script标签异步加载执行图 %}

## DOMContentLoaded和load

两者触发时机不一样，先触发DOMContentLoaded事件，在触发load事件
**DOMContentLoaded**
当DOM树构建完成后执行，DOM2级事件，可以绑定多次
**load**
当所有的节点和资源加载完成后执行，DOM0级事件，仅可绑定一次
