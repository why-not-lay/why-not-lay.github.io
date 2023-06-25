# 多进程架构
> 原文: [Multi-process Architecture](https://www.chromium.org/developers/design-documents/multi-process-architecture/)  
> 创建时间: 2023-06-16  
> 更新时间: 2023-06-16

本篇文章用于描述 Chromium 的高级架构以及它是如何在多进程类型中被划分的。

## 问题

构建一个永不崩溃或挂起的渲染引擎是几乎不可能的。构建一个完全安全的渲染引擎也是几乎不可能的。

在某些方面， 2006 年左右的网络浏览器状况就像过去单用户、协作多任务的操作系统类似。在这样的操作系统里，一个操作错误的应用可能会使整个系统崩溃，在浏览器里，一个操作错误的网页也会如此。只需一个渲染引擎或插件的出错，就会使整个浏览器和所有当前运行的标签失效。

现代操作系统变得更为强大，因为它们把应用运行在相互独立的进程中。一个应用程序的崩溃通常不会损害其他应用程序或操作系统的完整性，而且每个用户访问其他用户的数据都受到限制。Chromium 的架构旨在实现这种更强大的设计。

## 架构概览

Chromium 使用多进程是为了保护整个应用免受渲染引擎或其他组件错误和故障的影响。它限制每一个渲染进程对其他进程和系统其余部分的访问。在某些方面，这给页面浏览带来了操作系统中内存保护和访问控制的好处。

我们把运行 UI、管理渲染器和其他进程的主进程称为 “浏览器进程” 或 “浏览器”。同样地，处理网页内容的进程被称为 “渲染器进程” 或 “渲染器”。渲染器使用 [Blink](https://www.chromium.org/blink/) 开源布局引擎来解释铺设 HTML。

![multi-process-architecture-arch](	https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/multi-process-architecture/multi-process-architecture-arch.png)

### 管理渲染器进程

每个渲染器进程有一个全局 `RenderProcess` 对象，用于管理与父浏览器进程之间的通信和维护全局状态。浏览器给每个渲染器进程维护一个相应的 `RenderProcessHost`，用于管理渲染器的浏览器状态和通信。浏览器与渲染器之间的通信是使用 [Mojo](https://chromium.googlesource.com/chromium/src/+/HEAD/mojo/README.md) 或 [Chromium's legacy IPC system](https://www.chromium.org/developers/design-documents/inter-process-communication/)。

### 管理 frame 和文档

每个渲染器进程有一个或多个 `RenderFrame` 对象，它们对应于包含内容文档的 frame。浏览器进程中相应的 `RenderFrameHost` 管理与文档相关联的状态。每个 `RenderFrame` 被赋予一个路由 ID，这用于区分相同渲染器中的不同文档或 frame。这些 ID 在一个渲染器里是独一无二的，但在浏览器中却不是，所以要识别一个 frame 需要一个 `RenderProcessHost` 和路由 ID。从浏览器到渲染器中特定文档的通信是通过这些 `RenderFrameHost` 对象实现的，它们知道如何通过 Mojo 或 传统 IPC 发送信息。

## 组件和接口

在渲染器进程中：

- `RenderProcess` 处理 Mojo 设置和与浏览器中相应的 `RenderProcessHost` 的传统 IPC。每个渲染器进程正好有一个 `RenderProcess`。
- `RenderFrame` 对象与它在浏览器进程中相应的 `RenderFrameHost` (通过 via)以及 Blink 层通信，这个对象在一个标签或 subframe 中代表着一个网页文档的内容。

在浏览器进程中：
- `Browser` 对象代表一个顶层浏览器窗口。
- `RenderProcessHost` 对象代表单个浏览器的浏览器端 ↔ 渲染器 IPC 连接。每个渲染器进程正好有一个 `RenderProcessHost` 在浏览器进程中。
- `RenderFrameHost` 对象封装了与 `RenderFrame` 的通信，`RenderWidgetHost` 处理浏览器中 `RenderWidget` 的输入和绘制。

有关嵌入如何运作的更多详细信息，请参阅 [How Chromium displays web pages](https://www.chromium.org/developers/design-documents/displaying-a-web-page-in-chrome/) 设计文档。

## 共享渲染进程

一般而言，每个新窗口或标签会在一个新进程里打开。浏览器将生成一个新进程并指示它创建一个 `RenderFrame`，它可能在页面中创建更多的 iframes (可能在不同进程中)。

有时，在标签或窗口之间共享渲染器进程是必要的。例如，一个页面应用可以使用 `window.use` 创建另一个新的窗口，如果新文档同源，那么它必须共享相同进程。如果进程数太多，Chromium 存在一些策略来分配新标签到现有的进程中。这些考虑和策略在 [Process Model](https://chromium.googlesource.com/chromium/src/+/main/docs/process_model_and_site_isolation.md) 里有所描述。

## 检测崩溃或操作错误的渲染器

每个与浏览器进程相连的 Mojo 或 IPC 连接会监测进程句柄。如果这些句柄被发出信号，这代表渲染器进程已经崩溃，那么受影响的标签和 frame 会被告知崩溃。Chromium 展示一张 "sad tab" 或 "sad frame" 图片来提醒用户渲染器已经崩溃。页面能通过点击刷新按钮或开始新的导航来重载该页面。当这种情况发生时， Chromium 注意到没有渲染器进程，然后创建一个新的进程。

## 渲染器沙盒化

已知渲染器正运行在一个独立的进程，我们可以通过 [sandboxing](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/design/sandbox.md) 来限制它对系统资源的访问。例如，我们可以确保渲染器是通过 Chromium 的网络服务对网络唯一访问。同样地，我们可以使用主机操作系统的内置权限来限制它对文件系统的访问，或对用户显示和输入的访问。这些极大地限制一个局限的渲染器进程能够完成什么。

## 回收内存

随着渲染器运行在独立的进程中，它会直接将隐藏的标签设为低优先级。通常情况下， Windows 上最小化的进程会自自动将内存放入 “可用内存“ 池中。在少内存的情况下，Windows 会在交换出高优先级内存之前将这部分内存交换到磁盘中，有助于保持用户可见程序具有更高相应。我们能应用同样的规则到隐藏的标签中。当一个渲染器进程没有顶层标签时，我们可以释放该进程的 “工作集“大小作为对系统的提示，以便在必要的时候将该内存交换到磁盘上。因为我们发现用户在两个标签之间切换时，减少工作集大小也会降低标签切换性能，所有我们逐渐释放这部分内存。这意味着如果用户切换回最近一个使用的标签，该标签的内存比最近较少使用的标签更可能被分页。有足够内存来运行所有程序的用户必定不会注意到这个过程：Windows 只有在需要时才会实际回收这些数据，所以在有足够内存的情况下，不会有内存上的影响。

这有助于我们在少内存的情况下获得更好的内存占用率。和很少使用的后台标签相关的内存会完全被换掉，而前台标签的数据就可以完全加载到内存中。相比之下，一个单进程的浏览器会让所有标签数据随机分布在内存中，这不可能将已使用的和未使用的数据分离得如此干净，最终浪费了内存和性能。

## 额外的进程类型

Chromium 也将其他组件分割成独立的进程，有时是以特定平台的形式。例如，它现在有一个独立的 GPU 进程，网络服务和存储服务。沙盒化的实用进程也能被用于小型或有风险的任务，这被用作满足 [Rule of Two](https://chromium.googlesource.com/chromium/src/+/master/docs/security/rule-of-2.md) 安全的一种方式。

## 参考说明

本文所引用的图片均出自于 [原文 (Multi-process Architecture)](https://www.chromium.org/developers/design-documents/multi-process-architecture/)