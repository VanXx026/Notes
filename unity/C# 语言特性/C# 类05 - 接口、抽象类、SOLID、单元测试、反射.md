# 类05 - 接口、抽象类、SOLID、单元测试、反射

> 接口和抽象类是面向对象设计最关键、最精妙的部分。
>
> 具体类 => 抽象类 => 接口：越来越抽象，内部实现的东西越来越少。
> 具体类是完全实现了内部逻辑的类。
> 抽象类是未完全实现内部逻辑的类（可以有字段和非public成员）。专门作为基类来使用，也具有解耦功能。
> 接口是完全未实现内部逻辑的“类”（“纯虚类”：只有public类型的函数成员）。为解耦而生，方便单元测试。



## 1. SOLID设计原则

SOLID设计原则贯穿面向对象设计始终，遵守SOLID设计原则，是学习设计模式的基础，可以让软件更加稳定、灵活和健壮。

### 1. Single Responsibility Principle(SRP) 单一职责原则

**一个类或者一个模块只做一件事。**

让一个类或者一个模块专注于单一的功能，减少功能之间的耦合程度。这样在需要修改某个功能的时候，就不会影响到其他的功能。

![img](https://pic2.zhimg.com/80/v2-e8cc5291a9ced8251ea7c47f0b26ec51_720w.webp)

### 2. Open Closed Principle(OCP) 开放关闭原则

**对扩展开放，对修改关闭。**

对于一个类，它的内部逻辑除了存在bug之外，不要轻易地修改。如果需要新的需求，就对这个类进行扩展（继承、接口）。

![img](https://pic3.zhimg.com/80/v2-8b271d9e5ae3e5fb8335fe0a3fb4aa36_720w.webp)

### 3. Liskov Substitution Principle(LSP) 里氏替换原则

**所有基类出现的地方都可以用派生类替换而不会让程序产生错误。**

因为派生类可以拓展基类的功能，但基类原有的功能是不变的。这与 is a 概念和多态的现象有关。

### 4. Interface Segragation Principle(ISP) 接口隔离原则

**接口应该精简。**

对于不同的功能模块应该分别使用不同的接口，这也是解耦的方式之一，如果使用一个胖接口，那么这个接口的可复用型就会下降。

![img](https://pic2.zhimg.com/80/v2-9f346457f5419daa85ed4997b011a759_720w.webp)

### 5. Dependence Inversion Principle(DIP) 依赖倒置原则

**高层模块不应该依赖底层模块，而是依赖于抽象接口，通过抽象接口使用对应的低级模块。**

其实就是使用接口解耦的过程。

![img](https://pic3.zhimg.com/80/v2-83b0400eb2e7fae0a23fbb235897d716_720w.webp)



## 2. 抽象类与开放关闭原则

抽象类是内部逻辑未完全实现的类。最大的特点就是“未完全”。

因为未完全，因此这个类需要加上修饰符 abstract。
因为未完全，没有被实现的类需要加上修饰符 abstract。
因为未完全，抽象类中有可以有其他的实现的方法。
因为未完全，因此抽象类不能实例化。

```c#
namespace Example
{
    // 因为抽象类中可能存在方法没有实现，因此抽象类不能实例化
    // 两个作用：
    // 1 - 作为基类派生出派生类
    // 2 - 作为父类类型引用子类实例，这时抽象类变量就可以去调用子类已经实现的抽象方法
    abstract class Student
    {
        // 访问限制不能是private，如果其他类都无法访问它，那么谁来实现它呢？
        abstract public void Study();
        
        // 抽象类中可以有具体实现的方法
        public void Eat()
        {

        }
    }
}
```

首先我们来看看这个例子，看看抽象类是如何优化我们的代码结构的：

```c#
namespace Example
{
    class Car
    {
        public void Run()
        {
            Debug.Log("Car is running");
        }

        public void Stop()
        {
            Debug.Log("Stopped");
        }
    }

    class Truck
    {
        public void Run()
        {
            Debug.Log("Truck is running");
        }

        public void Stop()
        {
            Debug.Log("Stopped");
        }
    }
}
```

在这个例子中，Car和Truck的区别只有Run方法，Stop方法没有区别。但是在书写代码的过程中，代码可以Ctrl + C + V是绝对大忌。因此我们的代码需要优化：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;
using Example;

public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        // Vehicle v = new Vehicle(); // 报错：无法创建抽象类或接口“Vehicle”的实例
        Vehicle v = new Car();
        v.Run();
    }
}

namespace Example
{
    abstract class Vehicle
    {
        // 当然这里使用虚方法virtual也可以，使不使用虚方法在于Vehicle这个类需不需要生成实例
        public abstract void Run();

        public void Stop()
        {
            Debug.Log("Stopped");
        }
    }

    class Car : Vehicle
    {
        public override void Run() // virtual 和 abstract 子类方法都需要显式声明override重写
        {
            Debug.Log("Car is running");
        }
    }

    class Truck : Vehicle
    {
        public override void Run()
        {
            Debug.Log("Truck is running");
        }
    }

    abstract class NewVehicle: Vehicle
    {
        // 如果抽象类的子类不实现其抽象方法，那么子类必须也声明为抽象类
    }
}
```

通过抽象类，我们将Run方法和Stop方法进行了封装，并且将Run设置为抽象方法，在子类中进行实现。这是符合开放关闭原则的代码结构，将已经确定的结构我们不再改动（Vehicle），对于扩展（Car、Truck）我们通过继承来开放，重写我们的需求（Run方法）。

当然，我们还可以更过分一点，让Stop方法也变成抽象方法，这时VehicleBase就是一个完全的抽象类，然后让Vehicle继承VehicleBase，在Vehicle中实现Stop方法，就像这样：

```c#
namespace Example
{
    // 抽象类中完全由抽象方法组成
    abstract class VehicleBase
    {
        public abstract void Run();
        public abstract void Stop();
    }

    abstract class Vehicle : VehicleBase
    {
        //public abstract void Run(); // 由VehicleBase继承得到，因此不需要再书写

        public override void Stop()
        {
            Debug.Log("Stopped");
        }
    }

    class Car : Vehicle
    {
        public override void Run()
        {
            Debug.Log("Car is running");
        }
    }

    class Truck : Vehicle
    {
        public override void Run()
        {
            Debug.Log("Truck is running");
        }
    }
}
```

当然，完全抽象的抽象类在实际开发中是很少见的，因为抽象类不是这么用的。这时，如果我们需要一个类完全由抽象方法组成，那么我们会使用接口，这个部分我们先引出接口这个概念：

```c#
namespace Example
{
    // 抽象类中完全由抽象方法组成
    //abstract class VehicleBase
    //{
    //    public abstract void Run();
    //    public abstract void Stop();
    //}

    // 不如考虑使用接口来实现这样的需求
    // 接口名通常前面需要接一个I表明这是一个接口
    // 接口中的方法不需要显式声明为 abstract 和 public，因为接口就是干这个的，没有异义
    interface IVehicle
    {
        void Run();
        void Stop();
    }

    abstract class Vehicle : IVehicle
    {
        public abstract void Run();

        public void Stop()
        {
            Debug.Log("Stopped");
        }
    }

    class Car : Vehicle
    {
        public override void Run()
        {
            Debug.Log("Car is running");
        }
    }

    class Truck : Vehicle
    {
        public override void Run()
        {
            Debug.Log("Truck is running");
        }
    }
}
```



## 3. 接口

### 1. 接口的基本概念

接口是完全未实现内部逻辑的“类”（“纯虚类”：只有public类型的函数成员，**也可以存在属性**）。为解耦而生，方便单元测试。

接口是更加抽象的一种使用，在接口中，抽象方法不需要显式表明 abstract ，甚至连访问级别也不需要显式声明。
因为接口中的抽象方法一定是public的，不像抽象类中的抽象方法可以是public、protected 也可以是internal。因此在接口中声明一个方法可以不加访问级别修饰符（默认是public），但是要加，就一定是public。

为什么一定是public？public这个访问级别决定了接口的本质：服务的消费者和服务的提供者之间的契约。契约即合同，对于双方合同应当是绝对透明可见的。实现接口的类就是服务的提供者，如果接口中的需求对于类不可见，那还有什么建立契约的必要吗？就好比甲方提出的需求乙方不清不楚一样，是非常危险且荒谬的。对于类来说，不可见（private、protected、internal都有不可见的可能性存在）就没有办法完成接口内部提出的需求（方法），接口不得到完全的实现是会报错的。

当然，接口和抽象类类似，自定义接口类型的变量可以去引用实现了该接口的类的实例，进而达到多态的效果，而这种用法对于解耦、依赖反转来说非常有用。

### 2. 依赖反转（依赖倒置）原则

面向对象是对现实世界的抽象，既然在现实世界中存在分工合作，那么在面向对象中，也存在类与类、对象与对象之间的分工合作。而类与类、对象与对象之间的合作，就叫**依赖**。
存在依赖，那么就存在耦合。依赖越直接，耦合就越紧。

我们先来个例子看看什么是依赖，什么是耦合：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;
using Example;

public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        var engine = new Engine();
        var car = new Car(engine);
        car.Run(3);
        Debug.Log(car.Speed); // 输出结果为：30
    }
}

namespace Example
{
    class Engine
    {
        public int RPM { get; private set; }
        public void Work(int gas)
        {
            this.RPM = 1000 * gas;
        }
    }

    class Car
    {
        private Engine _engine;
        public int Speed { get; private set; }
        public Car(Engine engine)
        {
            _engine = engine;
        }

        public void Run(int gas)
        {
            // 实际上Run内的逻辑是：this.Speed = 1000 * gas / 100
            _engine.Work(gas);
            this.Speed = _engine.RPM / 100;
        }
    }
}
```

Car类和Engine类是依赖关系，Car依赖Engine类才能运行，因为Car类中的字段_engine是Engine类型的。一个类的运行需要另一个类作为前提，那么就存在依赖。

既然存在依赖，那么就一定存在耦合，而这个例子是紧耦合，耦合程度是比较大的。如果Engine类需求改变了，比如说Engine中的Work类逻辑变化，那么Car类有可能因此出错或者发生改变。而且这种情况需要在Engine类中直接修改逻辑，这其实不符合开放关闭原则。一旦某个类牵连的依赖类过多，那么直接修改内部逻辑就更加危险。

如何降低耦合度呢？使用接口，接口就是干这个的：接口是一个契约，接口约束了一组功能（方法），并且约束了功能的调用者只能调用这些功能，功能的调用者也无需关心这些功能是谁提供的，因为接口保证功能完全实现。正是因为调用者无需关心功能是谁提供的，因此在调用者内部，我们就不需要显式的指明是谁来提供这些方法，从而达到降低耦合度的效果。

比如说，人需要使用手机，手机可以打电话。我现在用的是小米手机，可以打电话；如果我的手机坏了，换了苹果，同样可以打电话，需求是不变的，但是手机类型变了。如果我们像上面一样只写了小米手机和我的依赖，那么人内部的逻辑就要改了，而且新建的苹果手机类区别也和小米手机并不大，这样的写法就很抽象了。

我们来试试手机的这个例子，和上面的例子差不多，使用接口就灵活很多了：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;
using Example;

public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        //Van van = new Van(new MiPhone());
        Van van = new Van(new HuaWeiPhone());
        van.UsePhone();
    }
}

namespace Example
{
    interface IPhone
    {
        void Call();
    }

    class MiPhone : IPhone
    {
        public void Call()
        {
            Debug.Log("MiPhone is calling...");
        }
    }

    class HuaWeiPhone : IPhone
    {
        public void Call()
        {
            Debug.Log("HuaWeiPhone is calling...");
        }
    }

    class Van
    {
        private IPhone _phone;
        public Van(IPhone phone)
        {
            _phone = phone;
        }

        public void UsePhone()
        {
            _phone.Call();
        }
    }
}
```

然后我们再来谈谈依赖反转原则。解耦在代码中的表现就是依赖反转。

依赖反转的意思是高层模块不应直接依赖底层模块（比如高层模块是人，底层模块是车），而是通过间接依赖的方式使用接口来使用底层模块。

那么为什么要叫依赖反转呢？依赖我们已经说过了，为什么叫反转呢？

看这个图，就可以明白了：

<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20230213142237082.png" alt="image-20230213142237082" style="zoom: 50%;" />

第一张图，也就是紧耦合的形式，人依赖于车，箭头是从上到下的形式。
第二张图，通过接口对人和车进行解耦，这时人依赖于接口，而两种车类实现接口的方法，因此箭头的方向就**反转**了，这就是依赖反转，也就是接口解耦。
第三张图，需求再次改变，这时通过两个接口，就有不同的人和车的组合，这时实现不同的组合不需要对类做过多改变，这时的耦合度就很低了。
最后的再逐步演化，就变成了我们公认的设计模式了。

### 3. 单元测试

单元测试是依赖反转的直接引用和受益者。

单元测试在看视频的时候我理解的是白盒测试，通过用例去测试某些模块是否能通过测试用例，进而测试程序的健壮性。目前来说仅做了解即可。

## 4. 接口隔离原则

接口隔离原则说的是一个功能应该对应一个接口，也就是说接口应该精简。

为什么接口应该精简，这是因为接口的特点决定的。接口规定功能的调用者不会多要，功能的提供者不会多给，因此功能的调用者只能拿到那么多方法，功能的提供者也必须提供那么多的方法。如果功能的提供者实现了一个胖接口，也就是有多余功能的接口，这不符合单一职责原则。多余的功能调用者也用不上，甚至会导致其他的问题产生。接下来，我们用一些例子来看看接口隔离原则的好处。

### 接口隔离原则的第一个例子（接口结构不合理）

下面的代码，我们有一个Driver类，依赖IVehicle运行，Car、Truck类实现Ivehicle接口，LightTank、MediumTank类实现ITank接口。

```c#
namespace Example
{
    class Driver
    {
        private IVehicle _vehicle;
        public Driver(IVehicle vehicle)
        {
            _vehicle = vehicle;
        }

        public void Drive()
        {
            _vehicle.Run();
        }
    }

    interface IVehicle
    {
        void Run();
    }

    class Car : IVehicle
    {
        public void Run()
        {
            Debug.Log("car is running");
        }
    }

    class Truck : IVehicle
    {
        public void Run()
        {
            Debug.Log("truck is running");
        }
    }

    interface ITank
    {
        void Run();
        void Fire();
    }

    class LightTank : ITank
    {
        public void Fire()
        {
            Debug.Log("LightTank fire!");
        }

        public void Run()
        {
            Debug.Log("LightTank is running");
        }
    }

    class MediumTank : ITank
    {
        public void Fire()
        {
            Debug.Log("MediumTank fire!");
        }

        public void Run()
        {
            Debug.Log("MediumTank is running");
        }
    }
}
```

接下来我有一个这样的需求：我的车路上追尾坏掉了，现在想换一台怎么撞都撞不坏的坦克（绷），坦克能开就行，不需要作为武器使用。目前的代码结构能用吗？显然是不行的，Driver类中依赖的是IVehicle类型，坦克和IVehicle无关，如果我们想要换一台坦克的话，必须更改Driver中的逻辑，这是我们不能接受的。

问题就出在我们的接口上：ITank这个接口并不符合接口隔离原则，ITank中的方法实际上还可以更精简。我们开坦克不一定非要使用武器对吧？那么我们不如将ITank拆成两个接口：IWeapon、IVehicle。然后再由ITank实现IWeapon、IVehicle接口的方法。这样，结构就更好了，也更加说得过去。

修改后如下：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;
using Example;

public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Driver driver = new Driver(new MediumTank());
        driver.Drive(); // 输出结果：MediumTank is running"
    }
}

namespace Example
{
    class Driver
    {
        private IVehicle _vehicle;
        public Driver(IVehicle vehicle)
        {
            _vehicle = vehicle;
        }

        public void Drive()
        {
            _vehicle.Run();
        }
    }

    interface IVehicle
    {
        void Run();
    }

    class Car : IVehicle
    {
        public void Run()
        {
            Debug.Log("car is running");
        }
    }

    class Truck : IVehicle
    {
        public void Run()
        {
            Debug.Log("truck is running");
        }
    }

    interface IWeapon
    {
        void Fire();
    }

    interface ITank : IVehicle, IWeapon
    {

    }

    class LightTank : ITank
    {
        public void Fire()
        {
            Debug.Log("LightTank fire!");
        }

        public void Run()
        {
            Debug.Log("LightTank is running");
        }
    }

    class MediumTank : ITank
    {
        public void Fire()
        {
            Debug.Log("MediumTank fire!");
        }

        public void Run()
        {
            Debug.Log("MediumTank is running");
        }
    }
}
```

### 接口隔离原则的第二个例子（调用者使用接口类型过胖）

第二个例子是需求与提供不匹配导致的错误。功能提供者根据接口合理地提供了所需的功能，功能调用者错误地使用了接口类型导致多要了一些功能。

这个例子我们自己实现了一个可以迭代的类ReadOnlyCollection，一个类如果可以迭代则需要实现IEnumerable接口，因此在这个例子中，Sum方法错误地选择了ICollection类型作为参数，导致我们ReadOnlyCollection类型的实例roc无法使用Sum方法，因为Sum方法所使用的参数类型过大了（胖），ICollection中有的方法ReadOnlyCollection没有实现，因此需求和提供不匹配。

```c#
public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        int[] nums1 = { 1, 2, 3, 4, 5 };
        ArrayList nums2 = new ArrayList() { 1, 2, 3, 4, 5 };
        Debug.Log(Sum(nums1));
        Debug.Log(Sum(nums2));

        var roc = new ReadOnlyCollection(nums1);
        foreach(var n in roc)
        {
            Debug.Log(n); // 可以正确输出，说明ReadOnlyCollection实例可以被迭代
        }

        Debug.Log(Sum(roc)); // 报错：无法从"ReadOnlyCollection"转换为"System.Collections.ICollection"
    }

    static int Sum(/*ICollection nums*/ IEnumerable nums) // 这里的参数选择的不合理，如果需要使用迭代器，只需实现IEnumerable即可，对于当前的需求来说，ICollection这个接口太胖了
    {
        int sum = 0;
        foreach (var n in nums)
        {
            sum += (int)n;
        }
        return sum;
    }
}

class ReadOnlyCollection : IEnumerable
{
    private int[] _array;

    public ReadOnlyCollection(int[] array)
    {
        _array = array;
    }

    // 实现IEnumerable接口的方法GetEnumerator，即必须能获取到迭代器
    public IEnumerator GetEnumerator()
    {
        return new Enumerator(this);
    }

    // 创建一个内部迭代器类，继承IEnumerator接口
    public class Enumerator : IEnumerator
    {
        private ReadOnlyCollection _collection;

        private int _head; // 下标

        public Enumerator(ReadOnlyCollection collection)
        {
            _collection = collection;
            _head = -1;
        }

        // Current属性：当前迭代到的对象
        public object Current
        {
            get
            {
                object o = _collection._array[_head];
                return o;
            }
        }

        // 迭代器++
        public bool MoveNext()
        {
            if (++_head < _collection._array.Length)
            {
                return true;
            }
            else
            {
                return false;
            }
        }

        // 迭代器重置
        public void Reset()
        {
            _head = -1;
        }
    }
}
```

### 接口隔离原则的第三个例子（接口的显式实现）

这个例子我觉得非常合适的讲清楚了什么时候需要使用到接口的显式实现：

一个暖男杀手，在平时并不会表现出杀戮的一面，大多情况下都是暖男，因此在这里，如果我们的变量类型是WarmKiller的时候，是不应该能出现Kill的需求的，只有在我们是杀手的情况下，才会出现Kill的需求。因此我们将IKiller接口显式实现内部的Kill方法，这样一来，只有在类型是IKiller的变量引用WarmKiller实例时，Kill方法才可以正常地被调用，因为现在是杀手嘛。

```c#
public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        WarmKiller wk = new WarmKiller();
        wk.Love();
        //wk.Kill(); // 报错：找不到这个方法

        IKiller killer = new WarmKiller();
        killer.Kill(); // 可以正常被调用
    }
}

interface IGentleman
{
    void Love();
}

interface IKiller
{
    void Kill();
}

class WarmKiller : IGentleman, IKiller
{
    public void Love()
    {
        Debug.Log("I love you forever...");
    }

    // 显式实现接口方法：只有在当类型变量为IKiller引用WarmKiller实例时，才可以看到这个方法
    void IKiller.Kill()
    {
        Debug.Log("Let me kill the enemy.");
    }
}
```

## 5. 反射

### 1. 反射的概念

反射(Reflection)，字面意思就是在镜子中看到的那个虚像。
意思就是，如果给了一个对象，反射可以在**不用new操作符**的情况下，甚至**不需要显式指明是什么类型**的情况下，创建出一个同类型的对象（镜子里的虚像），还能访问对象内的成员。
就像镜子里站着一个人，我不管这个人是什么品种的人，镜子总能照出一个一模一样的人，而不需要和镜子说你给我照一个什么样的人出来。

这种不需要使用new操作符，不需要知道类型就可以得到一个对象的特点，就使得耦合的程度甚至可以弱到忽略不计。之前我们在提到解耦合的时候，常常通过使用接口实现依赖倒置来解耦合。尽管如此，也还是需要指明接口的类型，也还是需要使用到new操作符。

反射在C#语言中非常重要，单元测试、依赖注入、泛型编程都是基于反射机制的。

### 2. 为什么需要反射

程序的逻辑有时候不是在编译阶段就能决定的，有时候逻辑是要到了用户和程序的运行时，即交互阶段才能确定。这时候程序是动态的。 如果我们在程序的编译阶段即静态的时候要去枚举用户有可能做的操作，那么程序就会变得非常臃肿，有可能你会写出很多的ifelse语句，这是不可能的。因此我们的程序应该有一种可以以不变应万变的能力，而这种能力就是反射。

下面还是人和车和坦克的例子，通过这个例子我们来看看反射是怎么写的：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;
using System.Reflection; // 使用反射相关的命名空间
using Example;

public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        ITank tank = new LightTank();
        // ============ 以下的内容就是反射 ============
        // 没有使用 new 操作符
        // 没有显式的声明LightTank类型
        // 只通过一个对象tank,就可以得到另一个一模一样的LightTank类型的对象,并且可以调用其中的方法
        var t = tank.GetType(); // Type类：关于这个类的一些静态表述
        object o = Activator.CreateInstance(t); // 通过激活器创建实例
        MethodInfo fireMi = t.GetMethod("Fire");
        MethodInfo runMi = t.GetMethod("Run");
        fireMi.Invoke(o, null); // LightTank fire!
        runMi.Invoke(o, null); // LightTank is running
    }
}


namespace Example
{
    class Driver
    {
        private IVehicle _vehicle;
        public Driver(IVehicle vehicle)
        {
            _vehicle = vehicle;
        }

        public void Drive()
        {
            _vehicle.Run();
        }
    }

    interface IVehicle
    {
        void Run();
    }

    class Car : IVehicle
    {
        public void Run()
        {
            Debug.Log("car is running");
        }
    }

    class Truck : IVehicle
    {
        public void Run()
        {
            Debug.Log("truck is running");
        }
    }

    interface IWeapon
    {
        void Fire();
    }

    interface ITank : IVehicle, IWeapon
    {

    }

    class LightTank : ITank
    {
        public void Fire()
        {
            Debug.Log("LightTank fire!");
        }

        public void Run()
        {
            Debug.Log("LightTank is running");
        }
    }

    class MediumTank : ITank
    {
        public void Fire()
        {
            Debug.Log("MediumTank fire!");
        }

        public void Run()
        {
            Debug.Log("MediumTank is running");
        }
    }
}
```

### 3. 反射的作用

因为目前的编程水平不足，在Unity开发中没有办法接触到这两个内容，并且Tim老师的例子中我使用Unity很难复现，因此这部分的内容暂且搁置。

#### 1. 依赖注入



#### 2. 插件编程（更松的耦合）



