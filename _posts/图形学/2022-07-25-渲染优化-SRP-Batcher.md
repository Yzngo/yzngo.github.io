

### SRP Batcher 原理

1. SRP Batcher并没有减少 [[DrawCall]] 的数量，只是效率更高，优化了每一次Drawcall的耗时。


### 合批条件

1. `只需要Shader变体相同，不需要材质相同`。
2. 渲染的物体必须是一个 [[Mesh]] 或者 [[SkinnedMesh]]。不能是粒子。
3. [[Shader]] 代码必须兼容 SRP Batcher。
4. 不能用代码修改 [[MaterialPropertyBlock]], 会破坏SRP Batcher。



### Shader兼容 SRP Batcher

1. 必须声明所有内建引擎properties 在一个名为"UnityPerDraw"的CBUFFER里。


### 注意事项

1. 如果项目启用了SRP Batcher，[[GPU instancing]] 就不再生效，可以使用代码做GPU实例化。
