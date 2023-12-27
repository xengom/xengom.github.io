---
title: Spring Boot 구동 시, ClassLoader Error 발생
category: troubleShooting
tags:
- spirng
- springboot
- classLoader
---

# 에러 로그
```
java.lang.IllegalStateException : Failed to introspect Class [org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration] from ClassLoader
...
Caused by : java.lang.NoClassDefFoundError : javax/servlet/DispatcherType
...
Caused by : java.lang.ClassNotFoundException : javax.servlet.DispatcherType
``` 

# 원인
Maven pom 에서는 tomcat의 scope가 provided로 설정되어 있으나,  Eclipse와 다르게 IntelliJ의 Run Configuration는 provided 태그를 읽어들이는 설정을 별도로 해야한다.
이 경우 해당 설정을 제대로 해놓지 않아 tomcat을 잡아주지 못해 생기는 문제로 보인다. 

# 해결 방안
* scope 제거
	굳이 scope가 필요하지 않은 경우 (내장tomcat으로 돌려도 상관 없는 경우) 제거하고 돌리면 정상구동한다.

	
```
<!-- AS-IS -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-tomcat</artifactId>
	<scope>privided</scope>
</dependency>

<!-- TO-BE -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-tomcat</artifactId>
</dependency>
```

* Run Configuration 에 Add Dependencies with "provided" scope to classpath 옵션 추가
	Run/Debug Configurations > 해당  Configuration > Modify options > ‘ADD DEPENDENCIES WITH "PROVIDED" SCOPE TO CLASSPATH’ 옵션 추가 > maven reload > run
	상기 과정을 통해 구동하면 정상구동한다.


# 참조
https://stackoverflow.com/questions/53714699/failed-to-introspect-class-org-springframework-security-config-annotation-web-c
> We were seeing this locally because when running Spring Boot application from IntelliJ we didn't have the option 'Include dependencies with "Provided" scope' in the Run/Debug Configuration ticked.
