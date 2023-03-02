# STL Stack And Queue



## 栈 Stack

### 定义

```c++
stack<T> stack;
stack<int> stack;
stack<int> stack1(stack) // 拷贝
```



### 常用操作

```c++
// 向栈顶添加元素
void push(T element);

// 从栈顶去除一个元素
void pop();

// 返回栈顶元素
T& top();

// 判断栈是否为空
bool empty();

// 返回栈的大小
int size();
```





## 队列 Queue

### 定义

```c++
queue<T> queue;
queue<int> queue;
queue<int> queue1(queue); // 拷贝
```



### 常用操作

```c++
// 往队尾添加元素
void push(T element);

// 从队头移除一个元素
void pop();

// 返回队尾元素
T& back();

// 返回队头元素
T& front();

// 判断队列是否为空
bool empty();

// 返回队列的大小
int size();
```

