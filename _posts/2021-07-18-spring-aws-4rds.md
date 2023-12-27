---
title: Spring 프로젝트 AWS 배포(4)RDS 접근
category: cloneCoding
tags:
- RDS
- properties
---

# RDS 접근하기

(3)에선 깃 프로젝트를 서버에 올리기만 한거고,

이젠 DB 접속을 해야된다..



## RDS 테이블 생성

우리가 테스트 코드 수행할 때 나오는 쿼리와

schema-mysql.sql파일에 있는 쿼리를 RDS 콘솔에서 실행해주면 된다.



인텔리제이를 켜고 테스트를 돌리면

![테스트쿼리](/assets/images/13/TESTCODE.PNG)

요런 쿼리가 있을 것이다. 요걸 고대로 복사해서 RDS콘솔에 넣고 실행하면 된다.

**RDS에 반영되는데 약간의 시간이 걸린다**



다음은 schema-mysql.sql인데

이건 **Ctrl+Shift+N**을 입력해서 찾으면 된다

단축키 외우기 귀찮으면 그냥 Shift를 두 번 따닥 눌러보자

단축키계의 마스터 키라 엔간한건 저걸로 다 된다.

![SCHEMA](/assets/images/13/SCHEMA.PNG)

내용은 다음과 같다.

~~~sql
CREATE TABLE SPRING_SESSION (
                                PRIMARY_ID CHAR(36) NOT NULL,
                                SESSION_ID CHAR(36) NOT NULL,
                                CREATION_TIME BIGINT NOT NULL,
                                LAST_ACCESS_TIME BIGINT NOT NULL,
                                MAX_INACTIVE_INTERVAL INT NOT NULL,
                                EXPIRY_TIME BIGINT NOT NULL,
                                PRINCIPAL_NAME VARCHAR(100),
                                CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;

CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);

CREATE TABLE SPRING_SESSION_ATTRIBUTES (
                                           SESSION_PRIMARY_ID CHAR(36) NOT NULL,
                                           ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
                                           ATTRIBUTE_BYTES BLOB NOT NULL,
                                           CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
                                           CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID) REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;
~~~



## 설정

### 프로젝트 설정

테이블은 이제 끝났고 다음은 프로젝트 설정이다.

build.gradle에

~~~java
compile("org.mariadb.jdbc:mariadb-java-client")
~~~

를 추가해주고, 서버에서 쓸 환경설정 을 추가한다.



~~~properties
# resources/application-real.properties
spring.profiles.include=oauth,real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc
~~~

별 내용 없어보인다면 그게 맞는거다.

DB를 공공재로 만들고 싶은 사람이 아니라면 DB 접속정보를 git에 올릴 순 없기 때문에

공개되도 상관 없는 부분만 넣어준다.



### EC2설정

OAuth도 그렇지만 DB 접속도 따로 보호해야하니 EC2에 설정파일을 따로 둔다.

아까와 같이 app 디렉토리에 application-real-db.properties 파일을 생성한다.

~~~sh
vim ~/app/application-real-db.properties
~~~

내용은 다음과 같다

~~~properties
spring.jpa.hibernate.ddl-auto=none
spring.datasource.url=jdbc:mariadb://[rds주소:포트번호]/[DB이름]
spring.datasource.username=[DB계정]
spring.datasource.password=[DB PW]
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
~~~

**참고로 ddl-auto=none으로 안하면 쉘스크립트 실행할때마다 테이블 다시 만들어 버리는 불상사가 발생할 수 있으므로 꼭 넣어주자**

그리고 요것도 deploy.sh의 nohup 부분에 추가해준다.

~~~sh
nohup java -jar \
-Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties,classpath:/application-real.properties \
	-Dspring.profiles.active=real \
	$REPOSITORY/$JAR_NAME 2>&1 &
~~~
