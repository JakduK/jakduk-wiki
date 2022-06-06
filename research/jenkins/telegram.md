# 젠킨스 단독으로 telegram 알림 받기 설정

1. 플러그인 설치 : https://plugins.jenkins.io/groovy/
1. Job 생성
1. Job 설정
    1. 알림 받을 Job 등록 : Build after other projects are built > Projects to watch (Trigger only if build is stable)
    1. 빌드 작업 추가 : Add BuildStep > Execute system Groovy script


```groovy
import hudson.model.Result

def BOT_API_TOKEN = "my bot api token"
def CHAT_ID = "chat room id"

for (cause in build.causes) {
    if (cause.class.name.equals('hudson.model.Cause$UpstreamCause')) {
        def upstreamBuild = cause.upstreamRun
        def name = upstreamBuild.fullDisplayName.replaceAll('-', '\\\\\\\\-').replaceAll('#', '\\\\\\\\#')
        def url = upstreamBuild.absoluteUrl
        def result = upstreamBuild.result.equals(Result.SUCCESS) ? '빌드 성공' : '빌드 실패'
        def proc = [
            'curl', '-s', "https://api.telegram.org/bot${BOT_API_TOKEN}/sendMessage",
            '-H', 'content-type:application/json',
            '-d', "{\"chat_id\":\"${CHAT_ID}\", \"parse_mode\":\"MarkdownV2\", \"text\":\"[${name}](${url})\n*${result}*\"}"
        ].execute()
        def sout = new StringBuilder()
        proc.consumeProcessOutput(sout, sout)
        proc.waitForProcessOutput()
        println sout
        break
    }
}
```

# telegram chat_id 알아내기
https://web.telegram.org 접속해서 채팅방 선택후 현재 url에서 확인.
