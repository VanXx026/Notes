# Unity 委托Delegate 和 事件Event

笔记的内容是结合了BraverJoe和刘铁猛老师的委托和事件视频来做的，他们两个讲的非常好。



## 委托 Delegate

### 1. 什么是委托

委托**是一种类**Class，类是一种为 Reference Type（引用）的数据类型，所以委托也是**一种为引用类型的数据类型**。

同样的，类可以声明变量，可以创建实例，那么**委托也可以声明变量，创建实例**。

（数据类型就是string、int、float、bool、你通过class关键字创建的Player类，Player也是数据类型、你通过delegate创建的MyDelegate委托，MyDelegate也是数据类型）

如何证明委托是一种类？我们可以通过在命名空间中的Type类的方法IsClass来判断我们自定义声明的委托是不是一个类

```c++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;

public class DelegateEx_01 : MonoBehaviour
{
    private delegate void MyDelegate(); // 通过delegate关键字声明一个委托类型MyDelegate
    private MyDelegate myDelegate; // 数据类型：MyDelegate、变量名：myDelegate，就像类和对象一样，类名表示类类型。

    private void Start()
    {
        // 查看我们声明的委托MyDelegate是不是一个类
        Type type = typeof(MyDelegate);
        Debug.Log(type.IsClass);
    }
}

```

可以看到，委托确实是一种类。

![image-20220928125316609](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220928125316609.png)

说个题外话，C#中的数据类型有两大类：引用类型和值类型，大体分类如下图：

![image-20220928125552404](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220928125552404.png)



### 2. 委托为什么重要

先来点真实的：因为面试会考。大部分的公司面试的时候都会问到有关委托和事件的问题。

来点技术上的：事件的基本是委托、Lambda表达式的基础也是委托，Lambda表达式又是LINQ（不知道是啥）的基础，而这些东西又非常重要。所以学好委托，才能理解事件和Lambda表达式。



### 3. C语言的函数指针

**委托其实就是C/C++中函数指针的升级版**，我们可以发现，C#中对委托的声明其实和C/C++中对函数指针的声明非常像，同样都需要指明引用的方法/函数的返回值和参数表，只有这个方法/函数的返回值和参数表与委托和函数指针中声明的返回值和参数表才能被引用，即**类型兼容**。

下面是C语言中函数指针的小栗子：

```c++
#include <iostream>
using namespace std;
// typedef：将Calculator定义为一个类型 
typedef int (* Calculator)(int x, int y); // 函数指针声明

int Add(int x, int y)
{
	return x + y;
}

int Sub(int x, int y)
{
	return x - y;
}

int main()
{
	// 直接调用Add和Sub 
	cout << "直接调用Add:" << Add(200, 100) << endl;
	cout << "直接调用Sub:" << Sub(200, 100) << endl;
	// 通过函数指针间接调用Add和Sub
	Calculator calc01 = &Add;
	Calculator calc02 = &Sub;
	cout << "间接调用Add:" << calc01(200, 100) << endl;
	cout << "间接调用Sub:" << calc02(200, 100) << endl;
    /*得到的结果和直接调用是一样的*/
}
```

C#中，确实没有指针这部分概念，但是C#还是以委托的形式保留了C/C++中函数指针的这一部分功能。而Java中，为了安全性，禁止程序员直接访问内存地址，所以把指针相关的内容舍弃的非常彻底，也就没有函数指针对应的功能实体了。



### 4. 自定义委托的声明和创建

委托虽然说是一种类，但是委托的声明格式却不像类的声明那样，反而是像C/C++中函数指针的声明格式。

之所以这样做，有两个目的：

1. 为了照顾可读性
2. 保持C/C++的传统

委托的自定义声明的确很像C/C++的函数指针：

```c#
/// 声明一个委托MyDelegate，此时的MyDelegate是一种引用类型的数据类型，不同于值类型int之类的数据类型
// delegate：委托关键字
// void：委托要引用的方法的返回值
// 括号：委托要引用的方法的参数列表
private delegate void MyDelegate();
```

然后，我们根据这个委托声明一个对象（或者说变量）

```c#
/// 声明一个MyDelegate类型的变量myDelegate
private MyDelegate myDelegate;
```

我们在脚本内有三个方法：Teleport、ChangeColor、Log

```c#
// 随机变换位置
public void Teleport()
{
    Vector2 curPos = transform.position;
    curPos.x = UnityEngine.Random.Range(-5f, 5f);
    transform.position = curPos;
}

// 随机变换颜色
public void ChangeColor()
{
    // 注意：由于UnityEngine和System这两个命名空间下都有Random类，所以这里会报错，手动写一下命名空间就行了
    spriteRenderer.color = new Color(UnityEngine.Random.value, UnityEngine.Random.value, UnityEngine.Random.value);
}

// 输出日志信息
public void Log()
{
    // 显示的是格林尼治时间    
    Debug.Log("Current Time is:" + DateTime.UtcNow);
}
```

我们的需求是，装载了这个脚本的金币，在我们按下space键之后，按照顺序依次执行这三个方法。

我们可以通过直接调用的方法：

```c#
private void Update()
{
    if (Input.GetKeyDown(KeyCode.Space))
    {
        // 直接调用三个方法
        Teleport();
        ChangeColor();
        Log();
    }
}
```

我们也可以通过委托来间接调用这些方法：

首先，我们对声明的MyDelegate类型的变量myDelegate初始化：

```c#
// 这是标准的写法
myDelegate = new MyDelegate(Teleport); // 实例化括号内填的是返回值和参数表与这个委托类型声明时兼容的方法，写方法名即可
// 还有比较便利的写法
myDelegate = Teleport; // 这种写法更能表达出myDelegate引用了Teleport方法的意思
```

那么，现在问题来了，初始化我确实是初始化了，但是我希望的是既然我三个方法返回值都是void，参数表也是空的，那么怎么样用这一个委托变量来存储我三个方法的引用呢？

可以这样吗？

```c#
myDelegate = new MyDelegate(ChangeColor);
myDelegate = new MyDelegate(Log);
```

答案很显然是不行的，这就相当于先给a赋值为100，然后又给a赋值90，最终a为90是一样的，后面的实例化会把前面的实例化给覆盖掉。

而我们上面的初始化写法是【单播委托】的写法。

**单播委托**：一个委托只引用了一个方法的形式。这种写法委托只能装载一个方法的引用，所以很不方便。

为了解决我们这个问题，我们可以使用【多播委托】。

**多播委托**：一个委托引用多个方法的形式。

那么多播委托怎么初始化呢？很简单，使用 +=

```c#
// 这是标准语法的写法
myDelegate += new MyDelegate(Teleport);
// 也可以这样写 便利写法
myDelegate += ChangeColor;
myDelegate += Log;
```

这样，我们就可以把三个方法的引用都存储在同一个委托类型的变量内了。

初始化我们完成了，接下来就是通过委托实现间接调用了

```c#
private void Update()
{
    if (Input.GetKeyDown(KeyCode.Space))
    {
        /// 使用委托间接调用
        // 第一种写法，使用委托类型自带的方法Invoke，invoke有援助，帮忙的意思
        myDelegate.Invoke();
        
        // 第二种写法，像函数指针一样，直接写上委托变量，在后面加上一个括号表示调用
        myDelegate();
    }
}
```

这就是我们自定义委托的声明、创建变量、初始化和基本使用了。

然而在Unity里面我们一般不用自定义的委托，C#已经帮我们封装好了两个可用的委托类型：Action委托和Func委托，我们就不需要自己用delegate关键字自己去创建了。



### 5. 委托的缺点

这些缺点是**事件**、**观察者模式**出现的原因。

1. 委托使用不当会导致内存泄露和程序性能下降。
   - 委托存储了该对象方法（非静态）的引用，因为只是引用，所以要想调用这个方法，还得去对象里面拿，所以这个对象必须一直留在内存里面，等待委托去通过引用拿到对应的方法。委托用得太多的话，久而久之，就会造成内存泄漏导致性能下降。
   - 写多播委托有可能会犯 += 写成 = 的情况，人为地导致委托封装的方法被覆盖。
2. 滥用委托会导致代码可读性急剧性地下降，debug难度也很高。
   - 明明没有必要使用委托却盲目使用委托。
   - 出现委托变量的参数里面还有委托类型的参数这种嵌套委托的情况
3. 委托是方法层面上的耦合（如果把对象比作房间，那么委托这种方法级别上的耦合就相当于穿透墙壁去进行的耦合），违反设计模式，使用委托要谨慎。
4. 把模板方法、回调方法、异步回调和多线程纠缠在一起，不仅可读性非常差还很难维护。

简而言之，不要因为委托的优点而去滥用它，而是在深知它的优点和缺点的情况下，权衡利弊，合理地去使用它。



### 6. Action委托和Func委托

在大多数情况下，我们在写项目的时候，是**不需要自己拿delegate关键字去声明一个委托类型**的，因为C#类库（所以需要引用**System命名空间**才能用）已经为我们准备好了两个内置委托类型：Action委托和Func委托。

#### Action委托（无返回值）

Action委托**只能够**引用（或者说指向）一个**返回值为空**的目标方法，**参数列表有或者没有都可以**。

```c#
/// 泛化的Action委托类型
// in：Input，表示T是输入参数
// 尖括号内最多支持16个输入参数，也就是参数列表最大为16
// 但是不管参数有几个，没有都可以，引用的方法的返回值一定是void
Action<in T, ...>
```

比如还是我们自定义委托的例子，还是那三个返回值为void，参数列表为空的方法：Teleport、ChangeColor、Log

我们这次用Action来试试。

首先，声明一个Action委托类型的变量，变量名为action。

```c#
// 通过Action委托类型，创建一个Action类型的变量，返回值为void，参数列表为空
Action action;
```

然后，在OnEnable方法中，把我们方法的引用保存在action中（多播委托）

```c#
// 在awake之后，start之前调用
private void OnEnable()
{
    // 当然用简便写法也是可以的
    action = new Action(Teleport);
    action += new Action(ChangeColor);
    action += new Action(Log);

    // action01 = new Action<string, float>(SayHello);

    func = new Func<double, double, double>(Add);
}
```

最后，在Update方法中通过action间接调用就可以了。

```c#
private void Update()
{
    if (Input.GetKeyDown(KeyCode.Space))
    {
        action.Invoke();
        action();

        // action01.Invoke("UZI", 0);
        // action01("Faker", 3);

        func.Invoke(3f, 4f);
        func(30f, 40f);
    }
}
```



当然，Action委托类型也支持有参数的，比如我们写了一个返回值为void，参数列表为(string, float)的方法。

写法是差不多的，只是声明这个Action类型的变量不一样而已。

```c#
// 通过Action委托类型，创建了一个Action类型的变量，返回值为void，参数列表(string, float)
Action<string, float> action01;

// 在awake之后，start之前调用
private void OnEnable()
{
    action01 = new Action<string, float>(SayHello);
}

private void Update()
{
    if (Input.GetKeyDown(KeyCode.Space))
    {
        // 大胆！
        action01.Invoke("UZI", 0);
        action01("Faker", 3);
    }
}

public void SayHello(string playerName, float num)
{
    Debug.Log(string.Format("{0} has {1} championships now", playerName, num));
}
```



#### Func委托（有返回值）

Func委托**只能够**引用一个**有返回值**的方法，**参数列表有没有都可以**。

```c#
/// 泛化的Func委托类型
// in：Input，表示T是输入参数
// out：Output，表示是输出参数，在尖括号的最后一位
// 一定要有返回值，参数有没有都无所谓
Func<in T, ..., out TResult>
```

Func委托正好用在和Action委托不同的地方，这个委托类型一定要有返回值，不然就滚去用美国人的Action去，鉴不鉴啊。

我们写了一个返回值为double，参数列表为(double, double)的Add方法。

```c#
// 通过Func委托类型，创建了一个Func类型的变量，返回值为double，参数列表(double, double)
Func<double, double, double> func; // 最后一个是返回值的类型

// 在awake之后，start之前调用
private void OnEnable()
{
    func = new Func<double, double, double>(Add);
}

private void Update()
{
    if (Input.GetKeyDown(KeyCode.Space))
    {
        func.Invoke(3f, 4f);
        func(30f, 40f);
    }
}

public double Add(double x, double y)
{
    Debug.Log(x + y);
    return x + y;
}
```





### 7.  委托的一般使用情况【将委托作为参数传递到方法中】

在第一节我们已经说过了：委托是一种类，是引用类型的数据类型。既然委托是一种类，那么类可以声明变量，创建实例；我们的委托类型同样也可以声明变量，创建实例，当然这些都不是重点。

关键是，委托是一种引用类型，可以作为参数传递到方法中。

所以，委托的一般使用情况就是：**将委托作为参数传递到方法中**。

将委托作为参数传递到方法中，要的效果就是**动态调用委托引用的方法**。



我个人理解，模板方法和回调方法的不同在于：相对来说模板方法的逻辑是不完整的，回调方法的逻辑是完整的。模板方法依赖委托参数可选地**补全**逻辑，回调方法根据自己的逻辑决定要不要使用委托参数来**附加**逻辑。



#### 模板方法（使用Func）

模板模板，什么是模板，写过作文没有，什么叫套模板，什么叫背模板。就是在说，这个模板有大部分的内容都是固定下来的（一些华而不实的空话），剩下自己套进去的关键词是**不确定**的，要根据题目的具体情况具体分析。

那么，模板方法的意思就是：

这个方法是个模板，模板里有一处是不确定的，那么这个不确定的部分就要靠这个委托类型的参数引用的方法来给到你，所以模板方法中使用的委托参数一般都是要有返回值的**（Func）**，返回值交给方法，**填补**这个不确定的部分，让这个模板正常运行。

所以，模板方法就用在：

这个方法**有一小段逻辑是需要动态的去选择的（比如我想吃苹果，就返回苹果，想吃西瓜，就返回西瓜，这些选择就作为引用存在委托参数中）**，根据场景不同就选择不同的逻辑，并且整个方法需要依赖这个逻辑去运行（返回值）。



#### 回调方法（Action）

要想理解回调方法，首先需要理解什么是回调（打从学编程开始就偶尔能冷不丁地看到回调出现，但是就是不懂什么意思，难绷）。

先来讲一个例子：你走在大街上，“游泳健身了解一下”，游泳教练递给你一张传单，让你办卡。你带回了家，把传单放在了桌子上，很快就忘记了这件事。到了暑假，你想去游泳锻炼一下，突然记起之前收到过传单，就马上联系了那位游泳教练办了张卡。

当然，我有需求的情况下，我就会去找他办卡，当我没有这个需求的时候，我这辈子都不会去联系他。这时，我和这位游泳教练之间就形成了一种**回调关系**：**我需要的时候我就去找他，不需要的时候我就不找他。**

回调方法也是一样的，游泳教练就好比委托类型的参数，我就好比这个主逻辑方法。方法需要的时候，我就去找委托参数要方法来调用；方法不需要的时候，就不调用，我们的方法就可以通过逻辑判断要不要去调用这个方法。

所以说，回调方法在主逻辑中某些情况需要，某些情况不需要，大体上的逻辑是完整的，所以一般回调方法是没有返回值的，也就考虑使用Action委托类型来引用回调方法。



### 8. 委托一般使用情况实例

#### 模板方法实例 1

在这里，我需要在GameManager中实现得到每一个玩家信息（击杀数、死亡数、夺旗数）中最大的那个玩家的名字。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameManager : MonoBehaviour
{
    public static GameManager instance;
    public List<PlayerStatus> playerStatusList = new List<PlayerStatus>();

    private void Awake()
    {
        if (instance != null)
        {
            Destroy(gameObject);
        }
        instance = this;
        DontDestroyOnLoad(this);
    }

    private void Start()
    {
        Debug.Log("the most kill is " + GetTopKillNum());
        Debug.Log("the most death is " + GetTopDeathNum());
        Debug.Log("the most flag is " + GetTopFlagNum());
    }

    // 获取最高击杀数的玩家名
    public string GetTopKillNum()
    {
        string topName = "";
        int bestRecord = 0;

        foreach (PlayerStatus playerStatus in playerStatusList)
        {
            int tempNum = playerStatus.killNum;
            if (tempNum > bestRecord)
            {
                topName = playerStatus.playerName;
                bestRecord = tempNum;
            }
        }

        return topName;
    }

    // 获取最高死亡数的玩家名
    public string GetTopDeathNum()
    {
        string topName = "";
        int bestRecord = 0;

        foreach (PlayerStatus playerStatus in playerStatusList)
        {
            int tempNum = playerStatus.deathNum;
            if (tempNum > bestRecord)
            {
                topName = playerStatus.playerName;
                bestRecord = tempNum;
            }
        }

        return topName;
    }

    // 获取最高夺旗数的玩家名
    public string GetTopFlagNum()
    {
        string topName = "";
        int bestRecord = 0;

        foreach (PlayerStatus playerStatus in playerStatusList)
        {
            int tempNum = playerStatus.flagNum;
            if (tempNum > bestRecord)
            {
                topName = playerStatus.playerName;
                bestRecord = tempNum;
            }
        }

        return topName;
    }
}
```

使用委托，就可以做到如下的简化，可读性更强，维护起来也更加方便，玩家信息如果不仅仅只有击杀数、死亡数、夺旗数，还新增了其他信息的话，那么就只需补充一个简单的方法，再让这个委托引用就可以了。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public delegate int GetTopScoreDelegate(PlayerStatus player); // GetTopScoreDelegate委托类型，可以引用返回值为int，参数为PlayerStatus的方法

public class GameManager : MonoBehaviour
{
    public static GameManager instance;
    public List<PlayerStatus> playerStatusList = new List<PlayerStatus>();

    private void Awake()
    {
        if (instance != null)
        {
            Destroy(gameObject);
        }
        instance = this;
        DontDestroyOnLoad(this);
    }
    
    private void Start()
    {
        // 直接写方法名就可以了，在这里使用起来相当于省略了：
        // 使用GetTopScoreDelegate类型创建一个委托实例 GetTopScoreDelegate _delegate = new GetTopScoreDelegate(GetPlayerKillNum);
        // 或者 GetTopScoreDelegate _delegate = GetPlayerKillNum;
        Debug.Log("the most kill is " + GetTopNum(GetPlayerKillNum));
        Debug.Log("the most death is " + GetTopNum(GetPlayerDeathNum));
        Debug.Log("the most flag is " + GetTopNum(GetPlayerFlagNum));
    }

    /// 委托中引用的三个方法
    private int GetPlayerKillNum(PlayerStatus player)
    {
        return player.killNum;
    }
    private int GetPlayerDeathNum(PlayerStatus player)
    {
        return player.deathNum;
    }
    private int GetPlayerFlagNum(PlayerStatus player)
    {
        return player.flagNum;
    }

    // 获取某个玩家信息（killNum、deathNum、flagNum...）最高的玩家名
    public string GetTopNum(GetTopScoreDelegate _delegate)
    {
        string topName = "";
        int bestRecord = 0;

        foreach (PlayerStatus player in playerStatusList)
        {
            // 将委托作为参数，使用其返回值赋值给tempNum，根据委托中引用的方法不同，返回不同的玩家信息
            int tempNum = _delegate(player); // killNum、deathNum、flagNum...
            if (tempNum > bestRecord)
            {
                topName = player.playerName;
                bestRecord = tempNum;
            }
        }
        return topName;
    }
}
```

敢不敢再过分（简化）一点呢？敢！

使用Lambda表达式：

```c#
// Lambda表达式：() => 
(输入参数的名字) => 输出的返回值 
    
// 当然，Lambda表达式不止是可以简单的传一个返回值，可以还传一个代码块
(输入参数的名字) => {
    // Do Something...
    return 输出的返回值
}
```

实际上使用了Lambda表达式之后整段代码是这样的：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public delegate int GetTopScoreDelegate(PlayerStatus player); // GetTopScoreDelegate委托类型，可以引用返回值为int，参数为PlayerStatus的方法

public class GameManager : MonoBehaviour
{
    public static GameManager instance;
    public List<PlayerStatus> playerStatusList = new List<PlayerStatus>();

    private void Awake()
    {
        if (instance != null)
        {
            Destroy(gameObject);
        }
        instance = this;
        DontDestroyOnLoad(this);
    }

    private void Start()
    {
        // 使用Lambda表达式
        Debug.Log("the most kill is " + GetTopNum((player) => {
            // Do Something...
            return player.killNum;
        }));
        Debug.Log("the most kill is " + GetTopNum((player) => player.killNum));
        Debug.Log("the most death is " + GetTopNum((player) => player.deathNum));
        Debug.Log("the most flag is " + GetTopNum((player) => player.flagNum));
    }

    // 获取某个玩家信息（killNum、deathNum、flagNum...）最高的玩家名
    public string GetTopNum(GetTopScoreDelegate _delegate)
    {
        string topName = "";
        int bestRecord = 0;

        foreach (PlayerStatus player in playerStatusList)
        {
            // 将委托作为参数，使用其返回值赋值给tempNum，根据委托中引用的方法不同，返回不同的玩家信息
            int tempNum = _delegate(player); // killNum、deathNum、flagNum...
            if (tempNum > bestRecord)
            {
                topName = player.playerName;
                bestRecord = tempNum;
            }
        }
        return topName;
    }
}
```

当然，如果不是使用自定义委托，而是使用C#类库的Func委托（不要忘记了using System），我们连声明委托类型这一步都免去了：

**在这里我们甚至不需要额外声明一个Func委托变量，直接将其作为参数传递到方法中，在这个方法调用的时候，我们直接在参数中写上这个委托参数引用的方法名就可以了**

只有传参和使用的时候有区别，其余的初始化和引用都一样：

```c#
// 获取某个玩家信息（killNum、deathNum、flagNum...）最高的玩家名
public string GetTopNum(Func<PlayerStatus, int> _func)
{
    string topName = "";
    int bestRecord = 0;
    foreach (PlayerStatus player in playerStatusList)
    {
        // 将委托作为参数，使用其返回值赋值给tempNum，根据委托中引用的方法不同，返回不同的玩家信息
        int tempNum = _func(player); // killNum、deathNum、flagNum...
        if (tempNum > bestRecord)
        {
            topName = player.playerName;
            bestRecord = tempNum;
        }
    }
    return topName;
}
```



#### 模板方法实例2

##### 富文本

[(32条消息) Unity 富文本的使用_送命猫的博客-CSDN博客_unity 富文本](https://blog.csdn.net/qq_28112287/article/details/78320384)

Unity中，富文本是以H5标签的形式使用的，可以直接在字符串中应用，比如在Text组件中的字符串，或者说显示在控制台处的字符串，都是支持的。

```c#
// 注意：这里使用富文本等号左右两边不能加空格，识别不了
UIManager.instance.UpdatePlayerTitle(topKillName, "<color=orange>超神【击杀数最高】</color>", Logger.Log);
UIManager.instance.UpdatePlayerTitle(topDeathName, "<color=green>超鬼【死亡数最高】</color>", Logger.Log);
UIManager.instance.UpdatePlayerTitle(topFlagName, "<color=cyan>拔旗狂魔【夺旗数最高】</color>", Logger.Log);
```



##### 冒泡排序

在这里，我们希望通过冒泡排序将左栏的战绩根据击杀数、死亡数、夺旗数进行排序。在这里，我们如果不使用委托方法硬写的话，就会写出三种主逻辑都是冒泡排序，但是只有比较击杀数、死亡数、夺旗数的逻辑是不同的的方法。和我们的实例1所说一样，可维护性不好，代码结构也非常冗杂。所以我们仍然考虑使用委托作为参数来解决玩家信息比较的这一块不确定的地方。

冒泡排序结合委托参数：

```c#
// 冒泡算法实现：大的往后移动，升序
    // 传入一个可引用返回值为bool，参数列表为两个PlayerStatsus的Func委托，用于比较两个玩家信息
    public void BubbleSort(List<PlayerStatus> _playerStatusList, Func<PlayerStatus, PlayerStatus, bool> _Func)
    {
        bool isSwapped = true; // 两个对象是否进行了交换
        do
        {
            isSwapped = false;
            // 如果整个循环，从头到尾都没有再进行交换，那么说明排序完成，isSwapped为false，退出第一层循环
            for (int i = 0; i < _playerStatusList.Count - 1; i++)
            {
                // if (playerStatusList[i].killNum > playerStatusList[i + 1].killNum)
                // 通过委托传参来实现不同的玩家信息比较
                if (_Func(playerStatusList[i], playerStatusList[i + 1]))
                {
                    PlayerStatus temp = _playerStatusList[i];
                    _playerStatusList[i] = _playerStatusList[i + 1];
                    _playerStatusList[i + 1] = temp;

                    isSwapped = true;
                }
            }
        } while (isSwapped);
        UIManager.instance.UpdateLeftUI();
    }
```

下面的这三个方法即上面冒泡排序中Func委托参数_func所引用的三个方法：

```c#
// 这些方法用于排序中的比较，被BubbleSort中的Func委托所引用，为模板填补不确定的部分
public static bool CompareKill(PlayerStatus _playerA, PlayerStatus _playerB)
{
    return _playerA.killNum > _playerB.killNum;
}
public static bool CompareDeath(PlayerStatus _playerA, PlayerStatus _playerB)
{
    return _playerA.deathNum > _playerB.deathNum;
}
public static bool CompareFlag(PlayerStatus _playerA, PlayerStatus _playerB)
{
    return _playerA.flagNum > _playerB.flagNum;
}
```

冒泡排序的调用如下：

```c#
public void OnClickKillSortButton()
{
    // 这三个bool变量是为了实现升序和降序的逻辑
    isAscendingDeath = false;
    isAscendingFlag = false;
    isAscendingKill = !isAscendingKill;
    if (isAscendingKill) // 升序
    {
        // 剩下的两个方法中只需把最后一个委托类型参数的值修改为其他两个方法名CompareDeath、CompareFlag即可
        GameManager.instance.BubbleSort(GameManager.instance.playerStatusList, PlayerStatus.CompareKill);
        UpdateOrderButton(0, ascendingSprite);
    }
    else // 降序
    {
        GameManager.instance.playerStatusList.Reverse();
        UpdateLeftUI();
        UpdateOrderButton(0, descendingSprite);
    }
    UpdateRightUI();
}
```



##### Dotween

[DOTween - Documentation (demigiant.com)](http://dotween.demigiant.com/documentation.php)

一个UI动画插件，里面内置了很多动画插值方法，就不需要自己去写方法了，我一都没用过，但是听说蛮好用的。





## 事件 Event

### 1. 事件的定义

event，在牛津词典里面的解释是“能够发生(happen)的东西，尤其是某些重要的”。也就是说，如果一个东西作为主语，可以使用“发生”这个词来作为谓语动词的话，那么这个东西就是一个事件。

例如：苹果不能接“发生”，没有人会说苹果发生了，苹果就不是事件。而公司上市、产品发布，就可以接“发生”，那么它们就是事件。

准确的说，是以公司为主体的**上市**、以产品为主体的**发布**，这两个东西是**事件**。

上述是在现实世界中对事件的描述，那么在C#中充当什么样的角色。

在C#中，事件是一种类型的成员，为什么是类型的成员？因为凡是事件都是有**主体**的，没有公司，就不可能有上市这个事件发生。

在类和对象中的成员无非就属性和方法，每一种成员都具有自己特定的功能，属性的功能是存储和访问数据，方法的功能是DoSomething。

既然事件也是类型的成员，那么事件作为类型的成员，它的功能是什么呢？

事件提供的是**事件发生之后**的**通知**功能。所以，

**事件是一种类型的成员，是使对象或类具备通知能力的成员。**

比如：手机响铃了，响铃是一个事件，手机是事件的主体，响铃这个事件**通知**我们接下来要做什么，这是事件的功能。

而具体做什么，则需要看这个事件发生的时候，它**所携带的信息**：微信消息、短信、电话来电等。这个信息我们称为**事件参数(Event Args)**，我们根据事件参数决定接收到通知之后做什么：回微信信息，看短信，接电话等。当然，事件参数是可有可无的。

总而言之，**事件作为类或对象的成员，它的功能是通知别的类或者对象，并且可选地附带上事件参数。**



### 2. 事件模型中的两个“5”

根据现实世界事件的特点，以及开发中对于事件的运用，我们总结抽象出了事件模型。

#### 1. 事件模型在运作时的5个步骤

1. 有事件：我（类）要有一个事件（成员）
2. 有订阅：一群别的类关心、订阅我的事件
3. 发生：我的事件发生了
4. 通知：关心的类们被依次的通知到
5. 响应：被通知到的人，拿着事件提供的事件参数，做出响应。

#### 2. 事件模型的5个组成部分

1. 事件的拥有者
2. 事件
3. 事件的响应者
4. 事件处理器
5. **订阅关系+=**

我们可以借助例子来搞清楚事件模型：

闹钟响了你起床：闹钟响了是一个事件，这个事件的拥有者是你的手机，事件发生了之后，你的响应是起床，而你自然就是事件的响应者了，那么你和闹钟之间就产生了订阅关系。你关心这个闹钟到底有没有响，所以闹钟响了之后，就通知了你，所以你才知道，该起床了。

**事件的通知功能是需要以订阅作为前提的**，就好像你关注了一个UP主，你只有在关注了他，他发新视频的时候才会在动态里通知你，而你没有关注的话就看不到。这也就是为什么事件模型运作时订阅在事件发生之前。



### 3. 事件是基于委托的

似乎还有很多的人对事件有误会，认为事件就是特殊的委托类型字段。也会对此不解：为什么微软搞了个委托之后，还要搞一个事件出来呢？当然不是这样的。

事件是基于委托的：

#### 1. 类型兼容

谈到事件和委托的关系，我们要先来说说事件模型中的订阅关系。

订阅关系在事件模型中是最为重要的，上面也说了，没有订阅，事件的通知功能就没有作用了。在C#中订阅关系（事件 += 事件处理器）不只是表现为 += 这么简单，事件的订阅，虽然只是简单的+=操作符，却极大的解决了三个核心问题：

1. 事件发生后，通知的一定是订阅了这个事件的对象们
2. 事件处理器和事件的关系（本质就是事件处理器的返回值和参数列表是否和事件的委托类型描述的一致）
   - 其实就是说，事件处理器必须得和事件对应起来，不能随随便便的拿一个方法就当事件处理器去响应事件，就好比你肚子饿了就去吃饭，结果肚子饿了你去睡觉，事件发生了却得不到对应的回馈，这是不允许的。这种对应关系是对双方的制约：约束了事件可以向事件处理器传达什么样的信息，也约束了事件处理器能够处理什么样的信息。**这种约束是靠事件中的委托类型决定的。**
   - 这就类似于委托类型想要引用的方法必须和这个委托类型声明时所定义的返回值和参数列表相匹配。
   - 所以说，事件是基于委托的，事件依靠委托类型来做约束，使得事件处理器和事件相匹配。
3. 具体由事件响应者的哪个事件处理器来处理这个事件
   - 一个事件响应者可能会有多个满足约定的事件处理器，这时我们就可以通过 += 来添加订阅，只有被添加订阅的事件处理器，才被允许处理这个事件。

总而言之，**事件并非特殊的委托类型字段，它不是委托，它基于一个委托来为它的事件和事件处理器的匹配做约束。**

#### 2. 存储方法的引用

委托在事件的内部，为事件存储方法的引用。因为委托本身就有这样的功能，只要这个方法和委托声明时的返回值和参数列表相匹配，那么委托就可以引用这个方法，也可以说是存储了这个方法的引用。

同样，在事件中，委托被用来做事件和事件处理器之间的约束，只有满足事件内委托的返回值和参数列表，事件处理器才能订阅这个事件，即事件内部的委托才存储这个方法的引用，等到事件触发了，就通过委托去调用这个事件处理器。

所以，**事件基于委托在于事件依赖委托为他存储事件处理器方法的引用，以便事件触发时可以调用事件处理器。**



### 4. 事件的自定义声明和事件的完整/简略声明格式

这里我们写了一个例子，方便理解事件的实际应用：顾客到星巴克点单(Order)，服务员合计账单(TakeAction)，顾客结账(PayTheBill)。

在这个例子中，我们希望顾客的点单操作是一个事件，服务员订阅了顾客的点单事件，那么当点单事件触发的时候，服务员就会根据顾客的点单信息（事件参数）来为顾客合计账单（事件处理器）。

我们先写出我们的事件模型五个组成部分：

- 事件的拥有者【类】-----> Customer类
- 事件【event关键字】-----> OnOrder事件点单事件
- 事件的相应者【类】-----> Waiter类
- 事件处理器【匹配的方法】-----> TakeAction方法
- 订阅关系【+=】-----> +=



#### 1. 完整声明格式

根据我们的思路，我们应该有顾客Customer类、一个属于Customer类内部的点单事件OnOrder、一个Waiter类、一个属于Waiter类内部的事件处理器TakeAction方法、还有我们的事件参数类OrderEventArgs用于传递事件消息。

首先，我们先写上我们的顾客类：

- 注意属性和变量的区别：属性是变量的包装器，一般使用大驼峰

```c#
public class Customer
{
    // 字段/变量 小驼峰
    // public int a;

    // prop，属性，一般是大驼峰
    public float Bill { get; set; }

    // 通过事件的拥有者的内部逻辑来触发事件
    public void Order()
    {
        // DoSomething...
    }

    // 结账
    public void PayTheBill()
    {
        Debug.Log("I have to pay: " + Bill);
    }
}
```

我们希望顾客点单的时候传达给服务员的点单信息应该作为事件参数，让服务员在事件触发后做对应的处理。

所以我们定义一个OrderEventArgs作为点单事件的事件参数。

- 如果一个类是作为事件参数类来使用，那么就应该派生自EventArgs这个微软提供的事件参数基类，规范写法。

```c#
// 点单事件的事件参数类
public class OrderEventArgs : EventArgs
{
    public string CoffeeName { get; set; }
    public string CoffeeSize { get; set; }
    public float CoffeePrice { get; set; }
}
```

接下来，我们应该创建一个事件了。

首先，事件是基于委托的，我们应该先声明一个委托类型来供这个事件使用，这里我们就把这个委托类型声明在类外了：

- 我们声明了一个返回值为void，参数列表为(Customer, OrderEventArgs)的委托类型OrderEventHandler，因为他是专门为了OnOrder事件成员所声明的，所以我们加上EventHandler以做标识。

```c#
// .Net规定，如果这个委托类型是为了声明某个事件而准备的，那么取名应该为：事件名+EventHandler
// 用于事件的委托返回值一般都是void，因为事件处理器用于处理事件，一般没有返回值
// 为OnOrder事件声明的委托类型
public delegate void OrderEventHandler(Customer _customer, OrderEventArgs _e); // _customer:事件拥有者，谁点的餐; _e:事件参数，存储了一些有关点餐的消息
```

准备好事件依赖的委托类型后，我们就可以在Customer类内部声明我们的事件了。

- 声明委托之前，先得创建一个对应委托类型的变量，供事件内部使用
- 事件的完整声明，需要使用add和remove关键字表示添加事件处理器和移除事件处理器，这样写才能够建立起添加订阅和取消订阅的关系：事件 +=/-= 事件处理器
-  value关键字用于你现在不知道具体该赋什么值的时候，你可以用value关键字来表明，未来传进来的事件处理器/值会尝试填到这里来赋值。value关键字在属性prop的声明set关键字中也会用到。

```c#
public class Customer
{
    // 类的其他部分省略......

    /// 事件的完整声明格式【不常见】
    // 1. 声明一个委托类型的变量
    private OrderEventHandler orderEventHandler;
    // 2. 声明一个事件
    public event OrderEventHandler OnOrder
    {
        add
        {
            // 当我们不知道未来传进来的事件处理器/数值是多少的时候，可以用上下文关键字value来表示
            orderEventHandler += value; // 添加事件处理器
        }
        remove
        {
            orderEventHandler -= value; // 移除事件处理器
        }
    }
}
```

事件声明之后，我们需要有事件处理器对它进行订阅，否则事件就没用了。

现在我们开始写服务员Waiter类，以及它内部的事件处理器方法TakeAction：

- 其实这里用internal和public都行，internal限制只能在同一程序集中访问，public无限制，至于程序集是什么，我也不知道。。。

```c#
public class Waiter
{
    // 为什么快速修复创建的方法关键字是internal？
    // public void Test(Customer _customer, OrderEventArgs _e)
    internal void TakeAction(Customer _customer, OrderEventArgs _e)
    {
        float finalPrice = 0;
        switch (_e.CoffeeSize)
        {
            case "Tall":
                finalPrice = _e.CoffeePrice; // 中杯：原价
                break;
            case "Grand":
                finalPrice = _e.CoffeePrice + 3; // 大杯：原价+3
                break;
            case "Venti":
                finalPrice = _e.CoffeePrice + 6; // 超大杯：原价+6
                break;
        }

        _customer.Bill += finalPrice;
    }
}
```

然后我们在脚本类里面创建Customer和Waiter的实例，建立订阅关系：

```c#
public class EventEx_02 : MonoBehaviour
{
    Customer customer = new Customer();
    Waiter waiter = new Waiter();

    private void Start()
    {
        customer.OnOrder += waiter.TakeAction;
    }
}
```

最后，我们来写事件触发的逻辑。

我们知道，事件属于类，事件应不能在类的外部被调用，它应只能通过类内部的逻辑来触发（调用），即顾客类的Order点单方法：

- 可以发现，在Order方法内，我们还是通过委托类型变量去调用被订阅的方法，这更能体现出事件作为委托的包装器的功能所在，这种写法，事件作为对内部委托类型的限制，只允许外部用+=、-=去添加或删除引用，这使得委托的使用更加安全。

```c#
public class Customer
{
    // prop，属性，一般是大驼峰
    public float Bill { get; set; }

    /// 事件的完整声明格式【不常见】
    // 1. 声明一个委托类型的变量
    private OrderEventHandler orderEventHandler;
    2. 声明一个事件
    public event OrderEventHandler OnOrder
    {
        add
        {
            // 当我们不知道未来传进来的事件处理器/数值是多少的时候，可以用上下文关键字value来表示
            orderEventHandler += value; // 添加事件处理器
        }
        remove
        {
            orderEventHandler -= value; // 移除事件处理器
        }
    }

    // 通过事件的拥有者的内部逻辑来触发事件
    public void Order()
    {
        if (orderEventHandler != null)
        {
            OrderEventArgs e = new OrderEventArgs();
            e.CoffeeName = "Mocha";
            e.CoffeeSize = "Tall";
            e.CoffeePrice = 28;

            orderEventHandler(this, e);
        }
    }

	// 支付账单
    public void PayTheBill()
    {
        Debug.Log("I have to pay: " + Bill);
    }
}
```

最后，整个脚本的完整项目逻辑是这样的；

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;

// .Net规定，如果这个委托类型是为了声明某个事件而准备的，那么取名应该为：事件名+EventHandler
// 用于事件的委托返回值一般都是void，因为事件处理器用于处理事件，一般没有返回值
// 为OnOrder事件声明的委托类型
public delegate void OrderEventHandler(Customer _customer, OrderEventArgs _e); // _customer:事件拥有者，谁点的餐; _e:事件参数，存储了一些有关点餐的消息

public class EventEx_02 : MonoBehaviour
{
    Customer customer = new Customer();
    Waiter waiter = new Waiter();

    private void Start()
    {
        customer.OnOrder += waiter.TakeAction; // 订阅

        // 验证：事件只能通过事件拥有者内部的逻辑来触发，不能在时间拥有者的外部调用，直接如下调用会报错
        // customer.OnOrder();

        customer.Order();  // 顾客点餐
        customer.PayTheBill(); // 顾客支付
        // Debug.Log()输出为：I have to pay: 28
    }
}

public class Customer
{
    // 字段/变量 小驼峰
    // public int a;

    // prop，属性，一般是大驼峰
    public float Bill { get; set; }

    /// 事件的完整声明格式【不常见】
    // 1. 声明一个委托类型的变量
    private OrderEventHandler orderEventHandler;
    2. 声明一个事件
    public event OrderEventHandler OnOrder
    {
        add
        {
            // 当我们不知道未来传进来的事件处理器/数值是多少的时候，可以用上下文关键字value来表示
            orderEventHandler += value; // 添加事件处理器
        }
        remove
        {
            orderEventHandler -= value; // 移除事件处理器
        }
    }

    // 通过事件的拥有者的内部逻辑来触发事件
    public void Order()
    {
        if (orderEventHandler != null)
        {
            OrderEventArgs e = new OrderEventArgs();
            e.CoffeeName = "Mocha";
            e.CoffeeSize = "Tall";
            e.CoffeePrice = 28;

            orderEventHandler(this, e);
        }
    }


    public void PayTheBill()
    {
        Debug.Log("I have to pay: " + Bill);
    }
}

// 订单事件的事件参数类
public class OrderEventArgs : EventArgs
{
    public string CoffeeName { get; set; }
    public string CoffeeSize { get; set; }
    public float CoffeePrice { get; set; }
}

public class Waiter
{
    // 为什么快速修复创建的方法关键字是internal？
    // public void Test(Customer _customer, OrderEventArgs _e)
    internal void TakeAction(Customer _customer, OrderEventArgs _e)
    {
        float finalPrice = 0;
        switch (_e.CoffeeSize)
        {
            case "Tall":
                finalPrice = _e.CoffeePrice; // 中杯：原价
                break;
            case "Grand":
                finalPrice = _e.CoffeePrice + 3; // 大杯：原价+3
                break;
            case "Venti":
                finalPrice = _e.CoffeePrice + 6; // 超大杯：原价+6
                break;
        }

        _customer.Bill += finalPrice;
    }
}
```





#### 2. 简略声明格式

其实，完整声明格式的事件并不常见，而且写起来确实很麻烦，又要声明委托类型，还要创建委托变量，然后还要通过add和remove关键字来创建和移除事件处理器，确实是不太方便，所以事件有更加简便的写法：

- 你可以发现，除了多了一个event关键字，和我们声明委托类型的变量真的简直一模一样，而且这个时候，事件居然可以接除了+=、-=之外的其他符号，甚至可以代替委托类型变量来调用订阅的事件处理器。
- 这其实是因为语法糖的缘故，让事件变得过于像委托了，这种方便的写法使得很多程序员对于事件和委托的关系暧昧不清。实际上，如果使用反编译器查看，可以发现编译器帮我们创建了一个委托类型的变量，然后在运行的时候使用，只是我们在写代码的时候，看不见也调用不到，实际上是存在的。
  - 语法糖（Syntactic sugar）：计算机语言中添加的某种语法，这种语法对语言的功能没有影响，但是更方便程序员使用（比如这里的简略声明格式）。使用语法糖能够增加程序的可读性，减少程序的出错机会。它是一种更加便捷的写法，编译器会帮我们做转换，提高开发代码的效率，在性能上也不会带来损失。

```c#
public class Customer
{
	// 省略...

    /// 事件的简略声明格式【多见】
    // 微软把这种写法叫做"Field-like"，这种写法像字段一样，实际上事件不是字段，只是这个语法糖让它像字段
    // 和我们上面的OrderEventHandler委托类型字段的声明是不是非常像：private OrderEventHandler orderEventHandler;
    // 看起来只是用event关键字修饰了一下而已，这就非常容易使人误会事件就是一个委托类型的字段，实际上并不是。
    public event OrderEventHandler OnOrder;

    // 通过事件的拥有者的内部逻辑来触发事件
    public void Order()
    {
        if (OnOrder != null) // 判断事件是否为空，语法糖
        {
            OrderEventArgs e = new OrderEventArgs();
            e.CoffeeName = "Black Tea";
            e.CoffeeSize = "Grand";
            e.CoffeePrice = 23;
            // 原本声明了OrderEventHandler委托类型的字段，应该是使用这个字段的，现在使用简略写法，没有声明委托类型字段，就使用了事件名来代替
            OnOrder(this, e);
        }
    }

    public void PayTheBill()
    {
        Debug.Log("I have to pay: " + Bill);
    }
}
```



#### 3. 优化

最后，我们对整个代码进行了优化：

首先，我们使用了微软为我们提供的事件委托类型EventHandler来声明事件。

EventHandler委托类型是这样的：

```c#
namespace System
{
    public delegate void EventHandler(object sender, EventArgs e);
}
```

因为它的参数列表变成了(object, EventArgs)，所以我们Waiter类内部的TakeAction就和这个委托类型不匹配了。

我们的解决方法是，使用as关键字：

- as关键字：类似于强制类型转换，检查类型之间能不能类型转换，如果不能，返回null，而不是抛出异常。

```c#
public class Waiter
{
    // 原先的TakeAction方法已经和事件的委托类型EventHandler不匹配了
    public void TakeAction(object _sender, EventArgs _ea)
    {
        float finalPrice = 0;

        Customer customer = _sender as Customer; // 将object类型转为Customer类型
        OrderEventArgs e = _ea as OrderEventArgs; // 将EventArgs类型转为OrderEventArgs类型

        switch (e.CoffeeSize)
        {
            case "Tall":
                finalPrice = e.CoffeePrice; // 中杯：原价
                break;
            case "Grand":
                finalPrice = e.CoffeePrice + 3; // 大杯：原价+3
                break;
            case "Venti":
                finalPrice = e.CoffeePrice + 6; // 超大杯：原价+6
                break;
        }

        customer.Bill += finalPrice;
    }
}
```

最后，我们还对Order方法做了优化，使他可以通过传参来指明点单信息。

最后整个代码是这样的：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;

public class EventEx_02 : MonoBehaviour
{
    Customer customer = new Customer();
    Waiter waiter = new Waiter();

    private void Start()
    {
        customer.OnOrder += waiter.TakeAction;

        // 验证：事件只能通过事件拥有者内部的逻辑来触发，不能在时间拥有者的外部调用，直接如下调用会报错
        // customer.OnOrder();

        // 顾客点餐
        customer.Order("Mocha", "Grand", 32);
        customer.Order("Macaron", "Tall", 16);
        customer.Order("Latte", "Venti", 30);
        customer.Order("Macaron", "Tall", 16);
        
        customer.PayTheBill(); // 顾客支付
    }
}

public class Customer
{
    // prop，属性，一般是大驼峰
    public float Bill { get; set; }
    
    public event EventHandler OnOrder;

    // 通过事件的拥有者的内部逻辑来触发事件
    public void Order(string _name, string _size, float _price)
    {
        if (OnOrder != null) // 判断事件是否为空，语法糖
        {
            OrderEventArgs e = new OrderEventArgs();
            e.CoffeeName = _name;
            e.CoffeeSize = _size;
            e.CoffeePrice = _price;
            // 原本声明了OrderEventHandler委托类型的字段，应该是使用这个字段的，现在使用简略写法，没有声明委托类型字段，就使用了事件名来代替
            OnOrder(this, e);
        }
    }

    public void PayTheBill()
    {
        Debug.Log("I have to pay: " + Bill);
    }
}


// 订单事件的事件参数类
public class OrderEventArgs : EventArgs
{
    public string CoffeeName { get; set; }
    public string CoffeeSize { get; set; }
    public float CoffeePrice { get; set; }
}

public class Waiter
{
    // 原先的TakeAction方法已经和事件的委托类型EventHandler不匹配了
    public void TakeAction(object _sender, EventArgs _ea)
    {
        float finalPrice = 0;

        Customer customer = _sender as Customer;
        OrderEventArgs e = _ea as OrderEventArgs;

        switch (e.CoffeeSize)
        {
            case "Tall":
                finalPrice = e.CoffeePrice; // 中杯：原价
                break;
            case "Grand":
                finalPrice = e.CoffeePrice + 3; // 大杯：原价+3
                break;
            case "Venti":
                finalPrice = e.CoffeePrice + 6; // 超大杯：原价+6
                break;
        }

        customer.Bill += finalPrice;
    }
}
```



#### 4. 为什么不用多播委托？

现在有个问题，目前看来，事件的作用似乎和委托一样，事件触发之后，让内部的委托帮忙调用引用的方法（事件处理器），这样做似乎事件这个东西好像可有可无的样子。

事件当然不是可有可无的，微软还没傻到专门去写一个多余的东西给你用。

事件成员能够让程序之间的逻辑和对象之间的关系变得非常的理所当然，就应该是这样的，而且非常的安全。

而且可以防止“借刀杀人”的情况发生。

实际上，如果我们把事件声明时的event关键字去掉，居然是真的不会报错的。这时我们声明的就不是事件了，而是EventHandler委托类型的字段：

```c#
public EventHandler OnOrder;
```
这个时候运行脚本，你会发现结果是完全正确的。

别急，我们来分析一下，加了event关键字，和不加event关键字，逻辑上有什么区别。

我们之前也试过了，OnOrder如果是一个事件，那么它无论在完整声明格式，还是在简略声明格式中，在外部除了+= -=操作符可以使用之外，是不能够在外部直接调用的。它只能够在类的内部通过内部逻辑来触发事件，即使OnOrder是public类型。

比如这里，OnOrder是一个事件，你是没有办法在外部像这样直接调用OnOrder的，编译器会告诉你这样的错误：

**事件“Customer.OnOrder”只能出现在 += 或 -= 的左边(从类型“Customer”中使用时除外)**

```c#
public class EventEx_02 : MonoBehaviour
{
    Customer customer = new Customer();
    Waiter waiter = new Waiter();

    private void Start()
    {
        customer.OnOrder += waiter.TakeAction;

        // 验证：事件只能通过事件拥有者内部的逻辑来触发，不能在时间拥有者的外部调用，直接如下调用会报错
        OrderEventArgs e = new OrderEventArgs();
        e.CoffeeName = "Mocha";
        e.CoffeeSize = "Grand";
        e.CoffeePrice = 32;
        customer.OnOrder(customer, e);
    }
}
```

现在，我们把event关键字去掉，这时OnOrder就变成了一个EventHandler委托类型的字段了。

按道理来说，委托字段是可以在外部使用的，因为它本质上是一个函数指针，它的使用没有特别的限制，因为这里的OnOrder是public修饰的。

所以，OnOrder如果可以在外部调用的话，就会有借刀杀人的情况发生。

我们创建一个Customer类新的实例goodBro，在Start方法中这样写：

```c#
public class EventEx_02 : MonoBehaviour
{
    Customer customer = new Customer();
    Customer goodBro = new Customer();
    Waiter waiter = new Waiter();

    private void Start()
    {
        customer.OnOrder += waiter.TakeAction;
        goodBro.OnOrder += waiter.TakeAction;

        OrderEventArgs e = new OrderEventArgs();
        e.CoffeeName = "Mocha";
        e.CoffeeSize = "Grand";
        e.CoffeePrice = 32;
        customer.OnOrder(customer, e);

        OrderEventArgs e1 = new OrderEventArgs();
        e1.CoffeeName = "Suda";
        e1.CoffeeSize = "Tall";
        e1.CoffeePrice = 16;
        goodBro.OnOrder(customer, e1); // 借刀杀人，明明是goodBro应该支付的账单，却可以由customer来支付
        
        customer.PayTheBill(); // I have to pay: 51
        goodBro.PayTheBill();  // I have to pay: 0
    }
}
```

这种情况就非常不符合逻辑了，凭什么你goodBro的账单可以算在我customer的头上啊。原因就在于委托字段没有加以限制，使得其在外部可以随意的调用引用的方法，在安全性和逻辑上就不过关了。

你可能会有疑问，那把委托类型的字段改为private不就可以了吗？你傻啊，改成private的话，你怎么在外部使用+=存储其他方法的引用呢？

这就是事件和委托的区别，事件使用起来有限制，也更加的合理。委托因为源自函数指针，使用起来更多的是在功能上满足程序员的需要，而不是在逻辑上对应。

而微软正是为了解决内部public委托类型的字段有可能在类的外部被滥用、误用的情况，才推出了事件这种成员。



#### 5. 属性和事件的类比

属性和事件有很多相似的地方。首先，他们都是一个包装器。

属性是字段的包装器，事件是委托类型字段的包装器。（事件的完整声明和属性的完整声明结构非常类似）

```c#
// 字段/变量 小驼峰
private float bill;
// prop，属性，一般是大驼峰
public float Bill
{
    get
    {
        return bill;
    }
    set
    {
        bill = value;
    }
}
```

属性作为字段的包装器，它的作用就是保护字段不被滥用，我们可以在set和get当中添加一些逻辑来保护字段。而事件作为委托类型字段的包装器，同样起到的是一个保护的作用，保护委托类型的字段不被外界所滥用，只能去获取它的+= -=操作符来使用，如果你想直接调用，或者是用=操作符来赋值，编译器都直接报错。

包装器永远都不可能是被包装的东西的本身。所以**事件本质上就是一个委托类型字段的包装器。**



### 5. 事件的使用实例

使用实例可以是自动门：人物走到门周围，会自动开关门。门的检测（OnTriggerEnter、OnTriggerExit）作为事件，事件触发后门执行开门和关门的操作。

- 小贴士：在Scene窗口的W拖拽模式，你可以按住V键拖动物体的某个顶点，这个顶点会和其他的顶点自动进行对齐。

摆烂了，笔记懒得写了，结果委托和事件最终写了这么多字啊，如果不算代码起码也有八九千字了。

EventSystem类：事件拥有者

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;

// 事件拥有者：EventSystem
// 事件：onDoorOpen、onDoorExit
public class EventSystem : MonoBehaviour
{
    public static EventSystem instance;

    // 声明事件，使用简略声明格式
    public event Action<int> onDoorEnter;
    public event Action<int> onDoorExit;
    
    private void Awake()
    {
        if(instance != null)
        {
            Destroy(this);
        }
        instance = this;
        DontDestroyOnLoad(this); 
    }

    // 事件拥有者在内部触发事件
    public void OpenDoor(int _id)
    {
        if(onDoorEnter != null)
        {
            onDoorEnter(_id);
        }
    }
    public void CloseDoor(int _id)
    {
        if(onDoorExit != null)
        {
            onDoorExit(_id);
        }
    }
}
```

Door类：事件响应者

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// 事件的响应者：Door
// 事件处理器：DoorOpen、DoorClose
public class Door : MonoBehaviour
{
    [SerializeField] private int doorID;

    // 如果存在添加订阅的话，那么就应该补充上取消订阅，防止内存泄漏
    private void Start()
    {
        EventSystem.instance.onDoorEnter += DoorOpen;
        EventSystem.instance.onDoorExit += DoorClose;
    }
    // 当门被摧毁的时候取消订阅
    private void OnDisable()
    {
        EventSystem.instance.onDoorEnter -= DoorOpen;
        EventSystem.instance.onDoorExit -= DoorClose;
    }

    public void DoorOpen(int _id)
    {
        if (doorID == _id) // 只有当doorID == areaID时才开门，其余门不开
            LeanTween.moveLocalY(gameObject, 3.0f, 1.0f).setEaseInSine();
    }

    public void DoorClose(int _id)
    {
        if (doorID == _id)
            LeanTween.moveLocalY(gameObject, 1.0f, 1.0f).setEaseOutSine();
    }
}
```

TriggerArea：门的Trigger，用于检测

```c#
using UnityEngine;

public class TriggerArea : MonoBehaviour
{
    [SerializeField] private int areaID;

    private void OnTriggerEnter(Collider other)
    {
        EventSystem.instance.OpenDoor(areaID);
    }

    private void OnTriggerExit(Collider other)
    {
        EventSystem.instance.CloseDoor(areaID);
    }
}
```

PlayerMovement：玩家基本移动

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(Rigidbody))]
public class PlayerMovement : MonoBehaviour
{
    private Rigidbody rb;
    private Vector3 moveInput;
    [SerializeField] float moveSpeed;

    private void Awake()
    {
        rb = GetComponent<Rigidbody>();
    }

    private void Update()
    {
        moveInput = new Vector3(Input.GetAxis("Horizontal"), 0, Input.GetAxisRaw("Vertical"));
        LookAtCursor();
    }

    private void FixedUpdate()
    {
        rb.MovePosition(rb.position + moveInput * moveSpeed * Time.fixedDeltaTime);
    }

    private void LookAtCursor()
    {
        // Camera.main问题，应该声明一个类全局变量来存放Camera.main
        Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        Plane plane = new Plane(Vector3.up, Vector3.zero); // 这里是什么意思

        float distToGround = 0;
        if(plane.Raycast(ray, out distToGround))
        {
            Vector3 point = ray.GetPoint(distToGround);
            // point 是在平面上的一个点，LookAt这个点人物移动会有问题，所以使用rightPoint调整
            Vector3 rightPoint = new Vector3(point.x, transform.position.y, point.z);

            transform.LookAt(rightPoint); // 物体朝向鼠标点击位置
        }
    }
}
```

