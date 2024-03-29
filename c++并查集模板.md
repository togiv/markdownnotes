---
title: "C++并查集模板"
date: 2019-08-31T10:32:33+08:00
draft: true
tags: ["算法，自用"]
---
## 此代码只保存并查集的模板
详情请参见
<a href="https://blog.csdn.net/zjy_code/article/details/80634149">https://blog.csdn.net/zjy_code/article/details/80634149</a>

并查集是一种树形的数据结构，用于一些不相交的集合的查询和合并问题，使用起来非常高效和方便，例如有不同的集合，查询任意两个数据是否属于同一集合，如果不是属于同一集合就合并两个集合为一个集合。并查集主要运用了父节点来达到高效的目的。

+ 初始时每个元素的父节点都指向自己
+ 查询时查询两个元素的父节点是不是同一个，如果是同一个说明两个元素属于同一个集合，如果父节点不是同一个说明属于不同的集合，很显然初始化后每个节点都属于不同的集合
+ 如果两个节点属于不同的集合，那就将两个元素合并到同一个集合中，将其中一个元素的父节点直接指向另一个元素的父节点，具体方法及优化见下面代码
### 初始化
```c++
/*初始化*/
void init() {
	for (int i=0;i<size,i++){
		father[i]=i;  //每个元素的父节点都指向自己
	}
}
```
### 查询
> 中间函数得到父节点，用于后面的判断及合并，得到父节点有两种方式，一种是路径压缩一种是非路径压缩，所谓非路径压缩就是一直往上查父节点知道总的父节点（即父节点指向自己），非路径压缩会随着树结构越来越大，子节点越来越多，查询的会越来越慢。

> 路径压缩方式就是将路径压缩，避免树结构越来越深，查询越来越慢，具体做法是每次查询时若该节点不是父节点就将该节点的指向改为该节点的最终父节点（即将查询到的节点都打平，将原来纵向的改为横向的）
```c++
/*得到父节点*/   //非路径压缩
int get(){
	if(father[x]==x)
		return x;
}   return get(father[x]);

/*得到父节点*/   //路径压缩
int get(){
	if(father[x]==x)  // x 结点就是根结点
		return x;
	return father[x]=get(father[x])   // 返回父结点的根结点，并另当前结点父结点直接为根结点
}
```
### 合并
     
```c++
/*合并两个集合*/
void merge(int x, int y) {
    x = get(x);  //得到x的最终父节点
    y = get(y);  //得到y的最终父节点
    if (x != y) { // 不在同一个集合
        father[y] = x;  //合并
    }
```
## 有时候节点可能含有权值，合并的时候需要将权值一并合并
*以排队问题为例，不具有一般性*
### 初始化
```c++
/*带权并查集*/
void init() {
    for(int i = 1; i <= n; i++)  {
        father[i] = i, dist[i] = 0, size[i] = 1;
        //size 保存集合的大小，dist保存权值
    }
```
### 查找
```c++
/*查找*/  // 查的时候执行压缩，查一次压缩一次 

int get(int x) {
    if(father[x] == x) {
        return x;        
    }
    int y = father[x];
    father[x] = get(y);
    dist[x] += dist[y];  // x 到根结点的距离等于 x 到之前父亲结点距离加上之前父亲结点到根结点的距离
    return father[x];
```
### 合并
```c++
void merge(int a, int b) {
    a = get(a);
    b = get(b);
    if(a != b) {  // 判断两个元素是否属于同一集合
        father[a] = b; //a元素合并到b元素的后边，并改变父节点的指向
        dist[a] = size[b]; //被合并元素a权值都加上b所在集合的大小
        size[b] += size[a];//b集合的大小加上被合并集合a的大小
    }
```
## 并查集具体使用的可以根据需要加上自己需要的属性


