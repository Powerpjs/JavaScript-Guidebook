## 渲染层合并

概念不复杂，即是渲染层合并，我们将渲染树绘制后，形成一个个图层，最后把它们组合起来显示到屏幕。渲染层合并。前面也说过，对于页面中 DOM 元素的绘制是在多个层上进行的。在每个层上完成绘制过程之后，浏览器会将绘制的位图发送给 GPU 绘制到屏幕上，将所有层按照合理的顺序合并成一个图层，然后在屏幕上呈现。

对于有位置重叠的元素的页面，这个过程尤其重要，因为一量图层的合并顺序出错，将会导致元素显示异常。另外，这部分主要的是这涉及到我们常说的 GPU 加速的问题。

说到性能优化，针对页面渲染过程的话，我们希望的是代价最小，避免多余的性能损失，少一点让浏览器做的步骤。

明显，我们改的越深，代价越大，所以我们只改最后一个流程——合成的时候，性能是最好的。浏览器会为使用了 transform 或者 animation 的元素单独创建一个层。当有单独的层之后，此元素的重绘（Repaint）操作将只需要更新自己，不用影响到别，局部更新。所以开启了硬件加速的动画会变得流畅很多。

因为每个页面元素都有一个独立的渲染进程，包含了主线程和合成线程，主线程负责脚本的执行、CSS 样式计算、计算布局位置（Layout）、将页面元素绘制成位图（Paint）、发送位图给合成线程。合成线程则主要负责将位图发送给 GPU、计算页面的可见部分和即将可见部分（滚动）、通知 GPU 绘制位图到屏幕上。加上一个点，GPU 对于动画图形的渲染处理比 CPU 要快，那么就可以达到加速的效果。

注意不能滥用 GPU 加速，一定要分析其实际性能表现。因为 GPU 加速创建渲染层是有代价的，每创建一个新的渲染层，就意味着新的内存分配和更复杂的层的管理。并且在移动端 GPU 和 CPU 的带宽有限制，创建的渲染层过多时，合成也会消耗跟多的时间，随之而来的就是耗电更多，内存占用更多。过多的渲染层来带的开销而对页面渲染性能产生的影响，甚至远远超过了它在性能改善上带来的好处。

 

 

 

 