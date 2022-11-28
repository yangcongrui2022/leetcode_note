## KMP算法
### 应用场景
- KMP算法一般用于字符串匹配问题
- 例如：给出两个字串S，P需要判断P串是否为S串的子串
### 前缀表
- 前缀：包含第一个字符不包含最后一个字符
- 后缀：包含最后一个字符不包含最后一个字符  
  例如：**aaba**  
  前缀分别为：a, aa, aab  
  后缀分别为：a, ba, aba  
- 最长相等前后缀：记录前缀和后缀相等的长度，在这个例子中最长相等前后缀为a，长度为1
- 在KMP算法当中，用一个next数组记录每个字符的最长相等前后缀
  例如：**aabaa**  
  前缀分别为：a, aa, aab, aaba  
  后缀分别为：a, aa, baa, abaa  
  next数组为：a:0, aa:1, aab:0, aaba:1, aabaa:2    
  next = [0, 1, 0, 1, 2]  
### 前缀表在KMP算法中的作用
- 暴力解法中，我们需要两重循环遍历P串和S串，直到找到匹配的字串，时间复杂度为O(n*m)，n，m分别表示P串和S串的长度
- KMP算法的核心思想就是用前缀表记录已经匹配过的文本内容，使得当发生匹配冲突的时候，可以不需要重新遍历，而是通过前缀表回退到之前匹配成功过的位置继续匹配，next数组就是前缀表
- 具体原理参考<https://www.bilibili.com/video/BV1PD4y1o7nd/?spm_id_from=333.337.search-card.all.click><https://www.bilibili.com/video/BV1M5411j7Xx/?spm_id_from=333.788&vd_source=57d32b81e2ba68496de56c631fb1aa07>
### next数组的实现（前缀表实现）
- 构造next数组分为四步：  
1. 初始化  
  定义两个指针i，j  
  j指向前缀末尾位置，i指向后缀末尾位置  
  next数组初始化为0，j从0开始，i从1开始
2. 处理前后缀不相同的情况  
  当前后缀不相等并且j>0时（后续要回退到j-1的状态所以要保证j>0）  
  j回退到j-1的状态
3. 处理前后缀相同的情况  
  前后缀相同时，j向后移动一位
4. 更新next数组  
  将next数组更新为j
- 代码模板
```
int j = 0;
next[0] = 0;
for (int i = 1; i < m; i ++){
    while (j > 0 && s[i] != s[j]) j = next[j - 1];
    if (s[i] == s[j]) j ++;
    next[i] = j;
}
```
#### leetcode.28
- 链接<https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/>  
- leetcode解题代码
```
class Solution {
public:
    int strStr(string haystack, string needle) {
        int n = haystack.length(), m = needle.length();
        vector<int> next(m);
        int j = 0;// 初始化j
        next[0] = 0;// 初始化next数组
        for (int i = 1; i < m; i ++){// 初始化i
            while (j > 0 && needle[i] != needle[j]) j = next[j - 1];// 前后缀不相同时
            if (needle[i] == needle[j]) j ++;// 前后缀相同时
            next[i] = j;// 更新next数组
        }

        j = 0;
        for (int i = 0; i < haystack.size(); i++) {
            while(j > 0 && haystack[i] != needle[j]) j = next[j - 1];
            if (haystack[i] == needle[j]) j++;
            if (j == needle.size() ) {
                return (i - needle.size() + 1);
            }
        }
        return -1;
    }
};
```
#### leetcode.459
- 链接<https://leetcode.cn/problems/repeated-substring-pattern/>  
- leetcode解题代码
```
class Solution {
public:
    bool repeatedSubstringPattern(string s) {
        int n = s.size();
        vector<int> next(n);
        int j = 0;
        next[0] = 0;
        for (int i = 1; i < n; i ++){
            while (j > 0 && s[i] != s[j]) j = next[j - 1];
            if (s[i] == s[j]) j ++;
            next[i] = j;
        }
        return next[n - 1] != 0 && n % (n - next[n - 1]) == 0;
    }
};
```
## 字符串前缀哈希
### 应用场景
- 求两个字符串的子串是否相同
### 应用方法
- 字符串的映射    
  例如：有一个'abcdefgycr'的字符串，将其映射成某个哈希值并用数组h存下来  
  h[n]表示字符串第n位的哈希值  
  h[0]=0，h[1]='a'的哈希值，h[2]='ab'的哈希值...
- 哈希值的定义  
  例如：字符串'abcd'的哈希值是多少呢？  
  我们把'abcd'看成p进制的数，那么'abcd'则可以表示为  
  **a*p^3+b\*p^2+c\*p^1+d\*p^0**  
  但是这样映射的值可能过大，所以我们再将其取模q  
  这样就可以将字符串映射到0~q-1之间  
  一般情况下p=131，q=2^64，可以假定不会发生哈希冲突>.<（感兴趣的可以查一下）
- 定义一个区间[L, R]的哈希值  
  通过上述方式我们已经知道了h[L-1]和h[R]  
  通过h[R] - h[L-1]*p^(R-L+1)
### 实现方法
```
typedef unsigned long long ULL;// 可以省略取模的步骤了

const int P = 131;

ULL h[N], p[N];

// 初始化
p[0] = 1;// p^0 = 1
h[0] = 0;
// 前缀和定义前缀字符串哈希
for (int i = 1; i <= n; i ++){
    h[i] = h[i - 1] * P + str[i];
    p[i] = p[i - 1] * P;
}
// 计算字串[L, R]的哈希值
ULL get(int l, int r){
    return h[r] - h[l - 1] * p[r - l + 1];
}
```
#### leetcode.796
- 链接<https://leetcode.cn/problems/rotate-string/>  
- 解题思路：求两个字符串的哈希值，比较对应段是否相等  
  为了不用求解两次字符串哈希，可以将两个字符串拼接
- leetcode解题代码
```
typedef unsigned long long ULL;

const int N = 210, P = 131;
ULL h[N], p[N];

class Solution {
public:
    ULL get(int l, int r) {
        return h[r] - h[l - 1] * p[r - l + 1];
    }

    bool rotateString(string A, string B) {
        if (A.size() != B.size()) return false;
        string s = ' ' + A + B;
        int n = s.size() - 1;
        p[0] = 1;
        for (int i = 1; i <= n; i ++ ) {
            p[i] = p[i - 1] * P;
            h[i] = h[i - 1] * P + s[i];
        }

        for (int k = 1; k < A.size(); k ++ )
            if (get(1, k) == get(n - k + 1, n) && get(k + 1, A.size()) == get(A.size() + 1, n - k))
                return true;
        return false;
    }
};
```
解题参考：<https://www.acwing.com/>  
刷题顺序参考：<https://www.programmercarl.com/>
