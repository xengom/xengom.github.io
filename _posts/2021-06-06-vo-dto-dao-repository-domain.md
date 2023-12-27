---
title: VO vs DTO, DAO vs Repository, 그리고 Domain과 프로젝트 구조
category: devKnowledge
tags:
- spring
- DTO
- DAO
- VO
- Repository
- DDD
- Project
---

## 1. VO, DTO, DAO, Domain?
Java MVC패턴 공부할때 가장 많이 나오는 용어들인데 그냥 관성적으로 쓰기만하고 실질적으로 뭐가뭔지 이해가 잘 안가서 정리한다.
###  VO vs DTO
거의 같은 의미로 사용되는 경우가 많은 것 같고, 현재 다니는 학원에서도 그냥 같은거라고 퉁치고 넘어가는 듯하다.

하지만 인프런에 스프링 강의를 올리시는 김** 개발자님은

단순히 데이터 값을 전달하기 위한 용도로 사용되는 경우 = DTO

도메인 주도 설계에서 이야기하는 갮 객체(Value Object)의 경우 = VO

로 사용하신다 하시길래...한번 찾아봤다.
#### VO
데이터 그 자체로 의미 있는 것을 담는 객체이다.

DTO와 거의 동일하나 **Read-Only!!**라는 듯..?
#### DTO
전송되는 데이터의 컨테이너이다.

DTO는 같은 시스템내 보다는 **다른 시스템으로 데이터를 전달하는 작업을 처리하는 객체**(ex. DB에 데이터전송)





###  DAO vs Repository
사실 이 부분은 DDD(Domain Driven Design)에 대한 공부가 좀 더 필요한 듯하다... 추후 추가 포스팅할 예정..

공부하고 있는 책에 따르면 ibatis나 MyBatis에서는 DAO로 DB Layer에 접근하고, JPA에서는 Repository로 접근한다고 한다.
#### DAO
영구저장소(Oracle, MySQL,....etc...)에 접근할 때 특정 API사용시 저장소를 바꿀 때 API에 따라 구현을 변경해야 하기 때문에 API와 로직 사이에서 어댑터같은 역할을 해줄 것이 필요로 해 생긴 것이 DAO.

즉 DAO는 110V 돼지코랑 같은 거다.(아마도)

![돼지코](/assets/images/4/pignose.PNG)

		
		
		
#### Repository
원칙적으로 객체의 상태를 **관리**하는 저장소이다. 즉, Entity 그 자체를 저장하고 불러오는 역할이다.
여기서 Repository는 영구저장소라는 의미가 아니라 **관리**하는 저장소이기 떄문에 객체(Entity)에 대한 CRUD를 수행할 수 있으면 된다.

따라서 일반적으로는 Repository의 인터페이스를 도메인 로직에 넣어둔다.

![돼지코](/assets/images/4/repository.PNG)

뭣도 모르고 학원 수업진행할 때 따라만든 Repository이다.


### Domain
소프트웨어로 해결하고자 하는 문제의 영역이다.

예를 들어 쇼핑몰을 만든다 치면 쇼핑몰 그 자체가 도메인이 된다.

도메인은 또 하위 도메인으로 나뉠 수 있다.(ex. 상품, 회원, 주문, 배송 etc..)

이 부분은 추후 좀 더 공부해서 정리하는 걸로...

## 2. 프로젝트 구현시 폴더 구조
프로젝트 구조는 정형화되어있다기보다 프로젝트를 진행할 때마다 상황과 규모에 따라, 유지보수/확장이 쉬운 구조를 사용한다.

프로젝트의 요구사항이 추가되면서 기능이 증가하면 Controller, DTO, Entity, Service 등으로 패키지를 분할하는 방식.

### 간단한 구조(프로젝트 초기)

> + Project
> 	+ Controller
> 	+ DTO
> 	+ Entity
> 	+ Service

### 도메인 단위로 분할
위 프로젝트가 더 커짐에 따라 여러 도메인이 추가되면 도메인 단위로 상위 패키지 개념을 추가한다.
ex) 단순 게시판에 회원 기능 추가

> + Project
> 	+ BBS
> 		+ Controller
> 		+ DTO
> 		+ Entity
> 		+ Service
> 	+ member
> 		+ Controller
> 		+ DTO
> 		+ Entity
> 		+ Service

### 기능별 분할
하지만 위와 같이 구성시, BBS의 Controller에서 회원 관련 서비스를 자주 사용하게 되고, 엔티티도 연관관계가 생기면 다음과 같이 변경된다.

> + Project
> 	+ controller
> 		+ BBS
> 		+ member
> 	+ service
> 		+ BBS
> 		+ member
> 	+ entity
> 		+ BBS
> 		+ member

즉 프로젝트가 성장함에 따라 프로젝트 구조도 상황에 맞춰 성장시킬 수 있어야 한다.


### 프로젝트에서 DTO의 위치
특정 컨트롤러만 사용하는 DTO면 해당 컨트롤러와 한 패키지 안에 생성하고, 여러 클래스가 공유하는 경우에는 외부에 별도의 패키지를 생성한다.














+ 참고
	+ [Repository와 DAO차이점](https://bperhaps.tistory.com/entry/Repository%EC%99%80-Dao%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90)
	+ [인프런 프로젝트 폴더 구조 질문](https://www.inflearn.com/questions/16046)
	+ [인프런 domain과 repository 질문](https://www.inflearn.com/questions/111159)
