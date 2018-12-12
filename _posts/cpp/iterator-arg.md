---
title: "iterator 인자로 넘기기"
date: 2017-10-06 05:23:53
categories: C++
tags:
- C++
---

iterator를 인자로 받고 싶은 경우가 있다.
이때 단순히 `void f(iterator it)`와 같이 인자를 받아서는 안 된다.
인자를 넘기는 방법은 두 가지가 있다.
<!-- more -->

### 타입을 명확히 넘겨주기

vector나 set 등의 라이브러리에  정의되어 있는 iterator는 해당 클래스 내부적으로 정의된 것이다.
따라서 함수에 `void f(iterator it)`와 같이 넘겨주면, 전역에서 정의되지 않은 타입이므로 컴파일에 실패한다.
따라서 `vector<int>`에서 정의한 iterator를 넘겨주고 싶다면 다음과 같이 넘겨주어야 한다.

```c++
void f(vector<int>::iterator it);
```



어떤 타입을 받는 vector인지 넘겨주어야 하는 이유는, template로 만들어진 다형 함수/클래스는 각기 다른 함수/클래스와 다름없기 때문이다.
어떤 타입을 받든 동작하게 하고 싶다면, 다음과 같이 넘겨주면 될 것이다.

```c++
template <typename T>
void f(<vector<T>::iterator it);
```


### [iterator_traits](http://en.cppreference.com/w/cpp/iterator/iterator_traits)

`std::iterator_traits`를 사용하는 방법이다.
아무 iterator나 받아서 그 값을 읽거나, iterator 간의 거리를 재는 등의 연산을 할 때 유용하다.
`template`를 사용해서 인자를 자유롭게 받은 뒤, 해당 인자에서 trait을 읽어오는 것이다.

아래 예제는 iterator를 넘겨받아 그 값을 출력한다.

```c++
#include <iostream>
#include <iterator>
#include <vector>
 
template <typename BidirIt>
void my_print(BidirIt it)
{
    typename std::iterator_traits<BidirIt>::value_type val = *it;
    std::cout << val << std::endl;
}
 
int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};
    auto i = v.begin();
    while(i != v.end())
        my_print(i++);
    
    return 0;
}
```

