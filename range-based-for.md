---
title: range based for
date: 2017-03-01 20:06:13
tags:
- ps
- C++
categories:
- languages
---

다음과 같이 Python의 for each 구문과 비슷한 C++의 for문을 range based for이라고 한다.
이는 C++11에서부터 도입된 것으로, 벡터, 리스트, 배열 등 다양한 자료구조에서 순차적으로 원소에 접근할 때 사용한다.

```C++
priority_queue<pair<int, int>> pq;
for (const auto& pair : students) {
    // range based for : C++11
    pq.emplace(pair.second, pair.first);
}
```
