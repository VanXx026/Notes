# 剑指 Offer 58 - II. 左旋转字符串

#### [剑指 Offer 58 - II. 左旋转字符串](https://leetcode.cn/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/)



## 1. 局部反转+整体反转 （三次反转）

这题印象非常深刻，非常有意思，如果硬做的话，我记得这题还挺麻烦的，但是偏偏这题有个奇技淫巧：

题目中将字符串划分成了两个部分，那么我们先将这**两个字符串反转一次，然后再整个大字符串再反转一次**。

比如：lrloseumgh => umghlrlose

lrlose umgh => esolrl hgmu => umgh lrlose

```c++
class Solution {
public:
    // 左闭右闭的反转某段字符串
    void reverse(string& s, int left, int right) {
        while(left < right) {
            swap(s[left], s[right]);
            left++;
            right--;
        }
    }

    string reverseLeftWords(string s, int n) {
        reverse(s, 0, n-1); // 反转左边的字符串
        reverse(s, n, s.length()-1); // 反转右边的字符串
        reverse(s, 0, s.length()-1); // 反转整个字符串
        return s;
    }
};
```



