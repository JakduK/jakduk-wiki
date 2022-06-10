# 젠킨스 단독으로 telegram 알림 받기 설정

1. 플러그인 설치 : https://plugins.jenkins.io/groovy/
1. 알림 Job 생성
1. 알림 Job 설정  
    **Build after other projects are built** 체크 > **Projects to watch** 잡 등록 > **Trigger even if the build fails** 선택  
    **Add BuildStep** > **Execute system Groovy script**
    ```groovy
    import hudson.model.Result
    import jenkins.model.Jenkins

    BOT_API_TOKEN = ""
    CHAT_ID = ""

    for (cause in build.causes) {
        if (cause.class.name.equals('hudson.model.Cause$UpstreamCause')) {
            def upstreamBuild = cause.upstreamRun
            // upstream 빌드가 삭제되었으면 upstreamBuild == null
            if (upstreamBuild) {
                if (Jenkins.getInstanceOrNull()) {
                    def url = "${Jenkins.getInstanceOrNull().getRootUrl()}${upstreamBuild.url}"
                    def name = escapeSpecialLetter(upstreamBuild.fullDisplayName)
                    def result = upstreamBuild.result.equals(Result.SUCCESS) ? [marker: "🟢", message: "Build succeed."] : [marker: "🔴", message: "Build failed."]
                    send("[${result.marker} ${name}](${url})\n${escapeSpecialLetter(result.message)}\n${escapeSpecialLetter("${upstreamBuild.durationString} elapsed.")}")
                } else {
                    send("Jenkins service has not been started, or was already shut down, or we are running on an unrelated JVM, typically an agent.")
                }
            }
        }
    }

    def send(message) {
        def proc = [
            'curl', '-s', "https://api.telegram.org/bot${BOT_API_TOKEN}/sendMessage",
            '-H', 'content-type:application/json',
            '-d', "{\"chat_id\":\"${CHAT_ID}\", \"parse_mode\":\"MarkdownV2\", \"text\":\"${message}\"}"
        ].execute()
        def sout = new StringBuilder()
        proc.consumeProcessOutput(sout, sout)
        proc.waitForProcessOutput()
        println sout
    }

    def escapeSpecialLetter(str) {
        return str.replaceAll(/([#-.])/, '\\\\\\\\$1')
    }
    ```

# telegram chat_id 알아내기
https://web.telegram.org 접속해서 채팅방 선택후 현재 url에서 확인.
