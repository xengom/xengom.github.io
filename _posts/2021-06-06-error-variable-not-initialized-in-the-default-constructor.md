---
title: 'error: variable not initialized in the default constructor'
toc: false
tags:
- springboot
- testcode
- spirng
- lombok
category: Springboot
---

HelloResponseDto.java
~~~java
package com.ch.board.BootBoard.web.dto;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public class HelloResponseDto {
    private final String name;
    private final int amount;
}
~~~


HelloResponseDtoTest.java
~~~java
package com.ch.board.BootBoard.web.dto;

import org.junit.Test;
import static org.assertj.core.api.Assertions.assertThat;

public class HelloResponseDtoTest {
    @Test
    public void 롬복_기능_테스트(){
        //given
        String name = "test";
        int amount = 1000;

        //when
        HelloResponseDto dto = new HelloResponseDto(name,amount);

        //then
        assertThat(dto.getName()).isEqualTo(name);
        assertThat(dto.getAmount()).isEqualTo(amount);
    }
}
~~~

책을 보며 공부하다가 책에 나온 코드 그대로 테스트 코드 진행 시  
variable name not initialized in the default constructor
variable amount not initialized in the default constructor
두 에러가 떴다.

책 지은이의 깃헙 Issue를 찾아보니 책의 코드는 Gradle4를 기준으로 설정되어 있으므로 Gradle버전을 낮추라 한다.
[깃헙](https://github.com/jojoldu/freelec-springboot2-webservice/issues/2)

근데 굳이 gradle버전을 낮추기보단 현재 쓰고있는 gradle에 맞게 설정을 바꾸는게 맞지 않으려나 생각해서 좀 더 구글링 한 결과
[henry.hong 블로그](https://tube-life.tistory.com/14)

능력자 분의 블로그 포스팅을 보고 해결법을 찾았다.
~~~java
//기존
compile('org.projectlombok:lombok')
//추가
testCompile "org.projectlombok:lombok"
annotationProcessor('org.projectlombok:lombok')
testAnnotationProcessor('org.projectlombok:lombok')
~~~

![결과](/assets/images/SpringBoot/3/testresult.PNG)

보아하니 따로 testCompile용 lombok 의존성을 넣어줘야 하나보다..
