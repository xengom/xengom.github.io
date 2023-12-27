---
title: Spring Boot에서 Test Code 작성하기
toc_sticky: true
category: devKnowledge
tags:
- springboot
- testcode
- TDD
- JUnit
toc: true
toc_label: 목차
---

## 1. 실무에서 쓰는 테스트방법(기법)
### TDD
직접 해본적은 없지만 정보처리기사 실기를 공부할 때 애자일 방법론, 그중에서도 XP(eXtreme Programming)기법에서 TDD(Test-Driven-Development)는 기억에 남아있다.애초에 그 단원에서 서술형으로 나올만한 게 TDD정도였기 때문에..

TDD는 말 그대로 테스트가 주도하는 개발인데 그림으로 보는게 이해가 빠르다

![TDD](/assets/images/2/TDD.png)

레드 그린 사이클이라 하던데 그것까진 잘 모르겠고, 진행과정은 다음과 같다.
1. 실패하는 테스트를 먼저 작성
2. 테스트만 통과하는 코드를 작성
3. 테스트가 통과하면 코드를 리팩토링(동작을 바꾸지 않고 내부 구조만 개선)

맞는지는 모르겠지만 내 기준으로는 이렇게 들린다.
1. 일단 구현하고자하는 기능 코드를 구글링해서 긁어오고
2. 짜집기해서 합쳐서 아무튼 돌아가게만 만든다음
3. 잘 돌아가면 다듬어서 이쁘게 만든다



### 단위테스트
단위 테스트는 TDD의 1번을 의미한다.(기능 단위의 테스트 코드 작성)
TDD랑은 다르게 테스트 코드를 작성하는 것만을 의미하므로 공부 순서는 단위테스트 -> TDD 순이 되겠다.

## 2. 테스트 코드를 작성하는 이유

뭐 여러 이유가 있지만 간단하게 말하면 **문제를 빠른 단계에서 파악할 수 있게 도와준다**
테스트 코드가 없으면 코드 짤 때 
1. 코드 작성
2. 프로그램(서버)실행
3. postman으로 http요청
4. 요청 결과를 System.out.println()으로 검증
5. 결과가 다르면 다시 코드 수정

의 과정을 무한반복 해야하지만 테스트 코드를 사용하면 
+ 반복해서 서버를 올렸다 내렸다 할 필요도 없고 
+ System.out.println()으로 검증할 필요없이 자동으로 검증이 되며
+ 개발자가 만든 기능을 보호해준다.
  =>새 기능이 추가되었을 때 기존 기능에 문제가 생기는 경우를 예방해준다.

## 3. 테스트 코드 프레임워크

진짜 맨땅에서 헤딩하는 건 힘들기 때문에 테스트 코드 작성을 도와주는 프레임 워크가 존재
이 부분도 정보처리기사에서 열심히 외웠던 엑스피엔셀웨(xUnit STAF FitNessee NTAF Selenium Watir)등이 있지만 책에선 xUnit의 일종인 JUnit을 사용하기 때문에 JUnit으로 진행한다.

**대부분의 회사에서는 아직 JUnit4를 사용한다고 하니 JUnit4로 따라간다!**

## 4. 테스트할 src 코드 작성

### 메인 클래스 작성

![Mainclass](/assets/images/2/mainclass.PNG)

우선 처음 프로젝트를 생성할 때 설정한 [Group ID.프로젝트명]으로 패키지를 생성하고, [Application] 이라는 이름으로 Java 클래스를 생성해 다음과 같이 코드를 작성한다.

~~~java
package com.ch.board.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}
~~~

앞으로 Application.java는 이 프로젝트의 메인 클래스로 사용될 예정이다.

+ @SpringBootApplication
  + 다음을 자동으로 설정해줌
    + 스프링 부트
    + 스프링 Bean 읽기와 생성
  + 이 어노테이션이 있는 위치부터 설정을 읽어가기 때문에 항상 프로젝트의 최상단에 위치해야 함.

+ SpringApplication.run
  + 내장 WAS실행
    + 톰캣 설치 불필요
    + 반드시 사용하여야할 필요는 없지만 내장WAS사용시 배포가 쉽기 때문에 사용권장

### Controller 작성

메인 클래스를 담은 패키지 아래에 [web]패키지를 새로 생성하고, [HelloController] 클래스를 작성한다.

** [web]패키지는 앞으로 컨트롤러 관련 클래스를 담을 패키지로 사용된다.**

~~~java
package com.ch.board.BootBoard.web;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("hello")
    public String hello(){
        return "hello";
    }
}
~~~

+ @RestController
  + 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어줌
  + 메소드마다 @ResponseBody를 선언했던 것을 한번에 사용할 수 있게 해줌
+ @GetMapping
  + Get요청을 받을 수 있는 API 생성
  + RequestMapping(method=RequestMetho.GET)과 같음.


## 5. 테스트 코드 작성

![TDD](/assets/images/2/testclass.PNG)

HelloController의 패키지와 같은 패키지를 main이 아닌 test아래에 생성 후 HelloControllerTest 클래스를 작성한다.

~~~java
package com.ch.board.BootBoard.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception{
        String hello = "hello";
        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }
}

~~~

+ @RunWith(SpringRunner.class)
  + JUnit에 내장된 실행자 외의 실행자(SpringRunner라는 스프링 실행자)사용
  + 스프링 부트 테스트 와 JUnit 사이의 연결자 역할
+ @WebMvcTest
  + Web(Spring MVC)에 집중할 수 있는 어노테이션
  + @Controller, @ControllerAdvice 등 사용가능해짐
  + @Service, @Component, @Repository는 사용할 수 없다
+ @Autowired
  + 스프링이 관리하는 빈(Bean)주입
+ private MockMvc mvc
  + 웹 API 테스트할 떄 사용
  + HTTP GET, POST등에 대한 API 테스트가 가능
+ mvc.perform(get("/hello"))
  + MockMvc를 통해 /hello 주소로 HTTP GET 요청
  + 체이닝 지원! 검증기능 이어서 선언 가능
+ .andExpect()
  + mvc.perform 결과 검증
+ status().isOk()
  + HTTP Header의 Status 검증(200, 404, 500error같은 것)
  + 여기선 .isOk()이기 떄문에 200인지 아닌지 검증
+ content().string(hello)
  + 응답 본문의 내용 검증
  + Cotroller에서 hello를 리턴하기 때문에 이 값이 맞는지 검증



테스트에 통과하면 다음과 같이 Tests passed가 뜬다

![TDD](/assets/images/2/pass.PNG)

실제로도 잘 출력되는 것을 확인할 수 있다.

![TDD](/assets/images/2/result.PNG)





이 글은 [스트링 부트와 AWS로 혼자 구형하는 웹 서비스]을 참조하여 작성되었습니다.
