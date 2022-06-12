
```groovy
import jenkins.model.Jenkins
import hudson.model.Result

UPSTREAM_JOBS = [
    "job 1",
    "job n",
]

BOT_API_TOKEN = "bot api token"
CHAT_ID = "chat id"

node {
    properties([
        pipelineTriggers([
            upstream(upstreamProjects: UPSTREAM_JOBS.join(','), threshold: Result.FAILURE)
        ])
    ])
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
            def url = "${jenkins.rootUrl}${upstreamBuild.url}"
            def title = escapeSpecialLetter(upstreamBuild.fullDisplayName)
            def marker = getMarker(upstreamBuild.result)
            def message = escapeSpecialLetter("Build ${upstreamBuild.result.toString().toLowerCase()}.")
            def elapsed = escapeSpecialLetter("`${upstreamBuild.durationString} elapsed.`")
            return [
                "[${marker} ${title}](${url})",
                message,
                elapsed
            ].join("\n")
        }
    } else {
        return "Jenkins service has not been started, or was already shut down, or we are running on an unrelated JVM, typically an agent."
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
