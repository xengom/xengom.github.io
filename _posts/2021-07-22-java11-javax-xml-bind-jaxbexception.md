---
toc: false
category: JPA
tags:
- xml
- JPA
- maven
- JAXB
title: Java11버전 사용시 javax/xml/bind/JAXBException 에러
---

![에러](/assets/images/15/error.PNG)

JPA실습 시 초장부터 에러 발생

JAXB는 XML파일을 Java Object 형식으로 바꿔주는 역할을 하는데

Java11버전부터는 내장되어있지 않기 때문에 따로 추가해줘야 함

~~~xml
<dependency> 
    <groupId>javax.xml.bind</groupId> 
    <artifactId>jaxb-api</artifactId> 
    <version>2.3.0</version> 
</dependency>
~~~



물론 그래들 사용할땐 이런 에러 없었다..



[참고](https://yoonemong.tistory.com/254)
