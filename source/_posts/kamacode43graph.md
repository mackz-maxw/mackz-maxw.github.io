---
title: 代码随想录 | 刷题-图论1
categories: leetcode
lang: zh-CN
---

### 卡码网 98. 所有可达路径
给定一个有 n 个节点的有向无环图，节点编号从 1 到 n。请编写一个程序，找出并返回所有从节点 1 到节点 n 的路径。每条路径应以节点编号的列表形式表示

输入描述
> 第一行包含两个整数 N，M，表示图中拥有 N 个节点，M 条边
> 后续 M 行，每行包含两个整数 s 和 t，表示图中的 s 节点与 t 节点中有一条路径

输出描述
> 输出所有的可达路径，路径中所有节点之间空格隔开，每条路径独占一行，存在多条路径，路径输出的顺序可任意。如果不存在任何一条路径，则输出 -1。
> 注意输出的序列中，最后一个节点后面没有空格！ 例如正确的答案是 `1 3 5`,而不是 `1 3 5 `， 5后面没有空格！

```cpp
#include<iostream>
#include<vector>
#include<list>
using namespace std;
vector<vector<int>> result;
vector<int> path;
void dfs(const vector<list<int>> &vnode, int n){
    // if(path.empty())path.push_back(1);
    int cur_i = path.back();
    if(cur_i == n){
        result.push_back(path);
    }else{
        list<int> n_i = vnode[cur_i];
        if(!n_i.empty()){
            for(int i : n_i){
                path.push_back(i);
                dfs(vnode, n);
                path.pop_back();
            }
        }
    }
}
int main(){
    int n, m;
    cin >> n >> m;
    vector<list<int>> vnode(n+1);
    for(int i = 0; i < m; i++){
        int node, next_i;
        cin >> node >> next_i;
        vnode[node].push_back(next_i);
    }
    path.push_back(1);
    dfs(vnode, n);
    if(result.empty()){
        cout << -1;
        return 0;
    }
    for(auto &path : result){
        for(int i = 0; i < path.size()-1; i++){
            cout << path[i] << ' ';
        }
        cout << path.back() << endl;
    }
    return 0;
}
```