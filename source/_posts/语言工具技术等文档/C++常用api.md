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



arrary

```
int number[5];//= {1,2,3,4,5};    //数组初始化
int *number=new int[5];        //数组初始化

swap(number[i],number[j]);
```

include<algorithm>

```
sort(array,array+length);     //0-(length-1)数组排序
bool cmp(const int &a,const int &b) return a>b;   //没有官方封装的快
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

v.size();    v[0].size();
v.empty();  
v.push_back(x);
v.pop_back(x);  
v.clear(); //清空元素，但不回收空间

swap(v[i],v[j]);
sort(result.begin(),result.end());  //排列
```

#include<stack>
```
stack<int> s;
s.empty();
s.push(value);
s.top();
s.pop();
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
map<int,int> m;   //初始化
m[key]=val;     //赋值
m.erase(key);   m.erase(iter);   //去掉-key或指针

for(auto iter=m.begin();iter!=m.end();iter++){
    cout << iter->first << " : " << iter->second << endl; //注意是"->"不是".""
}
//map中的元素按照key顺序排列，在对顺序有要求的问题中使用map，查找速度上慢于哈希实现的unordered_map
```

#include<priority_queue>
```
priority_queue<int> Q;  //大顶堆
priority_queue<int,vector<int>,less<int>> Q;  //大顶堆
priority_queue<int,vector<int>,greater<int>> Q;//小顶堆

priority_queue<pair<int,int>> pQ;    //pair
pQ.push(make_pair(value1,value2));
Q.top();
Q.empty();
Q.size();
Q.pop();
```

#include<set>
```
unordered_set<int> hash;
unordered_set <int> hash(_vector.begin(), _vector.end());
hash.find(val)!=hash.end();
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
s.pop();
```


#include<multiset>  //自动排序的set
```
multiset<int, greater<int> > leastNums;  //从大到小排序

leastNums.end();  //最后一个元素之后的迭代器，不是最后一个元素
leastNums.insert(val);
leastNums.erase(least.begin());
result=vector<int> (leastNumbers.begin(),leastNumbers.end());
```


