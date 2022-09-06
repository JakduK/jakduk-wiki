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

    SUCCESS_WEB_HOOKS = [
        "your web hook url 1",
        "your web hook url 2",
        "..."
    ]

    FAILURE_WEB_HOOKS = [
        "your web hook url 1",
        "your web hook url 2",
        "..."
    ]

    pipeline {
        agent any

        stages {    
            stage("Send") {
                steps {
                    script {
                        sendForUpstreamBuilds(currentBuild)
                    }
                }
            }
        }
    }

    def sendForUpstreamBuilds(build) {
        for(cause in build.getBuildCauses()) {
            if (cause._class.contains("UpstreamCause")) {
                def result = getResult(cause)
                send(result, WEB_HOOKS)
                if (result.status == Result.SUCCESS) {
                    send(result, SUCCESS_WEB_HOOKS)
                } else if (result.status == Result.FAILURE) {
                    send(result, FAILURE_WEB_HOOKS)
                }
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
                return [
                    status:Result.FAILURE, 
                    title:"[${Result.FAILURE}] ${cause.upstreamProject} not found."
                ]
            } else if (!upstreamBuild) {
                def url = "${jenkins.rootUrl}${upstreamProject.url}"
                def title = "[${Result.FAILURE}] ${cause.upstreamProject} #${cause.upstreamBuild}"
                return [
                    status:Result.FAILURE, 
                    title:title, 
                    url:url, 
                    message:"Build not found."
                ]
            } else {
                def url = "${jenkins.rootUrl}${upstreamBuild.url}"
                def title = "[${upstreamBuild.result}] ${upstreamBuild.fullDisplayName}"
                def elapsed = "_${upstreamBuild.durationString} elapsed._"
                def startedBy = "${upstreamBuild.getCauses().collect {"_${it.shortDescription}_"}.join(",\n")}."
                return [
                    status: upstreamBuild.result,
                    title: title,
                    url: url,
                    message: [
                        elapsed,
                        startedBy,
                    ].findAll {it}.join("\n")
                ]
            }
        } else {
            return [
                status:Result.FAILURE, 
                title:"Jenkins service has not been started, or was already shut down, or we are running on an unrelated JVM, typically an agent."
            ]
        }
    }

    def send(result, webHooks) {
        def color = "\"color\": \"${getStatusColor(result.status)}\""
        def title = "\"title\": \"${escapeSpecialLetter(result.title)}\""
        def url = result.url ? "\"title_link\": \"${result.url}\"" : null
        def message = result.message ? "\"text\": \"${escapeSpecialLetter(result.message)}\"" : null
        for (webHook in webHooks) {
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
    def getStatusColor(status) {
        switch (status) {
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
