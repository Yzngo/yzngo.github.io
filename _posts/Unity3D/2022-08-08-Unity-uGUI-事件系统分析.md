![image-20230205200932624](https://cdn.jsdelivr.net/gh/yzngo/ImageHosting/img/202302052009661.png)



## Event System

1. Event System 组件本身仅仅作为整个事件系统的管理器，或者不同事件系统组件的辅助器。
2. 如果不需要处理玩家的输入事件，可以把 Event System 删掉。
3. **Event System 并非uGUI专用的，也可以用于其他用途。**
4. 工作流程：Unity输入系统检测到用户输入，Input Modules 处理用户输入，从摄像机的屏幕位置上通过 Raycasters 检测用户点击到的对象（通过坐标计算或射线检测），把输入数据存储到 Event Data 中，再通过 Messaging System 通知对象响应事件。



## Input Modules - 输入模块

![image-20230205201030135](https://cdn.jsdelivr.net/gh/yzngo/ImageHosting/img/202302052010165.png)



1.  Input Modules 承担了整个事件系统的业务逻辑，是事件系统的核心模块。
2. 默认的 `StandaloneInputModule` 对接的是 `Unity.Input`
3. 主要功能为：
    - 处理输入系统提供的输入事件
    - 管理事件状态
    - 发送事件到场景对象
4. 同一时刻只能有一个 Input Module 被激活，且必须和 Event System 组件放在同一个物体上。



## Raycasters - 射线投射模块

### Graphic Raycaster

1. 供 UI 对象使用。必须挂载在 [[Canvas]] 上，管理他下面的所有子UI物体的点击响应方式。
2. 通过**坐标计算**检测点击到的对象。
4. `Graphic Raycaster` 只对该Canvas下的点击交互起作用，而 `Physics` 类里面的API不影响UI上面的交互。
5. 遇到一些交互部件没响应时，可以检查这里是否出问题了。
6. 可以设置完全忽略输入的方式来彻底取消点击响应，还可以指定阻止对某些layers进行响应。


### Physics (2D) Raycaster

1. 供 3D (2D) physics 物体使用。必须挂载在 [[Camera]] 上。
2. 通过[[射线碰撞检测]]进行 3D (2D) 物体的点击判断。
3. 2D 以层级次序排序，3D 以距离摄像机的远近排序。

##### 3D物体响应事件需要满足的条件

1. **Physics (2D) Raycaster 挂载到相机上。**
2. 响应物体挂载[[碰撞体]]组件。
3. 编写实现 uGUI 的[[事件接口]]的脚本挂载到物体上。
4. 物体不被UI遮挡。

## Message System - 消息分发模块

1. 没有一个组件叫 Message System，它是指整个消息系统。
2. Message System消息系统用来代替 Unity 内置的 SendMessage。设法解决(address)了一些 SendMessage 存在的问题。
3. 消息系统一般用于松耦合的关系。比如：脚本响应触摸事件等。
4. 消息系统可以独立于uGUI使用。


### 自定义消息

```cs
using UnityEngine.EventSystems;
// 自定义消息接口
public interface ICustomMessageTarget : IEventSystemHandler
{
    // messaging system 可以调用的函数
    void Message1();
    void Message2();
}
// 查找实现了某消息接口的物体
```


### 接收消息

```cs
 // MonoBehaviour 实现接口以接收消息
public class CustomMessageTarget : MonoBehaviour, ICustomMessageTarget
{
    public void Message1()
    {
        Debug.Log ("Message 1 received");
    }

    public void Message2()
    {
        Debug.Log ("Message 2 received");
    }
}
```

### 触发消息

1. 通过messaging system 发送消息要指明消息类型+目标 GameObject，**目标GameObject上所有实现了该消息接口的组件都能够收到消息**。
2. 发送消息时可以指定一些自定义数据。也可以指定触发范围，仅在自己身上触发，还是包括parent 或者 children ？

```cs
using UnityEngine.EventSystems;
ExecuteEvents.Execute<ICustomMessageTarget>(targetObject, pointerEventData, (x,y)=>x.Message1());
ExecuteEvents.ExecuteHierarchy();
```



### Event Data - 事件参数

##### BaseEventData

1. 是所有事件数据的基类

##### PointerEventData

1. 点位事件数据类，为数据类的核心类，存储了大部分事件系统逻辑需要的数据，包括按下位置，松开与按下时间差，拖拽位移差，点击的物体等等，承载了所有输入事件需要的数据。


##### AxisEventData 

1. 轴类输入数据类，如手柄摇杆数据等。



## Event Trigger 组件

1. 事件触发器也需要 Image 组件用于接收事件。
2. 使用Image接收事件，使用EventTrigger触发业务函数。

### 使用 Event Trigger 动态绑定事件

```cs
private void Start()
{
    // 先给按钮添加EventTrigger
    EventTrigger et1 = button.AddComponent<EventTrigger>();
    et1.triggers = new List<EventTrigger.Entry>();
    EventTrigger.Entry entry1 = new EventTrigger.Entry();
    // 指定事件类型，PointerDown 指鼠标按下事件
    entry1.eventID = EventTriggerType.PointerDown;
    entry1.callback.AddListener(OnBtnEvent);   //  响应函数为OnBtnEvent
    // 将Entry添加到事件列表中，可以添加多个
    et1.triggers.Add(entry1);
}
```

### Event Trigger 示例

1. 使用EventTrigger对图片进行点击，拖拽，移入移出

```cs
  var self = _image.gameObject.AddComponent<EventTrigger>();
 
  self.triggers = new List<EventTrigger.Entry>(3)
         {
               new EventTrigger.Entry()
               {
                       eventID = EventTriggerType.BeginDrag,
                       callback = new EventTrigger.TriggerEvent()
                },
                new EventTrigger.Entry()
                {
                       eventID = EventTriggerType.Drag,
                       callback = new EventTrigger.TriggerEvent()
                 },
                new EventTrigger.Entry()
                 {
                      eventID = EventTriggerType.EndDrag,
                      callback = new EventTrigger.TriggerEvent()
                 }
         };
   
 
 
        //开始拖拽
        self.triggers[0].callback.AddListener((d) =>
        {
            objDrag = Instantiate(self.gameObject, self.transform.parent);
            objDrag.GetComponent<Image>().raycastTarget = false;
        });
        //拖拽中
        self.triggers[1].callback.AddListener((d) =>
        {
            if (objDrag != null)
            {
                Vector3 pos;
                if (RectTransformUtility.ScreenPointToWorldPointInRectangle(panel, Input.mousePosition, UICamera,out pos))
                {
                    objDrag.transform.position = pos;
                }
            }
         
        });
        //拖拽结束
        self.triggers[2].callback.AddListener((d) =>
        {
           
            
        });
```
