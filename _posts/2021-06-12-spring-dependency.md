---
title: 스프링 의존관계 설정
category: Spring
tags:
- spirng
- dependency
- DI
---

스프링 의존관계를 설정하는 방법은 크게 두 가지가 있다.
## 1. 컴포넌트 스캔과 자동 의존관계

@Component 어노테이션이 있으면 스프링 빈으로 자동 등록된다.

주로 @Repository @Service @Controller를 많이 쓰는데 각각 자세히 살펴보면 @Component를 포함하고 있는 어노테이션이라 자동으로 등록되는 것이다.

생성자에 @Autowired를 사용해 객체 생성 시점에 스프링 컨테이너에서 해당 스프링 빈을 찾아서 주입하는 방식이라 매우 간단하다.(어노테이션 하나만으로 의존성 주입(DI)가 끝나버리니까..)

참고로 스프링은 스프링 빈을 등록할 때, 싱글톤이 디폴트라고 하더라..



## 2. 자바 코드로 직접 스프링 빈 등록

@Service @Repository @Autowired를 제거하고 (Controller의 어노테이션은 제거하지 않는다.) 빈을 설정할 별도의 클래스 파일을 생성한다.

~~~java
@Configuration
public class SpringConfig{
    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }
    
    @Bean
    public MemberRepository memberRepository(){
        return new MemoryMemberRepository();
    }
}
~~~

물론 어노테이션 한줄 넣고 끝나는 것 보다 복잡하지만 다 일장일단이 있다고 한다...



## 3. 참고

1. XML
   XML로 설정하는 방식도 있지만 최근엔 사용하지 않는다고 한다.

2. DI(의존성 주입)

   + 필드주입 : 필드에다 @Autowired를 박아버린다.
     
		 예시 : @Autowired private MemberService memberService;
     중간에 설정을 바꾸거나 하기 어려워 거의 사용하지 않는다

   + setter주입 : setter를 통해 주입한다. (setter에 @Autowired)
   
     setter가 public으로 열려 있어야한다..
     사실 한번 조립되면 (동적으로 )바꿀일이 거의 없기 때문에 public하게 열어둘 필요가 없다.

   + 생성자주입 : 가장 권장하는 방식으로, lombok 사용 시 @RequiredArgsConstructor 어노테이션을 사용하면 final이 선언된 모든 필드를 인자값으로 하는 생성자를 만들어줌.
    직접 생성자를 만들지 않고 어노테이션을 쓰는 이유는 클래스의 의존 관계가 변경될 때 마다 생성자 수정하는 수고를 덜기 위함이다.
    
3. 따라서 정형화된 Cotroller, Service, Repository는 컴포넌트 스캔방식(어노테이션)을 사용하고,
   정형화되지 않거나 상황에 따라 구현 클래스를 변경해야하면 스프링 빈을 직접 등록한다.



출처 : [인프런 스프링 입문](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8)
