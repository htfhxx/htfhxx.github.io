---
title: C++常用API
date: 2019-09-03
author: 长腿咚咚咚
toc: true
mathjax: true
categories: 语言工具技术等文档
tags:
	- C++常用API
---

Trick
```
第一个判浮点数相等要用fabs(x -y) <= eps，不能用x==y，const double eps = 1e-6;
判断奇偶数，用x%2!=0 而不是x%2==1，因为c++里余数可以是奇数
```

常用
```
rand()%100; 生成0-99的随机数
INX_MAX
INT_MIN

// 遍历
for(int i=0;i<n;i++) a[i]
for(char c: a) c
```

arrary
```
int number[5];//= {1,2,3,4,5};    //数组初始化
int *number=new int[5];        //数组初始化
int[][] dp = new int[5][5];   //数组初始化

swap(number[i],number[j]);
```

#include<algorithm>
```
sort(array,array+length);     //0-(length-1)数组排序
static bool cmp(const int &a,const int &b) return a>b;   //没有官方封装的快 静态成员函数与普通成员函数的根本区别在于：普通成员函数有 this 指针，可以访问类中的任意成员；而静态成员函数没有 this 指针，只能访问静态成员（包括静态成员变量和静态成员函数）。
sort(array,array+length,cmp);  //0-(length-1)数组排序-降序
```

#include<math>
```
fabs(number); //绝对值,abs只针对整数
sqt(number); //开方
```

#include<string>
```
str.size();  str.length();
memset(a,0,sizeof(a));
string subS=str.substr(begin_index,length);  //获取子串
string subS=str.substr(begin_index);
position = str.find("jk");    
int b = to_string(a);
```

#include<vector>
```
//初始化vector
vector<int> v;  
vector<int> v(n+1,-1); //n+1个数，初始化-1
vector<int>({1,2,3,4});//直接加入元素
vector<int>(); //返回一个空的容器
vector<vector<int>> v;
vector<vector<int>> v(n,vector<int>(n,0));
vector<string> board(n,string(n,'.'));   

v.size();    v[0].size();
v.empty();  判断是否空
v.push_back(x);
v.pop_back(x);  
v.clear(); //清空元素，但不回收空间
v.erase(idx); // 清空某个元素
v.erase(idx, last) // 清空[idx, last) 的区间

swap(v[i],v[j]);
sort(result.begin(),result.end());  //排列
count(nums.begin(), nums.end()); //计数
reverse(nums.begin(),nums.end());   //翻转
reverse(nums.begin(),nums.begin()+k);   //翻转前k个
```

#include<stack>
```
stack<int> s;
s.empty();
s.push(value);
s.top();
s.pop();
s.size();
```

#include<queue>
```
queue<int> q;
q.empty();
q.push(value);
q.front();
s.pop();
```

#include<map>
```
map<int,int> m;   //初始化 int类型的value初始化默认为0
m[key]=val;     //赋值
m.erase(key);   m.erase(iter);   //去掉-key或指针
m.find(key)!=m.end();

for(auto iter=m.begin();iter!=m.end();iter++){
    cout << iter->first << " : " << iter->second << endl; //注意是"->"不是".""
}
for(auto &m : M){
    cout<<m.first<<endl;
}
//map中的元素按照key顺序排列，在对顺序有要求的问题中使用map，查找速度上慢于哈希实现的unordered_map
```

#include<priority_queue>
```
priority_queue<int> Q;  //大顶堆
# priority_queue<int,vector<int>,less<int>> Q;  //大顶堆
priority_queue<int,vector<int>,greater<int>> // 小顶堆 Q;//小顶堆

priority_queue<pair<int,int>> pQ;    //pair
pQ.push(make_pair(value1,value2));
Q.top();
Q.empty();
Q.size();
Q.pop();
```


===========================================================================

#include<set>
```
unordered_set<int> hash;
unordered_set <int> hash(_vector.begin(), _vector.end());
hash.find(val)!=hash.end();

hash.insert(val);

*(hash.begin())   // 首个值
```

#include<list>
```
list<int> l;
l.begin();  l.end();
l.size();
list<int>::iterator current=l.begin();
l.erase(current);
l.push_back(val);
```

#include<deque>  //双向队列
```
deque<int> q;
q.push_back(value);
q.empty();
q.front();
q.back();
q.pop_back();
q.pop_front()
```


#include<multiset>  //自动排序的set
```
multiset<int, greater<int> > leastNums;  //从大到小排序

leastNums.end();  //最后一个元素之后的迭代器，不是最后一个元素
leastNums.insert(val);
leastNums.erase(least.begin());
result=vector<int> (leastNumbers.begin(),leastNumbers.end());
```