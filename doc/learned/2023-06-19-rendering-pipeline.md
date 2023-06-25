# 渲染管线简述

> 创建时间: 2023-06-19  
> 更新时间: 2023-06-21

本文是基于 [Life of a Pixel](https://www.youtube.com/watch?v=K2QHdgAKP-s&ab_channel=BlinkOn) 的总结，下文内容主要关于 Chrome 是如何将 HTML 逐步渲染成实际交互的页面。

## 渲染流程总览

![rendering-pipeline-summary](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/rendering-pipeline.png)
<center style="font-size:12px;color:#C0C0C0">图 1: 渲染流程</center>

## DOM 解析 (parsing)

这个过程负责将 HTML 文档转化为 DOM

### DOM (Document Object Model)

<div style="background: white">

![DOM](	https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/DOM.svg)

</div>
<center style="font-size:12px;color:#C0C0C0">图 2: DOM 节点结构</center>

DOM 以树型结构存储在内存中，树中的节点有着如下属性: 
- parent: 指向父亲节点
- firstChild: 指向第一个孩子节点
- lastChild: 指向最后一个孩子节点
- next: 指向下一个兄弟节点
- previ: 指向前一个兄弟节点

DOM 是文档在 Chrome 内部的抽象表示，同时也通过 V8 对外暴露文档查询和修改的 API。

<div style="background: white">

![composed-tree](	https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/shadow-tree.svg)

</div>
<center style="font-size:12px;color:#C0C0C0">图 3: composed tree</center>

对于包含 Shdow Tree 的 DOM Tree 被称为 Composed Tree，而 Shadow Tree 在 DOM Tree 中挂载的节点被称作 shadow host，每个 shadow host 和 Shadow Tree 一一对应。

值得注意的是，在 Shadow Tree 的根节点被称作 shadow root，但 shadow root 和 shadow host 并不存在树结构中的父子关系。

<div style="background: white">

![flat-tree](	https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/flat-tree.svg)

</div>
<center style="font-size:12px;color:#C0C0C0">图 4: composed tree</center>

为了后续 [layout](#layout) 流程的处理，Composed Tree 会通过 Tree Flattening 操作被转化为 Flat Tree。

## 样式解析 (style)

这个过程根据 CSS 来得出各个 DOM 节点的计算样式

![style-process](	https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/style-process.png)

<center style="font-size:12px;color:#C0C0C0">图 5: style</center>

## 布局 (layout)

这个过程负责计算各个 DOM 节点的几何信息，包括节点的坐标及其对应矩形区域的长和宽。
> 对于字体的几何信息是通过 HarfBuzz 库来计算。

在这个计算的过程中，会相应的构建一棵布局树。

### 布局树 (Layout Tree)

![layout-tree](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/layout-tree.png)

<center style="font-size:12px;color:#C0C0C0">图 6: layout tree</center>

布局树中的每一个节点会依据节点的类型执行相应的布局算法，这些节点类型，如图中的 LayoutBlockFlow, LayoutText 等类型均是 LayoutObject 的子类。

![layout-obj-dom](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/layout-obj-dom.png)

<center style="font-size:12px;color:#C0C0C0">图 7: layout object</center>

值得注意的是，DOM 节点和 LayoutObject 并不是一一对应的。对于 display: none 的节点是不会生成 LayoutObject，而对于内联节点则会额外生成一个匿名 LayoutObject。

### 布局引擎 (Layout Engine)

![layout-tree-legacy](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/layout-tree-legacy.png)

<center style="font-size:12px;color:#C0C0C0">图 8: legacy layout engine</center>

在传统布局的过程中，会使用传统布局引擎进行布局，这个过程的输入和输出数据均存储在同一个 LayoutObject 中，这使得每次的布局都是对同一棵布局树进行操作，输入和输出数据的杂糅也使得更新的时候小心辨析数据。

![layout-engine-NG](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/layout-engine-NG.png)

<center style="font-size:12px;color:#C0C0C0">图 9: layoutNG</center>


新的布局引擎 LayoutNG 将输入和输出的数据完全分离，每个 LayoutObject 都有一个布局算法类 NGLayoutAlgorithm，该类以 LayoutObject 和 NGConstraintSpace 为输入，输出是不可变的 NGPhysicalFragment，并起到缓存作用。

## 绘制 (paint)

这个过程记录绘制的操作 (paint ops)，而不是生成像素。

![paint](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/paint.png)

<center style="font-size:12px;color:#C0C0C0">图 10: paint</center>

绘制的过程会生成绘制操作列表 (paint ops)，并存储在 DisplayItem 对象里。

### [层叠上下文 (stacking context)](./2023-06-20-stacking-context.md)

![stacking-order](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/stacking-order.png)

<center style="font-size:12px;color:#C0C0C0">图 11: staking order</center>

在绘制的过程中会依据一定的[规则](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_positioned_layout/Understanding_z-index/Stacking_context#%E5%B1%82%E5%8F%A0%E4%B8%8A%E4%B8%8B%E6%96%87)生成层叠上下文，而在每个层叠上下文中，会依据相应的层叠顺序 (stacking order) 生成绘制操作。

## 光栅化 (raster)

这个过程会根据 DisplayItem 的绘制操作 (paint ops) 生成位图数据。

![raster](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/raster.png)

<center style="font-size:12px;color:#c0c0c0">图 12: raster</center>

光栅化的过程会通过 Skia 调用 OpenGL 的接口来实现加速渲染的效果。由于渲染器进程是被沙盒化，无法直接进行系统调用，所以这个过程会在 GPU 进程执行。

![gpu-process](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/gpu-process.png)

<center style="font-size:12px;color:#c0c0c0">图 13: gpu process</center>

绘制操作 (paint ops) 会被封装在 GPU 指令里，然后通过 IPC 传输到 GPU 进程，之后就会在 GPU 进程里执行光栅化的过程，这样做能够很好的隔绝光栅化过程出错时带来的危害。

## 分层与合并

为加快渲染速度，尽可能地保持每秒 60 帧的刷新，渲染的过程会加入分层与合并这两个步骤。

### 分层 (compositing assignments)

![compositing assignments](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/compositing-assignments.png)

<center style="font-size:12px;color:#c0c0c0">图 14: compositing assignments</center>

该过程会依据 DOM 样式等情况进行分层并创建相应的层列表 (layer list)，每一层后续会独立绘制。

![prepaint](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/prepaint.png)

<center style="font-size:12px;color:#c0c0c0">图 15: prepaint</center>

在分层后并不会立即进入绘制阶段，而是进入预绘制 (prepaint) 阶段，这个阶段会创建属性树 (property trees) 以供后续使用。


### 合并

![commit](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/commit.png)

<center style="font-size:12px;color:#c0c0c0">图 16: commit</center>

在绘制完成后，会在合成器线程里对各图层 (layer) 及属性树进行比对并更新。

![tiling](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/tiling.png)

<center style="font-size:12px;color:#c0c0c0">图 17: tiling</center>

由于图层会比较大，每次光栅化整个图层会耗费比较多的资源，并且这其中的某些部分是不会在屏幕上展示，所以这部分没有必要进行光栅化。未解决这一问题，在合成器线程中会将图层进行分片 (tile)，这些分片会由多个光栅化线程进行处理，处理的优先级由图层与展示区域的距离决定。

> 光栅化的过程是在 GPU 进程处理。

![tiling-draw](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/tiling-draw.png)

<center style="font-size:12px;color:#c0c0c0">图 18: draw</center>

在所有的分片光栅化后，合成器线程会生成 “绘制四边形” (draw quads)，每个四边形代表相应分片在屏幕上的绘制区域，随后这些四边形会依据图层列表合并成帧 (CompositorFrame)，该帧被提交给渲染器进程。

![activate](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/activate.png)

<center style="font-size:12px;color:#c0c0c0">图 19: activate</center>

在合并器线程中有两棵树，pending tree 是处理上述 commit 和 tiling 的过程，而 active tree 是进行后续的 draw/display 操作。

![display](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/rendering-pipline/display.png)

<center style="font-size:12px;color:#c0c0c0">图 20: display</center>

在 activate 后，渲染器进程里处理合成后的帧 (CompositorFrame) 会被发送到 GPU 进程中进行处理并在屏幕中展示。

## 参考说明

[图 1 出处](https://docs.google.com/document/d/1aitSOucL0VHZa9Z2vbRJSyAIsAz24kX8LFByQ5xQnUg/edit)

[图 2 出处](https://chromium.googlesource.com/chromium/src/+/master/third_party/blink/renderer/core/dom/README.md#node-and-node-tree)

[图 3 出处](https://chromium.googlesource.com/chromium/src/+/master/third_party/blink/renderer/core/dom/README.md#shadow-tree)

[图 4 出处](https://chromium.googlesource.com/chromium/src/+/master/third_party/blink/renderer/core/dom/README.md#flat-tree)

[图 5 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_371)

[图 6 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_528)

[图 7 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_573)

[图 8 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_573)

[图 9 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_653)

[图 10 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_814)

[图 11 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_859)

[图 12 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_1003)

[图 13 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_1059)

[图 14 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_1394)

[图 15 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_1462)

[图 16 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_1535)

[图 17 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_1578)

[图 18 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_1619)

[图 19 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_1691)

[图 20 出处](https://docs.google.com/presentation/d/1boPxbgNrTU0ddsc144rcXayGA_WF53k96imRH8Mp34Y/edit#slide=id.ga884fe665f_64_1718)

[DOM](https://chromium.googlesource.com/chromium/src/+/master/third_party/blink/renderer/core/dom/README.md#dom)