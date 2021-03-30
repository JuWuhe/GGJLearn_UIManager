一月底被大佬们带飞了GGJ2021，写了一点点代码，总结并学习一下

- UI框架
- SceneManager框架

# UI框架

## UIType类

包含Name和Path两个属性，即该UI的名字和Prefabs的相对路径

> 因为在UIManager类中，用了Resources.Load方法，因此Path应该是**保存在Resources文件夹**下的Prefabs的相对路径

## BasePanel类

所有UIPanel类的父类，是一个抽象类

有管理进入操作、暂停操作、继续操作、退出操作的方法

> 举例：当设置界面弹出时，主界面上的按钮应该失去点击效果，因此需要暂停和继续操作

## UIManager类

管理所有UI信息，和所有UI的创建、销毁

### 存储UI信息

```c#
private Dictionary<UIType, GameObject> dicUI;
```

## PanelManager类

Panel管理器，采用栈存储UIPanel，入栈即显示该Panel，栈顶元素即为当前最前界面，非栈顶元素处于暂停状态

## UI Tool 类

管理UI，获取子组件、子对象组件

### 获取或添加一个组件 

```c#
public T GetOrAddComponent<T>() where T : Component
{
    //T：组件类型
    if (activePanel.GetComponent<T>() == null)
    {
        activePanel.AddComponent<T>();
    }

    return activePanel.GetComponent<T>();
}
```

### 获取子对象

```C#
public GameObject FindChildGameObject(string name)
{
    //name：子对象名称
    Transform[] trans = activePanel.GetComponentsInChildren<Transform>();

    foreach (Transform item in trans)
    {
        if (item.name == name)
        {
            return item.gameObject;
        }
    }
    Debug.LogWarning($"{activePanel.name}里没有{name}");
    return null;
}
```

### 获取子对象的组件

```c#
public T GetOrAddComponentInChildren<T>(string name) where T : Component
{
    GameObject child = FindChildGameObject(name);
    if (child)
    {
        if (child.GetComponent<T>() == null)
        {
            child.AddComponent<T>();
        }

        return child.GetComponent<T>();
    }

    return null;
}
```

## 具体Panel类的创建

继承BasePanel类，并将相对路径传入构造函数、重写方法

并且在OnEnter方法中通过UITool类中实现的方法获取Button子对象的Button组件来实现添加onClik监听器

```c#
UITool.GetOrAddComponentInChildren<Button>("ButtonName").onClick.AddListener(() =>
{
    //点击事件
});
```

# SceneManager

## SceneState类

场景状态，抽象类，有OnEnter方法和OnExit方法，分别是进入执行的操作和退出执行的操作

## SceneSystem类

场景状态管理

```c#
private SceneState sceneState;

//设置当前场景，并进入当前场景
public void SetScene(SceneState state)
{
    sceneState?.OnExit();
    sceneState = state;
    sceneState?.OnEnter();
}
```

["?."是什么？](#?.)

## 具体Scene类的创建

继承SceneState类，重写OnEnter和OnExit方法

并用PanelManager生成该场景初始UI面板

```c#
// 开始场景
public class StartScene : SceneState
{
    private readonly string sceneName = "Start";
    private PanelManager panelManager;
        
    public override void OnEnter()
    {
        panelManager = new PanelManager();
        if (SceneManager.GetActiveScene().name != sceneName)
        {
            SceneManager.LoadScene(sceneName);
            SceneManager.sceneLoaded += SceneLoaded;
        }
        else
        {
            panelManager.Push(new StartPanel());
        }
    }

    public override void OnExit()
    {
        SceneManager.sceneLoaded -= SceneLoaded;
    }

    // 场景加载完毕之后执行
    private void SceneLoaded(Scene scene, LoadSceneMode load)
    {
        panelManager.Push(new StartPanel());
    }
}
```

# GameRoot类

管理全局的一些东西，例如场景切换，需要绑定在Object上

采用单例模式

例：游戏开始时展示Start场景

```c#
private void Start()
{
    SceneSystem.SetScene(new StartScene());
}
```

# 总结

## 参考教程

[友人aa - [Unity编程]这大概是最好理解的UI框架了吧](https://www.bilibili.com/video/BV1Bz4y1D7rL)

<div id="?."></div>

[C#中 ??、 ?、 ?: 、?.、?[]](https://www.cnblogs.com/niyl/p/12987610.html)