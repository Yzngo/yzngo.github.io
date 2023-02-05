## Canvas

1. Canvas负责合并画布上的元素。在同一个Canvas中，将相同覆盖层级，相同材质，相同图集的元素进行合并，从而减少drawcall。（Canvas里如果两个元素重叠，就认为它们是上下层关系，将所有重叠的层计算完毕后，将第0层元素合并，再将第1, 2, 3...层的元素合并，依次类推）
2. 隐藏Canvas时，最好是把Canvas组件禁用，这样就不会给GPU发送drawcall命令，Canvas也就不会显示。这样也不会丢失它的网格缓存（Vertex Buffer）不会导致网格重新构建，实际上，disable掉Canvas组件不会触发昂贵的 OnDisable/OnEnable 回调，它只是简单的开始重新渲染已经构建好的网格。只是要注意，记得 disable 掉每帧都会执行的代码。 


## Canvas - Render Mode

1. `Screen Space - Camera`
    - 最常用的渲染模式。
    - 对Z轴不为0的元素，会单独提取出来渲染，不参与网格合并，使用时要注意，可能会成为一个性能点。
    - **优先级由由高到低：SortingLayer >> Order in Layer >> UI 层级**

2. `Screen Space - Overlay`
    - 这种模式下空间上的前后位置不再对元素起作用。
    - 常用在纯UI系统的区域内，这种模式下，Camera排序有别于其他模式，SortOrder参数在排  序时被重点用到，[[SortOrder]] 参数的值越大，越后渲染，越显示在前面。

3. `World Space`
    - 适用于放在3D世界中的UI元素。
    - 适用于制作 [[叙事UI]]，
    - 如果是World Space的UI，则 [[RectTransform]] 的 width 和height的单位不是像素，而是米。
    - 1Unit并不等于1米，要看  Canvas Scaler 中的设置。


## Canvas Renderer

1. Canvas下的任何可见UI对象都带有Canvas Renderer组件。
2. Canvas Renderer 是 Canvas 和渲染的连接组件。通过 Canvas Renderer 才能把网格绘制到 Canvas 画布上。

## Canvas Group

1. 可以使用Canvas Group 组件对 [[uGUI]] 控件进行**打组**。
2. 可以控制打组对象的透明度, 输入控制, Raycast等

## Canvas Scaler

1. 不同分辨率屏幕适配组件，用于指定画布中元素的比例大小。
2. 只能添加到Root-Canvas上。
4. 常用组合 `Scale With Screen Size /  Expand  /  100`
