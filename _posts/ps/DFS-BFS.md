---
title: 'DFS, BFS'
date: 2017-03-14 11:01:11
categories: ps
tags:
- ps
---
그래프의 두 가지 탐색법 DFS, BFS.

### DFS

#### 재귀 방식

방문을 했는지 알기 위해서, check array를 만들어둔다.
i번 노드를 방문한 경우, check[i]를 1로 만든다.

방문은 재귀적으로 이루어진다.
즉, 스택이 사용된다.
현재 노드에서 이어진 노드를 방문한 뒤, 다시 한번 해당 함수를 호출한다.
<!-- more -->
```C++
#include <vector>
#include <algorithm>
using namespace std;

vector<int> ary[NODE_NUM];
int check[NODE_NUM];

void dfs(int now){
    visit[now] = 0;
    printf("%d ", now);
    for (int i = 0; i < ary[now].size(); i++){
        int node = ary[now][i];
        if (!visit[node]) {
            dfs(node);
        }
    }
}
```

#### 비재귀 방식

스택에 앞으로 방문해야 하는 노드들을 직접 푸시해둔다.
방문한 경우, 해당 노드를 버리면 되고, 방문해야 하는 경우는 같은 방식으로 반복해준다.

```C++
int check[101];
vector<int> neighbor[101];

int DFS(int start){
    int num = 0;
    stack<int> s;
    s.push(start);
    while(!s.empty()){
        int now = s.top();
        s.pop();
        if (check[now]) continue;
        check[now] = 1;
        for(int i = 0; i < neighbor[now].size(); i++){
            s.push(neighbor[now][i]);
        }
        num++;
    }
    return num;
}
```

### BFS


```C++
#include <queue>
int check[1001];

void bfs(int now){
    check[now] = 1;
    queue<int> q;
    q.push(now);
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        printf("%d ", u);
        for (int i = 0; i < ary[u].size(); i++){
            int node = ary[u][i];
            if(!visit[node]){
                check[node] = 1;
                q.push(node);
            }
        }
    }
}
```
