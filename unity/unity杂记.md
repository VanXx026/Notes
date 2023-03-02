# unity

```note
主要是为了记录一些比较零散的知识点，怕自己忘记了
```



## Universal Render Pipeline (通用渲染管道)

这个东西我目前还没搞懂是干什么的，貌似和游戏里那些设置画质的东西有关，网友说这玩意儿可以让你的游戏画面牛逼一点

怎么开呢？普通的3D项目可以通过Window - Package Manager - Unity Registry - 搜索URP import获取

然后你需要从Edit - Project Settings - 找到Graphics将URP Assets 置入相应位置 - 找到Quality将URP Assets render 置入相关位置

有的素材支持URP，有的不支持，需要仔细看清楚到底支不支持



## 快捷键： move Tool（拖动模式）下按v键；shift+ctrl+直接拖动

这时鼠标会像磁铁一样吸住物体，移动到底部拖动的时候会自动贴合其他物体。比如一棵树，使用v键的话就可以拖动树根轻松的贴合地面，不需要自己对准。

**反转了，ctrl+shift+直接拖还更好用，逆天**



## 单例模式

单例模式的目的是让**一个类在游戏运行期间有且只有一个实例**。

好处：**其他脚本可以很方便地使用到这个类的方法**，只存在一个实例的话管理起来也很方便，减少了内存开销。

用法：大多数实现全局管控的类，比如各种资源管理器（**Manager），生命周期在游戏中永不销毁的对象可以使用单例模式

写法：

```c#
public class MouseManager : MonoBehaviour
{
    //static 保证了它可以通过类访问，而不是通过实例访问。
    //private set 保证了它只允许在 MouseManager 内部赋值，外部只能读取。
    //
    public static MouseManager Instance { get; private set; }
 
    private void Awake()
    {
        //因为在切换场景的时候，这个场景的MouseManager会被销毁；
        //而从另一个场景切换回来的时候，unity并不是恢复到这个场景，而是新建了一个场景，所以会新建一个MouseManager
        //而单例模式的目的是为了使整个游戏进程中该类只存在一个实例，所以检测到Instance非空，就要销毁。
        if (Instance != null)
        {
            Destroy(gameObject);
        }
        Instance = this;
    }
}

//当其他脚本想要调用MouseManager中的方法和属性的时候（当然是public的才可以），就变得非常简单了。
MouseManager.Instance.xxx();
```

单例模式是一个非常常用的设计模式，当你有一个类是实现全局监控或管理某个东西的时候，就可以使用单例模式。

### 泛型的单例模式

单例模式的泛型大体上的写法如下：这里我写了一个单例模式的泛型用于创建各种各样的Manager

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/*where：用于对参数T做出一定的约束
//where T: struct ：类型参数必须为值类型
//Where T: class ：类型参数必须为引用类型
//where T: new() ：类型参数必须有一个公有、无参的构造函数。当于其它约束联合使用时,new()约束必须放在最后。
//where T: <base class name> ：类型参数必须是指定的基类型或是派生自指定的基类型
//where T: <interface name>	：类型参数必须是指定的接口或是指定接口的实现。可以指定多个接口约束。接口约束也可以是泛型的
*/

//单例模式泛型
//在这里将T限定为只能是Singleton的基类型或是派生类类型，这就限制了不是什么阿猫阿狗都可以使用Singleton类，而是需要继承Singleton类才可以使用
public class Singleton<T> : MonoBehaviour where T : Singleton<T>
{
    private static T instance;

    //提供外部访问
    public static T Instance
    {
        get { return instance; }
    }

    //判断当前类型的单例是否已经初始化过了
    public static bool IsInitialized
    {
        get { return instance != null; }
    }

    //protected：只允许自己和继承者访问这个方法
    //virtual：虚函数，允许继承者重写这个函数
    protected virtual void Awake()
    {
        if (instance != null)
            Destroy(gameObject);
        else
            instance = (T)this;
    }

    //存在多个同类型的单例时销毁该单例
    protected virtual void Ondestroy()
    {
        if (instance == this)
        {
            instance = null;
        }
    }
}
```

这样的话，如果我们想创建一个**Manager类用来管理各种东西，就可以直接继承Singleton类，实现了代码的复用性。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//GameManager类继承Singleton实现单例模式，不需要再写重复的代码
public class GameManager : Singleton<GameManager>
{
    public CharacterStates playerStates;

    public void RegisterPlayer(CharacterStates player)
    {
        playerStates = player;
    }
}
```





## C# 委托delegate 事件 event

[C# 中的委托和事件](https://www.cnblogs.com/SkySoot/archive/2012/04/05/2433639.html)

### 委托

我个人理解的委托其实就是一个函数指针，可以将函数绑定到这个委托上（前提是返回值，参数类型、参数个数都得对得上）

委托是引用类型的类，作为对函数的引用，定义了函数的类型，使得可以将函数当作另一个函数的参数来进行传递，使得程序具有更好的可扩展性。

```c#
public delegate void GreetingDelegate(string name);
 
class Program
{
    private static void EnglishGreeting(string name)
    {
        Console.WriteLine("Good Morning, " + name);
    }
 
    private static void ChineseGreeting(string name)
    {
        Console.WriteLine("早上好, " + name);
    }
 
    private static void GreetPeople(string name, GreetingDelegate MakeGreeting)
    {
        //简写
        MakeGreeting(name);
        //正常写法
        MakeGreeting.Invoke(name);
    }
 
    static void Main(string[] args)
    {
        GreetPeople("Liker", EnglishGreeting);
        GreetPeople("李志中", ChineseGreeting);
        Console.ReadLine();
    }
}
```

事实上，还可以将函数名直接给委托绑定上，也可以通过+= 和 -= 增加函数的绑定和去除函数的绑定。这叫**多播（multicast）**。

```C#
static void Main(string[] args)
{
    GreetingDelegate delegate1;
    delegate1 = EnglishGreeting;
    delegate1 += ChineseGreeting; 
    delegate1("Liker");
    delegate1 -= ChineseGreeting;
    Console.ReadLine();
}
```

使用委托可以将多个方法绑定到同一个委托实例，当调用此实例时(这里用“调用”这个词，是因为此变量代表一个方法)，可以依次调用所有绑定的方法。

### 事件

我个人理解就是，事件对委托进行了一个封装，限制了使用委托的范围，也就是：事件应该由事件发布者触发（也就是在发布者类内部触发），订阅者负责绑定在委托上，等待事件触发控制方法运行。事件使用**发布-订阅（publisher-subscriber）**模型。

其实就更像是UP主发视频了，然后通知所有关注的人看视频（看视频这个行为可以视为关注的人的方法，毕竟看的人不一样，看视频的方式也不一样）。

```C#
using System;
 
class Program
{
    static void Main(string[] args)
    {
        Publishser pub = new Publishser();
        Subscriber sub = new Subscriber();
        pub.NumberChanged += new NumberChangedEventHandler(sub.OnNumberChanged);
        pub.DoSomething(); // 应该通过DoSomething()来触发事件
        pub.NumberChanged(100); // 不恰当使用
    }
}
 
/// <summary>
/// 定义委托
/// </summary>
/// <param name="count"></param>
public delegate void NumberChangedEventHandler(int count);
 
/// <summary>
/// 定义事件发布者
/// </summary>
public class Publishser
{
    private int count;
 
    public NumberChangedEventHandler NumberChanged; // 声明委托变量
 
    //public event NumberChangedEventHandler NumberChanged; // 声明一个事件
 
    public void DoSomething()
    {
        // 在这里完成一些工作 ...
 
        if (NumberChanged != null) // 触发事件
        { 
            count++;
            NumberChanged(count);
        }
    }
}
 
/// <summary>
/// 定义事件订阅者
/// </summary>
public class Subscriber
{
    //约定俗称的规定，就是订阅事件的方法的命名，通常为“On 事件名”
    public void OnNumberChanged(int count)
    {
        Console.WriteLine("Subscriber notified: count = {0}", count);
    }
}
```



## cinemachine

是一个可以提供多种摄像机的一个包（？，不知道能不能这样叫），在packet manager中可以搜索到。



## 相机跟随的小技巧

如果不想改变相机跟随物体的轴心点，又想改变相机跟随物体的点位，可以在物体中创建一个子空物体，再将空物体作为相机跟随的物体即可。这样就可以自由改变你想跟随的点位。



## 环境雾化

在Window - Rendering - Lighting - Environment 中可以找到fog选项，打开可以进行场景雾化的设置，推荐使用Linear mode。



## post processing

在unity中，是一个牛逼的提升画质的插件，如何开启呢？

首先在层级管理栏处鼠标右键 - volume - global volume - 直接new一个profile，然后在主相机的Camera组件下的rendering下勾选post processing即可。然后就可以在global volume物体上添加override实现各种画质效果。



## poly brush工具

poly brush 是一个可以用于修改多边形素材的一个工具，多用于修改和制作地图，也可以给地图上色，在地图上快速放置一些素材等。

在packet manager中可以搜索找到



## probuilder工具

这个工具可以用来自己动手创建各种形状的模型，同时也可以配合poly brush来制作poly风格的地图。



## Shader Graph 实现遮挡剔除

create - shader - URP - Unlit Shader Graph，可以通过这个玩意儿来创建一个material，接着点击这个Unlit Shader Graph 进入一个界面配置，太专业了我看不懂。

[Unity3D游戏开发教程|Core核心功能09:Shader Graph 遮挡剔除](https://www.bilibili.com/video/BV1i5411P7bd/?spm_id_from=333.788)

可以用这个来做一个当人物被遮挡后的半透明材质。



## 利用Shader Graph 制作传送门

UV是贴图坐标，相当于空间的XY坐标概念，shader组件中的uv输入，就是贴图的意思。

shader中常规uv，是自然过渡的颜色

对应关系：

U|V

X|Y|Z

R|G|B|A

不得不说，这玩意实在是太吊了，功能非常强大，unity还内置了一个这么变态的玩意真的牛皮



## 关于场景内容遮挡鼠标点击地面的问题

有两种方法：

1. 将遮挡物的Layer设置成：ignore raycast
2. 如果脚本中使用的是射线检测碰撞体的话，直接取消勾选遮挡物的Collider组件。





## [RequireComponent(typeof(**组件名**))] 脚本自动为物件挂载组件

使用[RequireComponent(typeof(**组件名**))]可以在类中**自动添加定义的该组件**

适用于这个脚本需要被很多同类型的物件使用的时候

```c#
[RequireComponent(typeof(NavMeshAgent))]
public class EnemyController : MonoBehaviour
{
    private NavMeshAgent agent;

}	
```



## 协程Coroutines

[Unity3D协程介绍 以及 使用](https://blog.csdn.net/huang9012/article/details/38492937)

协程不是进程，协程过程的同步操作仍在主线程上执行。

协程是使用 [IEnumerator](https://docs.microsoft.com/en-us/dotnet/api/system.collections.ienumerator) 返回类型和正文中某处包含的[yield](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/yield) return 语句声明声明的方法。

通过下面的实验可以知道，**普通方法的执行是一帧内完成的，而协程通过 yield return null 的控制，让方法在当前帧暂停，在下一帧继续执行。**实际上还有yield return new WaitForSeconds(.1f)，也就是隔0.1s后执行，null是指下一帧执行。

```c#
public class Test : MonoBehaviour
{
    private void Start()
    {
        //协程
        StartCoroutine(SayHelloCoroutlinesWay());	//1. 协程开始调用
        //普通函数调用
        SayHelloNormalWay();
    }

    //普通方法
    void SayHelloNormalWay()
    {
        for (int i = 0; i < 5; i++)
        {
            Debug.Log("normal hello" + Time.time); //普通方法在start方法调用的第一帧就全部执行完了
        }
    }
    
    //协程方法    
    IEnumerator SayHelloCoroutlinesWay()
    {	
        //2. 进入协程方法内
        for (int i = 0; i < 5; i++)
        {
            Debug.Log("Coroutlines hello" + Time.time);
            yield return null; //3. 从下一帧开始继续从该行执行，也就是说这一帧执行到这里任务就完成了，等待下一帧继续循环
            //4. 继续从这里开始循环
        }
        //5. 重复进行五帧的输出后，方法结束
    }
}
```

游戏当中的实例应用：

```c#
//当敌人被点击时调用该方法，获取到敌人的gameObject信息
    private void EventAttack(GameObject target)
    {
        if (target != null)
        {
            attackTarget = target;
            //启动协程
            StartCoroutine("MoveToAttackTarget");
            //为什么这里需要用协程而不是直接将协程的代码写到这里来？
            //因为unity的特性是普通的方法只给一帧的时间执行
            //你如果在这里使用while的话，意味着你要在一帧之内完成这段朝敌人方向前进的路程
            //然而agent有速度限制，也就是说无法在一帧内到达，所以会整个程序陷入死循环卡死
            //但是使用协程的话，这一帧这段位移没做完没有关系，我这一帧先移动一小段，然后下一帧再继续执行这个while
            //直到player和enemy的位移小于1，那么循环结束，接着执行接下来的攻击指令
        }
    }

    //协程
    IEnumerator MoveToAttackTarget()
    {
        agent.isStopped = false;

        transform.LookAt(attackTarget.transform.position);

        //当满足条件时，则一直朝敌人方向前进
        while (Vector3.Distance(attackTarget.transform.position, transform.position) > 1)
        {
            agent.destination = attackTarget.transform.position;
            yield return null; 
        }

        //↓当Player和Enemy距离小于1时执行
        agent.isStopped = true;

        //Attack
        if (lastAttackTime < 0)
        {
            anim.SetTrigger("Attack");
            //重置冷却时间
            lastAttackTime = 0.5f;
        }
    }
```





## [Header("标题名")] HeaderAttribute

用于在Inspector中的某些字段添加一个标题。

```c#
using UnityEngine;

public class Example : MonoBehaviour
{
    [Header("Health Settings")]
    public int health = 0;
    public int maxHealth = 100;

    [Header("Shield Settings")]
    public int shield = 0;
    public int maxShield = 0;
}

//在Inspector中效果是这样的：
Health Settings //加粗
health      =======
maxHealth   =======
Shield Settings
shield      =======
maxShield   =======
```





## 动画机技巧 使用一个空State代替该层之下的Layer

好吧，其实我没听太懂

8:37左右

[Unity3D游戏开发教程|Core核心功能:13Enemy Animator设置敌人的动画控制器｜Unity中文课堂_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV15V411H7Qz/?spm_id_from=pageDriver) 



## OnDrawGizmosSelected()

就是一个unity自带的可以画出辅助线框的方法，这个方法只有当你点击了该对象才会显示出你画的线框。

有什么用呢？

比如说画一个敌人的巡逻范围，索敌半径，爆炸的半径球体之类

```c#
//用于画出敌人的侦察范围和索敌范围，可选择性的
    private void OnDrawGizmosSelected()
    {
        Gizmos.color = Color.blue;
        //画出当前怪物的视野半径,DrawWireSphere是画出球体的线框
        Gizmos.DrawWireSphere(transform.position, sightRadius);
    }
```



## C#中乘法的开销要比除法低，尽可能用乘法

如题



## NavMesh.SamplePosition() 敌人自动巡逻方法防止一头攒死在不可移动的点位上

```c#
//随机得到一个新的wayPoint
    void GetNewWayPoint()
    {
        float randomX = Random.Range(-patrolRange, patrolRange);
        float randomZ = Random.Range(-patrolRange, patrolRange);

        //y值保留当前位置的y值，因为地图上有地形高度差
        Vector3 randomPoint = new Vector3(guardPos.x + randomX, transform.position.y, guardPos.z + randomZ);
        //FIXME:可能出现问题
        //没有考虑到地图杂物多时，所选择点位为不可移动的点位
        // wayPoint = randomPoint;

        //敌人巡逻时规避不可移动的点位，防止当前randomPoint取点导致敌人一头攒死在杂物上
        NavMeshHit hit;
        //SamplePosition：在指定范围内找到导航网格上离randomPoint最近的一个可以移动的点，如果找到了就是true，反之false
        //areaMask:在navigation中的area处可以查到1是Walkable
        wayPoint = NavMesh.SamplePosition(randomPoint, out hit, patrolRange, 1) ? randomPoint : transform.position;
    }
```



## [HideInInspector]

```c#
//想public调用却又不想在Inspector中出现
[HideInInspector]
public bool isCritical;
```



## ScriptableObject

这个类不同于MonoBehavior，脚本继承之后可以用于写各种数值项（比如说人物的攻击啊、防御啊、血量啊之类的），可以根据这个脚本单独的创建出一个配置项，这样就可以集中的修改人物的数值。

然后呢，你就可以使用继承了这个类的脚本创建多个具有相同配置项的文件来管理人物的数值（主角的，怪物的）。

官方文档是这么说的：

> 如果要创建不需要附加到游戏对象的对象，则可以从中派生的类。
>
> 这对于仅用于存储数据的assets最有用。

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//帮助我们在create菜单中创建一个子集菜单
//就是在Assets目录下右键点击出来的菜单的create子选项下创捷一个新的选项
//Assets -> Create -> Character States -> Data 创建出来的文件默认叫New Data
[CreateAssetMenu(fileName = "New Data", menuName = "Character States/Data")]
//继承ScriptableObject类
public class CharacterData_SO : ScriptableObject
{
    [Header("States Info")]
    public int maxHealth;
    public int currentHealth;
    public int maxDefence;
    public int currentDefence;
}
```

在这里配合[CreateAssetMenu]使用可以方便地创建一个继承了ScriptableObject的类的实例，这个实例可以用于配置各种各样的数值信息，其实和通用渲染管道的那个设置项是一个类型的文件。



## Animator Controller 参数中的bool 和 trigger

我理解的trigger就是一个阀门，在动画机中连接两个状态的单向线段（Transition）中，Trigger作为参数，在脚本中触发setTrigger()触发的时候，就会把阀门打开使动画从该状态流向下一个状态，随后再自动把阀门关闭。他会自己实现从开到关的过程。



然后bool呢就相当于一个按钮，就好像控制灯开关的按钮，你要开灯，就要把按钮拨成on的状态，然后灯就会一直亮，同理这个时候就会一直执行true状态的动画，然后你想关灯了，就把按钮拨成off的状态，灯就一直处于灭的状态，同理这个时候就会一直执行false状态的动画。在脚本中需要获取到这个bool参数，然后手动控制参数。



理解好这两个逻辑就没问题了。





## ref out 关键字

这两个关键字都是标识方法的参数的关键字

ref：标记该参数是引用类型的，传入方法的是参数的地址，不是参数的值，所以方法会对这个参数造成实际的变化。传入参数之前，**必须初始化**。

out：和ref类似，方法执行完后，参数的值会受到方法的影响。传入参数之前，**可以不初始化**



## Vector3.SqrMagnitude() 和 Vector3.Distance()

都可以拿来算两点之间的距离，SqrMagnitude的开销要比Distance小，如果可以尽量使用SqrMagnitude，如果是小型项目的话影响不大，但是可读性方面那我还是觉得我们Distance牛逼~





## 接口实现观察者模式的订阅和广播功能

[设计模式学习笔记-观察者模式 - Wang Juqiang - 博客园 (cnblogs.com)](https://www.cnblogs.com/wangjq/archive/2012/07/12/2587966.html)

观察者模式：处理一个信息对应多个事件发生的情况。事件发生了，告知大家执行某些行为，实际上把“事件发生”和“执行行为”两个操作解除了绑定。

在实践中的一个应用是当玩家死亡，就会告知所有的敌人玩家死亡了，然后播放自己的胜利动画。当然使用的时候必须在初始化敌人就先订阅（加入频道），然后玩家死后广播（发布动态）敌人告知玩家已死，可以开香槟了。就好像up主和粉丝的关系，你关注了up主（订阅），up主发布动态让粉丝得知发新视频了。好像就是event和委托里面的**发布/订阅模式**

C#里的event事件系统使用的是观察者模式，unity中的所有事件（比如UI点击事件）也是应用的观察者模式。

- 观察者模式中的四个角色
  - 抽象主题(subject)：把所有观察者对象的引用集中保存，提供一个**接口**，可以增加和删除观察者对象
  - 具体主题：将有关状态存入具体观察者对象；在具体主题内部状态改变时，通知所有订阅了的观察者
  - 抽象观察者：为所有具体观察者定义一个**接口**，得到主题通知时更新自己
  - 具体观察者：实现抽象观察者所要求的更新接口，以便使本身的状态与主题状态协调。

在3DRPG的例子中，很显然GameManager就是一个具体主题类，而EnemyController则是一个具体观察者类，如果不懂的话可以翻代码来看看，有注释，因为代码太长了我就没贴到这里来。



## Animator Override Controller

可覆盖的Controller，这个东西非常吊，假如你已经创建了一个同类型的动画机，而且不需要做任何的参数和层级修改，只需要修改动画，那么就可以用Animator Override Controller，用这个甚至都不用自己复制出来一份改了。



## Animator Controller 动画机

在animator controller中，我们也可以为状态创建一个behavior脚本来控制动画状态的切换。

比如说可以用于敌人攻击时停止移动，受击时、眩晕时停止移动

这种脚本继承自StateMachineBehaviour，有不同于Behaviour的脚本提示：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class StopAgent : StateMachineBehaviour
{
    // OnStateEnter is called when a transition starts and the state machine starts to evaluate this state
    //override public void OnStateEnter(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    //{
    //    
    //}

    // OnStateUpdate is called on each Update frame between OnStateEnter and OnStateExit callbacks
    //override public void OnStateUpdate(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    //{
    //    
    //}

    // OnStateExit is called when a transition ends and the state machine finishes evaluating this state
    //override public void OnStateExit(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    //{
    //    
    //}

    // OnStateMove is called right after Animator.OnAnimatorMove()
    //override public void OnStateMove(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    //{
    //    // Implement code that processes and affects root motion
    //}

    // OnStateIK is called right after Animator.OnAnimatorIK()
    //override public void OnStateIK(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    //{
    //    // Implement code that sets up animation IK (inverse kinematics)
    //}
}

```



## ExtensionMethod 扩展方法

拓展方法适用于当你需要为这个类添加新方法但是不能编辑这个类的情况。比如说向unity中的Transfrom类添加一些和位置、缩放、转向有关的方法。简单来说就是为一个修改不了的类添加更多自定义静态方法

拓展方法所在的**类必须是静态类**，他本身必须是**静态方法**，然后还需要用到**this**关键字

拓展方法的写法如下：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//不继承任何类
//static:随时随地都可以快速调用类内部方法
public static class ExtensionMethod
{
    //攻击扇面的余弦值，也就是说扇面为(-60, 60)
    private const float dotThresheld = 0.5f; //需要使用const，因为是在static类中 
    //this关键字：后面跟着的类就是需要扩展的类，就是可以直接通过一个实例的transform.IsFacingTarget()调用这个方法
    public static bool IsFacingTarget(this Transform transform, Transform target)
    {
        //target的方向
        var vectorToTarget = target.position - transform.position;
        vectorToTarget.Normalize();

        float dot = Vector3.Dot(transform.forward, vectorToTarget);

        return dot >= dotThresheld;
    }
}

```





## Partical System 粒子系统

在Hierarchy层级管理器中右键选择Effect - Partical System 可以创建一个partical System，可以用来做石头破裂之后的碎石效果，各种破碎的效果，还有各种特效：比如说下雪，光粒，身上的buff效果，加血加攻。



## 通过UI为敌人设置血条显示

通过UI里的Canvas画布可以为界面添加UI，Canvas有三种Render mode：

Screen Space - Overlay：在屏幕上，用来做界面UI

Screen Space - Camera：在摄像机视角上

World Space：在世界坐标上

如果要为敌人设置血条跟随在头上显示的话，就把Render mode设置在World Space上。血条的制作是通过两个嵌套的Image实现的，父oj在底，为红色；子oj在上，为绿色，Image Type设置为Filled，Fill Method设置为Horizontal(水平的)，然后通过脚本控制Fill Amount就可以实现实时的血条显示了。



## Update()、LateUpdate、FixedUpdate

Update()：按照游戏帧率更新，通常不是相同帧更新，用于游戏逻辑的更新

LateUpdate()：迟一帧更新，通常用于摄像机跟随和跟随着敌人的UI（血条蓝条之类）

FixedUpdate()：固定帧更新，默认是0.02s，可以在Edit - ProjectSetting - time - Fixedtimestep中修改，用于物理相关的更新

执行顺序：FixedUpdate - Update - LateUpdate





## 异步加载场景和加载进度条

SceneManager.LoadSceneAsync()

使用协程来帮助同场景的转换和不同场景的转换，异步加载场景保证了在使用该场景之前unity已经把场景给加载好了，避免造成切换场景卡顿。



## 关于OnTriggerStay()、OnTriggerEnter()、OnTriggerExit()

用于检测碰撞，如果要使用的话，必须要满足以下条件：

1. 两个go都包含Collider
2. 其中一个go需要勾选isTrigger
3. 其中一个go需要包含rigidbody



## PlayerPrefs 配合 JsonUtility.ToJson 保存玩家数据

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SaveManager : Singleton<SaveManager>
{
    protected override void Awake()
    {
        base.Awake();
        DontDestroyOnLoad(this);
    }

    public void SavePlayerData()
    {
        Save(GameManager.Instance.playerStates.characterData, GameManager.Instance.playerStates.characterData.name); 
    }

    public void LoadPlayerData()
    {
        Load(GameManager.Instance.playerStates.characterData, GameManager.Instance.playerStates.characterData.name);
    }

    public void Save(Object data, string key)
    {
        var jsonData = JsonUtility.ToJson(data); //将数据转化为Json字符串
        PlayerPrefs.SetString(key, jsonData); //将数据以键值形式建立起来
        PlayerPrefs.Save(); //保存
    }

    public void Load(Object data, string key)
    {
        if(PlayerPrefs.HasKey(key))
        {
            JsonUtility.FromJsonOverwrite(PlayerPrefs.GetString(key), data); //将数据重写回Object中
        }
    }
}

```





## TimeLine

TimeLine是一个unity自带的类似于pr一样的动画剪辑器，你可以在轨道上添加移动的关键帧，然后就可以随着时间线性的变化人物的位置或者转向等信息。




## UI Componet CanvasGroup

可以利用CanvasGroup中的Alpha属性来制作渐入渐出的专场效果。Alpha在我看来就是透明度



## 关于FreeLook相机的几个绑定模式 binding mode

千万千万别选错相机了，血的教训啊。

Lock To Target On Assign：本地空间，相机被激活或target赋值时的相对位置。

Lock To Target With World Up：本地空间，保持相机y轴朝上，yaw和roll为0。

Lock To Target No Roll：本地空间，锁定到目标物体，roll为0。

Lock To Target：本地空间，锁定到目标物体

World Space：世界空间

Simple Follow With World Up：相对于目标的位置，使用相机的本地坐标系，保持相机y轴朝上



## [Quaternion](https://docs.unity3d.com/cn/2020.2/ScriptReference/Quaternion.html).LookRotation

public static [Quaternion](https://docs.unity3d.com/cn/2020.2/ScriptReference/Quaternion.html) **LookRotation** ([Vector3](https://docs.unity3d.com/cn/2020.2/ScriptReference/Vector3.html) **forward**, [Vector3](https://docs.unity3d.com/cn/2020.2/ScriptReference/Vector3.html) **upwards**= Vector3.up);

forward：物体要朝向的方向；

upwards：向上方向的向量，默认为Vector3.up

具体的意思就是，当前物体z轴和forward向量对齐，y轴和upwards向量对齐所需的四元旋转量。



## [Physics](https://docs.unity3d.com/cn/2020.2/ScriptReference/Physics.html).OverlapBox

public static Collider[] **OverlapBox** ([Vector3](https://docs.unity3d.com/cn/2020.2/ScriptReference/Vector3.html) **center**, [Vector3](https://docs.unity3d.com/cn/2020.2/ScriptReference/Vector3.html) **halfExtents**, [Quaternion](https://docs.unity3d.com/cn/2020.2/ScriptReference/Quaternion.html) **orientation**= Quaternion.identity, int **layerMask**= AllLayers, [QueryTriggerInteraction](https://docs.unity3d.com/cn/2020.2/ScriptReference/QueryTriggerInteraction.html) **queryTriggerInteraction**= QueryTriggerInteraction.UseGlobal);

center：盒体的中心

halfExtents：盒体各个维度的大小

orientation：盒体的旋转

layerMask：投射射线所选择物体的Layer

queryTriggerInteraction：指定该查询是否应该命中触发器

具体应用：查找和一个盒体（甚至是一个平面）内的所有碰撞体，接下来，创建一个透明盒很有用。



## Dotween

**DOTween**是一款针对Unity的快速高效、类型安全的面向对象的**补间动画引擎**，并且对于C#用户做出了很多的优化。

这玩意是真好使，大概。

[Dotween常用方法详解](https://blog.csdn.net/zcaixzy5211314/article/details/84886663)



## Ezy-Slice

**支持使用一个面片对任意的 Convex Mesh 进行剖切。**

[GitHub - DavidArayan/ezy-slice：Unity3D Game Engine的开源网格切片器框架。用 C# 编写。](https://github.com/DavidArayan/ezy-slice)

用来做水果忍者、或者是合金装备崛起复仇的刀剑模式都很不错



## AnimatorStateTransition

Transition就是那个白线，定义状态机在何时以何种方式从一种状态切换为另一种状态。既然动画这玩意天天要用，不如现在就搞清楚每一个设置都能干啥。

- hasExitTime：勾选后，动画过渡将会有一个退出时间。**一般来说不勾选的话就可以在anim.SetXXX后即时触发动画**
  - exitTime：如果退出时间exitTime是0.5，那么就表示 `A -> B` 下，A如果一帧播放了50%的动画，那么就可以过渡到B状态，如果播放不足50%，则会等到播放到50%后才会开始过渡，exitTime < 1的情况下，A状态每次播放都会判断一次，而 > 1则只会判断一次。
- hasFixedDuration：确定过渡的持续时间是以秒还是标准化时间为单位。如果为true，以秒为单位；如果为false，以标准化时间为单位。好像一般都会勾选这个吧
  - transitionDuration：过渡持续时间，就是那个`\ \`斜着的玩意。
- transitionOffset：目标状态的开始时间。也就是B状态开始的时间，标准化时间为单位。增大Offset的话状态块会向左移。
- （进阶）interruptionSource：动画状态过渡打断。过渡就是那个`\ \`，不是整个过程都叫过渡。
  - 想用的话，可以看看这两篇文章
  - [Animator 状态机切换打断机制_lrh3025的博客-CSDN博客_状态机切换](https://blog.csdn.net/lrh3025/article/details/104012508?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)
  - [Unity Animator状态切换打断探索_问君能有几多发的博客-CSDN博客](https://blog.csdn.net/qq_37940787/article/details/108080877?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1.pc_relevant_default&utm_relevant_index=2)
- （进阶）orderedInterruption：过渡可被具有较高优先级的过渡中断。
- canTransitionToSelf：（存在前提：过渡是从AnyState到某状态）可允许(true)或禁止(false)在 AnyState 过渡期间向自身过渡。勾选的话，则在AnyState向该状态过渡的时候，如果触发了过渡条件，将允许再次从AnyState向该状态过渡，不会等待动画播放完。



## AnimationCurve 动画曲线

这玩意提供一个动画曲线，可以控制动画变化的速率，详情看unity的API。

目前我应用到的场景是运用这个动画曲线实现UI弹入弹出的效果。

```c#
//通过协程实现弹出窗口
IEnumerator ShowPanel(GameObject panel)
    {
        panel.SetActive(true);
        float timer = 0;
        while (timer <= 1)
        {
            //Animation.Evaluate(float time): 通过[0, 1]范围内的时间得到当前曲线的Yz
            panel.transform.localScale = Vector3.one * showCurve.Evaluate(timer);
            timer += Time.deltaTime * animationSpeed;
            yield return null;
        }
    }
```



## AsyncOperation 异步操作协同程序

[UnityEngine.AsyncOperation - Unity 脚本 API (unity3d.com)](https://docs.unity3d.com/cn/2020.2/ScriptReference/AsyncOperation.html)

可以用于制作加载条和对加载界面的控制。



## Unity2D素材的处理：Sprice Editor

在此之前每一个2D素材（Texture 2D）如果选择的Texture模式是Sprite，会有一个SpriteMode，说明这个精灵的模式是怎么样的：Single、Multiple、Polygon。

如果我们得到的素材是许多精灵组成的一个精灵图，我们想要获得里面的素材的话，就需要进行切割。SpriteMode需选择Multiple，然后点击SpriteEditor进入编辑界面，选择Slice，选择Slice方式，一般是选择根据像素点大小来选择，最后点击Apply即可。

顺带一提，2D素材的Inspector下有一个属性叫Pixels Per Unit：每个单元（即Scene中的一个网格）的像素点大小，太大了的话有的小素材放到Scene里就很小。



## Unity2D Tilemap 平铺地图

在New下的2D Object中可以找到Tilemap，创建后会得到一个Grid和子Go Tilemap，在Tilemap内你可以通过TilePalette来平铺式的绘制你的Scene，非常方便。

TilePalette（平铺调色板）：在Window下的2D子菜单可以找到TilePalette，新建一个Palette之后就可以拖动你刚刚切割好的精灵图放进去，就可以使用画笔将素材画到Scene中了，前提是配合Tilemap使用。

你画上去的内容实际上是在Grid的子Go Tilemap里



## Unity2D Tilemap Collider 2D

这碰撞体可太刁了，专门给Tilemap使用的，这个碰撞体可以自动地贴合你画上去的地图。



## Unity2D Transform.Scale.x 调整面朝方向

我擦，调整Scale的x居然可以改变面朝的方向：1就是素材本身的方向（向右），-1就是素材的反方向（向左），这样的特点刚好可以配合Input.GetAxisRow()使用来改变人物的面朝方向。



## Unity2D Cinemachine => Cinemachine Confiner 限制镜头的区域

Cinemachine中存在extension方法，其中有一个CinemachineConfiner，使用这个脚本可以将镜头在一个区域内限定死。2D中的应用场景是将背景贴图先使用一个PolygonCollider包起来，勾选isTrigger，然后拖进CinemachineConfiner中的BoundingShape2D即可将这个镜头限制在背景之内，防止镜头跟随人物超出背景的范围。也可以通过调整PolygonCollider的大小限制镜头的移动来限制玩家到达某个区域可以看到的信息。

顺带一提，PolygonCollider编辑时可以按住ctrl来减少一条边。



## Unity2D Physics Materials 2D 物理材质

Physics Materials 2D 是2D的物理材质，可供调整的参数就俩个：Friction（摩擦力）、Bounciness（弹力），将这个材质给到Collider上就可以改变这个碰撞体在和其他碰撞体碰撞的时候的一些情况。



## Unity 射线系统

这个老哥讲的还行：[简析Unity射线检测的概念与应用 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/421534861)

需要注意的是，**射线的初始位置Start如果被挡住的话，是不会被检测到的**，比如说人物站在地面上的检测，如果射线整条都在地面下，就检测不出来。



## Unity2D Audio系统

Audio Listener：耳朵。通常这个玩意儿在主摄像机上，没有这个就收不到声音。

Audio Source：播放器。通过在go上添加这个组件来得到管理一个AudioClip的播放器。通过AudioSource类的公开方法可以播放，暂停，停止这个AudioClip。

Audio Clips：播放的声音。就是你的音频文件，AudioSource 引用和使用 AudioClip 播放声音。



## Unity GetButton()、GetButtonDown()、GetButtonUp()

三个方法的参数都是buttonName，这个参数通常使用InputManager里所定义的名字，方便后期实现改键的功能。

GetButton(string buttonName)：**只要一直按下该按键，那么一直为true，松开即为false**

**下面两个方法都要求在Update()方法内调用**，如果进行某些物理运算需要用到FixedUpdate时，也要将这两个方法放在Update中，通过bool值来控制。因为FixedUpdate是固定帧率执行，默认为0.02s，即50帧。而Update则是根据电脑实际运行执行，帧率不定。而**下面两个方法，只会在一帧中为true或者为false，这一帧过去了就重置状态，等待下一次输入。**Update实际运行的帧率有时会比FixedUpdate大，有时也会小，这就导致**如果你将Input.GetButtonDown()方法放在FixedUpdate中执行时，你在0.02s内触发了按键为true，然而Update已经在0.02s内已经执行了一遍或两遍以上，这个true就被重置了，表现为按键按下之后没反应。**如果Update在0.02s内没有运行，这时按键就会触发。正是因为Update的帧率不定，Input.GetButtonDown()方法放在FixedUpdate执行时才会给人一种按键有时有反应有时没反应的感觉。

GetButtonDown(string buttonName)：按键按下时返回true，下一帧重置状态（false），如果你觉得人物基本点按的操作手感不好，多半是这个背锅。

GetButtonUp(string buttonName)：按键释放后的第一帧返回true，在用户按下键并再次释放键之前，它不会返回 true。



## Unity Object Pool 对象池 设计模式

对象池这个东西，说白了就是一个容器，里面装着一些对象，你想用的时候，就拿出来用，不用的时候就放回去。

这样做有什么好处呢？**节省资源，不需要在游戏进行时重复创建和销毁，给内存和cpu带来麻烦**

在Unity中，麦扣老师的教程实现人物冲刺的残影，或者一般的射击游戏的子弹，都是使用对象池的地方。这玩意还是很有用的，试想一下，如果你放任子弹随便的生成销毁，对于FPS游戏来说，短暂的帧率下降都会给玩家不好的游戏体验，而对象池在游戏加载的时候就已经创建好了，拿出来用直接就SetActive就可以，就不涉及到生成销毁带来的花销了。

对于对象池中的对象，在刚开始初始化的时候需要考虑到每一次被取出时，**自身的状态不应被上一次的取出所影响，所以对象的初始化需要写在OnEnable()方法中，顺带一提，OnEnable()方法会在每次对象启用（SetActive为true）执行一次**。

下面是麦扣老师教程中实现人物冲刺残影的对象池代码。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ShadowPool : MonoBehaviour
{
    public static ShadowPool Instance; //单例模式实例，使用单例模式调用对象池会方便一些
    public GameObject shadowPrefab; //残影预制体
    public int shadowCount; //残影数量

    //对象池本体，使用队列实现，因为队列的出队入队比较方便取出和放回
    private Queue<GameObject> availableObjects = new Queue<GameObject>(); 
   
    private void Awake()
    {
        if (Instance != null)
            return;
        Instance = this; //单例模式初始化

        //初始化对象池
        FillPool();
    }

    //初始化对象池
    public void FillPool()
    {
        for (int i = 0; i < shadowCount; i++)
        {
            //创建对象
            var newShadow = Instantiate(shadowPrefab);
            newShadow.transform.SetParent(transform);

            //刚创建出来的时候就要取消启用，返回对象池
            ReturnPool(newShadow);
        }

    }

    //对象返回到对象池中
    public void ReturnPool(GameObject go)
    {
        go.SetActive(false); //取消启用

        availableObjects.Enqueue(go); //入队
    }

    //从对象池中取出对象
    public GameObject GetFromPool()
    {
        //如果队列里面的对象不够用了，就扩充一倍
        if (availableObjects.Count == 0)
        {
            FillPool();
        }

        var shadow = availableObjects.Dequeue(); //出队

        shadow.SetActive(true); //启用shadow

        //还有另一种对象池里对象不够用的时候是直接返回一个新创建的对象
        return shadow; 
    }
}
```



## Unity2D 实现视觉差Parallax

Unity2D横板游戏中，为了让画面看上去立体一点，就需要背景相对地有一些偏移，看起来就像背景环境在随着人物移动被甩在后头一样。

原理实际上就是背景跟随相机移动，背景的位置 加上 相机的位置信息乘上一个跟随的比率就可以实现了，比较简单。

不过麦扣老师说实现视觉差好像有一个自带的插件，因为写的脚本效果还不错，我就没深入了解了。

还有一种实现视觉差的方法是直接设置镜头的投影方式Projection，为Perspective，原本是四方平行的Orthographic，设置后就和3D的相机一样的透视效果了，然后通过设置背景和平台的z轴，就可以实现视觉差了。



## Unity2D 实现单向平台

Unity2D自带的Platform Effector 2D 组件就可以实现了，使用条件是当前go拥有一个Collider2D，在这个Collider中勾选Used By Effector即可。



## Unity2D Tilemap Extras

是一个拓展来的，需要自己手动添加（Unity2020版在Preview中，打开Preview需要在ProjectSettings - PacketManager中设置）

详情可以看麦扣老师的视频：[Unity 2D教程:从《Robbie》学习开发03: Tilemap (Rule Tile)_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1FE411f7VJ?spm_id_from=333.999.0.0)

简单来说就是Tilemap中的一些拓展功能，其中在教程里使用到的是Tile中的RuleTile 和 Random Tile。

新建一个RuleTile或者RandomTile后，添加一个Sprite，就可以把这个Tile拖进你的Palette中使用了。使用的话当然在Palette中选中这个Sprite就行

RuleTile：规则平铺，制定对应Sprite的规则（九宫格内）。

RandomTile：随机平铺，输入随机平铺的Sprite的个数，然后分别添加进去即可。



## Unity2D Composite Collider 2D 复合碰撞器



## Unity2D Rigidbody CollisionDetection 碰撞检测 和 Interpolate 插值

CollisionDetection：

在Rigidbody2D组件里，有CollisionDetection，有Discrete（不连续的）和Continuous（持续的）两个选项。

字面意思了，使用Continuous可以更加顺滑的判断当前组件的go碰撞到了什么东西。

Interpolate：

定义如何在物理更新之间插值GameObject(游戏物体)的运动(当运动趋于不稳定时很有用)。

- None：没有插值运算
- Interpolate：根据GameObject在先前帧中的位置对运动进行平滑处理。
- Extrapolate：基于其在下一帧中的位置的估计来平滑运动。



## Unity 法线贴图 Normal Map

[Unity Shader-法线贴图（Normal）及其原理_puppet_master的博客-CSDN博客_unity 法线贴图](https://blog.csdn.net/puppet_master/article/details/53591167?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-53591167-blog-107810518.pc_relevant_paycolumn_v3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-53591167-blog-107810518.pc_relevant_paycolumn_v3&utm_relevant_index=2)

这个老哥的这段话把法线贴图的原理讲清楚了。

> ![img](https://img-blog.csdn.net/20161214010009820?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcHVwcGV0X21hc3Rlcg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
>
> 假如下面是我们的低面数模型，上面是我们的高面数模型，上面的模型在计算光照时，由于面数多，每个面的法线方向不同，所以各个面的光照计算结果都不同，就有凹凸的感觉了，而下面的低模，只有一个面，整个面的光照条件都是一致的，就没有凹凸的感觉了。我们如果把上面的高模的法线信息保存下来，类似纹理贴图那样，存在一张图里，再给低模使用，低模就可以有跟高模一样的法线，进而在计算光照时达到和高模类似的效果，这也就是常说的烘法线的原理。
> 



## Unity 传递动画参数推荐使用编号ID

因为使用字符串的话有时候有可能会出问题，所以可以用编号的形式来传，后续要改参数的时候也不会影响。

```c#
//AnimationId
private int groundId;

private void Awake()
{
    //获得动画参数的Id
    groundId = Animator.StringToHash("isOnGround");
}

private void Update()
{
    anim.SetBool(groundId, movement.isOnGround);
}
```



## Unity PostProcessing 

PostProcessing插件需要去Packet Manager处下载导入，导入后怎么用PostProcessing呢？首先需要在Main Camera处添加Post Process Layer，然后选择一个Layer，使得只有相同Layer的go下的后处理效果才能在这个相机中展现出来。

Main Camera设置好之后，可以创建一个空物体，然后添加组件Post Process Volume，一个配置后处理效果的组件。is Global可以选择是否全局使用该效果；新建一个profile后就可以进行相关后处理的设置了。当然，别忘记调整这个空物体的Layer保证效果正确触发。



## Unity 相机抖动

基于Cinemachine，在cinemachine下创建的虚拟相机的Extensions中可以添加拓展方法：Cinemachine Impulse Listener，脉冲监听器，简单来说就是给相机一个脉冲监听器，当它收到其他物体发射的脉冲信号时，就会造成相应的抖动效果。

物体可以通过添加组件：Cinemachine Collision Impulse Source，通过添加一个Raw Signal原始信号以及物体身上的Collision的Trigger被某个Layer的物体触发后，就会向相机发送脉冲信号，相机接收到脉冲信号，就会根据Raw Signal中的波来造成抖动效果。举个简单的例子，人物拾取物品时相机抖动，Collision Impulse Source中设置的Layer Mask就可以是Player，当人物触发了拾取物品的Trigger就会造成相机抖动。



## Unity AudioMixer



## 在vscode下，alt+shift可以多行选中一起写代码



## 在vscode下，alt加上下方向键可以对一行代码进行上下移动操作



## [SerializeField]让可序列化类型的私有成员也可以显示在组件面板上

具体什么是可序列化，看看官方文档吧。



## 层级管理器窗口按下Alt再点击箭头，可以展开所有子物体





# 暑期实习准备时接触到的小知识

## 1. Time.timeScale = 0时，Update还会运行吗？

是会的。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;

public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Time.timeScale = 0.0f; // 在Update调用之前将timeScale设置为0
    }

    private void Update()
    {
        Debug.Log("???"); // 可以正常输出
        Debug.Log(Time.deltaTime); // 输出为0
    }
}
```

当然，只知道现象是没有用的，我们得搞清楚Time.timeScale的原理：

首先有几个时间我们需要了解：

| 定义                                | Update                              | FixedUpdate            |
| ----------------------------------- | ----------------------------------- | ---------------------- |
| 游戏开始就计时                      | time                                | fixedTime              |
| 帧时间                              | deltaTime                           | fixedUnscaledDeltaTime |
| 游戏开始就计时（不受timeScale影响） | unscaledTime / realtimeSinceStartup | unscaledDeltaTime      |
| 帧时间（不受timeScale影响）         | unscaledDeltaTime                   | fixedDeltaTime         |

1. unscaledTime 和 realtimeSinceStartup 的区别？

   他们的区别只在编辑器中体现，当项目暂停时，unscaledTime的值不会增长，暂停结束后会将暂停时间加回去，而realtimeSinceStartup不管暂不暂停值都会增加。

2. 为什么fixedUnscaledDeltaTime受timeScale影响而fixedDeltaTime不受timeScale影响？
   具体我也不知道为什么，但是fixedDeltaTime是可以在项目中设置具体大小的，因此我觉得不受timeScale影响是可以理解的。fixedUnscaledDeltaTime的的确确是受timeScale影响的，可能是为了支持某种需求吧。

查看unity官方对Time.timeScale的介绍，总的来说有以下几点：

> 1. timeScale是时间流逝速度的缩放比例
> 2. timeScale为1.0时，时间是正常速度。timeScale为0.5时，时间流逝速度会降为正常速度的一半。
> 3. timeScale为0时，所有基于**帧率**的功能都将被暂停
> 4. Time.realtimeSinceStartup这个值不受timeScale影响
> 5. 修改timeScale时，推荐同时以相同比例修改Time.fixedDeltaTime，即 Time.fixedDeltaTime * timeScale。需要这个值的时候，也可以直接用fixedUnscaledDeltaTime
> 6. timeScale为0时，FixedUpdate函数不再执行，而Update和LateUpdate正常运行

以下是实践验证代码，创建一个简单的Cube，将脚本附在上面即可：

```c#
using UnityEngine;
using System.Collections;

public class TimeScaleDemo : MonoBehaviour
{
	private bool moveDirection = true;
	private bool scaleDirection = true;
	public Renderer rend;

	void Start()
	{
		rend = this.GetComponent<Renderer>(); // Cube的MeshRenderer
		StartCoroutine(ChangeColor());
	}

	void Update()
	{
		// Cube的平移：不使用Time.deltaTime
		this.transform.position += Vector3.right * 0.01f * (moveDirection == true ? 1.0f : -1.0f);
		if (this.transform.position.x > 4.0f || this.transform.position.x < 0.0f)
		{
			moveDirection = !moveDirection;
		}

		// Cube的缩放：使用Time.deltaTime
		this.transform.localScale += Vector3.one * Time.deltaTime * (scaleDirection == true ? 1.0f : -1.0f);
		if (this.transform.localScale.x > 2.0f || this.transform.localScale.x < 1.0f)
		{
			scaleDirection = !scaleDirection;
		}
	}

	void FixedUpdate()
	{
		// Cube的旋转：使用Time.fixedDeltaTime
		this.transform.rotation = Quaternion.Euler(Vector3.one * 45.0f * Time.fixedDeltaTime) * this.transform.rotation;
	}

	IEnumerator ChangeColor()
	{
		while (true)
		{
			rend.material.color = Random.ColorHSV();
			yield return new WaitForSeconds(1f); // WaitForSeconds 也和timeScale有关
			yield return new WaitForFixedUpdate(); // 也会受timeScale影响
			yield return null; // 如果返回非时间相关的值，timeScale不会影响到协程运行
		}
	}

	void OnGUI()
	{
		GUI.color = Color.white;

		GUIStyle buttonStyle = new GUIStyle(GUI.skin.button);
		buttonStyle.fontSize = 30;

		GUIStyle labelStyle = new GUIStyle(GUI.skin.label);
		labelStyle.fontSize = 30;

		GUILayoutOption[] options = new GUILayoutOption[] { GUILayout.Width(300), GUILayout.Height(100) };

		GUILayout.BeginHorizontal();
		if (GUILayout.Button("TimeScale = 0", buttonStyle, options) == true)
		{
			Time.timeScale = 0;
		}
		if (GUILayout.Button("TimeScale = 0.5", buttonStyle, options) == true)
		{
			Time.timeScale = 0.5f;
		}
		if (GUILayout.Button("TimeScale = 1", buttonStyle, options) == true)
		{
			Time.timeScale = 1;
		}
		if (GUILayout.Button("TimeScale = 2", buttonStyle, options) == true)
		{
			Time.timeScale = 2;
		}
		GUILayout.EndHorizontal();

		GUILayout.Space(50);
		GUILayout.Label("Time.timeScale : " + Time.timeScale, labelStyle);
		GUILayout.Space(50);
		GUILayout.Label("Time.realtimeSinceStartup : " + Time.realtimeSinceStartup, labelStyle);
		GUILayout.Space(50);
		GUILayout.Label("Time.time : " + Time.time, labelStyle);
		GUILayout.Label("Time.unscaledTime : " + Time.unscaledTime, labelStyle);
		GUILayout.Space(50);
		GUILayout.Label("Time.deltaTime : " + Time.deltaTime, labelStyle);
		GUILayout.Label("Time.unscaledDeltaTime : " + Time.unscaledDeltaTime, labelStyle);
		GUILayout.Space(50);
		GUILayout.Label("Time.fixedTime : " + Time.fixedTime, labelStyle);
		GUILayout.Label("Time.fixedUnscaledTime : " + Time.fixedUnscaledTime, labelStyle);
		GUILayout.Space(50);
		GUILayout.Label("Time.fixedDeltaTime : " + Time.fixedDeltaTime, labelStyle);
		GUILayout.Label("Time.fixedUnscaledDeltaTime : " + Time.fixedUnscaledDeltaTime, labelStyle);
	}
}
```

最后总结：

1. timeScale会影响FixedUpdate的运行速度，但不会影响Update和LateUpdate的运行速度。timeScale = 0时，FixedUpdate完全停止。
2. timeScale不会影响协程本身的运行速度，当timeScale = 0时，如果协程中返回的是WaitForSeconds或者WaitForFixedUpdate，那么协程就会停下。如果想不受影响，需要使用WaitFOrSecondsRealtime。
3. timeScale改变时，**会**对 time、deltaTime、fixedTime、fixedUnscaledDeltaTime 造成影响。
4. timeScale改变时，**不会**对 realtimeSinceStartup、unscaledTime、fixedUnscaledTime、unscaledDeltaTime、fixedDeltaTime 造成影响。
5. 在timeScale改变时，虽然Update函数不被影响，但是deltaTime和time是受影响的。使用时需要注意。
