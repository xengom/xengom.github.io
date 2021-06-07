---
title: build.gradle 파헤치기 (build.gradle 수동설정)
category: Springboot
tags:
- gradle
- spirng
- springboot
toc_sticky: true
toc: true
toc_label: 목차
---

## 1. 왜 Initializr를 사용하지 않는가

![Spring Initializr](/assets/images/1/Initializr.PNG)

학원 수업에서도 Initializr를 사용하고 있으나 자잘한 오류 발생시, 스스로 오류를 해결하기 어려웠습니다. 결국 구글링을 통해 build.gradle 파일을 수정하는 식으로 대부분의 오류는 해결할 수 있었지만 처음부터 build.gradle 로 설정했으면 오류가 생기지 않았겠지만..

또, build.gradle 을 모르는 상태로 프로젝트 진행중에 새로 추가해야할 의존성이 생겼을 경우 Initializr만 사용한다면 처음부터 프로젝트를 다시 생성해야 한다.

따라서 Spring Boot와 Gradle에 익숙해질 때까지는 Initializr 사용을 지양하고 하나씩 추가하는 식으로 진행하겠다.




## 2. 그레이들 프로젝트 생성
### 기본 상태
우선 프로젝트 생성 후 build.gradle은 다음과 같이  자바 개발에 있어 가장 기초적인 설정만 되어있는 상태이다.
 ~~~java
plugins {
    id 'java'
}

group 'com.ch.board'
version '1.0-SNAPSHOT'
sourceCompatibility = 1.8
    
repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version:'4.12'
}

 ~~~

## 3.설정 추가

기본 build.gradle의 상단에 프로젝트의 플로그인 의존성 관리를 위한 설정을 추가합니다.(인텔리제이의 플러그인 관리가 아님)
### spring-boot-gradle-plugin
~~~java
buildscript {
    ext {
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}
~~~

+ ext : build.gradle에서 사용하는 전역변수 설정
  springBootVersion 라는 전역 변수를 생성하고 '2.1.7.RELEASE'라는 값을 삽입해 주었고, 이 변수를 통해 spring-boot-gradle-plugin라는 스프링 부트 그레이들 플러그인(2.1.7.RELEASE 버전)을 의존성 주입하겠다는 의미이다. 

### 필수 플러그인
위에서 선언한 플로그인 의존성들을 적용할 것인지를 결정하는 코드이다.
~~~java
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframwork.boot'
apply plugin: 'io.spring.dependency-management'
~~~

+ io.spring.dependency-management : 스프링 부트의 의존성들을 관리해주는 플러그인
  이 4 개의 플러그인들은 필수적이기 때문에 항상 추가하면 된다.

### 나머지
~~~java
repositories {
    mavenCentral()
    jcenter()
}
dependencies {
    compile('org.springframework.book:spring-boot-starter-web')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
~~~

+ repositories : 각종 의존성(라이브러리)를 어떤 원격 저장소에서 받을 것인지 설정

  두 저장소 모두 사용해보기

  + mavenCentral
    이전부터 많이 사용하는 저장소
    본인이 만든 저장소를 업로드하기 위해 많은 과정 및 설정을 요해 점차 공유가 잘 안된다.
  + jcenter
    mavenCentral의 문제점을 개선
    라이브러리 업로드가 간단
    jcenter에 업로드하면 mavenCentral에도 업로드되도록 자동화 가능

+ dependencies : 프로그램 개발에 필요한 의존선 선언
  Ctrl+Space를 누르면 자동완성이 가능하나 특정 버전 명시하면 안됨(맨위에 미리 버전을 전역 변수로 지정해주었기 때문) ->  버전 관리를 쉽게 하기 위함.

### 추가된 모든 코드

~~~java
buildscript {
    ext {
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group 'com.ch.board'
version '1.0-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
    mavenCentral()
}
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}

~~~

작성이 완료되면 우상단에 gradle 아이콘이 뜨므로 클릭하여 설정을 반영해준다.

![gradleload](/assets/images/1/GradleLoad.PNG)

설정이 전부 반영되었으면 우상단 gradle탭으로 들어가 의존성들이 잘 받아졌는지 확인

![Gradle tab](/assets/images/1/GradleTab.PNG)










이 글은 [스트링 부트와 AWS로 혼자 구형하는 웹 서비스]을 참조하여 작성되었습니다.
