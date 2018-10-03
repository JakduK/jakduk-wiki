<!-- TITLE: Let's Encrypt -->
<!-- SUBTITLE: Let's Encrypt, Https, TLS -->

# Letsencrypt 간편 셋팅
* https://certbot.eff.org/ 접속
* OS, 웹서버 선택
* 안내하는데로 진행한다.
* crontab으로 자동 갱신
```sh
crontab -e 0 0,12 * * * python -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew
```

# 와일드카드 인증서 제공을 하는데 직접 해보진 않음. 
* https://certbot.eff.org/docs/using.html#dns-plugins