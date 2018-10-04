<!-- TITLE: macOS -->
<!-- SUBTITLE: macOS -->

# 사파리 플러그인 완전 삭제
## Mojave
모하비부터 플러그인을 앱으로 취급
## High Sierra
https://www.lifewire.com/how-to-view-and-remove-safari-plug-ins-2260895 /Library/Internet Plug-Ins 디렉토리에서 삭제

# Windows 키배열 키보드 사용하기
* 시스템 환결설정 > 키보드 > 보조 키 > Option/Command 키 상호변경.
* Windows와 Alt 키캡을 뽑아 자리를 바꾼다.

# brew
* https://docs.brew.sh/Formula-Cookbook#homebrew-terminology 용어 설명
* https://github.com/Homebrew/homebrew-cask-versions cask설치시 하위 버전 패키지 지원

# Terminal
## 단축키
* Control + u : Remove a current line
* Control + w : Remove a word
* Control + k : Delete from cursor to the end of the line
* Control + a : Move cursor to the beginning of the line
* Control + e : Move cursor to the end of the line

## 커맨드 팁
* 현재 패스 복사 : pwd|pbcopy
* cd stack : cd -, cd --, cd --- ....

# Mojave
## 메뉴바만 다크모드 적용하기
다크모드 UI가 눈아픈 나같은 사람은 하이 시에라처럼 메뉴바만 다크모드를 적용해보자.
* https://www.tekrevue.com/tip/only-dark-menu-bar-dock-mojave/
* Aqua스타일 UI 항상 켬. 메뉴바만 화면 모드에 맞게 바뀜.
```
defaults write -g NSRequiresAquaSystemAppearance -bool Yes
```
* 위 설정 삭제
```
defaults delete -g NSRequiresAquaSystemAppearance
```