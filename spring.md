<!-- TITLE: Spring -->
<!-- SUBTITLE: Spring Framework -->

스프링부트 공식 문서의 Logging 파트를 보면
* https://docs.spring.io/spring-boot/docs/1.5.14.RELEASE/reference/htmlsingle/#howto-configure-logback-for-logging
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <include resource="org/springframework/boot/logging/logback/base.xml"/>
   <logger name="org.springframework.web" level="DEBUG"/>
</configuration>
```

이런 Logback 샘플이 있고, base.xml 를 include 하고 있다. 이에 대한 공식 문서에서의 간략한 설명은 다음과 같다.

> Spring Boot also provides some nice ANSI colour terminal output on a console (but not in a log file) using a custom Logback converter. See the default base.xml configuration for details.

ANSI 컬러로 멋지게 콘솔로 출력해준다고 한다. 자세히 설정을 보고 싶으면 base.xml을 보라고 한다.

그래서 실제 base.xml 내용이 뭔지 궁금해서 찾아봤다.
* https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/base.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
  Base logback configuration provided for compatibility with Spring Boot 1.1
-->

<included>
	<include resource="org/springframework/boot/logging/logback/defaults.xml" />
	<property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
	<include resource="org/springframework/boot/logging/logback/console-appender.xml" />
	<include resource="org/springframework/boot/logging/logback/file-appender.xml" />
	<root level="INFO">
		<appender-ref ref="CONSOLE" />
		<appender-ref ref="FILE" />
	</root>
</included>
```

먼저 눈에 들어오는건 root는 INFO 레벨로 로그를 찍고, CONSOLE과 FILE appender를 사용함을 알 수 있다. 
또한  console-appender.xml, file-appender.xml 을 또 include 하고 있다.
* https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml

내용을 보자.
* https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml defaults.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Default logback configuration provided for import, equivalent to the programmatic
initialization performed by Boot
-->

<included>
	<conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
	<conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
	<conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
	<property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
	<property name="FILE_LOG_PATTERN" value="${FILE_LOG_PATTERN:-%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

	<logger name="org.apache.catalina.startup.DigesterFactory" level="ERROR"/>
	<logger name="org.apache.catalina.util.LifecycleBase" level="ERROR"/>
	<logger name="org.apache.coyote.http11.Http11NioProtocol" level="WARN"/>
	<logger name="org.apache.sshd.common.util.SecurityUtils" level="WARN"/>
	<logger name="org.apache.tomcat.util.net.NioSelectorPool" level="WARN"/>
	<logger name="org.eclipse.jetty.util.component.AbstractLifeCycle" level="ERROR"/>
	<logger name="org.hibernate.validator.internal.util.Version" level="WARN"/>
</included>
```

defaults.xml엔 콘솔 컬러 설정과 패턴설정 등이 있다.

console-appender.xml을 보면
* https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/console-appender.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Console appender logback configuration provided for import, equivalent to the programmatic
initialization performed by Boot
-->

<included>
	<appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>${CONSOLE_LOG_PATTERN}</pattern>
		</encoder>
	</appender>
</included>
```

ch.qos.logback.core.ConsoleAppender 클래스로 CONSOLE라는 이름의 appender를 추가하고, ${CONSOLE_LOG_PATTERN} 패턴으로 로그를 출력하는데, ${CONSOLE_LOG_PATTERN}는 defaults.xml에 정의되어 있다.

file-appender.xml의 내용을 보자.
* https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/file-appender.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
File appender logback configuration provided for import, equivalent to the programmatic
initialization performed by Boot
-->

<included>
	<appender name="FILE"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<encoder>
			<pattern>${FILE_LOG_PATTERN}</pattern>
		</encoder>
		<file>${LOG_FILE}</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz</fileNamePattern>
			<maxFileSize>${LOG_FILE_MAX_SIZE:-10MB}</maxFileSize>
			<maxHistory>${LOG_FILE_MAX_HISTORY:-0}</maxHistory>
		</rollingPolicy>
	</appender>
</included>
```

ch.qos.logback.core.rolling.RollingFileAppender 클래스로 FILE란 이름의 appender를 추가하고, ${FILE_LOG_PATTERN} 패턴으로 로그를 출력하는데, ${FILE_LOG_PATTERN}는 defaults.xml에 정의되어 있다. 그 이외에 rotate 정책이 설정되어 있다.

자 이제 종합해보면 base.xml 는 컬러풀한 CONSOLE 로그와 rotate 가능한 spring.log 란 이름의 로그 파일을 생성 해준다.
