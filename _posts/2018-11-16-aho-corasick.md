---
title:   aho-corasick algorithm
date:    2018-11-16 11:00:00 +0900
categories: [dev, nlp]
tags: [algorithm, nlp]
---
## Aho-Corasick (아호-코라식)
* 참고
  * [http://m.blog.naver.com/kks227/220992598966](http://m.blog.naver.com/kks227/220992598966)
  * [https://www.slideshare.net/ssuser81b91b/ahocorasick-algorithm](https://www.slideshare.net/ssuser81b91b/ahocorasick-algorithm)

아호 코라식 알고리즘(Aho–Corasick string matching algorithm)은 Alfred V. Aho와 Margaret J. Corasick이 고안한 패턴 집합에 대한 매칭 알고리즘이다.  
O(m + n + k)의 시간 복잡도로 패턴 집합에 대하여 패턴 길이와 텍스트의 선형 시간에 탐색을 처리할 수 있게 된다. (m: 모든 패턴 길이 합, n: 텍스트 길이, k: 텍스트 내에 패턴의 발생 수)

간단히 실험해보고 싶다면 [https://github.com/WojciechMula/pyahocorasick](https://github.com/WojciechMula/pyahocorasick)

애니메이션으로 이해해보기 [http://blog.ivank.net/aho-corasick-algorithm-in-as3.html](http://blog.ivank.net/aho-corasick-algorithm-in-as3.html)
