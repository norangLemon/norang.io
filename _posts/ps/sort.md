---
title: 정렬
date: 2017-03-09 12:09:44
tags:
- ps
- sorting
- C++
---
다양한 고급 정렬을 C++로 직접 구현해보자.

### quick sort

퀵소트는 O(nlogn) 정렬 중에서, 평균적으로 괜찮은 성능을 보인다.
in-place sort이기 때문에, 값을 복사하는 시간이 추가로 들지 않기 때문이다.
다만, stable sort는 아니다.
<!-- more -->

```C++
#include <algorithm>
#include <vector>
using namespace std;

int getPartition(int start, int end, vector<int>& v){
    int pivot = v[end];
    int i = start, j = start;
    while(j < end){
        if (v[j] < pivot) swap(v[i++], v[j]);
        j++;
    }
    swap(v[i], v[j]);
    return i;
}

void quickSort(int start, int end, vector<int>& v){
    if (start >= end) return;
    int partition = getPartition(start, end, v);
    quickSort(start, partition-1, v);
    quickSort(partition+1, end, v);
}
```

### merge sort
merge sort는 inplace sort는 아니지만 대표적인 stable sort다.
역시 O(nlogn)이다.

```C++
#include <vector>
#include <algorithm>
using namespace std;

void merge(int start, int end, int mid, vector<int>& ary){
    vector<int> temp;
    for (int i = 0; i <= end-start; i++){
        temp.push_back(ary[start+i]);
    }
    int pivot_head = 0, pivot_tail = mid-start+1, pivot_new = start;
    while(pivot_head <= mid-start && pivot_tail <= end-start){
        if (temp[pivot_head] > temp[pivot_tail]) {
            ary[pivot_new++] = temp[pivot_tail++];
        } else {
            ary[pivot_new++] = temp[pivot_head++];
        }
    }
    while(pivot_head <= mid-start){
        ary[pivot_new++] = temp[pivot_head++];
    }
    while(pivot_tail <= end-start){
        ary[pivot_new++] = temp[pivot_tail++];
    }
}

void mergeSort(int start, int end, vector<int>& ary){
    if (start >= end) return;
    int mid = (end + start)/2;
    mergeSort(start, mid, ary);
    mergeSort(mid+1, end, ary);
    merge(start, end, mid, ary);
}
```


### count sort
정렬해야 하는 숫자의 범위가 작은 경우, O(n)으로 count sort를 할 수 있다.
범위가 큰 경우, radix sort를 하면 된다.
다만 radix sort의 경우, 상수값이 커서 정렬해야 하는 수가 아주 많지 않은 경우에는
평균적으로 다른 O(nlogn) 정렬보다 더 많은 시간이 소요된다.


```C++
#include <cstdio>

int cnt[10010]; // 1~10000의 숫자를 정렬하는 경우
int n, t;

int main() {
    scanf("%d", &n);
    int largest = 0;
    for(int i = 0; i < n; i++){
        scanf("%d", &t);
        cnt[t]++;
        largest = largest > t? largest : t;
    }
    for(int i = 1 ; i <= largest; i++){
        while(cnt[i]){
            printf("%d\n", i);
            cnt[i]--;
        }
    }
}
```
