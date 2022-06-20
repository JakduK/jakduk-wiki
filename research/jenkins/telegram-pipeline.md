# 젠킨스 단독으로 telegram 알림 받기 설정

1. 알림 Job 생성 (Pipeline)
2. 알림 Job 설정 
    - **Build after other projects are built** 체크 > **Projects to watch** 잡 등록 > **Trigger even if the build fails** 선택
    - **Pipeline** > **Pipeline Script**.
        - `BOT_API_TOKEN`, `CHAT_ID` 입력.
        - **Use Groovy Sandbox** 체크 해제.
    ```groovy
    import jenkins.model.Jenkins
    import hudson.model.Result

    BOT_API_TOKEN = "bot api token"
    CHAT_ID = "chat id"

    node {
        stage("Send") {
            script {
                sendForUpstreamBuilds(currentBuild)
            }
        }
    }

    def sendForUpstreamBuilds(build) {
        for(cause in build.getBuildCauses()) {
            if (cause._class.contains("UpstreamCause")) {
                def result = getResult(cause)
                send(result)
            }
        }
    }

    @NonCPS
    def getResult(cause) {
        def jenkins = Jenkins.getInstanceOrNull()
        if (jenkins) {
            def upstreamProject = jenkins.getItemByFullName(cause.upstreamProject)
            def upstreamBuild = upstreamProject?.getBuildByNumber(cause.upstreamBuild)
            if (!upstreamProject) {
                return escapeSpecialLetter("${cause.upstreamProject} not found.")
            } else if (!upstreamBuild) {
                def url = "${jenkins.rootUrl}${upstreamProject.url}"
                def title = escapeSpecialLetter("${cause.upstreamProject} #${cause.upstreamBuild}")
                return "[${title}](${url})" + escapeSpecialLetter(" not found.")
            } else {
                def trigger = upstreamBuild.getCause(UserIdCause)
                def url = "${jenkins.rootUrl}${upstreamBuild.url}"
                def title = escapeSpecialLetter(upstreamBuild.fullDisplayName)
                def marker = getMarker(upstreamBuild.result)
                def message = escapeSpecialLetter("Build ${upstreamBuild.result.toString().toLowerCase()}.")
                def startedBy = escapeSpecialLetter(trigger ? "${trigger.getShortDescription()}." : null)
                def elapsed = escapeSpecialLetter("`${upstreamBuild.durationString} elapsed.`")
                return [
                    "[${marker} ${title}](${url})",
                    message,
                    startedBy,
                    elapsed,
                ].findAll {it != null}.join("\n")
            }
        } else {
            return escapeSpecialLetter("Jenkins service has not been started, or was already shut down, or we are running on an unrelated JVM, typically an agent.")
        }
    }

    def send(message) {
        sh "curl -s -X POST -H 'content-type: application/json' -d '{\"chat_id\":\"${CHAT_ID}\",\"parse_mode\":\"MarkdownV2\",\"text\":\"${message}\"}' 'https://api.telegram.org/bot${BOT_API_TOKEN}/sendMessage'"
    }

    @NonCPS
    def escapeSpecialLetter(str) {
        return str.replaceAll(/([#-.])/, '\\\\\\\\$1')
    }

    @NonCPS
    def getMarker(result) {
        switch (result) {
            case Result.SUCCESS: return "🟢"
            case Result.FAILURE: return "🔴"
            default: return "🟡"
        }
    }
    ```
    
# 트러블 슈팅
## telegram chat_id 알아내기
https://web.telegram.org 접속해서 채팅방 선택후 현재 url에서 확인.
