### Task

1. 任务是可能出现高延迟的工作单元，作用是产生结果值或者希望的副作用。
2. **任务**代表一个异步工作，而**线程**是做这件工作的工作者。
3. 任务和委托类似，区别是委托同步执行，任务把委托从同步执行转变为异步执行，即从线程池中取一个工作者线程[[异步]]执行任务。
4. 启动一个任务，控制几乎立刻返回给调用者，无论这个任务要执行多少工作。
5. Task类：**System.Threading.Tasks**。
6. 任务并行库 TPL  -  Task Parallel Library。
7. `Task`承诺(Promise)在未来(Future)的某个时刻，其内部会含有值或错误。


### 基于任务的异步模式

1. TAP - Task-based Asynchronous Pattern。
2. TAP使用 async/await 让编译器来承担异步程序的复杂性，从而使程序员可以集中精力实现真正的业务逻辑。

#### async

1. 使用 async 修饰符声明的方法，其中可包含异步方法调用，并使用 await 表达式等待异步方法结束。
2. 具有 async 修饰的方法也会运行于调用者线程中，如果其中没有用 await 等待任何“可等待”的任务，则它会像普通方法一样在调用者线程中同步执行；仅仅使用 async 关键字并不会改变执行线程。
3. 从调用者的角度看，调用具有 async 关键字的方法与调用普通方法并无差别，它仅仅只是一个能返回异步类型的方法。

#### await

1. await 阻塞的是当前执行的方法，而非当前线程。在异步方法中遇到 await 关键字时，当前的执行路径会分成两条：一条为当前线程，它会立即返回到调用者继续执行后续的代码；一条为自动分配的子线程，这个子线程会等待 await 右边的异步任务完成，之后继续执行异步方法中的后续代码。
2. await 操作符会自动将逻辑上的返回值从 Task 对象中提取出来。

```cs
private static void PrintPageLength()
{
    Task<int> lengthTask = GetPageLengthAsync("http://google.com");
    Debug.Log(lengthTask.Result.ToString());
}

private static async Task<int> GetPageLengthAsync(string url)
{
    using (HttpClient client = new HttpClient())
    {
        Task<string> fetchTextTask = client.GetStringAsync(url);
        int length = (await fetchTextTask).Length;  
        return length;
    }
}
```


### Task API

##### 检查状态

```cs
// 检查任务的状态
task.IsCompleted
task.Status
```

##### 读取返回值

```cs
// 读取返回值，此操作会自动造成当前线程阻塞
task.Result
```

### 有效异步返回值类型

1. 对于一个异步方法，只有在作为事件订阅者时才应该返回void，在其他不需要特定返回值的情况下，最好将方法声明为返回`Task`，这样调用者可以等待操作完成，以及探测失败情况。
2. 相比异步任务，创建 Task 对象的开销基本可以忽略，所以不要返回 null，不然调用者还需要检查是否为 null。
3. await 支持的返回类型为可等待类型，即任何包含 **GetAwaiter()** 方法的类型。

##### Task

1. 如果一个异步方法没有返回值，就使用 Task。
2. 如果异步方法中的操作**一定异步完成**，或者可能缓存常见的结果Task对象，首选 `Task<TResult>`


```cs
// 不返回对象，但可以通过Task检查任务状态
Task
// 返回可供调用者使用的对象
Task<TResult>
```

##### ValueTask

1. 当异步方法无需执行异步任务而提前返回时，该类型可以提供轻量级的对象构造。
2. 如果操作**可能同步完成**并且不能有效的缓存所有常见的返回值，则`ValueTask<T>` 更合适。

```cs
using System.IO.Compression;
private static async ValueTask<byte[]> CompressAsync(byte[] buffer)
{
    if (buffer.Length == 0) {
        return buffer;
    }
    using MemoryStream memoryStream = new MemoryStream();
    using GZipStream gZipStream = new GZipStream(memoryStream, CompressionMode.Compress);)
    await gZipStream.WriteAsync(buffer, 0, buffer.Length);
    return memoryStream.ToArray();
}
```

##### IAsyncEnumerable

```cs
IAsyncEnumerable<T>
```

##### IAsyncEnumerator

```cs
IAsyncEnumerator<T>
```





### 任务取消

1. 任务会定期轮询 CancellationToken **令牌对象**，检查是否发出了取消请求。
2. CancellationToken 是 struct，可以拷贝值。

```cs
CancellationTokenSource cancellationTokenSource = new CancellationTokenSource();
Task task = Task.Run( () => Func(), cancellationTokenSource.Token);
// 取消任务
cancellationTokenSource.Cancel();
```
