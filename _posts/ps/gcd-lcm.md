---
title: '최대공약수, 최소공배수'
date: 2017-03-01 19:59:23
categories: ps
tags:
- ps
- google
---
이산 수학 시간에 배운 [유클리드 호제법](https://ko.wikipedia.org/wiki/유클리드_호제법)이다.
최악의 경우 O(log n) 정도의 시간복잡도.
Stack Overflow를 걱정하지는 않아도 된다.
<!-- more -->

```C++
#include <cstdio>
int x, y;
int gcd, lcm;

int f(int x, int y){
if(!y) return x;
return f(y, x%y);
}
int main() {
scanf("%d %d", &x, &y);
lcm = x*y;
gcd = f(x, y);
printf("%d\n%d", gcd, lcm/gcd);
return 0;
}
```
