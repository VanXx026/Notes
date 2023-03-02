# Unity 对话系统

其实整个对话系统或者说任务系统没有说特别难，就是一些非常简单的逻辑。其中任务系统可以用事件event来写，这样可能效果会更好一点。



## 小知识点

### 1. Image组件下有一个勾选项：Preserve Aspect（锁定比例）：这个选项勾选之后，使用的图片大小比例是多大就多大，不会拉伸变形。



### 2. Font 中的 BestFit勾选项

就是让文字之间自动调整，不要超出整个框的范围，如果文字量多，那么就会把文字大小缩小；如果文字量少，那么就会把文字大小变大。不过似乎在TextMeshPro中已经没有这个选项了



### 3. TextArea属性Attribute

这个通常给string使用，使用之后在inspector窗口中可以看到变大的string窗口，方便编辑，并且也支持回车换行，挺方便的。

```c#
[TextArea(1, 3)] // arg0: minLines:最小行数; arg1: maxLines:最大行数
public string talk;
```



### 4. 文字滚动效果

其实文字滚动的效果非常的简单，就是开一个协程，然后foreach循环取出每一个字出来，直接赋值在Text文本上，然后yield return new WaitForSeconds(ScrollTime)就可以了，当然如果想要提前结束滚动显示文本也很简单，停止协程，然后将文本全部赋值到Text上就可以了。
