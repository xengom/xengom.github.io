---
title: MySQL5InnoDBDialect설정시 테스트 실패
toc: false
toc_sticky: false
toc_label: false
tags:
- SQL
- JPA
- Dialect
- Oracle
- JDBC
category: troubleShooting
---

~~~java
package com.xengom.domain.posts;

import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest
public class PostsRepositoryTest  {
    @Autowired
    PostsRepository postsRepository;

    @After
    public void cleanUp(){
        postsRepository.deleteAll();
    }

    @Test
    public void 게시글저장_불러오기(){
        //given
        String title = "테스트 게시글";
        String content = "테스트 본문";
        postsRepository.save(Posts.builder()
                .title(title)
                .content(content)
                .author("xengom@gmail.com")
                .build());

        //when
        List<Posts> postsList = postsRepository.findAll();

        //then
        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }

}

~~~

위와 같은 테스트 코드를 실행하는데 Dialect설정 시 제대로 SQL을 실행하지 못하는 에러가 생겼다.

~~~properties
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
~~~

dialect MySQL설정은 위와 같았다.





구글링해본 결과 깃헙댓글에 다음과 같이 설정해주라 한다.

~~~properties
jpa:
  show-sql: true
  properties:
    hibernate:
      dialect: org.hibernate.dialect.MySQL5InnoDBDialect
datasource:
  hikari:
    jdbc-url: jdbc:h2:mem://localhost/~/testdb;MODE=MYSQL
~~~



Oracle DB 적용하고 싶으면(Oracle11g기준)

```properties
jpa:
  show_sql: true
  properties:
    hibernate:
      dialect: org.hibernate.dialect.Oracle10gDialect
datasource:
  hikari:
    jdbc-url: jdbc:oracle:thin:@localhost:1521/xe
    driver-class-name: oracle.jdbc.driver.OracleDriver
    username: DBID
    password: DBPW
```

출처 : https://github.com/jojoldu/freelec-springboot2-webservice/issues/67
