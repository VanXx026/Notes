# C# 泛型 partial类 枚举 结构体



## 1. 泛型(generic)

泛型这个东西实际上等同于接口在面向对象中的地位，非常重要。并且泛型在开发中也非常常用，这是因为泛型具有**正交性（大部分的数据类型都能使用泛型）**。

| 数据结构 |   泛型   |
| :------: | :------: |
|    类    |  泛型类  |
|   接口   | 泛型接口 |
|   委托   | 泛型委托 |
|   方法   | 泛型方法 |
|   属性   | 泛型属性 |
|   字段   | 泛型字段 |
|   ...    |          |

### 什么是泛型

泛型，全称可以是泛化数据类型，也就是说这个类型是泛泛而谈的，区别于我们之前声明的时候都是一个特化的具体的类型。

比如说，我喜欢听音乐，音乐这个类型就是泛化的，并没有指出是什么音乐类型，只要是音乐我都喜欢听。因此如果我要听音乐，我可以听jpop、可以听kpop、可以听华语流行等等，而jpop、kpop这些音乐类型就是特化的指明了的音乐类型了。

当然我们要听音乐的时候，总得落实到某一个特化的类型，某首歌来听，总不能说我听音乐，听的是一个泛化的概念吧。因此泛型的语义就可以说明泛型是不能够直接使用的，必须经过特化之后才可以使用。

### 为什么需要泛型

按照惯例，我们先来个例子说明泛型的作用：

有一个老板开了一个非常小的商店，只卖苹果，而目前他的打包盒中，也只是用来装苹果：

```c#
public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Apple apple = new Apple() { Color = "Red" };
        Box box = new Box() { Cargo = apple };
        Debug.Log(box.Cargo.Color); // Red
    }
}

class Apple
{
    public string Color { get; set; }
}

class Box
{
    public Apple Cargo { get; set; }
}
```

随着商店慢慢地做大，老板开始卖书了，同样书也需要打包盒给顾客装走，但是现在我们的打包盒Box是专门为苹果准备的，要知道贸然修改已经确定的类是大忌，现在应该怎么办呢？

第一种方法自然是多准备一个用来装书的盒子BookBox：

```c#
public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Apple apple = new Apple() { Color = "Red" };
        AppleBox box = new AppleBox() { Cargo = apple };
        Debug.Log(box.Cargo.Color); // Red

        Book book = new Book() { Name = "New Book" };
        BookBox bookBox = new BookBox() { Cargo = book };
        Debug.Log(bookBox.Cargo.Name); // New Book
    }
}

class Apple
{
    public string Color { get; set; }
}

class AppleBox
{
    public Apple Cargo { get; set; }
}

class Book
{
    public string Name { get; set; }
}

class BookBox
{
    public Book Cargo { get; set; }
}
```

这种方法目前来说乍一看好像没什么问题，万一老板经营得好，后面卖的东西越来越多，导致相应的盒子也越来越多，如果有一千种商品要卖，就要有一千种盒子来装对应的商品，这就会导致**类型膨胀**的问题，后续的类会变得非常多，导致代码很难维护。

第二种方法是只准备一种盒子，在盒子类中声明不同类型的属性：

```c#
public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Apple apple = new Apple() { Color = "Red" };
        Book book = new Book() { Name = "New Book" };

        Box box1 = new Box() { AppleCargo = apple };
        Box box2 = new Box() { BookCargo = book };
        Debug.Log(box1.AppleCargo.Color); // Red
        Debug.Log(box2.BookCargo.Name); // New Book
    }
}

class Apple
{
    public string Color { get; set; }
}

class Book
{
    public string Name { get; set; }
}

class Box
{
    public Apple AppleCargo { get; set; }
    public Book BookCargo { get; set; }
}
```

这种方法问题就更加明显了：box1中只用到了AppleCargo这个属性，box2中只用到了BookCargo这个属性，如果我们有一千种商品要卖，Box类里面就有一千种对应的属性，而且每次使用都只有一个属性是有用的，而且每次新增商品都需要去修改Box这个类，万一不卖某个商品了，得，还要去找到这个属性给他改掉，这就导致了**成员膨胀**的问题，并且还导致代码很难维护。

如果要指明类型的话需要很多的属性，那我们使用object类型的属性不就好了吗，什么类型的商品都可以使用object类型：

```c#
public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Apple apple = new Apple() { Color = "Red" };
        Book book = new Book() { Name = "New Book" };

        Box box1 = new Box() { Cargo = apple };
        Box box2 = new Box() { Cargo = book };
        Debug.Log((box1.Cargo as Apple)?.Color); // 类型转换，如果类型转换失败会返回null而不是异常
    }
}

class Apple
{
    public string Color { get; set; }
}

class Book
{
    public string Name { get; set; }
}

class Box
{
    public object Cargo { get; set; }
}
```

可以是可以，但是有个问题，这样创建出来的Box实例里面的Cargo属性不能直接访问到商品的属性，因为类型是object，object类里面哪有声明Color属性和Name属性啊，这时候就必须使用类型转换来得以访问商品属性。因此这种方法虽然不会导致成员膨胀，但是实际使用起来是及其不方便的。

没辙了，彻底没辙了，我们如何才能满足商品与盒子对应，并且在新增商品的时候维护起来更方便呢？

**泛型可以解决类型膨胀和成员膨胀的问题。**

```c#
public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Apple apple = new Apple() { Color = "Red" };
        Book book = new Book() { Name = "New Book" };

        Box<Apple> box1 = new Box<Apple>() { Cargo = apple };
        Box<Book> box2 = new Box<Book>() { Cargo = book };
        // 这时候的 box1.Cargo 和 box2.Cargo 都是强类型的，一个是Apple，一个是Book
        Debug.Log(box1.Cargo.Color); // Red
        Debug.Log(box2.Cargo.Name); // New Book
    }
}

class Apple
{
    public string Color { get; set; }
}

class Book
{
    public string Name { get; set; }
}

class Box<TCargo> // 类型参数TCargo
{
    public TCargo Cargo { get; set; }
}
```

使用泛型类，完美地解决了我们遇到的问题。
第一个例子中的类型膨胀问题，使用泛型类，我们只需要新增商品，在打包的时候只需指明打包的商品是什么类型就可以使用对应的盒子，不需要新增盒子类。
第二个例子中的成员膨胀问题，使用泛型类，在盒子类内部只需要声明一个对应的参数类型的属性，在商品打包的时候只需要指明商品的类型就可以建立对应商品类型的属性了，不需要新增商品属性。

以上就是一个使用泛型类的例子。



泛型接口在日常开发中是非常常用的（这是因为C#的很多数据结构都是基于泛型的，比如说数组、链表、字典这些。），因此接下来我们再来一个使用泛型接口的例子：

学校的学生都有一个独一无二的标识ID：

```c#
public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Student<int> stu = new Student<int>() { ID = 101, Name = "Van" };
        // 学校扩招了，原来的int类型的ID属性不够用了，现在使用无符号long类型
        Student<ulong> stu2 = new Student<ulong>() { ID = 1000000000001, Name = "Bily" };
    }
}

interface IUnique<TID>
{
    TID ID { get; set; }
}

class Student<TID> : IUnique<TID> // 实现接口的类也是一个泛型类，因为如果要实现泛型接口中的属性，这个类也必须要有对应的类型参数TID
{
    public TID ID { get; set; }
    public string Name { get; set; }
}

```

可以发现，使用泛型接口可以更加灵活地选择学生ID的类型。如果不使用泛型接口，如果想要改变学生ID的类型，不仅需要修改接口内ID属性的类型，而且还要在所有实现了IUnique接口的类中修改实现的ID属性的类型，非常麻烦。

这种情况是当接口被实现的时候，接口仍然是一个泛型接口，因此实现接口的类也是一个泛型类。还有一种用法是当接口被实现的时候，此时接口中的类型参数已经特化为某个类型，此时的类实现的是一个特化的接口，因此当前类不是泛型类。

```c#
public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Student stu = new Student() { ID = 100000000001, Name = "Van" };
    }
}

interface IUnique<TID> // 这里声明的还是一个泛型接口
{
    TID ID { get; set; }
}

class Student : IUnique<ulong> // 在实现接口的时候就把接口给特化了，因此这时实现的就是一个TID为ulong类型的特化接口，因此此时的Student类不是泛型类
{
    public ulong ID { get; set; }
    public string Name { get; set; }
}

// 同样，我们还可以特化一个int类型的接口
class Teacher : IUnique<int>
{
    public int ID { get; set; }
    public string subject { get; set; }
}
```



## 2. partial类

partial关键字只支持类、接口、结构体。

这个关键字能够拆分一个类、一个结构体、一个接口为不同的部分，每个部分都可以有自己的成员，编译时会将同类的部分合并。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;

public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Student student = new Student() { ID = 101, Age = 10, Name = "Van", Subject = "Math"};
        student.Info();
        student.Study();
    }
}

partial class Student // 使用partial关键字
{
    public int ID { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }

    public void Info()
    {
        Debug.Log(ID + Name + Age);
    }
}

partial class Student // 在另外一个类也可以使用其他类的属性和方法
{
    public string Subject { get; set; }
    public void Study()
    {
        Debug.Log(Name + " is studying " + Subject);
    }
}
```

partial类适用于当类的某一部分可能因为业务逻辑改变需要改变内部逻辑的地方，如果使用继承的方法新增派生类的话，这样在每次改动的时候都需要创建派生类也有点麻烦，因此这时可以使用partial类。

![image-20230215220301558](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20230215220301558.png)

## 3. 枚举类型(enum)

枚举类型实际上就是人为限定取值范围的一些整数，是值类型。注意：是类型，是需要创建变量来使用的。

```c#
public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Person person = new Person();
        person.Level = Level.Employee;
    }
}

public enum Level // 默认情况下是0 1 2 3
{
    Employee,
    Manager,
    Boss,
    BigBoss
}
public enum Level // 可以直接赋值，如果赋了值之后，下一个位置没有赋值的话，默认是上一个枚举值+1
{
    Employee = 100,
    Manager, // 101
    Boss = 300,
    BigBoss // 301
}


partial class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Level Level { get; set; }
}

```

枚举类型还有一种比特位用法，这种情况用于在枚举类型中一个变量不仅仅局限于一个枚举，而是满足多个枚举的情况下使用：

```c#
public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Person kunkun = new Person();
        kunkun.Skill = Skill.Sing | Skill.Dance | Skill.Rap | Skill.BasketBall; // 按位或，即按位+
        Debug.Log((int)kunkun.Skill);
        Debug.Log(kunkun.Skill & Skill.Rap); // 按位与，即每一位比较，相同即为1，最后 1111 & 0100 = 0100 -> Skill.Rap
    }
}

public enum Skill // 将每一个枚举值定义为2的倍数，与比特位相符
{
    Sing = 1,
    Dance = 2,
    Rap = 4,
    BasketBall = 8
}

partial class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Skill Skill { get; set; }
}
```



## 4. 结构体(struct)

结构体和类很相似，结构体类型是值类型。值类型和引用类型有什么区别我们在参数那篇笔记中已经说的很详细了，因此不再赘述。

我们可以用一个例子来讲述结构体的特点：

```c#
public class AbstractClass : MonoBehaviour
{
    private void Start()
    {
        Student stu = new Student() { ID = 101, Name = "Van" };
        object o = stu; // 装箱：将值类型转换为引用类型
        Student stu1 = (Student)o; // 拆箱：将引用类型转换为值类型
        // =======================================================
        Student stu2 = stu;
        stu2.ID = 102;
        stu2.Name = "Bily";
        // 因为是值类型，stu2 = stu 是直接进行了copy操作，新建了一块内存空间，而不是像引用类型那样，新建了一个指针指向stu的内存空间
        Debug.Log($"{stu.ID}, {stu.Name}"); // 101, Van
        Debug.Log($"{stu2.ID}, {stu2.Name}"); // 102, Bily
        // =======================================================
        stu.Speak();
        // =======================================================
        Student stu3 = new Student(103, "Mr.Banana");
        stu3.Speak();
    }
}

interface ISpeak
{
    void Speak();
}

struct Student : ISpeak // 结构体可以实现接口，但是不能派生自类或派生自结构体
{
    public int ID { get; set; }
    public string Name { get; set; }

    //public Student() // 报错：结构体不能包含显式的无参构造函数
    //{

    //}
    // 但是可以有显式的有参构造器
    public Student(int id, string name)
    {
        ID = id;
        Name = name;
    }

    public void Speak()
    {
        Debug.Log($"I'm {ID}, {Name}");
    }
}
```







