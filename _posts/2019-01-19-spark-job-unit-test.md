---
title: principles for the unit test of data processing job
date: 2019-01-19 12:50:00 +0900
categories: [dev, spark]
tags: [spark, unit test]
---

spark job 개발하다가 몇 가지 회고
- 결과물을 빨리 내겠다고 마음이 급해지니 제일 먼저 UnitTest 개발을 Skip 한다.
  - 여기서부터 코드에서 나쁜 냄새가 나기 시작한다.
- 가능한한 연산의 과정, 과정을 method로 나누고 UnitTest를 만들자.
  - 특히 numpy나 pandas로 matrix 연산을 하고 있다면,
  - 그리고 vector space가 크다면,
  - 더더욱 그렇게 해야 한다.
  - 안그러면 연산 결과에 대한 검증을 하기가 힘들다.
  - 단, numpy나 pandas 자체를 테스트하지는 말자.
- 연산결과의 테스트는 간단한 수치나 matrix를 pararellize해서 수행하고, 여러 건의 데이터가 필요한 경우에는 실데이터를 일부 sampling해서 로컬에 적재하여 사용한다.
  - 연산결과 검증을 sampling한 데이터로 하게되면, Expected Value를 계산하기 힘들 수 있다.
    - 예를들어 Tf-Idf Vector가 잘 만들어졌는지 보기위해 실제 콘텐츠 데이터를 사용하면, 몇백자 되는 콘텐츠를 보고 일일히 Tf-Idf Vector를 만들어보아야 하는 수고가...
    - 이런 경우는 그냥 간단히 단어 몇 개만 가지고 Tf-Idf Vector 만들어서 검증
