<!-- TITLE: Let's Encrypt -->
<!-- SUBTITLE: Let's Encrypt, Https, TLS -->

# Letsencrypt 간편 셋팅
1.  https://certbot.eff.org/ 접속
1.  OS, 웹서버 선택
1.  안내하는데로 진행한다.
	1. certbot 설치
	1. 인증서 설치
1.  crontab으로 자동 갱신
```sh
# 편집화면 진입
crontab -e 

# 편집화면에서 입력
0 0,12 * * * python -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew
```

# 와일드카드 인증서 
* https://certbot.eff.org/docs/using.html#dns-plugins
그러니까... 플러그인을 제공하는 업체가 아니면 도메인 소유자 인증을 수동으로 해야 한다는 말이다.
당연히 해당 업체 서비스를 사용하지 않기 때문에 수동으로 인증한다.
인증 방법은 certbot을 통해 생성된 TXT레코드를 내 도메인에 추가.
TXT레코드 반영이 꽤 오래 걸린다. 내 네트워크에서 `dig`로 TXT레코드가 출력되도 검증이 실패 할 수 있으므로 충분히 기다린다.
	**모든 3차 도메인과 2차 도메인에 대한 인증서를 발급한다.**
```
certbot certonly \
--manual \
-d *.domain.net \ # 3차 도메인
-d domain.net \ # 2차 도메인
--manual-public-ip-logging-ok \
--preferred-challenges dns-01 \
--server https://acme-v02.api.letsencrypt.org/directory
```

여기까지 발급이고... 문제는 갱신이다. 
업체에서 레코드 관리 API를 제공하지 않기 떄문에 자체 dns서버를 구축해서 TXT레코드 검증을 해야한다고 한다.
이런 준비가 안되어 있다면 신규 발급 과정과 동일하게 갱신해야 한다고 한다.
