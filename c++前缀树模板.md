---
title: "C++前缀树模板"
date: 2019-08-31T10:33:08+08:00
draft: true
tags: ["算法，自用"]
---
### 前缀树又称Tries树、字典树、单词查找树等，常用于快速检索，大量字符串的排序和统计等。
在计算机科学中，trie，又称前缀树或字典树，是一种有序树，用于保存关联数组，其中的键通常是字符串。与二叉查找树不同，键不是直接保存在节点中，而是由节点在树中的位置决定。一个节点的所有子孙都有相同的前缀，也就是这个节点对应的字符串，而根节点对应空字符串。一般情况下，不是所有的节点都有对应的值，只有叶子节点和部分内部节点所对应的键才有相关的值
+ 根节点不包含字符，除根节点外每个节点只包含一个字符。
+ 从根节点到某个节点，路径上所有的字符连接起来，就是这个节点所对应的字符串。
+ 每个节点的子节点所包含的字符都不同。

结构如下：
<img src="./前缀树.png">

代码实现:

```c++
/*数据结构：字典树(前缀树)*/
 
#include <iostream>
#include <string.h>
 
using namespace std;
/*字典树结点*/
class TrieNode{
public:
    /*结点最大分支树(由于这里假设所有输入是由26个小写字母组成，故MaxBranchNum为26)*/
    const static int MaxBranchNum = 26;
    /*指针数组，代表26个字母，如果包含第i个字母，那么nextBranch[i]!=NULL*/
    TrieNode* nextBranch[MaxBranchNum];
    /*是否是一个单词的结尾*/
    bool isEnd;
    /*当isEnd==true，该值保存对应的单词*/
    string word;
    /*该单词的出现次数*/
    int count;
 
    TrieNode():count(0),isEnd(false){
        memset(nextBranch,NULL,sizeof(TrieNode*)*MaxBranchNum);//初始化nextBranch数组
    };
    /*插入字母ch(相当于将ch在nextBranch中对应的指针实例化)*/
    void put(char ch){
        if(nextBranch[ch-'a']==NULL)nextBranch[ch-'a'] = new TrieNode();
    }
    /*获得字母ch对应的结点指针，当ch不存在时返回NULL*/
    TrieNode* get(char ch){return nextBranch[ch-'a'];}
    /*判断当前结点是否包含字母ch(相当于判断ch在nextBranch中对应的指针是否为NULL)*/
    bool containsKey(char ch){return nextBranch[ch-'a']!=NULL;}
    /*设置isEnd为true*/
    void setEnd(){isEnd = true;}
    /*判断是否是单词*/
    bool isWord(){return isEnd;}
};
/*字典树*/
class Trie{
public:
    Trie(){root = new TrieNode();}
    ~Trie(){destory(this->root);}
 
    /*插入单词*/
    void insert(const string word);
 
    /*查找前缀prefix，如果查找成功则返回最后的结点，否则返回NULL*/
    TrieNode* searchPrefix(const string prefix);
    /*查找单词word,如果查找成功返回word的出现次数，否则返回0;*/
    int search(const string word);
    /*如果存在任意单词以prefix为前缀，则返回true，否则返回false*/
    bool startsWith(const string prefix);
 
    /*打印当前字典树中的所有单词*/
    void printALL();
    /*打印以prefix为前缀的所有单词*/
    void printPre(const string prefix);
private:
    TrieNode* root;
    /*销毁以root为根的子树*/
    void destory(TrieNode* root);
    /*打印以root为根的所有单词*/
    void print(TrieNode* root);
};
 
/*插入单词*/
void Trie::insert(const string word){
    if(word.size()==0)return;
    for(int i=0;i<word.size();i++) //出现异常字符时直接返回
        if(word[i]<'a'||word[i]>'z')return;
    TrieNode* node = root;
    for(int i=0;i<word.size();i++){
        if(!node->containsKey(word[i]))
            node->put(word[i]);
        node = node->get(word[i]);
    }
    //此时的node指向最终的结点
    if(node->word.size()==0){
        node->word.assign(word);
    }
    node->count++;
    node->setEnd();
}
 
/*查找前缀prefix，如果查找成功则返回最后的结点，否则返回NULL*/
TrieNode* Trie::searchPrefix(const string prefix){
    TrieNode* node = root;
    for(int i=0;i<prefix.size();i++){
        if(prefix[i]<'a'||prefix[i]>'z')return NULL;
        if(node->containsKey(prefix[i]))
            node = node->get(prefix[i]);
        else
            return NULL;
    }
    return node;
}
 
/*查找单词word,如果查找成功返回word的出现次数，否则返回0;*/
int Trie::search(const string word){
    TrieNode* node = searchPrefix(word);
    return node?node->count:0;
}
 
/*如果存在任意单词以prefix为前缀，则返回true，否则返回false*/
bool Trie::startsWith(const string prefix){
    return searchPrefix(prefix)!=NULL;
}
 
/*打印以root为根的所有单词*/
void Trie::print(TrieNode* root){
    if(root==NULL)return;
    TrieNode* node = root;
    if(root->word.size()!=0)cout<<root->word<<endl;
    for(int i=0;i<root->MaxBranchNum;i++)
        print(root->nextBranch[i]);
}
/*打印当前字典树中的所有单词*/
void Trie::printALL(){
    print(this->root);
}
/*打印以prefix为前缀的所有单词*/
void Trie::printPre(const string prefix){
    if(prefix.size()==0)printALL();
    else{
        TrieNode* node = root;
        for(int i=0;i<prefix.size();i++){
            if(node->containsKey(prefix[i]))node = node->get(prefix[i]);
            else return;
        }
        print(node);
    }
}
/*销毁以root为根的子树*/
void Trie::destory(TrieNode* root){
    if(root==NULL)return;
    for(int i=0;i<root->MaxBranchNum;i++)
        destory(root->nextBranch[i]);
    delete root;
    root = NULL;
}
int main(){
    Trie* a = new Trie();
    a->insert("hello");
    a->insert("hello");
    a->insert("hi");
    a->insert("beauty");
    a->insert("helloworld");
    a->insert("word");
    cout<<a->search("hello")<<endl; //2
    a->printALL();
    a->printPre("h");
    delete a;
    return 0;
}
```
*原文链接：https://blog.csdn.net/bqw18744018044/article/details/82502435*

