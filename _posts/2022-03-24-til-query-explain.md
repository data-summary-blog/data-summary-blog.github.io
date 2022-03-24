---
layout: post
title: "TIL: Explain (query)"
categories: til
author: "Yongchan Hong"
---

# TIL: Explain (query)

이미 어느정도 알고 있는 거지만, 쿼리의 성능을 확인하기 위하여 `explain`을 쓸 수 있다.
Query 전에 explain을 붙이면 된다. EX: explain select * from aaa  
나온 값중 key를 통해 어떤 index를 사용하는지 알 수 있고, type을 통해 full scan인지, range인지 등을 알 수 있다.