<!-- TITLE: NAPT -->
<!-- SUBTITLE: NAPT, NAT, Network -->

# Network Address and Port Translation
NAT의 확장판 기술로 NAT, NAPT 각각 존재하는 기술이나 현대에서 말하는 NAT은 100% NAPT를 의미한다.
적은수의 외부 IP 주소를 통해 다수의 내부 IP 주소를 외부 네트워크에 연결시켜주는 기술.
* http://www.tcpipguide.com/free/t_IPNATPortBasedOverloadedOperationNetworkAddressPor.htm 상세한 내용 (안 읽음)
* https://www.cisco.com/c/en/us/about/press/internet-protocol-journal/back-issues/table-contents-29/anatomy.html

내부 IP 주소:포트로 출발한 패킷을 라우터가 외부 IP 주소:포트로 출발한 것처럼 패킷 헤더를 조작하고 맵핑 테이블에 저장한 다음 목적지로 보낸다.
외부 네트워크에서 패킷이 들어오면 라우터는 패킷의 목적지 포트를 보고 맵핑 테이블에서 내부 IP 주소:포트를 찾아서 패킷을 보낸다.
외부 IP 하나당 가능한 커넥션 수는 가용한 최대 포트수인 ~64K 이므로 대규모 사설망이라면 다수의 외부 IP 주소 pool이 필요하다.

NAPT가 마냥 좋은 것처럼 얘기하고 끝내는 곳이 대다수이다.
유명한 문제점으로 네트워크의 end to end원칙을 위배한다는 것(잘 모르겠다.), 많은 사람들이 보안에 도움이 되는 것으로 오해하고 있다는 것,
IPSec 보급을 막고 있다는 것, 인터넷 애플리케이션이 반강제로 TCP/UDP만 사용해야 한다는 것이다.
현대에서 ICMP, FTP active mode가 가능한 이유는 라우터에 별도로 기능을 넣었기 때문이고 이런식으로 라우터 내부가 점점 복잡해지고 있다고...
* https://community.extremenetworks.com/extreme/topics/nat-error-global-ip-addresses-exhausted-for-pool
  * 외부 IP 주소가 한개라면 한명이 포트스캔 혹은 토렌트를 사용해서 외부 IP 주소가 가용한 포트를 고갈시킬수 있는 가능성을 제기했다.
* https://otacon22.com/2013/01/15/network-address-translation-an-abomination-and-horror/
  * NAPT가 왜 사용하지 말아야 할 기술인지 10가지 이유로 설명한다.
* https://blog.webernetz.net/why-nat-has-nothing-to-do-with-security/
  * 보안 때문이라면 NAPT를 사용하지 말라고한다.
* https://networkingnerd.net/2011/05/05/i-hate-nat-or-do-i/
  * 나중에 읽어보겠음.

NAPT를 조사하면서 문제점도 알고 싶어서 하루 종일 구글링 한 결과,
한글로 된 글중에 이 기술이 뭐다하는 글만 복붙 한것처럼 잔뜩있고 한계와 문제를 똑바로 얘기하는 글은 한개도 못찾았다.🤯