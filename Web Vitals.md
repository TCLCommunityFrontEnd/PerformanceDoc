### Web Vitals

Web 指标是 Google 开创的一项新计划，旨在为网络质量信号提供统一指导，这些信号对于提供出色的网络用户体验至关重要。

网站所有者要想了解他们提供给用户的体验质量，并非需要成为性能专家。Web 指标计划为了简化场景，帮助网站专注于最重要的指标，即[核心 Web 指标（LCP FID CLS）](https://link.juejin.cn/?target=https%3A%2F%2Fweb.dev%2Fi18n%2Fzh%2Fdefining-core-web-vitals-thresholds%2F)。

Core Web Vitals 是应用于所有 Web 页面的 Web Vitals 的子集，是其最重要的核心

<img src="/Users/moriarty_c/Documents/PerformanceDoc/Pics/WebVitals1.jpg" width="50%">

#### LCP:  Largest Contentful Paint 最大内容绘制

LCP(Largest Contentful Paint)最大内容绘制，**可视区域中最大的内容元素**呈现到屏幕上的时间，用以估算页面的主要内容对用户的可见时间

LCP考虑的元素

- `<img>`元素
- 内嵌`<SVG>`元素内的`<image>`元素
- `<video>`元素
- 通过`url()`函数（而非使用CSS渐变）加载的带有背景图像的元素
- 包含文本节点或其他行内级文本元素子元素的块级元素

LCP元素的大小是指**视口上可见元素的大小**，**元素大小或者位置的变化不会产生新的LC候选元素**

![](/Users/moriarty_c/Documents/PerformanceDoc/Pics/WebVitals2.webp)

![](/Users/moriarty_c/Documents/PerformanceDoc/Pics/WebVitals3.webp)

在上方的两个时间轴中，最大元素随内容加载而变化。在第一个示例中，新内容被添加进 DOM，并因此使最大元素发生了改变。在第二个示例中，由于布局的改变，先前的最大内容从可视区域中被移除。

虽然延迟加载的内容通常比页面上已有的内容更大，但实际情况并非一定如此。接下来的两个示例显示了在页面完全加载之前出现的最大内容绘制。

![](/Users/moriarty_c/Documents/PerformanceDoc/Pics/WebVitals4.webp)

![](/Users/moriarty_c/Documents/PerformanceDoc/Pics/WebVitals5.webp)

在第一个示例中，Instagram LOGO加载得相对较早，即使其他内容随后陆续显示，但LOGO始终是最大元素。在 Google 搜索结果页面示例中，最大元素是一段文本，这段文本在所有图像或标志完成加载之前就显示了出来。由于所有单个图像都小于这段文字，因此这段文字在整个加载过程中始终是最大元素。

##### 速度指标

为了提供良好的用户体验，网站应该努力将最大内容绘制控制在**2.5 秒**或以内。为了确保您能够在大部分用户的访问期间达成建议目标值，一个良好的测量阈值为页面加载的**第 75 个百分位数**，且该阈值同时适用于移动和桌面设备。

<img src="/Users/moriarty_c/Documents/PerformanceDoc/Pics/LCP.png" width="25%">

##### LCP测量

```javascript
//使用PerformanceObserver
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log('LCP candidate:', entry.startTime, entry);
  }
}).observe({type: 'largest-contentful-paint', buffered: true});
//使用web-vitals库，当 LCP 可用时立即进行测量和记录。
import {getLCP} from 'web-vitals';
getLCP(console.log);
```

##### 优化方案

- [https://web.dev/optimize-lcp/](https://web.dev/optimize-lcp/)

导致LCP不佳的原因

- 缓慢的服务器响应速度
- 阻塞渲染的Javascript和CSS
- 缓慢的资源加载速度
- 客户端渲染

#### FID:  First Input Delay 首次输入延迟

发生输入延迟是因为浏览器的主线程正忙着执行其他工作，所以（还）不能响应用户。浏览器可能正忙于解析和执行由您的应用程序加载的大型 JavaScript 文件。在此过程中，浏览器不能运行任何事件侦听器。

第一次输入延迟通常发生在第一次内容绘制(FCP)和可持续交互时间(TTI)之间

<img src="/Users/moriarty_c/Documents/PerformanceDoc/Pics/WebVitals6.png" width="70%">

因为输入发生在浏览器正在运行任务的过程中，所以浏览器必须等到任务完成后才能对输入作出响应。浏览器必须等待的这段时间就是这位用户在该页面上体验到的 FID 值。

##### 速度指标

为了提供良好的用户体验，页面的 FID 应为**100 毫秒**或更短。

<img src="/Users/moriarty_c/Documents/PerformanceDoc/Pics/FID.png" width="25%">

##### FID测量

```javascript
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    const delay = entry.processingStart - entry.startTime;
    console.log('FID candidate:', delay, entry);
  }
}).observe({type: 'first-input', buffered: true});

import {getFID} from 'web-vitals';
// 当 FID 可用时立即进行测量和记录。
getFID(console.log);
```

##### 优化方案

https://web.dev/optimize-fid/

- 减小第三方代码的影响
- 减少Javascript执行时间
- 最小化主线程工作
- 保持较低的请求数和较小的传输大小

#### CLS：Cumulative Layout Shift 累积布局偏移

CLS 测量整个页面生命周期内发生的所有[意外](https://web.dev/cls/#expected-vs.-unexpected-layout-shifts)布局偏移中最大一连串的*布局偏移分数*。

请注意，只有当现有元素的起始位置发生变更时才算作布局偏移。如果将新元素添加到 DOM 或是现有元素更改大小，则不算作布局偏移，前提是元素的变更不会导致其他可见元素的起始位置发生改变。

##### 布局偏移分数

`布局偏移分数 = 影响分数 * 距离分数`

**影响分数**：测量*不稳定元素*对两帧之间的可视区域产生的影响。

<img src="/Users/moriarty_c/Documents/PerformanceDoc/Pics/WebVitals7.png" width="60%">

在上图中，有一个元素在一帧中占据了一半的可视区域。接着，在下一帧中，元素下移了可视区域高度的 25%。红色虚线矩形框表示两帧中元素的可见区域集合，在本示例中，该集合占总可视区域的 75%，因此其*影响分数*为`0.75` 。

**距离分数**：测量不稳定元素相对于可视区域位移的距离，是任何*不稳定元素*在一帧中位移的最大距离（水平或垂直）除以可视区域的最大尺寸维度（宽度或高度，以较大者为准）。

<img src="/Users/moriarty_c/Documents/PerformanceDoc/Pics/WebVitals8.png" width="60%">

在上方的示例中，最大的可视区域尺寸维度是高度，不稳定元素的位移距离为可视区域高度的 25%，因此*距离分数*为 0.25。

所以，在这个示例中，*影响分数*是`0.75` ，*距离分数*是`0.25` ，所以*布局偏移分数*是`0.75 * 0.25 = 0.1875` 。

##### 速度指标

为了提供良好的用户体验，页面的 CLS 应保持在 **0.1.** 或更少。

<img src="/Users/moriarty_c/Documents/PerformanceDoc/Pics/CLS.png" width="25%">

##### CLS测量

```javascript
import {getCLS} from 'web-vitals';

// 在所有需要汇报 CLS 的情况下
// 对其进行测量和记录。
getCLS(console.log);
```

##### 优化方案

https://web.dev/optimize-cls/

CLS 较差的最常见原因为：

- 无尺寸的图像
- 无尺寸的广告、嵌入和 iframe
- 动态注入的内容
- 导致不可见文本闪烁 (FOIT)/无样式文本闪烁 (FOUT) 的网络字体
- 在更新 DOM 之前等待网络响应的操作

1. 始终在您的图像和视频元素上包含尺寸属性，或者通过使用[CSS 长宽比容器](https://css-tricks.com/aspect-ratio-boxes/)之类的方式预留所需的空间
2. 除非是对用户交互做出响应，否则切勿在现有内容的上方插入内容。

#### 其他关键指标

##### First Contentful Paint (FCP)

浏览器首次绘制来自DOM的内容的时间，内容必须是文本，图片（包含背景图），非白色的Canvas或SVG，也包括正在加载中的web字体的文本

**白屏与首屏**

![](/Users/moriarty_c/Documents/PerformanceDoc/Pics/WebVistals9.jpeg)

![](/Users/moriarty_c/Documents/PerformanceDoc/Pics/WebVitals10.jpeg)

白屏时间：firstPaintEnd - performance.timing.navigationStart

首屏时间：firstPaintContentend - performance.timing.navigationStart

##### Time to Interactive (TTI)

表示网页第一次**完全达到可交互状态**的时间点，也就是最后一个长任务（50ms以上）完成的时间点，并且在随后5秒内网络和主线程是空闲的。

<img src="/Users/moriarty_c/Documents/PerformanceDoc/Pics/WebVitals11.png" width="60%">

0~3.8s:快速，3.9~7.3s:中等，7.3s以上:慢

##### Total Block Time (TBT)

总阻塞时间：度量了FCP和TTI之间的总时间

#### 测量Web Vitals

- 使用性能测试工具，如Lighthouse
- 使用web-vitals库
- 使用浏览器插件Web Vitals

#### 参考链接

- https://web.dev/vitals/
- https://juejin.cn/post/7026907443250593805
- 



