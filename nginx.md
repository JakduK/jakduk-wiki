<!-- TITLE: Nginx -->
<!-- SUBTITLE: Nginx -->

# 자주 사용하는 기능 셋팅
* https://www.nginx.com/resources/wiki/start/topics/examples/full/ nginx.conf 종합 샘플
* https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/ public 파일 리소스 호스팅
* https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/ reverse proxy
* https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/ load balancing
* https://docs.nginx.com/nginx/admin-guide/web-server/compression/ content-encoding gzip

# 트러블 슈팅
## proxy_pass
https://stackoverflow.com/a/49492644 `13: Permission denied` nginx 에러
뭔지는 알고 사용하자
* setenforce
* semodule
## x-forwarded-for, x-real-ip
nginx가 프록시 서버에 요청하기 때문에 프록시 서버 액세스 로그에 nginx ip를 찍는다.
클라이언트 ip를 담아 보내기 위해 x-forwarded-for, x-real-ip 헤더를 사용한다. 아직 de-facto.
서비스 서버에서 x-forwarded-for, x-real-ip 해더값을 로거에 넘기는지 또는 로거가 해당 헤더를 읽어 찍는 로직이 있는지 확인 해봐야한다.

```text
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

koa프레임워크로 nodejs서버를 돌리면서 morgan을 사용한다면 proxy=true설정만으론 클라이언트 IP주소를 제대로 못 찍는다.
"remote-addr"토큰에 별도 fallback로직을 넣어야 한다.

```javascript
morgan.token('remote-addr', req => req.headers['x-real-ip'] || req.headers['x-forwarded-for'] || req.connection.remoteAddress);
```

* http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header 
	* 이 설정은 `proxy_set_header` 디렉티브를 사용한다.
* http://nginx.org/en/docs/http/ngx_http_proxy_module.html#variables 
	* `x-forwarded-for`헤더에 넣을 값은 가진 변수이름은 `$proxy_add_x_forwarded_for`
## 설정 파일을 읽는 시점에 도메인 네임이 IP주소로 변환됨
도메인 주소의 맵핑 IP주소가 바뀌면 리로드 해야됨. 이를 해결하려면 resolver설정을 추가해야함.
>  A server name, its port and the passed URI can also be specified using variables:
>  `proxy_pass http://$host$uri;`
>  or even like this:
>  `proxy_pass $request;`
>  In this case, the server name is searched among the described server groups, and, if not found, is determined using a resolver.
* http://nginx.org/en/docs/http/ngx_http_core_module.html#resolver
