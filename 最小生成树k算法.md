---
title: "最小生成树k算法"
date: 2019-08-31T10:34:58+08:00
draft: true
tags: ["算法，自用"]
---
*最小生成树是应用于无向图的*

*在一个无向连通图中，如果存在一个连通子图包含原图中所有的结点和部分边，且这个子图不存在回路，那么我们称这个子图为原图的一棵生成树。在带权图中，所有的生成树中边权的和最小的那棵（或几棵）被称为最小生成树。*

> 最小生成树Kruskal算法的算法原理，它按照如下步骤求解最小生成树：
（1）初始时所有结点属于孤立的集合。
（2）按照边权递增顺序遍历所有的边，若遍历到的边两个顶点仍分属不同的集合（该边即为联通这两个集合的边中权值最小的那条）则确定该边为最小生成树上的一条边，并将这两个顶点分属的集合合并。
（3）遍历完所有边后，原图上所有结点属于同一个集合则被选取的边和原图中所有结点构成最小生成树；否则原图不连通，最小生成树不存在。

由上述步骤可以看出，k算法应用并查集可以大大简化流程提高效率。

完整代码：
```c++
#include<iostream>
#include<cstdio>
#include<algorithm>
#include<string>
#define max 100
using namespace std;

int n,m,tot=0,k=0;//n端点总数，m边数，tot记录最终答案，k已经连接了多少边 
int father[200010];//记录集体老大 
struct node
{
	int from,to,dis;//结构体储存边 
}edge[200010];
	
bool cmp(const node &a,const node &b)//sort排序 ，定义比较器，按权值排序
{
	return a.dis<b.dis;
}

/*初始化*/
void init(int size) {
	for (int i=0;i<size;i++){
		father[i]=i;
	}
}

/*得到父节点*/   //路径压缩
int get(int x){
	if(father[x]==x)  // x 结点就是根结点
		return x;
	return father[x]=get(father[x]);   // 返回父结点的根结点，并另当前结点父结点直接为根结点
}

/*合并两个集合*/     
void unionn(int x, int y) {
    x = get(x);
    y = get(y);
    if (x != y) { // 不在同一个集合
        father[y] = x;
    }
}
string to_String(int n)  //重写to_string 方法,旧的编译器不支持to_string
{  
    int m=n;  
    int i=0,j=0;  
    char s[max];  
    char ss[max];  
    while(m>0)  
    {  
        s[i++]= m%10 + '0';  
        m/=10;  
    }  
    s[i]='\0';  
  
    i=i-1;  
    while(i>=0)  
    {  
        ss[j++]=s[i--];  
    }  
    ss[j]='\0';  
  
    return ss;  
}

int main()
{	string str;
	cout<<"请输入点数和边数"<<endl;
	scanf("%d%d",&n,&m);//输入点数，边数 
	cout<<"输入每条边的信息，起始点，终止点，权重"<<endl; 
	for(int i=1;i<=m;i++)
	{
		cin>>edge[i].from>>edge[i].to>>edge[i].dis;//输入边的信息 
	}
	init(n); 
	sort(edge+1,edge+1+m,cmp);//按权值排序（kruskal的体现） 
	for(int i=1;i<=m;i++)//从小到大遍历 
	{
		if(k==n-1) break;//n个点需要n-1条边连接 
		if(get(edge[i].from)!=get(edge[i].to))//假如不在一个团体 
		{	
			str += "V" + to_String(edge[i].from) + "--->" + "V" + to_String(edge[i].to) + "   ";
			unionn(edge[i].from,edge[i].to);//加入 
			tot+=edge[i].dis;//记录边权 
			k++;//已连接边数+1 
		}
	}
	cout<<str<<endl;
	cout<<tot;
	return 0;
}
```
原文链接：[https://blog.csdn.net/qq_41754350/article/details/81460643](https://blog.csdn.net/qq_41754350/article/details/81460643)

