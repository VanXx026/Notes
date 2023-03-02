# Unity 动画系统



## 1. Animator 组件

![image-20220912200929196](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220912200929196.png)

Controller：添加该go的AnimationController，没什么好说的

Avatar：就像一个翻译，把互不兼容的骨骼结构转换为统一的unity肌肉系统，从而实现不同骨骼结构的动画复用。

Apply Root Motion：如果想让go基于动画本身运动（旋转和移动）而不是通过脚本控制时，你可以勾，不过一般都不勾。

Update Mode：选择该组件的更新方式以及使用什么时间标度，有三个选项：

- Normal：Animator和**Update**同步调用，并且**动画速度和时间标度相匹配**，使用这个模式可以简单的**实现慢动作**，比如我们将Time.timeScale改为0.5，那么动画播放将会比平时慢一倍。
- Animate Physics：Animator和**FixedUpdate**同步调用，**受timeScale影响**，如果这个go需要频繁地和其他go进行**物理**交互，那么可以用这个。
- Unscaled Time：Animator和**Update**同步调用，**忽略时间标度的影响**，不顾一切地**保持一倍速**播放动画，该模式可以用在**UI**上，防止timeScale改变的时候UI动画也受到影响。

Culling Mode：为动画选择的剔除模式，即当go在摄像机视角外动画渲染的选择，有三个选项

- Always Animate：在摄像机外也始终播放动画，即使在摄像机外也**不剔除**。
- Cull Update Transforms：只剔除Update Transform部分，动画**仍然进行后续帧**的计算。当go进入摄像机视角时，动画还在播放，似乎从未间断一样。
- Cull Completely：完全剔除，动画不进行任何计算，go离开摄像机视角便**完全静止**，直到go进入摄像机视角才播放下一帧。





## 2. 动画状态机之间的Transitions

Transitions：

- Solo：独奏，给该过渡加上之后，多个状态机之间只能通过该过渡进入/退出。
- Mute：静音，禁用该过渡。

Has Exit Time：是否需要退出时间

Settings：

- Exit Time：动画播放到何时才过渡到下一个状态，比如值是0.97，那么意味着动画播放了97%后开始过渡到下一个状态。

- Fixed Duration：如果勾选，那么过渡时间以秒为单位；未勾选，unity官方说的是过渡时间解读为源状态的标准化时间的一部分，我不懂是什么意思

- Transition Dration：过渡时间（就是时间轴里面蓝色的条），如果勾选了Fixed Duration，那么就是以秒为单位。

- Transition Offset：过渡偏移量，如果值为0.5，指过渡在目标状态动画的50%处开始过渡，前50%不参与过渡。

- Interruption Source：因为同一个动画控制器同一时间只允许一个过渡运行，但是可以打断过渡。这个选项选择当该过渡被打断时应该怎么做：（这里插播一个链接，讲的很清楚：[如何中断Unity动画状态机的转换过程？ (qq.com)](https://mp.weixin.qq.com/s?__biz=MzkyMTM5Mjg3NQ==&mid=2247534784&idx=1&sn=71076ea5980a126973f20135bdb69cc6&source=41#wechat_redirect)）

  - None：不打断过渡，默认值，一般情况下过渡不会被打断。
  - Current State：优先判断进入状态的过渡
  - Next State：优先判断退出状态的过渡
  - Current State then Next State：先判断进入状态的过渡，再判断退出状态的过渡
  - Next State then Current State：先判断退出状态的过渡，再判断进入状态的过渡

- Ordered Interruption：勾选的话，那么只有比目前这个过渡更高优先级的过渡才能打断；不勾选，优先级没有影响。（优先级就是这个，walk到idle的过渡优先级比walk到run高，所以如果勾选，那么walk到run的过渡在选择了CurrentState的情况下只能被walk到idle这个过渡打断）![image-20220912225413934](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220912225413934.png)

  ![image-20220912224248915](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220912224248915.png)

  

  ![Inspector 中显示的过渡设置和过渡图](file:///E:/Unity/2020.3.25f1c1/Editor/Data/Documentation/en/uploads/Main/AnimatorTransitionSettingsAndGraph.svg)

![image-20220912212754476](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220912212754476.png)





## 3. 一维动画树 1D BlendTree

Parameter：一维动画树就只有一个参数可选，用于控制一维动画树

Motion：动作，可选动画，也可套娃一个blendTree

Thresho：阈值，当到达对应动画的阈值，就完全正式开始播放这个动画。

一个钟一样的东西：改变对应动画的速度，默认是一倍速

一个灰白一半的人：镜像反转动画

Automate Thresholds：勾选，那么Motion内的阈值会自动等比例的划分，范围在0到1之间；不勾选，可以自定义阈值。

Compute Thresholds：（仅在未勾选Automate Thresholds时可选）根据你选择的数据设置阈值，也就是说不用你自己设：

- Speed：根据速度（速度的量级）设置每个运动的阈值。
- VelocityX：根据 velocity.x 设置每个运动的阈值。
- VelocityY：根据 velocity.y 设置每个运动的阈值。
- VelocityZ：根据 velocity.z 设置每个运动的阈值。
- Angular Speed(Rad)：根据角速度（弧度/秒）设置每个运动的阈值。
- Angular Speed(Deg)：根据角速度（角度/秒）设置每个运动的阈值。
- 例如，假设您有一段速度为每秒 1.5 个单位的行走动画、一段速度为每秒 2.3 个单位的慢跑动画以及一段速度为每秒 4 个单位的奔跑动画，则从下拉选单中选择 **Speed** 选项将基于这些值为三段动画设置参数范围和阈值。所以，如果将速度参数设置为 3.0，则会混合慢跑与奔跑，并略微偏向慢跑。

Adjust Time Scale：调整时间刻度，用于调整在Motion中为每一个动作设定的速度。似乎和游戏运行无关，只是为了调整动作的速度

- Homogeneous Speed：均匀的速度，不太懂什么意思，没什么用
- Reset Time Scale：全部重置为1

![image-20220912234032795](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220912234032795.png)



## 4. 二维混合树 2D BlendTree

Blend Type：不同的2D混合类型具有不同的用途，区别在于如何计算对每个Motion的影响：

- 2D Simple Directional：不适合用于复杂的Motion转换，两个Motion之前有度数的限制（180度），所以在同一方向上不应该有多个Motion，比如向前走和向前跑。
- 2D Freeform Directional：和2D Simple Directional类似，但是支持同一方向上的多个Motion，而且应在坐标原点也设立Motion，比如idle
- 2D Freeform Cartesian：上面两种都是当Motion表示不同方向运动时使用的。如果你的Motion不是表示不同方向的运动，那么可以用这个。这时X和Y可以表示不同的概念，比如线速度和角速度。
- Direct：让用户直接控制每个节点的权重，通常用在面部表情变化

剩下的各种选项都和一维的差不多，我就不重复说了。

![image-20220913233919988](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220913233919988.png)





## 5. 动画重定向

如果通用动画的话，动画下载下来直接改成humanoid就可以了，用它自己的骨架一般来说没什么问题，反而是使用已有角色的骨架会有问题。

AnimationType：动画的类型

- None：这个模型不用来播放动画
- Legacy：历史遗留，兼容旧版本unity的动画
- Generic：通用，不是人形的可用
- Humanoid：人形动画都用这个，这保证了人形动画之间的通用性

Avatar Definition：选用了Humanoid就会出现这个选项：

- Create from this model：根据这个模型原有的骨骼创建avatar（骨骼和unity肌肉对应的关系）
- Copy from other avatar：官方的说法是指向另一个模型设置的avatar。我的理解是根据其他的avatar来复制创建avatar，这个不好用，因为其他模型的骨骼和当前动画模型的骨骼很可能有差异（即使是人形），从而导致生成失败。

Skin Weight：设置可以影响单个顶点的最大骨骼数量，因为性能原因这个选项通常可以不用动。

Optimize Game Object：优化游戏对象。当我们使用了Avatar之后，播放动画就和模型文件上的骨骼无关了（此时AnimationClip保存的信息是肌肉的拉伸程度，不是骨骼的移动信息），勾选之后，这个模型上的骨骼文件就会被删掉。这个选项作为程序可以不用动。



## 6. 动画层 Layers

使用多层动画可以实现动画的结合，比如Base Layer层的idle状态可以和Arm Layer的Shoot状态集合，就变成了站着瞄准的动画效果。

![image-20220916111756852](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220916111756852.png)

每一个动画层右边都有一个齿轮样式的设置图标，点进去是这样的：

![image-20220916111827957](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220916111827957.png)

Weight：每个Layers的权重，通过权重来调整该Layers对动画的影响。在最顶层的Layer权重固定为1，不能修改。下面的Layer可以修改权重，且**越下面的Layer，优先级越大。**

- 假如这时最下面的Layer权重为1，那么它将会完全接管动画的播放，其他层的动画没有效果。
- 假如这时中间层权重为1，下面层权重为0，那么由中间层的动画接管。
- 假如这时中间层权重不为0，下面层权重不为1，那么中间层和下面层动画会融合播放。

Mask：Avatar Mask，蒙版，蒙版指定要应用动画的主体部分，比如你这层的动画是不是只有上半身要播放，下半身交给其他层，那么就在新建的AvatarMask中这样设置：（图在下面的Avatar Mask介绍中）

Blending：混合类型，指定如何应用动画。

- override：覆盖，选择override，该层动画将优先于它上面的Layers，也就是上面提到的情况，这是默认选项。
- additive：附加：选择additive，该层动画将和它上面的（**也选了additive的**）Layers的动画相加播放，而不会出现完全覆盖的情况。这个东西用起来比较复杂，暂时知道有什么用就行

Sync：同步，选中之后，会出现 'Source Layer' 的下拉框，选择你要和该层同步的另外一个动画层。同步之后，该层的状态机结构会完全相同，在该层除了每个状态内使用的动画可以修改外，对状态机的布局或结构修改将会对同步层做相同的修改。这个选项可以应用在复用动画层的结构上，比如你有正常状态下的行走/奔跑/跳跃，又有受伤状态下的行走/奔跑/跳跃，那么就可以考虑使用这个选项了。

Timing：通过 Timing 复选框，Animator 可调整同步层中每个动画所需的时间（由权重决定）。如果取消选中 Timing，则会调整同步层上的动画。该调整会将动画的长度拉伸到与原始层上的一致。如果选中该选项，则动画长度将在两个动画之间平衡（基于权重）。在两种情况下（选中和不选中），Animator 都将调整动画的长度。如果不选中，则原始层将是唯一母版。如果选中，则采用折中方案。

IK Pass：IK，inverse kinematic，反向运动。这个选项用于是否激活或停用反向动力学。具体说，是激活或停用任何附加在该动画层对应的go中的脚本的"OnAnimatorIK"方法。



顺便提一提Avatar Mask窗口的选项：

<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220920212216222.png" alt="image-20220920212216222" style="zoom:67%;" />

Humanoid：选中绿色，当avatar mask附加到动画层上时，这些绿色的位置就可以进行运动和使用IK；反之红色的地方不能运动和使用IK。

Transform：如果动画不使用人形 Avatar，或者需要更精确地控制对各个骨骼的遮罩，则可以选择或取消选择模型层级视图的相应部分：在层级窗口中选中需要被遮罩的骨骼。



## 7. Animation视图

![image-20220925103548777](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220925103548777.png)

右边的区域如果要拖动指示条，点击蓝色区域来拖动就可以了，其他地方拖不动

Preview：点不点都行，预览

红色按钮：关键帧录制，这个功能很吊，很方便你去调整物体的位置和碰撞体的参数，比自己手动调要方便得多，录制的时候上方的蓝色时间轴会变成红色

剩下的五个按钮，我想我不用介绍了。

Samples：采样率，就是帧率，Samples是60就意味着一秒60帧，只是时间轴的细化程度，不会影响到关键帧的位置（如果界面中没有，可以在右上角的三个小点添加）

顺带一提，右上角三个小点点开之后，可以看到一个Ripple的选项，这个选项勾选之后，启用波纹剪辑（逆天）：是一种用于移动和缩放所选关键帧的方法，选中的关键帧如果向左向右拖动，会影响其他的关键帧也一起移动，像波浪一样。而一般的效果则只是单单选中的关键帧可以移动而已。

准心一样的图标：Filter by Selection，根据选择过滤，这个选项让我们只在时间轴中显示我们选择的游戏对象，这在拥有**复杂骨骼动画的物体上非常方便的用来调试某些骨骼的动作**。（当然目前来说我很可能用不到）

菱形图标：加关键帧，没什么好说的

**子弹图标：**重点来了：Add Event，添加事件，这个东西真的非常常用，使用Collider做攻击判定的时候非常管用，还有比如左脚落地右脚落地时发出声音啥的，也可以在这个瞬间加个事件来实现。**动画事件中只能添加public方法，private不会显示**

- 如果是选中了一个物体：那么事件的界面是这样的：![image-20220925211409080](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220925211409080.png)
- 如果你选中的是一个AnimationClip：那么事件的界面是这样的：![image-20220925211541154](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220925211541154.png)

最后，左下角的Curves，曲线：当然，用过ae知道这是什么玩意，可以调整动画曲线，让动画动起来更加的合理而不是生硬。当然，你对导入的动画不满意，可以手动调整，配合filter by selection可以很方便的针对某个部位进行调整。当然，作为程序来说，这个技能可以说是不太必要的，动画师会帮我们解决的。



## 8. RootMotion

终于讲到根运动了，每次想调试使用动画都要搞大半天，使用动画都不能如愿达到效果。

在游戏开发中，常见的动画文件有两种：inplace动画（不带位移）；root motion动画（自带根位移）。

这种带位移的动画好处是解决了角色动画和位移不匹配的问题。

### 1. 勾选了Animator组件中的Apply Root Motion之后都发生了什么

我做了一个实验，两个方块，放在不同的位置（一个是(0,0,0)，一个是(4,0,0)），但使用同一个AnimationClip，作用是从(0, 0, 0)移动到(0,2,0)，也就是向上移动2米。

其中一个方块不勾选Apply Root Motion，另一个方块勾选，并且打开AnimationClip的动画循环，启动游戏：

- 不勾选：方块根据AnimationClip所记录的坐标值一帧一帧的移动，播放完了就从起点开始这样循环播放；

- 勾选：方块在(4,0,0)的位置逐渐地不断向上移动，动画结束后仍然在不停的向上移动，一直不停地向上移动。

然后，我让两个方块都不勾选Apply Root Motion，结果非常的Amazing啊：

- (0,0,0)的方块：和上面一样
- (4,0,0)的方块：在(0,0,0)点开始，和(0,0,0)点的方块做着一样的动画

虽然只是简单的实验，但是结果非常的匪夷所思啊，结合这位大佬的视频：[【Unity动画系统详解 十五】Unity中Root Motion的核心机制（勾选Animator组件中的Apply Root Motion后到底发生了什么）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1kZ4y1B7Tc/?spm_id_from=333.851.header_right.fav_list.click)，得出以下的结论：

**AnimationClip会根据文件中所记录的每一帧坐标值和角度值生硬地（是多少就是多少）改变游戏对象的坐标值和角度值；而启用了Root Motion，则是提取AnimationClip中记录的数据计算出相对位移和相对角度来移动游戏对象（只关心距离，并不关系起点和终点是多少）**

### 2. Generic动画中的root motion

Generic动画中也是有骨骼的，而且有一个虚拟的（看不见）根骨骼（俗称大根），一般在动画物体的原点，专门用来记录动画播放时的坐标值和角度值。

如果你使用Generic动画，比如小狐狸的行走，在不勾选ApplyRootMotion的情况下，动画还是走完之后瞬移回起点接着走。勾选之后，就可以一直向前走了，这和我们上面的实验效果是一样的。

所以

在Generic动画中，应用root motion指的是把动画中根骨骼节点上的绝对坐标和绝对角度，转换为游戏对象的相对位移和相对转角。

### 3. RootMotion（Generic）的基本设置

Root Transform Rotation

- Bake Into Pose：字面意思是把旋转当作动画（Pose）的一部分，而不是游戏对象移动的一部分。简单来说，勾选了表示不要将骨骼根节点的旋转当作root motion（在Generic中表示为大根中存储的坐标和角度值，即勾选后这里的值不会再发生改变了）的一部分来处理，而是当成普通的骨骼动画来处理**（只有动画在旋转，实际上游戏对象的rotation是没有变化的）**。
  - 说了这么多，那么什么时候我们需要勾选这个选项呢？
  - 当我们**不希望动画带动游戏对象旋转时**考虑勾选，也可以参考左边红/绿灯的提示。
- Base Upon：动画的方向基于：
  - original：原点，根据这个动画在创建时就设置好的原点
  - Root Node Rotation：基于unity计算的根结点，听说并不准（）
- Offset：偏移量，提供给你手动调整

其他两个垂直位移和水平位移选项和这个差不多，我就懒得写了，踏马的这个大佬说的贼清楚。总的来说就是，这个Bake Into Pose，就是你要不要把这个旋转啊位置啊什么的设置为是动画的一部分还是说你要用来影响到游戏对象。如果勾选了，那么就只有动画会变，游戏对象上的rotation和position都不会变；如果不勾选，说明就指望root motion带游戏对象完成位移或旋转了。

### 4. Humanoid动画中的root motion

在Generic动画中，我们可以通过动画文件中根骨骼描述的绝对坐标和绝对角度来转换为相对坐标和相对角度并以此来移动游戏对象。但到了Humanoid动画里，由于使用的是avatar系统，动画文件不再包含对具体骨骼的描述，就没有办法通过指定根骨骼来应用root motion。

unity通过计算骨骼的结构，得出模型的重心center of mass（Generic中就是计算的RootNode是吧），这个重心也可以被称为body transform。根据avatar系统的计算，这个body transform计算的位置和模型骨骼的hips的位置非常接近。（unity：什么嘛，我计算的还是蛮准的嘛）。接下来，unity会根据具体动画的运行计算重心在水平平面上的投影，并把这个投影当做root motion的根骨骼节点来对待，这个点被称作root transform。

![image-20220926235204531](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220926235204531.png)

Humanoid动画在应用root motion时，unity会把root transform上的位移应用在游戏对象上。我们可以把unity计算得到的root transform当成humanoid动画下代表root motion的根骨骼节点（我觉得实际上是对Generic的root motion方案做了一个模拟吧）。

总结如下：

在Humanoid动画中，unity会计算出一个root transform，root motion 会把动画文件中描述的root transform的绝对坐标和绝对角度，转换为相对位移和相对旋转角，并以此来移动游戏物体。在这里，avatar把互不兼容的骨骼结构下的root transform转换成了统一的body transform，这样，同一套带有root transform的动画就可以在不同骨骼结构的人形角色上表达位移。（不太懂后一句是什么意思。。。）

### 5. RootMotion（Humanoid）的基本设置

大体上和Generic的设置相差不大。

Bake Into Pose还是一样，如果勾选，就不会把动画上的位移和角度作用在游戏对象上，而是作用在动画本身。

Root Transform Rotation

- Base Upon：动画的方向基于
  - Original：动画文件设定的原点，一般可以信任选这个，有问题就把动画师纱了祭天吧（
  - Body Orientation：就是我们说的那个unity计算出来的body transform

Root Transform Position Y

- Base Upon：
  - Original：原点
  - Center of Mass：重心，也就是算出来差不多在盆骨位置的那个body Transfrom，Y轴选择重心所在的位置。
  - Feet：动画模型的脚，或者说是avatar系统下的脚，可以用，如果original效果不好的时候。

Root Transform Position XZ

- Base Upon：
  - Original：原点
  - Center of Mass：XZ选择重心所在的位置

![image-20220925220247864](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220925220247864.png)

```note
！！！！！！废弃的内容！！！！！！

在讲这些选项之前，我想先说说这两个概念：

Body Transform：

- 这玩意是角色身体的重心，和方向一起存储在AnimationClip中。

Root Transform：

- 这玩意是Body Transform在Y平面上的投影，并在运行时实时计算，同时将值应用在游戏对象上使其移动。也就是说，动画使得游戏对象会动的始作俑者就是这个逼。

我们可以通过Root Motion的三个选项来控制Body Transform 对 Root Transform投影的结果。

简单来说：**如果不设置Root Transform中的Bake Into Pose，动画中的曲线会影响物体的Root Transform，而勾选了Bake Into Pose以后，动画的曲线就不会影响物体的Root Transform。比如勾选了Root Transform Position (Y)的Bake Into Pose，那动画就不会影响物体的Y轴位置了。**

RootMotion有三种：

Root Transform Rotation、Root Transform Position(Y)、Root Transform Position(XZ)

当然，都看得懂，无非就是旋转，Y轴移动（垂直）、XZ轴移动（水平）

Root Transform Rotation：

- Bake Into Pose：选中后，游戏对象的方向会基于Body Transform（Pose），意味着AnimationClip不会影响物体旋转。这个选项建议只有动画开始和结束旋转相似，比如一般的直向的走、跑动画。右边的绿灯如果判断loop match，意思就是这个动画开始和结束的旋转匹配得上，建议你勾选。
- Base Upon：根运动旋转基于什么：
  - Original：使用动画AnimationClip本身的旋转
  - Body Orientation：根据身体的方向。即旋转基于身体的正前方
- Offset：偏移量，没什么好说的

Root Transform Position(Y)：

- Bake Into Pose：
- Base Upon：根运动旋转基于什么：
  - Original
  - Center of Mass
  - Feet
- Offset：偏移量，没什么好说的

Root Transform Position(XZ)

- Bake Into Pose：
- Base Upon：根运动旋转基于什么：
  - Original
  - Center of Mass
- Offset：偏移量，没什么好说的


```



## 9. Avatar 替身系统

Avatar系统用于解决Unity上的人形动画复用。

假设我们想要在A、B模型上复用动画的话：

1. 通过Avatar替身系统将模型A的骨骼和一套标准的unity肌肉结构对应起来，这种对应关系保存在A的avatar文件中，同时，骨骼信息也被保存在了Avatar文件中。有了Avatar文件之后，模型A的骨骼就不需要了。<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220927220904052.png" alt="image-20220927220904052" style="zoom:50%;" />
2. 同样的，我们把B的骨骼和同一套标准的unity肌肉结构对应起来，将对应关系和B的骨骼信息保存在Avatar文件中。<img src="C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220927221238910.png" alt="image-20220927221238910" style="zoom:50%;" />
3. 通过A的Avatar文件，把这个模型的AnimationClip从描述A的骨骼变化情况的文本，转化为描述unity肌肉拉伸程度的文本。![选用Generic No Avatar](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220927221751191.png)![选用Humannoid 有Avatar](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220927221650372.png)
4. 当我们要在B模型上使用这个动画的时候，就要通过B的Avatar文件将这个AnimationClip中对肌肉拉伸的描述，**翻译**成对B模型骨骼变化的描述，这样，这个动画就可以在B模型上播放了。
5. 这个例子好吊：日美两方都没有对应的翻译，但是他们都有荷兰语对本语的翻译，那么翻译流程就变成了，双方的荷兰语翻译（Avatar）将日语（A模型）和英语（B模型），都先翻译成荷兰语（unity肌肉描述），再由荷兰语翻译成日语和英语。

我们来看看Avatar窗口：

![image-20220927223656112](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220927223656112.png)

骨骼映射界面，avatar存储的骨骼信息，这些绿点就表示我们模型的骨骼，其中实线的是这个模型必须有的骨骼，虚线的是可选项，可以没有。在这个界面你可以对这个骨骼信息做一些调整。

![image-20220927224409741](C:\Users\Van\AppData\Roaming\Typora\typora-user-images\image-20220927224409741.png)

最上面的是对肌肉群的预览，下面的是设置，如果你的模型动作上有穿模的情况发生，可以尝试调整一下肌肉的极限位置。
