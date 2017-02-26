---
title: priority_queue
date: 2017-02-26 21:52:03
tags: 
- ps
- 자료구조
category: ps
---

> [priority_queue](http://en.cppreference.com/w/cpp/container/priority_queue)
> header: queue

```
template<
    class T,
    class Container = std::vector<T>,
    class Compare = std::less<typename Container::value_type>
> class priority_queue;
```

## 특징
* T 타입을 넣으면, `vector<T>`에 들고 있다가 큰 순서대로 뱉어준다.
작은 것부터 뱉게 하고 싶다면, `greater<T>`를 Compare 자리에 넣으면 된다.
* `priority_queue<T>`과 같이 선언하면, 뒤 두 값은 기본값으로 들어간다.

### 시간 복잡도
* 삽입, 검색에 O(log n)의 시간이 소요되며, 꼭대기를 참조하는 데에는 O(1)의 시간이 소요

### 주요 함수
* `top()`: 꼭대기 하나를 읽는다
* `pop()`: 꼭대기 하나를 뽑는다. **값을 리턴해주지 않으니 주의**
* `push(value)`: `value`를 넣는다.
* `emplace(value)`: 값을 넣되, T의 생성자 인수를 받아서 생성과 동시에 삽입한다.
