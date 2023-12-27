---
title: AWS RDS 사용 시, DB Navigator에러
toc: false
tags:
- mariaDB
- MySQL
- DB
- AWS
- RDS
- EC2
category: troubleShooting
---

책에서 하라는 대로 다 설정했는데도 DB Navigator에서 Connection error를 발생시켰다.

 ![Connection error](/assets/images/8/error.PNG)

딱 [이 사람](https://github.com/jojoldu/freelec-springboot2-webservice/issues/687)이랑 똑같은 경우..



한 두어시간 RDS 처음부터 다시 만들고, 파라미터 그룹 다시 만들고, 심지어 EC2까지 다시 만들고 별 지랄을 다해봤는데, 결국은 실패...

이쯤되면 DB Navigator에 문제 있는거 아닌가 싶어서 그냥 기존에 쓰던 인텔리제이 Database 접속기로 접속하니 접속이 된다...

 ![RDS Success](/assets/images/8/rds.PNG)



~~~sql
use [db이름]; #잘된다!

select @@time_zone, now(); #잘된다!

show variables like 'c%'; #잘된다!
~~~

여기까진 순조로웠다

아니, 일단 결과는 잘 나왔으니... 순조롭다고 생각했다..



문제는 실제로 테이블 만들어서 데이터 넣을 때 발생

~~~sql
CREATE TABLE test (
    id bigint(20) not null auto_increment,
    content varchar(255) default null,
    primary key (id)
)ENGINE=InnoDB;

insert into test(content) values ('테스트');

~~~

당연히 CREATE까지는 잘 됬는데 insert에서 1366, incorrect string value에러가 발생

구글링 해보니 한글 입력 에러란다..



혹시나해서 variables를 다시 보니

 ![parameter](/assets/images/8/parameter.PNG)

사진은 수정 후라  3행의 character_set_database가 utf8mb4로 제대로 되있는데 원래는 latin1로 되어있었다.

![parameter group](/assets/images/8/parametergroup.PNG)

분명 RDS상의 파라메터 그룹에는 제대로 설정되있고, 

Putty를 통해 RDS접속해서 조회해봐도 utf8mb4로 되어있었다.

빡치긴하지만... 구글링해서 수정..

~~~sql
ALTER DATABASE pleasedb CHARACTER SET = 'utf8mb4' COLLATE = 'utf8mb4_general_ci';
~~~

알아서 charater set은 utf8mb4로, collation부분은 utf8mb4_general_ci로 일괄 수정해준다.



근데..... 이래도 똑같은 에러가 또난다...

또 구글링해보니 뭐 /my.cnf파일을 수정하느니 뭐하니 하는데 귀찮아서 더 뒤적거리다가 답을 찾았다.

~~~sql
alter table test convert to charset utf8mb4;
~~~

테이블 자체를 utf8mb4로 바꿔버리기



![result](/assets/images/8/result.PNG)

어후... RDS 빡세다..







### 참조

+ [variable수정](https://velog.io/@jimin_lee/MySQL%EC%97%90%EC%84%9C-%ED%95%9C%EA%B8%80-%EC%9D%B8%EC%BD%94%EB%94%A9-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0/)
+ [테이블 charset 설정](https://redapply.tistory.com/entry/mysql-%ED%95%9C%EA%B8%80%EC%9E%85%EB%A0%A5%EC%97%90%EB%9F%AC-ERROR-1366-HY000-incorrect-string-value)
