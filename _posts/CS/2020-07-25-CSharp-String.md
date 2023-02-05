# C#中的string

### 几种不同的字符串

###### 插值字符串 $  

1. $@可以同时使用

```cs
string name = "XXX";
string school = "YYY";
Debug.Log($"demo {name} {school}");
```

1. 格式化字符串中的数值要.ToString()。为了解读内插字符串，系统需要创建由System.Object引用所构成的数组，以便将调用方所要输出的值放在这个数组里面，并交给由编译器所生成的方法去解读。但int属于值类型，要想把它当成System.Object来用，就必须装箱。此外，该方法的代码还需要调用ToString()，而这实际上相当于在箱子所封装的原值上面调用，也就是说，相当于编译器生成了这样的代码：

```cs
int i = 23;
Console.WriteLine(i);
// to -------------------------
int i = 23;
object o = i;
Console.WriteLine(o.ToString());
// 如果想避免装箱和拆箱，就需要提前把这些值手工地转换成string，然后传给WriteLine：
Console.WriteLine(i.ToString());
```

###### 无需转义逐字字符串 @

1. @字符串如果要输出 `"`, 需要 `""`转义，`""` 是唯一需要转义的，其他任何字符都不需要转义，而是直接输出。
2. $@可以同时使用

```cs
Debug.Log(@"demo    \\\         ");
```


##### 复合格式化字符串

1. 不同的语言语句顺序可能不同，所以本地化字符串不能简单的使用加法操作符连接。如果是运行时构造字符串，可以使用上边的插值字符串。
2. 但语言本地化一般会把字符串移动到某个资源文件中，这种情况下就没办法使用字符串插值了。在这种情况下，应该使用复合格式化。

```cs
Debug.Log(string.Format("demo {0} {1}", name, school);
```


### 字符串操作

##### 替换字符串

```cs
str = str.Replace(".prefab", "");
```

##### 比较字符串

```cs
// 不区分大小写比较两个字符串
// 左小右大<0，相等=0，左大右小>0
int comparison = string.Compare("str", "STR", true);
```

##### 检索字符(串)索引位置

```cs
// 返回"\\"字符在此实例中第一个出现的索引位置
// 从左到右搜索，下标是从0开始，如果未找到则返回-1.
int index = str.IndexOf("\\");
// 返回"\\"在此实例中最后一个出现的索引位置
// 从右向左搜索，第一次出现的"\\"的位置，如果未找到则返回-1.
int index = str.LastIndexOf("\\", 7);
```

##### 截取字符串

```cs
// 表示从下标7开始，截取长度为2的字符串
// Substring(7)表示从下标7开始，一直截取到字符串末尾
string subStr = str.Substring(7,2)。
```

### 解析数值字符串

1. 由于未定义从字符串到数值类型的转换，因此需要使用像TryParse()这样的方法。每个数值数据类型都包含一个TryParse()方法，允许将字符串转换成对应的数值类型。

```cs
bool success = int.TryParse("123", out int a);
bool success = double.TryParse("123.456", out double b);
bool success = float.TryParse("123.456", out float c);
```



### 格式化字符串基本语法

1. [复合格式设置 | Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/standard/base-types/composite-formatting)

```cs
// 十六进制输出数字
Debug.Log($"0x{42:X}");
```

### null和""空字符串

1. `null` 意味着变量无任何值，string不指向任何引用.
2. `""` 表示一个空字符串.
3. 可以给null和""不同的含义，比如将null解释称“家庭电话未知”，将""解释成“无家庭电话”。


### 字符串长度

1. 字符串长度不能更改，因为字符串不可变（immutable）。
2. 因为字符串不可变，所以每次调用字符串的接口都是返回一个新的字符串，必须接收返回值，不然没有意义，比如 `string text = text.ToUpper();`


### StringBuilder

#### 命名空间

1. `System.Text.StringBuilder`

#### string的缺点

1. [[String]] 的值是不可变的，每次对[[String]] 的操作都会生成新的 [[String]] 对象，然后指向新的对象。
2. 效率低下，大量浪费内存空间。


#### StringBuilder的优点

1. StringBuilder 是可变的，且是线程安全的。每次操作不会生成新的对象，而是修改缓冲区的数据。
2. 每个 StringBuilder 对象都有一定的缓冲区容量。


#### StringBuilder的局限性

1. StringBuilder 只能减少字符串中间状态的分配，不能免除字符串的内存分配。在执行 ToString() 时还是会重新分配一个字符串。
