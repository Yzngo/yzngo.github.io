

[Graphics.DrawMeshInstanced](https://docs.unity3d.com/2021.2/Documentation/ScriptReference/Graphics.DrawMeshInstanced.html)


----

### GPU Instancing是什么？

1. GPUInstancing是一种GPU优化手段，可以让CPU用一个 DrawCall 命令控制GPU绘制**某个网格**的大量实例，这些网格使用**相同的material**。
2. GPU Instancing 是显卡的一个特性，大部分图形API都能提供的一种技术，其表象为当我们绘制1000个物体时，它只将模型数据及1000个坐标提交给显卡，这1000个物体不同的位置、状态、颜色等将整合成一个Per Instance Attribute的Buffer提交给GPU，这使得GPU可以在着色器中区别对待传入的网格数据。这样做的好处是只需要提交一次就能绘制所有不同位置的物体，这大大减少了提交次数，对于绘制大量相同模型的情况这种技术可以提高效率，同时也能避免因合批而造成内存浪费。
3. 使用GPU instancing绘制的对象在Frame Debugger中显示为 Draw Mesh(instanced)



----

### 需要满足的条件

1. 需要Shader着色器支持GPU Instancing（需要根据Instancing索引提取的参数来改变顶点或颜色等数据）。
2. 所有待渲染的对象需要使用相同的着色器，材质球和网格，不能有骨骼动画。
3. 使用 GPU Instancing就不能再使用 [[SkinnedMeshRenderer]] 或 [[Animator]] 计算蒙皮网格了。
4. 和SRP Batcher不兼容，且SRP Batcher优先级更高。如果使用SRP，要使GPU Instancing生效，需要SRP的设置中取消SRP Batcher。或者通过代码使用GPU Instancing。
5. SkinnedMeshRenderers不支持GPU instancing。
6. 不支持WebGL 1.0。


### 适用场景

1. GPU Instancing更适合同一个模型渲染多次的情况


### Shader开启Instancing功能

1. URPAssets -> Show Additional -> 关闭SRP Batcher
2. Material 中勾上 Enable GPU Instancing
3. 如果只在一个坐标上渲染多次模型是没有意义的，我们需要将一个模型渲染到不同的多个地方，并且需要有不同的缩放大小和旋转角度，以及不同的材质球参数。
4. GPU Instancing功能为我们提供了一个叫 InstancingID 的变量，这个变量可以确定当前着色计算的是第几个实例。Vertex Shader 和 Fragment Shader 可以通过这个变量获取模型矩阵，颜色等不同的参数，（烘焙动画可以通过InstancingID改变当前实例动画帧的偏移量）。

##### 简单的Demo

```cs
Shader "SimplestInstancedShader"
{
    Properties
    {
        _Color ("Color", Color) = (1, 1, 1, 1)
    }

    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 100

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            // 开启多实例变量的编译选项
            #pragma multi_compile_instancing 
            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                // 顶点着色器输入数据的 InstancingID 定义
                UNITY_VERTEX_INPUT_INSTANCE_ID 
            };

            struct v2f
            {
                float4 vertex : SV_POSITION;
                // 片元着色器输入数据的 InstancingID 定义
                UNITY_VERTEX_INPUT_INSTANCE_ID 
            };

            // 定义需要用到的多实例变量参数
            UNITY_INSTANCING_BUFFER_START(Props) 
                UNITY_DEFINE_INSTANCED_PROP(float4, _Color)
            UNITY_INSTANCING_BUFFER_END(Props)

            v2f vert(appdata v)
            {
                v2f o;
                // 装配 InstancingID
                UNITY_SETUP_INSTANCE_ID(v); 
                // 输入到结构中并传给片元着色器
                UNITY_TRANSFER_INSTANCE_ID(v, o); 

                o.vertex = UnityObjectToClipPos(v.vertex);
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                // 装配InstancingID
                UNITY_SETUP_INSTANCE_ID(i);
                // 提取多实例中当前实例的Color属性变量值
                return UNITY_ACCESS_INSTANCED_PROP(Props, _Color); 
            }
            ENDCG
        }
    }
}
```
