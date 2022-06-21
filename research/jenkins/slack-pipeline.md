# 젠킨스 단독으로 Slack 알림 받기 설정

1. 알림 Job 생성 (Pipeline)
2. 알림 Job 설정 
    - **Build after other projects are built** 체크 > **Projects to watch** 잡 등록 > **Trigger even if the build fails** 선택
    - **Pipeline** > **Pipeline Script**.
        - `WEB_HOOKS` 입력.
        - **Use Groovy Sandbox** 체크 해제.
    ```groovy
    import jenkins.model.Jenkins
    import hudson.model.Result

    WEB_HOOKS = [
        "your web hook url 1",
        "your web hook url 2",
        "..."
    ]

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
                return [color:Result.FAILURE, title:"${cause.upstreamProject} not found."]
            } else if (!upstreamBuild) {
                def url = "${jenkins.rootUrl}${upstreamProject.url}"
                def title = "${cause.upstreamProject} #${cause.upstreamBuild}"
                return [color:Result.FAILURE, title:title, url:url, message:"Build not found."]
            } else {
                def url = "${jenkins.rootUrl}${upstreamBuild.url}"
                def title = upstreamBuild.fullDisplayName
                def message = "Build ${upstreamBuild.result.toString().toLowerCase()}."
                def elapsed = "_${upstreamBuild.durationString} elapsed._"
                def startedBy = "${upstreamBuild.getCauses().collect {"_${escapeSpecialLetter(it.shortDescription)}_"}.join(",\n")}."
                return [
                    color: upstreamBuild.result,
                    title: title,
                    url: url,
                    message: [
                        message,
                        elapsed,
                        startedBy,
                    ].findAll {it}.join("\n")
                ]
            }
        } else {
            return [color:Result.FAILURE, title:"Jenkins service has not been started, or was already shut down, or we are running on an unrelated JVM, typically an agent."]
        }
    }

    def send(Map map = [:]) {
        def color = "\"color\": \"${getResultColor(map.color)}\""
        def title = "\"title\": \"${map.title}\""
        def url = map.url ? "\"title_link\": \"${map.url}\"" : null
        def message = map.message ? "\"text\": \"${map.message}\"" : null
        for (webHook in WEB_HOOKS) {
            sh """
    curl ${webHook} \
    -s \
    -X POST \
    -H 'content-type: application/json' \
    -d '{
        \"attachments\": [
            {
                ${
                    [
                        color,
                        title,
                        url,
                        message
                    ].findAll {it}.join(',')
                }
            }
        ]
    }'
    """
        }
    }

    @NonCPS
    def getResultColor(result) {
        switch (result) {
            case Result.SUCCESS: return "#2eb886"
            case Result.FAILURE: return "#dc3545"
            default: return "#ffc107"
        }
    }

    @NonCPS
    def escapeSpecialLetter(str) {
        return str.replaceAll(/(["])/, '\\\\$1')
    }
    ```
