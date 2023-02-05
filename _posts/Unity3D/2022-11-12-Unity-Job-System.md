

[官方文档](https://docs.unity3d.com/Manual/JobSystem.html)

### blit 和 blittable type 是什么？

1. blit 是位块传送的意思。内存拷贝操作（memory copy operation）有时候被称作位块传送（block transfer），简称为 bit blit。专用于 block transfer 的硬件被称作位块传送器（blitter）。
2. 能够使用 block transfer 的类型在C#中被称作 blittable，即可位块传送的类型。
3. Blittable 类型是指在托管代码和本机代码中具有相同位级别表示形式的类型。在托管代码和本机代码间[[Marshal|封送]]时无需进行转换。
4. 由于 blittable 类型性能高，应首选这些类型。


### C#中的 blittable 类型有哪些？

1. blittable基础类型：byte, sbyte, short, ushort, int, uint, long, ulong, single, double
2. 由blittable基础类型构成的非嵌套一维数组，例如：`int[]`
3. 仅包含 blittable基础类型 的固定布局的值类型（farmatted value types that contain only blittalbe type）,如 Vector3
4. Object reference 不是 blittable 类型
5. 托管字符串（managed strings）不是 blittable 类型
6. char, bool 也不是 blittable 类型


### NativeContainer

1. NativeContainer 本身是一种托管值类型（struct），为本机内存提供相对安全的C#包装器，它们包含指向非托管分配内存的指针，这些非托管内存在常规的托管内存堆之外，可以被我们的C#代码访问，从而规避了默认的内存管理开销。
2. 与 JobSystem 一起使用时，NativeContainer 允许 JobSystem 访问与主线程共享的数据，而不是使用副本。
3. 定义在`Unity.Collections`命名空间中的NativeContainer类型 
    - `NativeArray` - 可以使用 `NativeSlice` 获取 NativeArray 的子集。
    - `NativeList` - 可调整大小的 `NativeArray`
    - `NativeHashMap` - 键值对
    - `NativeMultiHashMap` - 每个键可以有多个值的键值对
    - `NativeQueue` - 队列
4. 使用Demo
    ```cs
    // 初始化
    NativeArray<float> result = new NativeArray<float>(1, Allocator.TempJob);
    // 释放（处置）
    result.Dispose();
    ```


### Jobs

1. `Jobs` 不能和 objects 一起工作，使用的成员变量只能是 [[blittable]] 或 [[NativeContainer]] 类型。
2. 从主线程访问 `Jobs` 中的数据只能通过 `NativeContainer`，`blittable` 类型的数据，都工作在副本上

#### Job使用流程

1. Mono类中定义Job struct
2. 托管代码中写入 Job 需要的数据
3. 调度Job做运算，写回数据，用于游戏逻辑


#### Job 接口

| Interface                | 用法                                                       |
| ------------------------ | ---------------------------------------------------------- |
| IJob                     | 调度一个与主线程和其他Jobs并行的单独Job，仅执行一次        |
| IJobFor                  | Job接口中最灵活的一个接口 (for代表for循环)                 |
| IJobParallelFor          | 对于 NativeArray 中的每一项数据，分别并行执行一次Execute() |
| IJobParallelFprTransform | 类似IJobParallelFor，专门用于操作 Transforms               |

#### JobHandle

1. JobHandle可用于确保 job 完成了。
2. 也可以作为依赖传递给其他的 job，用于确保这些 job 按顺序依次执行。


#### IJob

1. IJob Demo  [IJob](https://docs.unity3d.com/ScriptReference/Unity.Jobs.IJob.html)

    ```c#
    public class ApplyVelocitySample : MonoBehaviour
    {
    // 每个 Job 都是实现 IJob 接口的 struct
    private struct VelocityJob : IJob
    {
        // Jobs声明了此Job可以访问的所有数据
        // 数据必须是 blittable，或者 NativeContainer
        
        // 通过把数据声明为ReadOnly, 多个Jobs允许并行的访问此数据
        [ReadOnly] public NativeArray<Vector3> velocity;
        // 容器默认是 read & write
        public NativeArray<Vector3> position;
        // Job需要能够在 worker threads 上独立的执行，需要知道的所有信息
        public float deltaTime;
    
        public void Execute()
        {
            for (var i = 0; i < position.Length; i++)
                position[i] += velocity[i] * deltaTime;
        }
    }
    
    public void Update()
    {
        // 没有垃圾回收，这样写没问题，每帧 new 值类型是没问题的
        // NativeContainer只是个壳子，里面有个指针
        var position = new NativeArray<Vector3>(500, Allocator.Persistent);
        var velocity = new NativeArray<Vector3>(500, Allocator.Persistent);
        for (var i = 0; i < velocity.Length; i++)
        {
            velocity[i] = new Vector3(0, 10, 0);
        }
        // 初始化Job的数据
        var job = new VelocityJob
        {
            deltaTime = Time.deltaTime,
            position = position,
            velocity = velocity
        };
        
        // 调度方法以扩展方法的形式实现
        
        // 1.在单一工作线程上调度 job, 返回的 JobHandle 可以使进行后续操作
        JobHandle jobHandle = job.Schedule();
        // 2.直接在主线程上调度 job，无并行性
        job.Run();
        // 确保 job 已完成
        // 不建议这样做，因为这样实际上丧失了并行性。
        jobHandle.Complete();
        
        // 读取 job 运算之后的结果
        Debug.Log(job.position[0].ToString());
        // Native arrays 必须手动释放
        position.Dispose();
        velocity.Dispose();
    }
    }
    
    ```
    

#### IJobFor

1. IJobFor Demo [IJobFor](https://docs.unity3d.com/ScriptReference/Unity.Jobs.IJobFor.html)

```cs
public class ApplyVelocityParallelForSample : MonoBehaviour
{
    private struct VelocityJob : IJobFor
    {
        [ReadOnly]
        public NativeArray<Vector3> velocity;
        public NativeArray<Vector3> position;
        public float deltaTime;
        public void Execute(int i)
        {
            position[i] += velocity[i] * deltaTime;
        }
    }

    public void Update()
    {
        var position = new NativeArray<Vector3>(500, Allocator.Persistent);
        var velocity = new NativeArray<Vector3>(500, Allocator.Persistent);
        for (var i = 0; i < velocity.Length; i++)
        {
            velocity[i] = new Vector3(0, 10, 0);
        }
        var job = new VelocityJob()
        {
            deltaTime = Time.deltaTime,
            position = position,
            velocity = velocity
        };

        // 直接在主线程上调度 job，第一个参数为执行的迭代次数
        job.Run(position.Length);
        
        // 之后的某个时间点，在单一工作线程上调度此job
        JobHandle scheduleJobDependency = new JobHandle();
        JobHandle scheduleJobHandle = job.Schedule(position.Length, scheduleJobDependency);
        // 在并行的 N 个工作线程上调度此job
        // 第二个参数为 batch size, 即每个batch的任务数量；这个参数怎样设置？
        JobHandle scheduleParallelJobHandle = job.ScheduleParallel(position.Length, 64, scheduleJobHandle);

        // It is not recommended to Complete a job immediately,
        // since that reduces the chance of having other jobs run in parallel with this one.
        // You optimally want to schedule a job early in a frame and then wait for it later in the frame.
        scheduleParallelJobHandle.Complete();

        Debug.Log(job.position[0].ToString());
        position.Dispose();
        velocity.Dispose();
    }
}

```

#### IJobParallelFor

1. IJobParallelFor Demo [IJobParallelFor](https://docs.unity3d.com/ScriptReference/Unity.Jobs.IJobParallelFor.html)

![image-20230205202743704](https://cdn.jsdelivr.net/gh/yzngo/ImageHosting/img/202302052027787.png)

1. Demo

```c#
 public class ApplyVelocityParallelForSample : MonoBehaviour
{
    private struct VelocityJob : IJobParallelFor
    {
        [ReadOnly]
        public NativeArray<Vector3> velocity;
        public NativeArray<Vector3> position;
        public float deltaTime;
        public void Execute(int i)
        {
            position[i] += velocity[i] * deltaTime;
        }
    }

    public void Update()
    {
        var position = new NativeArray<Vector3>(500, Allocator.Persistent);
        var velocity = new NativeArray<Vector3>(500, Allocator.Persistent);
        for (var i = 0; i < velocity.Length; i++)
        {
            velocity[i] = new Vector3(0, 10, 0);
        }
        var job = new VelocityJob
        {
            deltaTime = Time.deltaTime,
            position = position,
            velocity = velocity
        };

        JobHandle jobHandle = job.Schedule(position.Length, 64);
        jobHandle.Complete();
        Debug.Log(job.position[0].ToString());
        position.Dispose();
        velocity.Dispose();
    }
}
```


#### Parallel Job Batch Size

1. 对于简单的任务，比如把几个Vector3向量相加，，batch size 可以是 32 到 128
2. 对于昂贵的任务，batch size 最好小一些，甚至设为 1 都是可以接受的

### Tips

1. Don’t try to update NativeContainer contents.

     `nativeArray[0]++;` is the same as writing `var temp = nativeArray[0]; temp++`

    ```c#
    MyStruct temp = myNativeArray[i];
    temp.memberVariable = 0;
    myNativeArray[i] = temp;
    ```

    
