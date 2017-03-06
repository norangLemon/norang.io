---
title: stack
date: 2017-03-06 07:39:49
tags:
- ps
- 자료구조
- C++
categories:
- ps
- 자료구조
- languages
---

> [stack](http://en.cppreference.com/w/cpp/container/stack)
> header: stack

```C++
template<
    class T,
    class Container = std::deque<T>
> class stack;
```

<!-- more -->
### 특징
* 삽입, 제거에 O(1)시간 소요.
* FILO(First In Last Out)

### 주요 함수
* top(): 꼭대기 하나를 읽는다.
* pop(): 꼭대기 하나를 제거한다. **값을 리턴해주지 않으니 주의**
* push(value): `value`를 삽입한다.
* emplace(value): 값을 넣되, T의 생성자 인수를 받아서 생성과 동시에 삽입한다.

