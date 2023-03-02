# 剑指 Offer 05. 替换空格

[剑指 Offer 05. 替换空格 - 力扣（LeetCode）](https://leetcode.cn/problems/ti-huan-kong-ge-lcof/)



## 1. resize() + 双指针

其实很多**数组填充类**的问题，都可以先**预先给数组扩容带填充后的大小**，然后在**从后向前进行操作**。

遇到需要往数组扩充或者插入元素的话，都可以样做，这样做的好处是：不需要申请新的空间，也不需要对元素做麻烦的移动操作。
另外有个问题就是：从后往前做，会把前面的值覆盖掉吗，也就是说，right指针会比left指针快吗？

实际上不需要担心这样的问题：因为**最终两个指针会在从右往左数最后一个空格处重合。**

首先，我们遍历一次数组，将空格出现的次数记录下来，然后使用resize申请大一点的空间。空格出现了cnt次，我们就新申请2 * cnt个额外空间在尾部。那么我们在原来数组的末尾放一个指针，在申请额外空间后的数组的末尾放一个指针，那么两个指针之间的距离为 2 * cnt，每次遇到空格，两者的距离就会减少2，即cnt--，那么到最后一个空格，替换完字符后，cnt = 0，即两者之间的距离变为0，这时两个指针就会重合，后面就无需再操作了，因为已经没有空格需要替换了，所以right绝对不会超过left将原来字符串的信息覆盖掉，而且这种情况会导致数组指针溢出。

![替换空格](https://tva1.sinaimg.cn/large/e6c9d24ely1go6qmevhgpg20du09m4qp.gif)



```c++
class Solution {
public:
    string replaceSpace(string s) {
        int origin_n = s.size(); // 初始的字符串长度
        int cnt = 0;
        for(char i : s) {
            if(i == ' ') {
                cnt++; // 得知空格的个数
            }
        }
        s.resize(origin_n + 2 * cnt); // 每存在一个空格，就申请2个额外空间在尾部
        int left = origin_n - 1; // 将左指针设在初始字符串末尾
        int right = s.size() - 1; // 将右指针设在resize后的字符串末尾
        while(left < right) {
            if(s[left] == ' ') {
                s[right--] = '0';
                s[right--] = '2';
                s[right--] = '%';
                left--; 
            } else {
                s[right--] = s[left--]; 
            }
        }
        return s;
    }
};
```

