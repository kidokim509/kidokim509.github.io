---
title: 좋은 TEST CASE를 만드는 법
date: 2015-09-18 23:00:00 +0900
categories: [dev, retrospective]
tags: [test case, acceptance criteria]
---
이번 스프린트에서는 Test Case가 치밀하지 못한 덕에 테스트 및 버그 수정 기간이 예상보다 많이 늘어졌음

* Acceptance Criteria를 100% 반영할 것
* Exception Handling을 검증할 것
* 성능이 특히 중요한 경우, Benchmark할 수 있는 Test Case를 추가할 것
  * 기능적으로 문제가 없으나, 서비스에 반영하면 성능이 나오지 않아 이슈가 되는 것을 사전에 검증
  * Acceptance Criteria에 성능 목표가 있다면 더욱 훌륭함
