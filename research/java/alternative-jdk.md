<!-- TITLE: Alternative JDK -->
<!-- SUBTITLE: -->

# Oracle JDK 대안
* https://github.com/AdoptOpenJDK Adopt OpenJDK
* http://jdk.java.net Oracle OpenJDK
* https://www.azul.com Zulu OpenJDK

# macOS에서 brew로 설치하기
/Library/Java/JavaVirtualMachines 밑에 설치되기 때문에 사용하는데 지장없음
* Adopt OpenJDK
```sh
brew tap AdoptOpenJDK/openjdk

# 설치 버전 선택
# adoptopenjdk-openjdk8
# adoptopenjdk-openjdk9
# adoptopenjdk-openjdk10
brew install <version>
```

* Zulu OpenJDK

```sh
# 현재 Java10 설치됨
brew cask install zulu

# 하위 버전 설치 방법
brew tap homebrew/cask-versions
brew cask install zulu8
```
