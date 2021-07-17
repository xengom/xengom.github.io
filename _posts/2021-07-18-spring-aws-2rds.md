---
category: AWS
tags:
- EC2
- RDS
- MySQL
- MariaDB
title: Spring 프로젝트 AWS 배포(2)RDS설정
---

# RDS

EC2가 끝났으면 RDS설정이다.

졸업작품 프로젝트할 때는 무지성으로 EC2안에다가 MySQL깔아서 그냥 그 안에서만 해결했는데

Amazon에서 따로 지원해주는 DB  서비스가 있으므로 그걸 활용한다.



## RDS 생성

![RDS생성](/assets/images/11/RDSC.PNG)

먼저, EC2랑 똑같이 RDS를 검색해서 들어간다.



![RDS선택](/assets/images/11/RDSSEL.PNG)

데이터베이스 생성을 클릭하고 MariaDB를 선택한다.

Amazon Aurora가 클라우드에 가장 적합해서 많은 회사가 사용하는데

오라클이나 MSSQL은 호환이 안된다(즉 나중에 Aurora쓰고싶어도 쓰기 힘들다)고 하니 

나중에 호환을 생각해서 MySQL, MariaDB를 고른다.

밑에 버전도 나중에 파라미터 설정할때 필요하니 기억해두도록 하자.



참고로 JPA쓴다면 Oracle이나 MySQL로 공부했을텐데 ID자동 증가 값 설정할때

MySQL은 IDENTITY, Oracle은 SEQUENCE를 쓰기 때문에 서로 바꾸기 힘들다..

난 Oracle로 배웠지만 지금 공부하는 책은 MySQL이라서.. 

방언변경만으로 SQL바꾸는게 안되서 첨에 삽질많이했다..

만약 다른 사람들과 협업중이라면 SEQUENCE와 IDENTITY를 모두 지원하는 PostgreSQL도 고려해볼만 하다.



아, 그리고 밑에 템플릿 디폴트 값이 프로덕션으로 되어있는데 꼭 프리 티어를 선택하도록 하자.



![RDS설정](/assets/images/11/RDSSET.PNG)

DB의 이름과 아이디, 비밀번호 설정하는 곳이다.

본인이 원하는 이름, 아이디, 비밀번호를 넣으면 된다.



![RDS연결](/assets/images/11/RDSCON.PNG)

연결에선 퍼블릭 엑세스만 [예]로 바꿔주면 된다.

디폴트값이 아니오라서 꼭 바꿔줘야한다.

어처피 나중에 보안그룹 설정할 때 지정된 IP만 접근할 수 있도록 막을 거다.



![RDS추가](/assets/images/11/RDSADD.PNG)

추가 구성은 DB이름만 지정해주면 된다. 

헷갈리니 웬만하면 아까 위에서 설정한 DB 이름하고 맞춰준다.

마지막으로 생성을 누르면 RDS생성은 끝난다.





## RDS 설정

### 파라미터 그룹

#### 파라미터 그룹 생성

본인이 쓸 변수를 설정하는 곳 이다. 

![PG](/assets/images/11/PG.PNG)

왼쪽에서 파라미터 그룹을 선택하고, 생성 클릭



![PGC](/assets/images/11/PGC.PNG)

아까 RDS생성할 때 버전이랑 맞춰줘야한다.



#### 파라미터 편집

그룹을 만들었으니 안에 들어갈 파라미터를 편집해줘야한다.

![PGM](/assets/images/11/PGM.PNG)

파라미터 밑에 검색창에 파라미터 이름을 적으면 파라미터를 찾을 수 있다.

수정 버튼을 눌러서 수정하자.

수정할 파라미터는 다음과 같다

+ time_zone
  값을 Asia/Seoul로 수정
+ chracter_set
  character_set으로 시작하는 파라미터 전부 utf8mb4 로 값 수정
+ collation
  collation으로 시작하는 파라미터 전부 utf8mb4_general_ci 로 값 수정

참고로 utf8mb4는 uft8에 이모지까지 저장할 수 있는 거라고 한다.



#### RDS DB에 파라미터 그룹설정

RDS의 데이터베이스 메뉴로 돌아가서 아까 생성한 데이터베이스를 수정한다. 

![PGADD](/assets/images/11/PGADD.PNG)

파라미터 그룹을 방금 생성한 파리미터 그룹으로 지정해주면 된다.

만약 제대로 반영이 안되면 데이터베이스를 재부팅하면 된다.



### RDS-EC2연동

#### 보안그룹 설정

아까 EC2에서 보안그룹을 생성했듯, 여기서도 보안그룹을 만들어 줘야한다.

정확히는 아까 EC2 보안그룹을 불러오는거지만..

우선 RDS창은 그대로 두고 새 창으로 EC2로 돌아가서 보안그룹으로 들어간다.



![SG](/assets/images/11/SG.PNG)

EC2의 보안 그룹 아이디를 복사해서



![RDSSEC](/assets/images/11/RDSSEC.PNG)

RDS의 보안 그룹 설정으로 들어간 다음



![RDSSECIB](/assets/images/11/RDSSECIB.PNG)

인바운드 규칙을 수정한다.

유형은 둘 다 MYSQL/Aurora로

첫 번째 줄은 본인 IP, 두 번째 줄은 아까 복사한 EC2 보안 그룹 아이디를 넣어주면 된다.

이렇게 하면 RDS-개인PC, RDS-EC2 연동설정이 끝난거다.



#### 로컬환경 테스트

책에서는 Intelij의 Database Navigator를 사용하지만

6시간 내내 눈물의 똥꼬쇼를 해도 잘 연결이 안됐기 때문에... 난 그냥 Intelij 오른쪽에 있는 Database탭으로 연결했다.

자세히는 [여기](https://xengom.com/aws/aws-rds-db-navigator/)를 참조하면 된다.
