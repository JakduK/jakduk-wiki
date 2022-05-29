<!-- TITLE: VirtualBox -->
<!-- SUBTITLE: VirtualBox, CLI -->

# CLI 활용하기
자주 사용 할 커맨드

```text

# VM 목록
VBoxManager list vms

# VM 정보. MAC주소 및 하드웨어 스펙을 볼 수 있음.
VBoxManager showvminfo <vm name|uuid>

# VM headless 시작
VBoxManager startvm centos --type headless

# VM 상태 전환. reset은 뭔지 모르겠음
VBoxmanager controlvm pause|resume|reset|poweroff|savestate

```
