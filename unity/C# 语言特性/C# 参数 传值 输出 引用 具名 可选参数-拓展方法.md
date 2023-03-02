# C#中的各种参数

上次我们学到了C#中在类型中承载数据的四种成员：字段、属性、索引器、常量，这次我们来看看C#中的各种参数。



## 1. 传值参数

传值参数也叫值参数。开发中99%的情况我们使用的都是传值参数。

值参数不需要带任何修饰符，在函数中充当的是一个局部变量，是一个函数中使用完就扔的过客，对于他的实参来说，他只是一个副本，因此对于值参数的操作永远不会影响到变量的值。

另外，值参数和值类型是不同的玩意，值类型是数据类型中的一种，包括结构体（我们使用的float、int这些也算作结构体类型）和枚举。值参数是参数中的一种。

<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20230203102453368.png" alt="image-20230203102453368" style="zoom:50%;" />

传值参数中的参数本质上也是数据，所以传值参数可以是值类型的，也可以是引用类型的。

说到引用类型，我想先说一下C#中引用类型的构成。这对接下来理解引用类型的传值参数以及引用类型的引用参数有帮助。

**C#中的引用类型变量，其实是存储在栈空间中，指向堆内存某个位置的地址变量。**这块堆内存的数据实际上就是引用类型真正的那个数据类型（对象）。意思就是本质上引用类型的变量就是一个地址。

> 栈：我们平时调用函数，main函数调用功能函数，功能函数再调用下属的函数，其实就是一个压栈的过程。Main函数始终是最后出栈的，而最后调用的函数和他的参数，会最先出栈，这也是我们程序的执行顺序。
>
> 堆：堆一般由程序员分配释放，若程序员不释放，目前很多语言都有自动的垃圾收集器，这样也方便非常方便我们编程。我们很多的实例化之后的类和泛型，其实都是存储在堆空间中的。

### 1. 传值参数 => 值类型

值类型的传值参数想必就不需要过多介绍了，实际上这种类型的参数就是实参的一个副本，在函数内使用这个参数就和实参无关了。

### 2. 传值参数 => 引用类型，用参数创建新对象

**传值参数的一个非常重要的特点就是一定会创建一个实参的副本，同样对于引用类型的传值参数也不例外**，这时形参 —— 实参的一个副本，也就是实参指向的对象的地址了。那么现在的情况就是，有两个地址变量，一个实参一个形参，都指向了同一个对象。接下来我们需要使用形参创建新对象，需要使用new关键字，因为形参是新创建出来的一个副本，因此形参就理所当然的修改自己的地址，指向创建的这个新对象了，那么使用形参来创建新对象，实际上就和实参无关了。如图：

<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20230203150221695.png" alt="image-20230203150221695" style="zoom:67%;" />

### 3. 传值参数 => 引用类型，用参数操作对象

这种情况下，实参和形参两个变量同样都指向了实参的对象，如果使用形参对对象进行修改的话，那么改变的就是对象中的值，对象还是那个对象没有变。如图：

<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20230203150716675.png" alt="image-20230203150716675" style="zoom:67%;" />

在C#中，所有的引用类型都基于Object这个基类，在这个基类中有一个方法GetHashCode()可以获取到一个对象的唯一标识，通过这个方法我们就可以验证经过形参修改后的对象还是不是原来的那个对象：

```c#
public class ClassNamespace : MonoBehaviour
{
    class Student
    {
        public string Name { get; set; }
    }

    void Start()
    {
        Student stu = new Student() { Name = "Van" };
        Debug.Log("实参指向对象修改前的哈希值: " + stu.GetHashCode() + " Name: " + stu.Name);
        ValueArg(stu);
        Debug.Log("实参指向对象修改后的哈希值: " + stu.GetHashCode() + " Name: " + stu.Name);
    }

    void ValueArg(Student stu)
    {
        Debug.Log("形参指向对象修改前的哈希值: " + stu.GetHashCode() + " Name: " + stu.Name);
        stu.Name = "Niko";
        Debug.Log("形参指向对象修改后哈希值: " + stu.GetHashCode() + " Name: " + stu.Name);
    }
}
```

输出情况：

<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20230203151808520.png" alt="image-20230203151808520" style="zoom: 80%;" />

首先，实参和形参在对象修改前的哈希值是一样的，说明实参和形参指向的对象是同一个。
其次，形参在修改前和修改后哈希值没有发生改变，说明是改变的是原来指向的对象，并没有创建新对象。
最后，形参和实参在对象修改后的哈希值仍然是一样的，说明形参在函数内的操作直接是直接影响实参指向的对象的。
结论：引用类型的传值参数会创建实参的副本，并且这个副本同样指向同一个对象，并且在函数内的操作直接影响该对象。



## 2. 引用参数（ref修饰符）

> 这里可能说法有些不严谨，实参和形参这两种说法应该是传值参数专有的，在这里对变量和参数作实参和形参的表述有些不准确，能理解就行。

在函数调用中，传值参数不需要添加任何修饰符，引用参数则需要使用`ref`修饰符来声明这个参数是一个引用参数。

**与传值参数不同，引用参数并不创建一个新副本，而是直接使用变量的内存空间的值（值或地址）。**因此，在变量可以作为引用参数传递之前，这个变量必须明确存在赋值。

在使用时，不仅在函数定义时的参数表上要使用引用参数时加上ref修饰符，在函数调用填写参数的时候对应的参数也需要加上ref修饰符。

### 1. 引用参数 => 值类型

这种情况也是比较简单的，既然不存在副本而是使用变量的内存空间，那么在函数内对参数做出修改的话，变量也会受到影响了。

### 2. 引用参数 => 引用类型，用参数创建新对象

这种情况就和传值参数不同了，由于传值参数中形参就是一个副本，因此对这个副本创建新对象，和实参指向的对象并不冲突；而引用参数中，由于形参不是实参的副本，它直接使用实参的内存空间，因此指向的是原来的那个对象的堆内存。那么这时形参使用new关键字创建一个新对象，返回的地址值直接就赋值在了实参的内存空间中，替代了原本的地址，因此旧对象被新对象给覆盖。

<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20230203154053174.png" alt="image-20230203154053174" style="zoom:80%;" />
图中的引用参数使用虚线框起来，表示实际上并没有创建变量的副本，而是直接使用变量的内存空间。

### 3. 引用参数 => 引用类型，用参数操作对象

和第二种情况一样，形参直接使用实参的内存空间，因此使用形参来修改对象的值，最终是使用的实参内存空间中的地址信息找到的对象，所以修改的是实参所指对象的值。

<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20230203154510240.png" alt="image-20230203154510240" style="zoom:80%;" />

同样，我们可以通过例子来验证一下：

```c#
public class ClassNamespace : MonoBehaviour
{
    class Student
    {
        public string Name { get; set; }
    }

    void Start()
    {
        Student stu = new Student() { Name = "Van" };
        Debug.Log("实参指向对象修改前的哈希值: " + stu.GetHashCode() + " Name: " + stu.Name);
        ValueArg(ref stu); // 函数调用时也需要添加ref修饰符
        Debug.Log("实参指向对象修改后的哈希值: " + stu.GetHashCode() + " Name: " + stu.Name);
    }

    void ValueArg(ref Student stu) // 参数表中使用引用参数时也需要添加ref修饰符
    {
        Debug.Log("形参指向对象修改前的哈希值: " + stu.GetHashCode() + " Name: " + stu.Name);
        stu.Name = "Niko";
        Debug.Log("形参指向对象修改后哈希值: " + stu.GetHashCode() + " Name: " + stu.Name);
    }
}
```

![image-20230203155112277](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20230203155112277.png)

说明参数确实是直接使用了变量的内存空间，通过内存空间的地址修改了变量所指的对象。

### 4. 引用类型的传值参数和引用参数在用参数操作对象上有什么区别？

引用类型使用传值参数操作对象，是通过复制了变量存储的对象的地址值，从而操作对象。
<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20230203155729739.png" alt="image-20230203155729739" style="zoom:50%;" />

引用类型使用引用参数操作对象，是通过直接使用了变量的内存空间，获取到对象的地址值 ，从而操作对象。
<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20230203155702477.png" alt="image-20230203155702477" style="zoom: 50%;" />

从结果上看，使用传值参数和引用参数没有区别。

从实现方式上看，是存在区别的：
使用传值参数的方式，是依赖于副本中的地址找到对象的；
使用引用参数的方式，是直接使用变量中的地址找到对象的。



## 3. 输出参数（out修饰符）

用`out`修饰的形参是输出参数。

类似于引用参数，输出参数不创建副本，而是直接访问变量的内存空间，使用其中的值（值或地址）

但是和引用参数不同的是，**输出参数不一定需要变量有明确的赋值**，因为输出参数和引用参数的作用不同，引用参数是使用这个参数来达到函数中的某些功能，而输出参数是使用这个参数来作为函数的输出之一。虽然输出参数在函数调用前不需要明确的赋值，**但是在函数返回之前，也就是输出之前必须要有明确的赋值。**

同样，输出参数也是分值类型和引用类型的，因为输出参数与引用参数很类似，因此变量与参数的原理是完全一样的，区别就在于，输出参数在方法调用中必须赋值，不然就意义了。

和引用参数一样，如果需要使用输出参数，在方法声明的参数表上，哪个参数需要使用输出参数，就加上out修饰符。同样，在方法调用的参数对应位置上也需要加上out修饰符。



## 4. 数组参数（params修饰符）

C#中有string.Format方法可以接受任意多个的参数来格式化：
```c#
string string.Format(string format, params object[] args)
```

可以发现，这个函数的**最后一个参数**使用了params修饰符，也就是说这个数组参数的作用是接受任意多个相同类型的参数传入方法中处理。

因此，我们可以得知：数组参数适用于这个方法可以接受一组相同类型，并且用处也相同的数量不确定的参数一并传入方法中处理。

特点：数组参数必须是参数表中的最后一个，且由params修饰。

如果我们现在需要写一个方法，用于数据的总和计算，通常来说我们是这样写的：

```c#
public class ClassNamespace : MonoBehaviour
{
    void Start()
    {
        // 在方法调用前，我们需要得到数组类型的数据
        int[] scores = { 1, 2, 3 };
        int sum = CalSum(scores);
        Debug.Log(sum);
    }

    int CalSum(int[] scores) // 传值方式传入一个数组类型的数据
    {
        int sum = 0;
        foreach (int score in scores)
        {
            sum += score;
        }
        return sum;
    }
}

```

如果使用数组参数的话，我们就可以直接将数据作为参数传入方法中：

```c#
public class ClassNamespace : MonoBehaviour
{
    void Start()
    {
        int sum = CalSum(1, 2, 3); // 直接将数据作为参数，在这里不同于引用参数和输出参数，就不需要写params修饰符了
        Debug.Log(sum);
    }

    int CalSum(params int[] scores) // 这里需要声明为数组参数
    {
        int sum = 0;
        foreach (int score in scores)
        {
            sum += score;
        }
        return sum;
    }
}
```

当然，这个例子显然不是数组参数的使用场景，这样反而更麻烦了些，可读性也很差，这个例子只是为了方便理解。



## 5. 具名参数

具名参数，就是在方法调用的时候写上方法声明时对应的参数名。

```c#
public class ClassNamespace : MonoBehaviour
{
    void Start()
    {
        PrintInfo(name: "Van", age: 20); // 在调用时每一个参数都写上参数名
    }

    void PrintInfo(string name, int age)
    {
        Debug.Log(name + ", " + age + "desu");
    }
}
```

乍一看好像没什么用，实际上有两个好处：

1. 提高了代码的可读性，可以一下就知道这个参数是做什么的
2. 参数之间的书写顺序可以改变，比如这里的age和name就可以写反

总之我觉得是没什么用的功能（）



## 6. 可选参数（不推荐使用）

参数因为具有默认值而变得"可选"。

```c#
public class ClassNamespace : MonoBehaviour
{
    void Start()
    {
        PrintInfo(); // 因为这个方法有默认值了，所以可以不写参数
    }

    void PrintInfo(string name = "Van", int age = 20)
    {
        Debug.Log(name + ", " + age + "desu");
    }
}
```

但是这种方法有个问题，如果你想使用name的默认值，但是age不使用，写成这样：

```c#
PrintInfo(18); // 会报错：参数1：无法从int转换为string
```

这时候就可以使用具名参数：

```c#
PrintInfo(age: 18); // 正常输出
```



## 7. 拓展方法（别名：this参数）

拓展方法为什么叫扩展方法？为什么需要扩展方法呢？

首先，在开发中，C#类库提供的方法不是永远都够用的，有时候你可能需要让string类型多一些方法来快速地使用，这时候我们又不能去修改C#类库的源码，那么这时候我们就需要使用拓展方法了。

1. 首先拓展方法就是拓展的方法，他首先是一个方法，这个方法必须是公有的、静态的，即public static修饰的。
   - 因为这个方法必须全局可用，并且不应该是某个对象的方法，而是基于类来说的，因此是公有且静态的。
2. 这样还没完，方法是公有的静态的还不能说明它是拓展方法，拓展方法的参数列表的第一个参数必须由this修饰，因此拓展方法又叫this参数。
   - 这个this参数实际上是指明拓展的是哪个类型的方法，如果是拓展string类型，那么第一个参数就是this string typeName
3. 这样还没完，这个方法还必须由一个静态类（一般类名为[SomeType]Extension）来统一收纳对SomeType类型的拓展方法

还是得来个例子才能体现这个拓展方法的用法：

我们现在想要将x保留小数位的前四位，但是在写的时候发现，double类型没有内置Round方法，我们只能从System命名空间里面找到Math类的方法Round来使用。

```c#
public class ClassNamespace : MonoBehaviour
{
    void Start()
    {
        double x = 3.14159;
        double y = System.Math.Round(x, 4);
        Debug.Log(y);
    }
}
```

那我们能不能给double类型拓展一个Round方法呢？

```c#
public class ClassNamespace : MonoBehaviour
{
    void Start()
    {
        double x = 3.14159;
        // 如果把光标移到这个Round上面，可以看到这样一行：
        // (拓展) double double.Round(int digits)
        double y = x.Round(4); // 使用拓展方法之后，this参数对应的那个类型就可以通过使用拓展方法了
        Debug.Log(y);
    }
}

static class DoubleExtension // 静态类：作为工具类，不能实例化
{
    public static double Round(this double value, int digits) // 第一个参数通过this指明拓展的类型，其本身也可以作为正常的参数在方法内使用
    {
        return System.Math.Round(value, digits);
    }
}
```

这就是拓展方法，实际上拓展方法在开发中用的挺多的，当然目前来说是知道有这个东西就可以了。

