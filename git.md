<!-- TITLE: Git -->
<!-- SUBTITLE: Git -->

# 꿀팁
## 브랜치 이름 패턴으로 일괄 삭제
* 패턴 포맷은 shell `wildcard` 또는 `glob 패턴`
	* https://en.wikipedia.org/wiki/Glob_(programming) glob pattern
```sh
git branch --list <parttern> | xargs git branch -d
```
* 예제) `hotfix/` 전체 삭제
```sh
git branch --list hotfix/* | xargs git branch -d
```