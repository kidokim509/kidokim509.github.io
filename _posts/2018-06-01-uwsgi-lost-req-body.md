---
title: nginx + uwsgi + flask로 rest api 개발 시 post method의 request body 유실 케이스
date: 2018-06-01 23:19:00 +0900
categories: [dev, flask]
tags: [flask, uwsgi]
---

## 이슈
nginx + uwsgi + flask로 rest api 서버를 구성. spring webflux로 만든 다른 api 서버에서 flask로 post 요청을 보내면 flask app에서는 request body가 보이지 않음. nginx access로그에서는 보임. 특이한 것은 다른 client (예를 들어 postman)에서 같은 명령을 보내면 잘 됨. 반대로 client는 동일한데 flask를 flask run (즉, werkzeug)로 띄우면 이것도 잘 됨.

## 원인
현재까지 파악한 원인은 문제가 되는 경우에 http header를 보면 Content-Length와 Transfer-Encoding이 모두 들어옴. 원래 HTTP/1.1 스펙을 보면 두 Header는 같이 사용해서는 안되고, 둘 중에 하나는 무시해야 함. 
> Messages MUST NOT include both a Content-Length header field and a non-identity transfer-coding. If the message does include a non- identity transfer-coding, the Content-Length MUST be ignored. (https://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html)

**Update** Transfer-Encoding chunked만 header에 있을 때에도 처리 못함. 결국 uwsgi가 chunked request 처리를 못하는 것임. 아래 이슈에서 몇 가지 솔루션을 제시하는데, 해봤는데 안됨.
https://github.com/pallets/flask/issues/367

## 더 알아봐야 할 사항
uwsgi쪽에서 처리를 제대로 못하는 것으로 보임. 또, webflux는 http client를 netty를 쓰고 있는데, 어느 시점에 Content-Length와 Transfer-Encoding이 붙는 지도 살펴보아야 함. Transfer-Encoding은 netty client에서 붙이는 것 확인. Content-Length는 Nginx에서 붙임.
