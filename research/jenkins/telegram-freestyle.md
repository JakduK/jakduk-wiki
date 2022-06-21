# 젠킨스 단독으로 Telegram 알림 받기 설정

1. 플러그인 설치 : https://plugins.jenkins.io/groovy/
1. 알림 Job 생성 (FreeStyle project)
1. 알림 Job 설정  
    **Build after other projects are built** 체크 > **Projects to watch** 잡 등록 > **Trigger even if the build fails** 선택  
    **Add BuildStep** > **Execute system Groovy script**
    ```groovy
   import hudson.model.Result
   import jenkins.model.Jenkins

   BOT_API_TOKEN = "bot api token"
   CHAT_ID = "chat id"

   sendForUpstreamBuilds(build)

   def sendForUpstreamBuilds(build) {
       for (cause in build.causes) {
           if (cause.class.name.equals('hudson.model.Cause$UpstreamCause')) {
               def jenkins = Jenkins.getInstanceOrNull()
               if (jenkins) {
                   // upstream 프로젝트가 삭제되었으면 upstreamProject == null
                   def upstreamProject = jenkins.getItemByFullName(cause.upstreamProject)
                   // upstream 빌드가 삭제되었으면 upstreamBuild == null
                   def upstreamBuild = upstreamProject?.getBuildByNumber(cause.upstreamBuild)
                   if (!upstreamProject) {
                       send(escapeSpecialLetter("${cause.upstreamProject} not found."))
                   } else if (!upstreamBuild) {
                       def url = "${jenkins.rootUrl}${upstreamProject.url}"
                       def title = escapeSpecialLetter("${cause.upstreamProject} #${cause.upstreamBuild}")
                       def message = escapeSpecialLetter(" not found.")
                       send("[${title}](${url})${message}")
                   } else {
                       def url = "${jenkins.rootUrl}${upstreamBuild.url}"
                       def title = escapeSpecialLetter(upstreamBuild.fullDisplayName)
                       def marker = getMarker(upstreamBuild.result)
                       def message = escapeSpecialLetter("Build ${upstreamBuild.result.toString().toLowerCase()}.")
                       def elapsed = escapeSpecialLetter("`${upstreamBuild.durationString} elapsed.`")
                       def startedBy = escapeSpecialLetter("${upstreamBuild.getCauses().collect {"`${it.shortDescription}`"}.join(",\n")}.")
                       send([
                           "[${marker} ${title}](${url})",
                           message,
                           elapsed,
                           startedBy
                       ].join("\n"))
                   }
               } else {
                   send(escapeSpecialLetter("Jenkins service has not been started, or was already shut down, or we are running on an unrelated JVM, typically an agent."))
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
       if (!str) {
           return str;
       }
       return str.replaceAll(/([#-.])/, '\\\\\\\\$1').replaceAll(/(["])/, '\\\\$1')
   }

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

## Scripts not permitted to use method xxx.xxx.xxx xxx. Administrators can decide whether to approve or reject this signature.
젠킨스 관리 > In-process Script Approval > Approve 클릭 > Signatures already approved: 목록에 추가.
