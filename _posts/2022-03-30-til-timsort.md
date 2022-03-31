---
layout: post
title: "TIL: Tim Sort"
categories: til
author: "Yongchan Hong"
---

# TIL: Tim Sort
Python에서 sorted()나 sort()를 쓰면 어떤 알고리즘을 쓸까? 바로 Tim Sort다. Tim Sort는 Insertion Sort와 Merge Sort를 섞어서 만든 Sort로 최선의 시간 복잡도 O(N), 평균 시간 복잡도 O(NlogN), 최악 시간 복잡도 O(NlogN)을 가지고 있다. Mergesort보다 최선 시간 복잡도가 나은데, 어떻게 다른걸까?

Tim Sort는 우리의 배열이 어느정도 정렬이 되어있지않을까 하는 생각으로, 정렬을 해야하는 작은 덩어리로 잘라 Insertion Sort로 정렬한뒤 Merge Sort로 병합하자는 것이다. 먼저, 덩어리가 2^n이 될때까지 (2^n인 이유는 Merge sort가 최초에 2개씩 병합하기 때문) binary insertion sort를 진행한다. 맨 앞의 두 원소를 보고 증가/감소하는 지 판단하고 삽입해야할 위치를 찾아간다. 그리고 2^n이 되면 이 덩어리를 minrun이라 하는데, 그 이후에 추가적으로 증가하는 친구들이 있다면 더 붙여서 run을 만든다. 그 이후부터는 증가였다면 감소하는 형태로 반대의 run을 만들어준다. 그렇게 run들이 만들어진다면, 이제 이걸 가지고 merge sort를 한다. 사실 이미 어느정도 정렬이 되어있기에, merge sort의 divide and conquer에서 divide과정은 생략할 수 있다. 2개의 run을 병합할때는 커지는거/작아지는거 순으로 되어있는데 작아지는 것을 반대로 하면 반대로 커지는거 처럼 되고, 따라서 작은 run을 복사를 하고 포인터로 사용해 둘을 합치는 정렬을 한다. 

간단하게 Tim sort에 대해서 알아보았다. 

### Reference
https://d2.naver.com/helloworld/0315536
