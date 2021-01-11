---
title: python tips
date: 2018-11-24 02:59:00 +0900
categories: [dev, python]
tags: [python]
---

## 3rd party library를 패키징하여 사용하기
```shell
$ pip install -r requirements.txt -t ./libs
$ cd ./libs && zip -r libs.zip .
```
이렇게 만든 libs.zip 파일을 python 코드에서
```python
import sys
sys.path.insert(0, 'libs.zip')
```
하면 모듈들을 import 해서 사용할 수 있다.
이 방식을 사용하면 python 프로젝트를 uber-jar 처럼 패키징해서 편하게 배포할 수 있다.
예를 들어 PySpark 잡으로 만들 때 편리하다. 필요한 패키지를 일일히 Spark Worker Node들에 설치하지 않아도 된다.
