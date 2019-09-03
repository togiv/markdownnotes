---
title: "拓扑排序c++"
date: 2019-08-31T10:34:43+08:00
draft: true
tags: ["算法，自用"]
---
# 拓扑排序c++

对一个`有向无环图`(Directed Acyclic Graph简称DAG)进行拓扑排序，是将G中所有顶点排成一个线性序列，使得图中任意一对顶点u和v，若边(u,v)∈E(G)，则u在线性序列中出现在v之前。 
拓扑排序对应施工的流程图具有特别重要的作用，它可以决定哪些子工程必须要先执行，哪些子工程要在某些工程执行后才可以执行。为了形象地反映出整个工程中各个子工程(活动)之间的先后关系，可用一个有向图来表示，图中的顶点代表活动(子工程)，图中的有向边代表活动的先后关系，即有向边的起点的活动是终点活动的前序活动，只有当起点活动完成之后，其终点活动才能进行。通常，我们把这种顶点表示活动、边表示活动间先后关系的有向图称做顶点活动网(Activity On Vertex network)，简称`AOV网`。 

一个AOV网应该是一个有向无环图，即不应该带有回路，因为若带有回路，则回路上的所有活动都无法进行（对于数据流来说就是死循环）。在AOV网中，若不存在回路，则所有活动可排列成一个线性序列，使得每个活动的所有前驱活动都排在该活动的前面，我们把此序列叫做拓扑序列(Topological order)，由AOV网构造拓扑序列的过程叫做拓扑排序(Topological sort)。AOV网的拓扑序列不是唯一的，满足上述定义的任一线性序列都称作它的拓扑序列。
- - -
拓扑排序的过程：
> + 先找到所有入度为0的节点，入`栈 `，`入栈表示即将被输出`，弹出栈顶元素，并打印，这样就保证了，入度为0的节点先被打印
>+ 接着将与该节点直接后继所有节点的`入度都减1`，若减1之后入度为0，则入栈，接着重复该过程，直到栈为空

> *从上述过程可以看出，若有分支，拓扑排序过程会将该分支打印完毕才会打印下一分支，若无入度为0的节点则不存在拓扑排序*

完整代码：
- - -
```c++
//topological_sort.h代码
#pragma once
//#pragma once是一个比较常用的C/C++杂注，
//只要在头文件的最开始加入这条杂注，
//就能够保证头文件只被编译一次。

/*
拓扑排序必须是对有向图的操作
算法实现：
（1）Kahn算法
（2）DFS算法
采用邻接表存储图
*/
#include<iostream>
#include<string>
#include<stack>
using namespace std;
//表结点
struct ArcNode {
    ArcNode * next; //下一个关联的边
    int adjvex;   //保存弧尾顶点在顶点表中的下标
};
struct Vnode {
    string data; //顶点名称
    ArcNode * firstarc; //第一个依附在该顶点边
};

class Graph_DG {
private:
    int vexnum; //图的顶点数
    int edge;   //图的边数
    int * indegree; //每条边的入度情况
    Vnode * arc; //邻接表
public:
    Graph_DG(int, int);
    ~Graph_DG();
    //检查输入边的顶点是否合法
    bool check_edge_value(int,int);
    //创建一个图
    void createGraph();
    //打印邻接表
    void print();
    //进行拓扑排序,Kahn算法
    bool topological_sort();
    //进行拓扑排序，DFS算法
    bool topological_sort_by_dfs();
    void dfs(int n,bool * & visit, stack<string> & result);
};
//topological_sort.cpp代码
#include"topological_sort.h"
#define max 100  
  
string to_String(int n)  //重写to_string方法，以免旧的编译器不支持
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

Graph_DG::Graph_DG(int vexnum, int edge) {// 初始化图
    this->vexnum = vexnum; //顶点数
    this->edge = edge;  //边数
    this->arc = new Vnode[this->vexnum];//邻接表
    this->indegree = new int[this->vexnum];//入度
    for (int i = 0; i < this->vexnum; i++) {
        this->indegree[i] = 0; //所有入度都初始化为0
        this->arc[i].firstarc = NULL;//第一个邻接边初始化为空
        this->arc[i].data = "v" + to_String(i + 1);//节点命名
    }
}
//释放内存空间
Graph_DG::~Graph_DG() { // 析构函数
    ArcNode * p, *q;
    for (int i = 0; i < this->vexnum; i++) {
        if (this->arc[i].firstarc) {
            p = this->arc[i].firstarc;
            while (p) {
                q = p->next;
                delete p;
                p = q;
            }
        }
    }
    delete [] this->arc;
    delete [] this->indegree;
}
//判断我们每次输入的的边的信息是否合法
//顶点从1开始编号
bool Graph_DG::check_edge_value(int start, int end) {
    if (start<1 || end<1 || start>vexnum || end>vexnum) {
        return false;
    }
    return true;
}
void Graph_DG::createGraph() { //创建图，邻接表
    int count = 0;   //count用来计数
    int start, end;
    cout << "输入每条起点和终点的顶点编号（从1开始编号）" << endl;
    while (count != this->edge) {
        cin >> start;
        cin >> end;
        //检查边是否合法
        while (!this->check_edge_value(start, end)) {
            cout << "输入的顶点不合法，请重新输入" << endl;
            cin >> start;
            cin >> end;
        }
        //声明一个新的表结点
        ArcNode * temp = new ArcNode;
        temp->adjvex = end - 1; //弧尾顶点下标
        temp->next = NULL;  // 下一关联的边为空
        //如果当前顶点的还没有边依附时，
        if (this->arc[start - 1].firstarc == NULL) {
            this->arc[start - 1].firstarc = temp;
        } //挂上第一条依附的边
        //如果当前顶点有边依附，
        else {
            ArcNode * now = this->arc[start - 1].firstarc;
            while(now->next) {
                now = now->next;
            }//找到该链表的最后一个结点
            now->next = temp; //挂到邻接表的最后

        }
        ++count;
    }
}
void Graph_DG::print() {
    int count = 0;
    cout << "图的邻接矩阵为：" << endl;
    //遍历链表，输出链表的内容
    while (count != this->vexnum) {
        //输出链表的结点
        cout << this->arc[count].data<<" ";
        ArcNode * temp = this->arc[count].firstarc;
        while (temp) {
            cout<<"<"<< this->arc[count].data<<","<< this->arc[temp->adjvex].data<<"> ";
            temp = temp->next;
        }
        cout << "^" << endl;
        ++count;
    }
}

bool Graph_DG::topological_sort() {
    cout << "图的拓扑序列为：" << endl;
    //栈s用于保存栈为空的顶点下标
    stack<int> s;
    int i;
    ArcNode * temp;
    //计算每个顶点的入度，保存在indgree数组中
    for (i = 0; i != this->vexnum; i++) {
        temp = this->arc[i].firstarc;
        while (temp) {
            ++this->indegree[temp->adjvex]; //该节点的所有直接后继节点的入度都加1，用邻接表来实现方便快捷
            temp = temp->next;
        }

    }

    //把所有入度为0的顶点入栈
    for (i = 0; i != this->vexnum; i++) {
        if (!indegree[i]) {
            s.push(i); 
        }
    }
    //count用于计算输出的顶点个数
    int count=0;
    while (!s.empty()) {//如果栈为空，则结束循环
        i = s.top();
        s.pop();//保存栈顶元素，并且栈顶元素出栈
        cout << this->arc[i].data<<" ";//输出拓扑序列
        temp = this->arc[i].firstarc;
        while (temp) {
            if (!(--this->indegree[temp->adjvex])) {//  该节点的所有直接后继的入度都减一 如果入度减少到为0，则入栈，同样是用邻接表来实现
                s.push(temp->adjvex);
            }
            temp = temp->next;
        }
        ++count;
    }
    if (count == this->vexnum) {
        cout << endl;
        return true;
    } 
    cout << "此图有环，无拓扑序列" << endl;
    return false;//说明这个图有环
}
//基于DFS的拓扑排序，仅供参考
//bool Graph_DG::topological_sort_by_dfs() {
//    stack<string> result;
//    int i;
//    bool * visit = new bool[this->vexnum];
//    //初始化我们的visit数组
//    memset(visit, 0, this->vexnum);
//    cout << "基于DFS的拓扑排序为：" << endl;
//    //开始执行DFS算法
//    for (i = 0; i < this->vexnum; i++) {
//        if (!visit[i]) {
//            dfs(i, visit, result);
//        }
//    }
//    //输出拓扑序列，因为我们每次都是找到了出度为0的顶点加入栈中，
//    //所以输出时其实就要逆序输出，这样就是每次都是输出入度为0的顶点
//    for (i = 0; i < this->vexnum; i++) {
//        cout << result.top() << " ";
//        result.pop();
//    }
//    cout << endl;
//    return true;
//}
//void Graph_DG::dfs(int n, bool * & visit, stack<string> & result) {
//
//        visit[n] = true;
//        ArcNode * temp = this->arc[n].firstarc;
//        while (temp) {
//            if (!visit[temp->adjvex]) {
//                dfs(temp->adjvex, visit,result);
//            }
//            temp = temp->next;
//        }
//        //由于加入顶点到集合中的时机是在dfs方法即将退出之时，
//        //而dfs方法本身是个递归方法，
//        //仅仅要当前顶点还存在边指向其他不论什么顶点，
//        //它就会递归调用dfs方法，而不会退出。
//        //因此，退出dfs方法，意味着当前顶点没有指向其他顶点的边了
//        //，即当前顶点是一条路径上的最后一个顶点。
//        //换句话说其实就是此时该顶点出度为0了
//        result.push(this->arc[n].data);
//
//}
//main.cpp文件
#include <iostream>

/* run this program using the console pauser or add your own getch, system("pause") or input loop */

#include"topological_sort.h"
bool check(int Vexnum, int edge) {
    if (Vexnum <= 0 || edge <= 0 || ((Vexnum*(Vexnum - 1)) / 2) < edge)
        return false;
    return true;
}//检查输入的图是否合法，数学结论
int main() {
    int vexnum; int edge;


    cout << "输入图的顶点个数和边的条数：" << endl;
    cin >> vexnum >> edge;
    while (!check(vexnum, edge)) {
        cout << "输入的数值不合法，请重新输入" << endl;
        cin >> vexnum >> edge;
    }
    Graph_DG graph(vexnum, edge);
    graph.createGraph();
    graph.print();
    graph.topological_sort();

    system("pause");
    return 0;

}
```
原文链接：[https://blog.csdn.net/qq_35644234/article/details/60578189](https://blog.csdn.net/qq_35644234/article/details/60578189)


