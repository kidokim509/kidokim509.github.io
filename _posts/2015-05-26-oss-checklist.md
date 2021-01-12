---
title: 오픈소스 분석 체크 리스트
date: 2015-05-26 23:00:00 +0900
categories: [dev, retrospective]
tags: [open source, oss]
---
* 왜?
  * 어떤 문제를 해결하기 위한 기술인가?
  * 기존 기술의 어떤 문제를 해결하기 위해 나왔는가?
* 어떻게?
  *문제를 어떻게 해결하는가?
  * 기존에 유사한 시도는 없었는가?
    * 기존의 유사한 시도가 가지고 있던 한계점은 어떤 것들이 있었나?
    * 신기술에서는 이를 어떻게 해결하고 있나?
* 아키텍처
  * 성능
    * 성능 결정 요인은?
    * 예상 병목 구간은?
    * 한계 수치
    * 장애 대비
      * SPOF는 없는가?
      * 데이터 Overflow에 대한 처리는?
      * HA 구성은 가능한가?
      * 데이터 유실에 대한 대비는?
  * 시스템 확장
    * 확장 가능한 형태인가?
  * 운영
    * 모니터링 방안
    * 어떤 경우에 Down-time이 발생하는가?
  * 동시성 처리
    * Thread Pool
    * Lock System
* Getting Started
  * 설치 및 설정
    * 설정에 아키텍처 체크 항목의 많은 힌트가 있다.
  * Tutorial
  * Hack
* 코드 분석