---
title: Where절에서 집계함수 사용불가.
category: troubleShooting
tags:
- Oracle
- Oracle11g
- SQL
- SQLDeveloper
- SubQuery
---

## 문제발생. WHERE절의 집계함수

학원 서브쿼리 수업진행중 강사(라고 쓰고 틀딱 학생이라 읽은다)와 똑같이 쿼리문 진행했는데 ORA-00934 "group function is not allowed here"에러가 단체로 발생하는 참사...?(사실 참사도 아니다 매일같이 에러내고 강사가 못 고쳐서 학생들이 고치고 강사를 가르친다.)가 발생해서 찾아봤다..

강사가 작성한 코드는 다음과 같다.

~~~sql
select name as "이름", sum(price*amount) as "매출"
from buytbl
where sum(price*amount) >= (select sum(price*amount) 
                            from buytbl 
                            where name='한라산');
~~~

ORA-00934로 검색해보니 WHERE절에는 집계함수와 그룹함수를 사용할 수 없다고 한다. 아무 생각없이 따라할 때는 그러려니 했지만 곰곰히 생각해보니 서브쿼리문 때고 보면 가격*수량, 즉 매출을 기준으로 이름과 매출을 출력하라... 인데 말이 안된다. 사실상 매출의 총 합이 기준 이상이면 출력되고 아님 말고...인 상태이기 때문에 의도와는 다르다는거다.



## 해결. HAVING절 활용

우선 HAVING절을 활용한 코드는 다음과 같다

~~~sql
select name as "이름", sum(price*amount) as "매출"
from buytbl
group by name
having sum(price*amount) >= (select sum(price*amount) 
                            from buytbl 
                            where name='한라산');
~~~

name을 기준으로 그룹을 만들어주고 HAVING절을 이용해 매출을 비교하니 잘 출력된다.

![sqlresult](/assets/images/5/sql.PNG)
