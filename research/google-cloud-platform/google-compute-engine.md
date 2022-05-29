<!-- TITLE: Google Compute Engine -->
<!-- SUBTITLE: Google Cloud Platform, Google Compute Engine -->

# VM 인스턴스 만들기
* "이름" 항목은 인스턴스 생성후 변경 불가. 신중히 지어야함.
* "ID 및 API 액세스" 항목은 인스턴스 내에서 GCP API 허용범위 설정하는 것.

# VM 인스턴스 사양 변경하기
* VM 정지 -> 수정 -> 머신 유형 항목에서 선택

# swapfile로 메모리 부족 해소
최소 사양 VM(0.6GiB 램)은 경량 Node.js 서버 애플리케이션 조차 npm install 커맨드가 불가능하다.
메모리 부족으로 npm 프로세스를 죽여버린다.
swapfile을 만들어야 한다. swapfile은 성능 하락을 야기하여 리눅스 서버에서는 사용하지 않는다고 한다.
만드는 방법은 여기 https://wiki.jakduk.com/wiki/Linux#swapfile
