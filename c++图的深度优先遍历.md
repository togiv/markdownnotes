---
title: "C++图的深度优先遍历"
date: 2019-08-31T10:33:46+08:00
draft: true
tags: ["算法，自用"]
---
### 图的深度优先遍历(dfs)
图的深度优先遍历和宽度优先遍历类似
*首先创建一个`集合或者数组`标记节点有没有被遍历过*
+ 与图的宽度优先遍历不同的是，深度优先遍历从根节点出发，一条道走到黑才会回来走另一条路，这也称之为`回溯`，如果该节点的直接后继节点没被遍历过，就将此节点的`全部后继节点`加入`堆`,加入堆表示`即将被遍历`，堆结构是先进后出的结构，先加入的节点会后遍历，符合深度优先遍历的规则
+ 接着从堆中弹出一个节点，先打印该节点，再将该节点该节点的所有直接后继放入堆中,重复此过程直到后继节点为空或全部加入到堆中。
### 完整代码（邻接矩阵表示）
```c++
#include <iostream>
#include<vector>
#include<queue>
#include<stack>

using namespace std;

typedef int T;                    //顶点的值类型 int, char, double and so on
struct Node
{
    T value;                      //顶点值
    int numbers;                  //顶点编号
    bool operator==(const Node &other)
    {
        if((this->numbers==other.numbers)&&(this->value==other.value))
            return true;
        else
            return false;
    }
    Node(int a1=0,int a2=0)
    {
        numbers=a1;
        value=a2;
    }
};

struct Edge
{
    int num1;                     //起始顶点
    int num2;                     //终止顶点
    int weight;                   //边所对应的权值
};

class Graph
{
private:
    int vertex_nums;               //顶点数
    int edge_nums;                 //边数
    vector<Node> vertex;           //顶点表，强制设为编号从0开始
    vector<Edge> edge;             //边表
    vector<vector<int> > edge_maze;    //邻接矩阵，存储各个顶点连接关系
public:
    Graph(int n1=10,int n2=10)
    {
        vertex_nums=n1;
        edge_nums=n2;
        for(int i=0;i<vertex_nums;i++)
        {
            Node a;
            a.numbers=i;
            a.value=0;
            vertex.push_back(a);
        }

        for(int i=0;i<vertex_nums;i++)    //邻接矩阵初始化
        {
            vector<int> temp(vertex_nums,0);
            edge_maze.push_back(temp);
        }

        cout<<"please input the edges about the graph: vertex_number1 vertex_number2 weight"<<endl;
        int x,y,z;
        for(int i=0;i<edge_nums;i++)
        {
            cin>>x>>y>>z;
            Edge temp_edge;
            temp_edge.num1=x;
            temp_edge.num2=y;
            temp_edge.weight=z;
            edge.push_back(temp_edge);
            edge_maze[x][y]=1;
            edge_maze[y][x]=1;
        }
    }

    void print_edge_maze()
    {
        cout<<endl;
        cout<<"输出邻接矩阵"<<endl;
        cout<<"   ";
        for(int i=0;i<vertex_nums;i++)
            cout<<"v"<<i<<" ";
        cout<<endl;

        for(int i=0;i<vertex_nums;i++)
        {
            cout<<"v"<<i<<"  ";
            for(int j=0;j<vertex_nums;j++)
            {
                cout<<edge_maze[i][j]<<"  ";
            }
            cout<<endl;
        }
    }

    void print_edge_vec()
    {
        cout<<endl;
        cout<<"输出边表："<<endl;
        cout<<"起始顶点-->结束顶点  边权值"<<endl;
        for(int i=0;i<edge_nums;i++)
        {
            cout<<"   v"<<edge[i].num1<<"   -->"<<"   v"<<edge[i].num2<<"        "<<edge[i].weight<<endl;
        }
    }

    //基于DFS非递归遍历图
    void DFS_Graph_noniterator(int start_vertex_num=0)
    {
        vector<int> if_output(vertex_nums,0);     //指示顶点是否被遍历
        Node first(start_vertex_num);
        stack<Node> s;

        cout<<endl;
        cout<<"遍历顶点路径如下"<<endl;
        cout<<"v"<<first.numbers<<"---->";
        if_output[start_vertex_num]=1;            //标记已被访问
        s.push(first);

        while(!s.empty())
        {
            Node temp=s.top();
            int i;
            for(i=0;i<edge_nums;i++)
            {
                if((edge[i].num1==temp.numbers)&&(if_output[edge[i].num2]==0))
                {
                    cout<<"v"<<edge[i].num2<<"---->";
                    if_output[edge[i].num2]=1;            //标记已被访问
                    Node push_node(edge[i].num2);
                    s.push(push_node);
                    break;
                }
                else if((edge[i].num2==temp.numbers)&&(if_output[edge[i].num1]==0))
                {
                    cout<<"v"<<edge[i].num1<<"---->";
                    if_output[edge[i].num1]=1;            //标记已被访问
                    Node push_node(edge[i].num1);
                    s.push(push_node);
                    break;
                }
                else
                    continue;
            }
            if(i==edge_nums)
                s.pop();
        }
        cout<<"end"<<endl;
    }


    //基于BFS非递归遍历图
    void BFS_Graph_nonrecursive(int start_vertex_num=0)
    {
        vector<int> if_output(vertex_nums,0);     //指示顶点是否被遍历
        Node first(start_vertex_num);
        queue<Node> qu;
        qu.push(first);
        if_output[start_vertex_num]=1;            //入队即表示即将被遍历
        cout<<endl;
        cout<<"遍历顶点路径如下"<<endl;
        while(!qu.empty())
        {
            Node temp=qu.front();
            qu.pop();
            cout<<"v"<<temp.numbers<<"---->";

            //遍历所有边，找到与队列头结点连接的顶点，将所有与头顶点连接的顶点加入队列
            for(int i=0;i<edge_nums;i++)
            {
                if(edge[i].num1==temp.numbers)
                {
                    if(if_output[edge[i].num2]==0)           //未被遍历
                    {
                        Node temp_node(edge[i].num2);        //将连接点加入队列
                        qu.push(temp_node);
                        if_output[edge[i].num2]=1;           //入队即表示即将被遍历
                    }
                }
                else if(edge[i].num2==temp.numbers)
                {
                    if(if_output[edge[i].num1]==0)           //未被遍历
                    {
                        Node temp_node(edge[i].num1);        //将连接点加入队列
                        qu.push(temp_node);
                        if_output[edge[i].num1]=1;
                    }
                }
                else
                    continue;
            }
        }
        cout<<"end"<<endl;
        cout<<"遍历结束"<<endl;

    }
};

int main()
{
    Graph graph(8,10);
    graph.print_edge_vec();
    graph.print_edge_maze();
    //graph.BFS_Graph_nonrecursive();
    graph.DFS_Graph_noniterator();
    return 0;
}
```
原文地址：[https://blog.csdn.net/weixin_43011522/article/details/82495556](https://blog.csdn.net/weixin_43011522/article/details/82495556)

