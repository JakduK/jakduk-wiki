<!-- TITLE: Linux -->
<!-- SUBTITLE: Linux -->

# 디렉토리 구조
https://www.tldp.org/LDP/Linux-Filesystem-Hierarchy/html/opt.html 잘 요약된 글

*/opt
  *최초 리눅스 벤더들의 옵션 패키지라 부르는 벤더가 제공하는 프로그램을 담는 디렉토리였다. 그래서 optional -> opt. 정식명칭은 optional add-on software packages.
  *패키지 매니저없이 수동 설치 할 프로그램은 여기로.
*/usr/local
  *리눅스 표준 구조로 설치할 프로그램은 여기로.
*/sbin, /usr/sbin
  *시스템 관리자(Administrator) 전용 프로그램

# systemd
*http://linux.systemv.pe.kr/centos-7-systemd-이해하기/
*https://www.opentechguides.com/how-to/article/centos/169/systemd-custom-service.html 유닛 파일 만들기
*위치

            Table 1.  Load path when running in system mode (--system).
           ┌────────────────────────┬─────────────────────────────┐
           │Path                    │ Description                 │
           ├────────────────────────┼─────────────────────────────┤
           │/etc/systemd/system     │ Local configuration         │
           ├────────────────────────┼─────────────────────────────┤
           │/run/systemd/system     │ Runtime units               │
           ├────────────────────────┼─────────────────────────────┤
           │/usr/lib/systemd/system │ Units of installed packages │
           └────────────────────────┴─────────────────────────────┘

*부팅 서비스 등록
```sh
systemctl enable <service>
```

*부팅 서비스 제거
```sh
systemctl disable <service>
```

*https://superuser.com/questions/513159/how-to-remove-systemd-services 서비스 제거
```sh
systemctl stop [servicename]
systemctl disable [servicename]
rm /etc/systemd/system/[servicename]
rm /etc/systemd/system/[servicename] #symlinks that might be related
systemctl daemon-reload
systemctl reset-failed
```

# 방화벽
리눅스 설치 직후 방화벽 데몬으로 인해 다 막혀있다고 봐야 한다.
이것저것 설정했는데 외부에서 접속이 안되면 firewalld 서비스 정지후 접속하여 방화벽 때문인지 확인해보자.
22번은 기본으로 열려 있음.
3000/27017번 같은 특정 어플리케이션 포트는 직접 열어줘야함.
리눅스VM 생성후 mongodb 27071포트, nodejs서버 3000포트 접속안되서 시간낭비 많이했음

*https://www.lesstif.com/pages/viewpage.action?pageId=22053128 방화벽 헬퍼 안내 firewall-cmd, firewalld
*systmctl stop firewalld 한뒤 외부에서 접근 시도
*firewall-cmd --get-active-zone 활성화된 존 확인
*firewall-cmd --permanent --zone=public --add-port=3000/tcp 활성화된 존에 포트개방

# nc (netcat) 포트 테스트
* BSD : nc -z host port
* Linux : nc -zv host port

# 타임존 변경

* 시간보기 커맨드

```sh
date
```

*/etc/localtime 심볼릭 링크가 타임존을 가리키고 있다.
*/etc/localtime 심볼릭 링크를 /usr/share/zoneinfo 안에서 원하는 타임존을 선택해서 업데이트한다.

```sh
ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```

* 타임존 변경 헬퍼 사용
```sh
timedatectl set-timezone Asia/Seoul
```

# 시간 sync
https://www.howtogeek.com/tips/how-to-sync-your-linux-server-time-with-network-time-servers-ntp How to Sync Your Linux Server Time with Network Time Servers (NTP)

* ntpd 확인
```sh
systemctl status ntpd
```

* ntpd 설치
```sh
yum install -y ntpd
```

* sync 실행
```sh
/usr/sbin/ntpdate pool.ntp.org
```

# 로그 검색 커맨드 팁
## tail
*tail -n+<줄번호> : <줄번호> 번째부터 출력
*-A 옵션 : after의미로 검색된 줄 아래로 포함할 줄수
*-B 옵션 : before의미로 검색된 줄 위로 포함할 줄수

## less
*more의 기능 강화 커맨드
*more보다 less 사용 습관 들이는 게 좋음

# curl
* form data
```sh
curl -X POST -d 'name=value&name2=value' <host>
```

* cookie
```sh
curl --cookie 'name=value' <host>
```

* header
```sh
curl -H 'name:value' <host>
```

* json data
```sh
curl -X POST -H 'content-type: application/json' -d '{"name":"value"}' <host>
```

# gzip/gunzip
* 디렉토리 압축

gzip만으로 디렉토리 압축 불가능. tar와 함께 사용해야 한다.
tar로 패키징하고 gzip으로 압축.
```sh
tar -cv directory | gzip > archive.tar.gz

# tar에 gzip압축 옵션(z)을 포함하고 있어서 gzip압축하려면 그냥 tar를 사용하면 된다.
tar -zcvf archive.tar.gz directory/ 
```

# tar
구글링 좀 그만하려고 기록

* f : in/out을 파일로 지정
* z : gzip알고리즘 사용
* v : verbose
* x : extract
* c : create new archive
* l : 내용물 열람

# RHEL/EPEL 패키지 대신 최신 패키지 설치
https://ius.io

* https://ius.io/GettingStarted/#ius-release 적절한 레포지토리 설정을 다운로드받아 설치
* https://github.com/iuscommunity-pkg/ 패키지 검색
* 기존 것을 삭제하고 검색 결과 패키지이름으로 설치

```sh
#git 2.x 설치 예시
#root 계정이라 치고

wget https://centos7.iuscommunity.org/ius-release.rpm
rpm -i ius-release.rpm
yum -y remove git
yum -y install git2u
</source>
=swapfile=
===만들기/장착===
<source>
 # 당연히 root 권한으로 해야함.
 # swapfile 만들기
 # fallocate 커맨드를 사용하는 예제가 많은데 CentOS는 dd커맨드로 swapfile을 만들어야함.
 # 1024K 블럭 1024개 = 1GiB. 크기는 마음대로.
 dd if=/dev/zero of=/swapfile bs=1024K count=1024
 mkswap /swapfile
 swapon /swapfile
```

* 리붓하면 사라지므로 유지시키기위해 /etc/fstab 파일에 아래 내용을 추가한다.
```sh
 /swapfie swap swap defaults 0 0
```

* https://www.centos.org/docs/5/html/Deployment_Guide-en-US/s1-swap-adding.html
* https://www.cyberciti.biz/faq/linux-add-a-swap-file-howto/

> 1. if=/dev/zero : Read from /dev/zero file. /dev/zero is a special file in that provides as many null characters to build storage file called /swapfile1.
> 2. of=/swapfile1 : Read from /dev/zero write storage file to /swapfile1.
> 3. bs=1024 : Read and write 1024 BYTES bytes at a time.
> 4. count=524288 : Copy only 523288 BLOCKS input blocks.
## 리사이즈
```sh
swapoff /swapfile
dd if=/dev/zero of=/swapfile bs=1M count=1024 oflag=append conv=notrunc
mkswap /swapfile
swapon /swapfile
```

* https://askubuntu.com/a/927870

## 삭제
```sh
swapoff -v /swapfile
rm /swapfile
```

https://www.centos.org/docs/5/html/5.2/Deployment_Guide/s2-swap-removing-file.html

## 계정 관리 (CentOS)
* 계정 추가 : useradd 계정이름
* 계정이 속한 그룹 목록 : groups
* 계정의 그룹 변경 : usermod -G 그룹이름 계정이름
* 계정 삭제 : userdel -r(홈 디렉토리 삭제) 계정이름
* 그룹에 계정 추가 : gpasswd -a 계정이름 그룹이름
* 그룹에서 계정 제거 : gpasswd -d 계정이름 그룹이름
* 그룹 추가 : groupadd 그룹이름
* 그룹 삭제 : groupdel 그룹이름
* 그룹 목록 보기 : cat /etc/group

## sudo
https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos

/etc/sudoers 파일은 반드시 visudo 커맨드로 변경해라
admin권한을 부여하려면 wheel그룹에 추가해라
적용이 안된다면 %weel ALL=(ALL) ALL 코멘트 해제해라