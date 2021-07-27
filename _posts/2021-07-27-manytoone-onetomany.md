---
title: 연관관계 매핑(ManyToOne, OneToMany)
category: JPA
tags:
- OneToMany
- ManyToOne
- 연관관계
- mappedBy
- Mapping
- 양방향
- 단방향
---

국비학원 JPA깔짝이며 가르칠 때 배웠으나 너무 대충 가르친다고 찌르니까 강사가 사실 JPA 하나도 모른다고 실토 후, 그냥 MyBatis로 강의진행하고 수료했기 때문에, 따로 공부하면서 정리한다.

웹 개발할 때, 연관관계, 뭐 더 쉽게 얘기하면 외래 키 설정할 때 은근 이해하기 힘들다.

특히 가르치는 사람이 본인도 모르는 걸 가르친다면 말이다.

다시 생각해도 빡치네



## 외래키란

아무튼 쉽게 정리하면 외래키는 두 개의 테이블은 이어주는 역할을 하는데

구체적으로는 소속이 적힌 이름표 같은거라 보면 된다.

왜 그런거 있지 않나.

훈련소 때 이름표에 몇 대대 몇 중대 몇 소대 몇 번 누구누구라 적혀있는 거

외래키가 바로 그 이름표라 생각하면 이해하기 쉽다 (적어도 나는 그렇다)

즉 A라는 테이블이 B의 키를 외래키로 들고 있다면 A는 B에 속한 자식 테이블이다.



### 예시

Team 과 Member로 예를 들어보자.

Team 에는 Member가 소속된다.

즉 Member테이블에 써있는 멤버들은 Team의 키를 외래키로 가지고 있어야한다.

여기서 다루고자 하는 내용은 이 외래키를 매핑하는 방법이다.



#### 방법1. 객체를 테이블에 맞추는 방법

엔티티로 표현하면 다음과 같다.

~~~ java
@Entity
 public class Member {
 @Id @GeneratedValue
 @Column(name = "MEMBER_ID")
 private Long id;
 @Column(name = "USERNAME")
 private String name;
 @Column(name = "TEAM_ID")
 private Long teamId;
 …
 } 
~~~



~~~java
@Entity
 public class Team {
 @Id @GeneratedValue
 @Column(name = "TEAM_ID")
 private Long id;
 @Column(name = "TEAMNAME")
 private String name;
 …
 }
~~~

Java보단 DB에 맞춘 방법이다.

이 방법을 사용하면 DB에 가깝게 설계할 수 는 있으나 이런 식으로 코딩하면 JPA를 쓸 이유가 없다.

실제로 위 엔티티를 가지고 데이터를 저장하려면 다음과 같이 해야한다.

~~~java
//팀 저장
 Team team = new Team();
 team.setName("TeamA");
 em.persist(team);
 //회원 저장
 Member member = new Member();
 member.setName("member1");
 member.setTeamId(team.getId());
 em.persist(member);
~~~

팀 따로 멤버 따로.. 매우 매우 번거롭다.

객체 지향적인 Java를 쓰는데 DB에서만큼은 객체지향적이지 못하다.

여기서 나온 것이 객체 연관관계다.



#### 방법2. 객체 연관관계 사용

객체 연관관계는 단방향과 양방향이 있다.

##### 단방향 연관관계

~~~java
@Entity
 public class Member {
 @Id @GeneratedValue
 private Long id;
 @Column(name = "USERNAME")
 private String name;
 private int age;
// @Column(name = "TEAM_ID")
// private Long teamId;
 @ManyToOne
 @JoinColumn(name = "TEAM_ID")
 private Team team;
 …
~~~

@ManyToOne과 @JoinColumn 어노테이션을 사용해서 매핑해주면 된다.

이 경우 데이터를 저장할 때 다음과 같이 참조 저장할 수 있게 된다.

~~~java
 //팀 저장
 Team team = new Team();
 team.setName("TeamA");
 em.persist(team);
 //회원 저장
 Member member = new Member();
 member.setName("member1");
 member.setTeam(team); //단방향 연관관계 설정, 참조 저장
 em.persist(member);
~~~

즉 Member에서 언제든 Team을 꺼내쓸 수 있다는 말이다.

드디어 훈련병한테 너 소속어디야 물어봤을 때 대답할 수 있게되었다.



##### 양방향 연관관계

![연관관계 차이](/assets/images/16/ex.PNG)

엄밀히 말하면 양방향은 양방향이라기보단 단방향으로 서로서로 연결된다는 느낌이다.

당연하게도 Team 입장에서 소속된 Member는 여러 명 이므로 컬렉션에 추가한다.

~~~java
@Entity
 public class Team {
 @Id @GeneratedValue
 private Long id;
 private String name;
 @OneToMany(mappedBy = "team")
 List<Member> members = new ArrayList<Member>();
 …
 }
~~~

여기서 특이점은 mappedBy이다.

왜 member->Team 단방향에서는 mappedBy가 없었는데 여기선 생겼을까?



## mappedBy

mappedBy를 쓰는 이유는 간단하다.

어떤 테이블을 주로 쓸꺼냐를 정해주는거다.

먼저 객체와 테이블에서 관계를 맺는 차이는 다음과 같다.

+ 객체 연관관계 (단뱡향 연관관계 2 개)
  + Member -> Team 연관관계(단방향)
  + Team -> Member 연관관계(단방향)
+ 테이블 연관관계(양방향 연관관계 1 개)
  + Member <-> Team 연관관계(양방향)

위에서도 언급했듯이 객체 연관관계에서의 양방향 연관관계는 서로 단방향 연관관계를 설정해서 양방향으로 한 것에 불과하다.

당연하게도 양방향 관계를 만들어주기 위해 단방향 관계를 2 개 만들어줘야 한다는 거다.

반대로 테이블에서는 외래키 하나로 두 테이블 연관관계를 모두 관리할 수 있다.

즉, 객체 연관관계에서의 양방향 연관관계는 어느쪽을 연관관계의 Owner로 할 것인가를 정해줘야 한다는 것이다.



### 연관관계 Owner

두 단방향 연관관계 중에 Owner를 정해주고, Owner만이 외래 키를 관리(등록, 수정)할 수 있게 해야한다.

이 주인을 정해주는 것이 mappedBy 어노테이션이다.

물론 주인은 테이블 연관관계랑 비슷하게 실제 외래키를 들고 있는 MEMBER쪽으로 설정하는 것이 좋다. 

그래야 안헷갈리니까...

즉, mappedBy어노테이션은 Team에 넣어주면 된다.



출처

[자바ORM표준JPA프로그래밍](https://www.inflearn.com/course/ORM-JPA-Basic)
