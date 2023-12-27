---
title: ORA-00903 Invalid table name / Error executing DDL
toc: false
tags:
- Oracle
- SQL
- ORA-00903
- DDL
category: troubleShooting
---

스프링 연관관계를 통해 게시판에 쓰일 5 개의 테이블을 생성하는데 Comment와 View, 2 개의 테이블만이 생성이 제대로 되지 않았다.



 **↓ View Entity**

```java
@NoArgsConstructor
@AllArgsConstructor
public class View {

    @Id
    @GeneratedValue(generator = "viewSeq")
    @Column(name = "viewNum")
    private Long num;

    private String title;

    @Lob
    private String content;

    private int like;

    private int unlike;

    private int activation;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "MemberNum")
    private Member member;

    @OneToMany(mappedBy = "view", orphanRemoval = true)
    private List<Comment> comments;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "BoardNum")
    private Board board;

    @OneToMany(mappedBy = "view", orphanRemoval = true, cascade = CascadeType.ALL)
    private List<Upload> uploads;

}
```

Comment와 View는 연관관계가 있기 때문에 구조가 단순한 Comment보단 복잡한 View에서 에러가 발생할 것이라 생각해서 구글링한 결과 원인은 다음과 같았다.

1. like는 SQL 예약어
2. view도 SQL 예약어

사실 기사 공부하면서 둘 다 오질라게 써먹긴 했는데 막상 실제로 코딩하니 다 까먹고.... 설마 이것 때문이겠어 했는데 진짜 이것 때문이라 허탈하다..
