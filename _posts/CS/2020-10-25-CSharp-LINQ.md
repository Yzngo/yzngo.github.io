### LINQ

1. Language Integrated Query 语言集成查询。
2. `System.Linq` 命名空间为 `IEnumerable<T>` 接口提供了50多个扩展方法，以实现标准查询操作。

```cs
using System.Linq;
```

### 标准查询操作符

1. 大多数标准查询操作符不一定在赋值的时候即求值，确切的说，操作符中只是描述了返回的集合中应该有什么，而没有描述具体应该如何遍历数据项并生成新集合。
2. 只有在需要遍历集合中的项时，才会真正对表达式进行求值。
3. **查询操作符的谓词应该只对条件求值，不应该有任何副作用**（比如打印到控制台）。


### 查询表达式

1. **用着的时候再看**
2. Demo

```cs
IEnumerable<string> selection = 
    from word in words
        where word.Contains('*')
        select word;
        
```



----

### 过滤(筛选)操作符 

##### Where

1.  `System.Linq.Enumerable.Where()`
2. 从集合中筛选出符合[[CS-Delegate#Predicate|谓词]]要求的元素。
3. Where方法的结果是一个对象，它封装了根据一个给定谓词对一个给定序列进行筛选的操作。
4. 由于 `IEnumerable<T>.Where()` 输出的是一个新的 `IEnumerable<T>` 集合。
5. Where主要用于减少集合项的数量。
6. 可综合运用 Where 和 Select 来获得原始集合的子集。

```cs
string[] words = {"zero", "one", "two", "three", "four"};
// 注意和Select区分
words.Where(word => word.Length > 3);
// 结果： "zero", "three", "four"
words.Where((word, index) => index < word.Length);
// 结果： "zero", "one", "two", "three"
```

### 投影(投射)操作符 

##### Select

1. 依照给定的规则，由序列中的每一项生成另一个元素。即：返回从源元素到结果元素的一对一投影。
2. Select可以对数据进行完全的转换。

```cs 
string[] words = {"zero", "one", "two", "three", "four"};
words.Select(word => word.Length);
// 4，3，3，5，4
words.Select((word, index) => index.ToString() + ": " + word);	
// "0: zero"......
```


----


### 聚合操作符 

```cs


```
##### Sum 
##### Count

1. 使用 Count() 对元素计数。
2. **有Count属性的集合应首选属性，而不要用LINQ的Count()方法，可能会遍历整个集合**。

##### Average 
##### Min 
##### Max 
##### Aggregate

```cs
string[] words = {"zero", "one", "two", "three", "four"};
int[] numbers = {0, 1, 2, 3, 4};

numbers.Sum();    
// 求和   10
numbers.Count();     
// 计数   5
numbers.Count(n => n > 2);    
// 符合要求的为3,4 一共2个，结果为2
numbers.Average();    
// 求平均 2
words.Min(word => word.Length); 
// 求符合selector的最小的项     3
words.Max(word => word.Length); 
// 求符合selector的最大的项
numbers.Aggregate("seed", 
  (current, item) => current + item, 
              result => result.ToUpper()); 
// 结果 "SEED01234"
	
```



### 连接操作符 

##### Concat

```c#
string[] words = {"zero", "one", "two", "three", "four"};
int[] numbers = {0, 1, 2, 3, 4};

// 连接操作符 ---------------------------------------------------
numbers.Concat(new[] {2,3,4,5,6});		// 连接 0,1,2,3,4,2,3,4,5,6
    
```



### 转换操作符 

##### ToArray 
##### ToList 
##### ToDictionary 
##### ToLookup

```c#
string[] words = {"zero", "one", "two", "three", "four"};
int[] numbers = {0, 1, 2, 3, 4};
// 转换操作符 ---------------------------------------------------
numbers.ToArray();			// 转换为 int[]
numbers.ToList();			// 转换为 List<int>
words.ToDictionary(w => w.Substring(0,2));	// 每个键只能出现一次，否则会抛出异常
words.ToLookup(word => word[0]);
```



### 元素操作符 

##### ElementAt 
##### First 
##### Last 
##### Single

```c#
    // 元素操作符 --------------------------------------------------
    string[] words = {"zero", "one", "two", "three", "four"};
    int[] numbers = {0, 1, 2, 3, 4};
    
    words.ElementAt(2);				// "one"
    words.ElementAtOrDefault(10);		// null
    
    words.First();							// "zero"
    words.First(w => w.Length == 3);				// "one"
    words.First(w => w.Length == 10);				// 异常
    words.FirstOrDefault(w => w.Length == 10);		// null
    
    words.Last();				// "four"
    words.Single();					// 异常，不止一个
    words.SingleOrDefault();			// 异常，不止一个
    words.Single(word => word.Length == 5);			// "three"
    words.Single(word => word.Length == 10);			// 异常
    words.SingleOrDefault(w => w.Length == 10);			// null
    
```



### 相等操作符 

##### SequenceEqual

```c#

    // 相等操作符 --------------------------------------------------
	words.SequenceEqual(new []{"zero", "one", "two", "three", "four"});
    words.SequenceEqual(new[] {"zero", "one", "two", "three", "four"}, StringComparer.OrdinalIgnoreCase);


```



### 生成操作符 

##### DefaultIfEmpty 
##### Range 
##### Repeat

```c#
    // 生成操作符 --------------------------------------------------
    numbers.DefaultIfEmpty();	// 序列不为空，返回原始序列，否则返回含有单个元素的序列，其中的元素是相应类型的默认值
    Enumerable.Range(15, 2);		// 15，16
    Enumerable.Repeat(26, 2);		// 26，26
    
```



### 分组操作符 

##### ToLoopup 
##### GroupBy 
##### Take 
##### Skip 
##### TakeWhile 
##### SkipWhile

```c#

	// 分组操作符 --------------------------------------------------
	string[] words = {"zero", "one", "two", "three", "four"};
    int[] numbers = {0, 1, 2, 3, 4};
	
    words.ToLookup(word => word[0]);

    words.GroupBy(word => word.Length);			// 括号内为分组的键
    words.GroupBy(word => word.Length, word => word.ToUpper());
    
    words.Take(2);			// 取前2个
    words.Skip(2);			// 跳过前2个
    words.TakeWhile(word => word.Length <= 4);	// 取，直到不满足谓词要求 "zero", "one", "two"
    words.SkipWhile(word => word.Length <= 4);	// 跳过，直到不满足谓词要求 "three", "four"
```






### 检查操作符 

##### All 
##### Any 
##### Contains

```c#
    // 检查操作符 --------------------------------------------------
    words.All(word => word.Length > 3);				// 是否任意一个都满足	false
    words.Any();						// 存在元素					true
    words.Any(word => word.Length == 6);			// 是否存在一个满足		false
    words.Contains("FOUR");					// 是否包含			false
    words.Contains("FOUR", StringComparer.OrdinalIgnoreCase);		// 是否包含  true
    
```


### 集合操作符 

##### Distinct 
##### Intersect 
##### Union 
##### Except

```c#
    // 集合操作符 --------------------------------------------------
    string[] abbc = {"a", "b", "b", "c"};
    string[] cd = {"c", "d"};
    
    abbc.Distinct();		// 去重  "a", "b", "c"
    abbc.Intersect(cd);		// 交集  "c"
    abbc.Union(cd);			// 并集  "a", "b", "c", "d"
    abbc.Except(cd);		// 差集  "a", "b"
    cd.Except(abbc);		// 差集  "d"
    
```



### 排序操作符 

##### OrderBy 
##### Reverse

```c#
    words.OrderBy(w => w);
    words.OrderBy(w => w[1]);
    words.OrderByDescending(w => w.Length);
    words.OrderBy(w => w.Length).ThenBy(w => w);
    words.OrderBy(w => w.Length).ThenByDescending(w => w);
    words.Reverse();
```
