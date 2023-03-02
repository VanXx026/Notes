# C# 字段 属性 索引器 常量



这四种成员都是用于表达数据的。

C#的类型就是指类或者结构体，而C#类型的成员就有：

<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20230130184347780.png" alt="image-20230130184347780" style="zoom:50%;" />



## 1. 字段（Field）

定义：字段是一种表示与对象或类型（类与结构体）关联的变量。
通俗点来说，字段就是成员变量。**所以字段是声明在类体里面的。**
如果是与对象相关联的变量，那么就是实例字段，也就是一般的成员变量了。
如果是与类型相关联，就是静态字段了，这个字段和整个类有关，即使有很多个实例，这些实例对应的该字段都是一样的，这个字段由static修饰。

字段的英文名是 field，其实就是指的存放数据的那块“土地”，也就是空间。

### 1. 字段的初始化

1. 如果字段在声明的时候没有进行初始化，那么字段会获得其类型的默认值，比如int即0，Object即null。

   ```c#
   // 类内
   private int age; // 没有自发初始化，实际上系统帮你初始化为默认值0
   ```

2. 实例字段（一般的成员变量）初始化的时机为**对象创建时**

   - 这个也很好理解，毕竟实例字段是属于实例的，当然是实例创建的时候才会初始化

   - 如果我们将实例字段初始化为这样：

   - ```c#
     private int age = 100;
     ```

   - 其实和在实例构造函数这样赋值是等价的：

   - ```c#
     private int age;
     public ClassNamespace()
     {
         age = 100;
     }
     ```

3. 静态字段（static修饰的字段）初始化的时机为**当前类型被第一次加载时**

   - 静态字段是属于类型的，不管当前类型有没有实例，静态字段都是有意义的，所以在类型被加载的时候，静态字段理应初始化。

   - 我们可以这样初始化静态字段：

   - ```c#
     private static int amount = 10;
     ```

   - 和在静态构造函数中赋值是等价的：

   - ```c#
     private static int amount;
     // 静态构造函数
     static ClassNamespace()
     {
         amount = 10;
     }
     ```

### 2. 只读字段

只读的字段可以通过readonly修饰符修饰，这种字段唯一的初始化机会只有在声明时直接初始化（也等价于在构造函数中赋值）。
只读字段可以是实例只读字段，也可以是静态只读字段：

```c#
// 像这种紫色的修饰符，都是连在一起写的，顺序随意，但是一定要在类型修饰符之前全部写完
public readonly int age;
private static readonly int amount;
// 只读字段同样可以直接在声明时初始化
// 也可以在实例构造函数或者静态构造函数中赋值
```



## 2. 属性（Property）

定义：属性是一种**用于访问对象或类型的特征**的成员，特征反映了状态。**属性是字段的自然拓展。**

实际上属性和字段都可以用于表示数据，但是在对类外暴露数据的情况下，属性要比字段安全。
属性中的定义提到了访问特征的一个特性，这里的意思是属性作为表示数据的成员，他可以对数据进行逻辑上的限制，以达到表现某种类型的特征。比如表示人的类Person，你可以将年龄数据设为属性，在属性中限制人类最多不能超过200岁，因为人类的年龄本身就不会过高，这是人类的特征。此外，这种限制数据的做法也可以保护数据不被非法值（外部赋值为200+）所污染。

在类中，表示数据的部分都应该考虑到数据的安全问题，如果使用public的字段，那么这个字段在类外就可以随意的被修改，很容易造成bug，因此在Java中，为了保护字段的安全，通常来说会使用private来修饰字段，并且提供两个公开的getter和setter方法作为外部接口来使用这个字段，而C#中的属性这个成员，其实就是为了方便使用这种方法（语法糖）进化而来的。

### 1. 属性的定义

属性有完整声明和简略声明两种。

#### 完整声明

在完整声明当中，我们首先得有一个私有的字段，然后才能声明该字段的一个属性：

```c#
// 在visual studio中，可以敲propfull然后按两下tab就可以给出完整声明的模板

// 私有字段age
private int age;
// age字段的公开属性Age
public int Age
{
    get { return age; }
    set { age = value; } // value是c#提供给属性内部使用的参数，其实就是setter中的传入参数
}
```

在完整声明中，我们就可以窥探到Java中保护数据方法的样子了，这种方法比起传统的办法有什么好处呢？
之前的方法是比较冗余的，如果在类外需要修改这个字段，必须要调用setter方法：person.SetAge(10)
而使用属性的话，就不需要写调用方法的语法了，而是直接赋值的方式：person.Age = 10。而这种写法就必须要求属性是公开的，不然没法用，因此大部分的属性都是公开的。

如果使用反编译工具查看的话，你会发现除了我们的私有字段age和公开属性Age之外，还多生成了两个方法，这两个方法就是getter和setter。

#### 简略声明

简略声明就是声明属性的语法糖了，毕竟如果使用完整声明去写属性（毕竟一个类型不可能只有一个属性），会比较冗杂。
这属于是语法糖的语法糖了（绷）

```c#
// 在visual studio中，可以敲prop然后按两下tab就可以给出简略声明的模板
public int Age { get; set; }
```

如果使用反编译工具来查看这个类，你会发现实际上编译器除了getter和setter生成了之外，还多生成了一个字段：
< Age > BackingField : private int32
这个就是我们Age这个属性封装的私有字段，也就是存放年龄数据的内存空间。

但是简略方法有一个缺点：在get和set中没办法添加额外的逻辑来限制数据。

#### 静态声明

静态声明其实也是一样的，完整声明和简略声明都可以，但是完整声明需要注意，提供的字段和声明的属性都必须带上static修饰符。

```c#
// 完整声明
private static int age;
public static int Age
{
    get { return age; }
    set { age = value; }
}

// 简略声明
public static int Age { get; set; }
```

### 2. 属性与字段的关系

在完整声明中，我们就可以看出，实际上属性大多数情况是字段的包装器，起到对字段的封装效果，更加安全。
在实际开发中，如果当前的数据是需要暴露在外的，建议**永远**使用属性来承载这个数据，即字段永远都应是private或protected的。



## 3. 索引器（indexer）

这个玩意比较少见，也比较少用，我是没咋见过。**拥有索引器这个成员的类型，基本上都是集合类型。**

定义：索引器可以使**对象**能够用与数组相同的方式（下标索引）进行遍历

用法：通常使用在集合类型中对类型进行索引，比如在Student类中定义一个索引器，可以通过下标来检索每一个student对象。

写法：

写法有点类似属性，但是在声明时有所不同。

```c#
// 声明成绩字典[string, int]
private Dictionary<string, int> scoreDictionary = new Dictionary<string, int>();

// indexer tab tab
// 索引器声明
// public : 公开的成员
// int? : 成员类型为可空的int型
// this : 表示当前类，在外可以通过 this[subject] 的方式来得到 int? 类型的返回值
// [string subject] : 通过 string 类型的变量 subject 来索引字典中的值
public int? this[string subject]
{
    get { 
        if(this.scoreDictionary.ContainsKey(subject))
        {
            return this.scoreDictionary[subject];
        }
        else
        {
            return null;
        }
    }
   set { 
        if(!value.HasValue)
        {
            throw new System.Exception("Score cannot be null");
        }
        if(this.scoreDictionary.ContainsKey(subject))
        {
            this.scoreDictionary[subject] = value.Value;
        }
        else
        {
            this.scoreDictionary.Add(subject, value.Value);
        }
    }
}

// 调用
Student stu;
debug.Log(stu["Math"]); // this[subject]
```



## 4. 常量（constant）

定义：常量是表示常量值的类成员，在编译的时候直接将值替代常量标识符，就不需要额外的再访问常量存放的内存空间。

**常量隶属于类型**，没有实例常量这种说法。“实例常量”的角色由只读实例字段来担当。但是有区别，字段终究只是变量，使用时是需要访问内存空间的，而常量则不需要，所以常量终归效率上是高一些。

常量的声明也很简单，加上const修饰符即可。

### 1. 各种“只读”的应用场景

- 为了提高程序的可读性和执行效率 —— 常量
- 为了防止对象的某个值被改变 —— 只读实例字段
- 向外暴露不允许修改的数据 —— 只读属性（静态或非静态），功能与常量有些重叠
- 当希望成为常量的值其类型不能被常量声明接受时（想要声明类/自定义结构体的常量时，因为const只能修饰一些基本的int double这些类型） —— 静态只读字段