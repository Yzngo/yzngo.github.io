

### 移动

1. 适用于既没有物理系统，对移动又没有特殊要求的情况。
2. 不易处理障碍物阻挡的问题。

```cs
private float speed = 10f;
float v = Input.GetAxis("Horizontal");
float h = Input.GetAxis("Horizontal");
float vOffset = v * Time.deltaTime * speed;
float hOffset = h * Time.deltaTime * speed;


// 世界坐标系下移动
Vector3 moveDelta = new Vector3(vOffset, 0, hOffset);
transform.position += moveDelta;
transform.Translate(moveDelta, Space.World);

// 朝物体z轴方向移动
transform.position += transform.forward;	
transform.Translate(Vector3.forward, Space.Self);

// 朝世界坐标z轴方向移动
transform.position += Vector3.forward;
transform.Translate(Vector3.forward, Space.World);

```

3. 同时设置位置和旋转

```cs
transform.SetPositionAndRotation();
```

### 缩放


### 旋转

- Unity使用左手坐标系，所以旋转正方向是**顺时针**方向
- rotation 考虑到了所有父级叠加的旋转量，是世界空间的最终旋转量。

##### 直接设置旋转角度

```cs
// localRotation 是 inspector面板看到的旋转量，相对于父对象。
transform.localRotation = Quaternion.Euler(0, 0, -30);
```

##### 累计旋转量

- 自身累积（cumulative）旋转量，增量旋转
```cs
void Update()
{
    // 每秒旋转22.5度
    transform.Rotate(0f, 22.5f * Time.deltaTime, 0f);
}
```

##### 绕某点累积旋转

- 绕某点累积（cumulative）旋转的方法：
```cs
// point为世界空间中的点
// axis为旋转轴
// angle为旋转角度
transform.RotateAround(point, axis, angle);
```

##### 看向某个对象

```cs
transform.LookAt(attackTarget.transform);
```

##### 分次应用旋转

- [[四元数]]相乘，应用多次旋转，结合父级的旋转量
```cs
// 四元数相乘的顺序，先应用自身N，再应用父级... ...321（顶级）。
// 所以乘法顺序为 rotation = 123.....N。
Quaternion rotation = Quaternion.Euler(0, 0, -30);
transform.localRotation = grandTrans.localRotation * parentTrans.localRotation * rotation;
```



### 坐标变换

##### 本地坐标变换到世界坐标

1. 要注意 w 分量的取值， w=0 会忽略 position

```cs
// 使用 TransformPoint 函数
Vector3 worldPos = transform.parent.TransformPoint(localPosition);

// 使用 localToWorldMatrix 矩阵
// 变换向量 w = 0， 变换坐标 w = 1
worldPos = transform.localToWorldMatrix * new Vector4(localPos.x, localPos.y, localPos.z, 1);

// 使用 Matrix4x4 静态类构建TRS矩阵变换 （ Translation-Rotation-Scale）
Matrix4x4 trs = Matrix4x4.TRS(position, rotation, scale);
worldPos = trs * new Vector4(localPos.x, localPos.y, localPos.z, 1);
```

##### 世界坐标变换到本地坐标

```cs
Vector4 worldPos = new Vector4(x, y, z, 1);

localPos = transform.InverseTransformPoint(worldPos);
localPos = transform.worldToLocalMatrix * worldPos;
```

##### 本地向量变换成世界空间向量

```cs
transform.TransformDirection();
```

##### 世界空间向量变换成本地向量

```cs
transform.InverseTransformDirection();
```

### 修改自己的层级

1. 修改层级关系的几个函数

```cs
transform.SetAsFirstSibling();    // 层级最上（最先渲染）
transform.SetSiblingIndex(i);
transform.SetAsLastSibling();    // 层级最下（最后渲染）
```


### 操作子对象

```cs
// 获取子对象的个数
int childCount = transform.childCount;

// 通过名字获取子对象
Transform child = transform.Find("Text");

// 遍历第一层子对象(不包括自身和孙子节点)
foreach(Transform child in transform)
{
    Debug.Lod("Child: " + child.name);
}

// 通过名字查找子孙对象
private Transform DeepFindChild(Transform root, string name)
{
    Transform result = null;
    result = root.Find(name);
    if (result == null) {
        foreach(Transform transform in root)
        {
            result = DeepFindChild(transform, name)
            {
                if (result != null) {
                    return result;
                }
            }
        }
    }
    return result;
}
// 遍历所有子孙对象（包括自身和孙子节点）
Transform[] children = GetComponentsInChildren<Transform>();
foreach(Transform child in children)
{
    Debug.Log("Child or Self: " + child.name);
}
```

### 获取组件

```cs
Animator anim = GetComponent<Animator>();
// 获取自身和其所有子孙对象身上的所有组件
Animator[] animators = GetComponentsInChildren<Animator>();
```

### 不能直接对 transform.position.y赋值

1. transform.position是一个属性，提供了get和set方法，transform.position实际上相当于调用了一个函数，该函数返回了一个Vector3[[CS-值类型]]的拷贝，不是一个变量（variable）直接修改一个这样的临时值是无意义的。
2. 归根结底是因为transform.position返回的是一个值类型的临时量，不是因为它是一个属性（其实提供了set方法）
