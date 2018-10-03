<!-- TITLE: Node.js -->
<!-- SUBTITLE: Node.js, JavaScript -->

# 사용법
## PM2
* max_restarts, min_uptime 옵션 이해하기
* https://stackoverflow.com/a/49666928
max_restarts 옵션은 최대 재시작 횟수를 지정하는 옵션이 아님.
min_uptime 옵션에 의존하는 옵션으로,
min_uptime 시간 동안 max_restarts 횟수만큼 실패하면 status : errored 되는 것.

# 버그 추적
## Node.js
* Date.toLocaleString() : 파라미터로 로케일을 받는데 포맷팅 안됨.
* https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Date/toLocaleString

```text
> console.log(new Date().toLocaleString('en-GB', { timeZone: 'UTC' }));
9/23/2018, 1:07:07 PM
undefined
> console.log(new Date().toLocaleString('ko-KR', { timeZone: 'UTC' }));
2018-9-23 13:07:16
undefined
> console.log(new Date().toLocaleString('ar-EG', { timeZone: 'UTC' }));
2018-9-23 13:07:40
undefined
> console.log(new Date().toLocaleString('ja-JP', { timeZone: 'UTC' }));
2018-9-23 13:07:48
undefined
> process.version
'v10.9.0'
> 
> 
```

## PM2
* wait_ready 옵션 : 옵션을 켜면 애플리케이션 구동 실패해도 status : online 을 표시함.
  * https://github.com/Unitech/pm2/issues/3147 오래된 심각한 이슈인데 대응을 안함