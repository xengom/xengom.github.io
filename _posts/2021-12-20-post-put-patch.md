---
title: POST, PUT, PATCH의 차이
category: HTTP
tags:
- post
- put
- patch
- http
- restful
- restfulapi
---

내가 학원에서 배운 웹은 거의 무조건 GET과 POST로만 실습을 진행했었다.
물론 수업 후반부에 가서야 "Restful API(GET, POST, PUT, DELETE)로 CRUD를 해야되요."라는 언급정도는 해주었지만 써보진 않았기에 대체 왜 Restful를 써야하는지, 어떤 역할을 하는지 뭣도 모른 상태로 수료해버렸고... 막연한 기억만 가지고 진행하기엔 너무 지식이 미천했기 때문에 따로 조사를 해봤다.

## 메서드의 종류
먼저 메서드는 크게 5가지가 있다.(물론 더 있긴 하지만 잘 쓰진 않는다 카더라)
1. GET
2. POST
3. PUT
4. PATCH
5. DELETE

+ GET
	먼저 GET은 다들 알다시피 조회(Read)하는데 주로 쓰는 메서드이다.
물론 조회말고 GET으로 요청도 보낼 수 있지만 쿼리를 통해 데이터를 전달하기 때문에 주소창에 어떤 데이터를 어떻게 넘겼는지 적나라하게 드러나는 단점이 있다. (ex. https://www.google.com/search?q=Restful) 
POST처럼 메세지 바디를 통해서 데이터 전달할 수도 있다곤 하지만 지원하지 않는 곳이 더 많다고하니... 얌전히 POST를 쓰는 것이 낫다.

+ POST
	POST는 메시지 바디에 데이터를 담아 보낸다. 
주로 새로운 리소스를 생성(등록)하는데 사용하고, 그 외에도 애매하면 우선 POST라고 생각하면 된다.

+ PUT
	학원에서는 수정(update) = PUT이다. 라고 가르쳤지만 (다시 생각해도 정말 야매강사였다.) 정확히는 update라기보단 replace라고 한다. 즉 아예 대체를 해버린다는 것이다.
	예를 들어 '이름', '학번', '주소' 가 있는 데이터에서 새로운 '주소'만 지정해서 PUT을 보내면, 주소는 변경되지만 이름/학번은 NULL이나 Default로 지정해놓은 값이 되어버린다.

+ PATCH
	이게 가장 수정에 가까운 메서드이다. 변경된 부분만 반영해준다.

+ DELETE
	이름만 봐도 알다시피 삭제 메서드이다.

여기까지 공부하고나니 한 가지 의문이 생겼다. 대체 업데이트할 때는 무슨 메서드를 써야 할까? 단순히 저 다섯 가지만 본다면 당연히 PATCH지! 라고 할 수도 있지만 대부분의 예제에서는 POST를 사용하는 경우가 많다. 무난하기도 하지만, 사실상 PUT이 아예 리소스를 Replace해버리기 때문에 어쩔 수 없이 POST를 사용하는 듯 했다. 대체 이 3 가지의 차이가 무엇일까?

## POST, PUT, PATCH의 차이
결론부터 말하자면 POST와 PATCH는 Idempotent(멱등)하지 않고, PUT은 Idempotent한 메서드이다.
이게 뭔 개소리냐 싶겠지만 잘 보면 이해가 간다. 
**Idempotent는 몇 번을 실행하더라도 같은 값이 나온다**는 의미인데, POST나 PUT은 일부를 수정할 수 있는 만큼 값이 변할 가능성이 있지만 PUT은 몇 번을 실행하더라도 아예 데이터를 대체해버리기 때문에 멱등한 것이다.

더 쉽게 설명하면 다음과 같다.
+ POST
	1. 식별자(Identifier) 없이 Data가 들어오면 새 식별자를 생성하고 Data를 등록한다.
	2. 식별자 없는 Data가 **또** 들어오면  새 식별자를 **또** 생성하고 Data를 등록한다.
	
	예를 들면 아래와 같이 식별자(id)가 없이 데이터를 POST로 **두 번** 보내면
	```json
	POST /member
	{
		"name": "세종대왕",
		"class" : "조선"
	}
	```
	아래와 같은 결과가 나온다.
	```json
	HTTP/1.1 200 OK
	{ “id”: 1, “name”: “세종대왕”, “class”: "조선" }

	HTTP/1.1 200 OK
	{ “id”: 2, “name”: “세종대왕”, “class”: "조선" }
	```
	즉, 식별자만 다른 같은 데이터가 두 개 생성된다.
	
+ PUT
	PUT은 Idempotent하기 때문에 몇 번을 보내도 같은 결과만이 남는다.
	1. Data가 들어왔을 때 식별자가 존재하면 해당 리소스를 업데이트(Replace)한다.
	2. Data가 들어왔을 때 식별자가 존재하지 않으면 새 식별자를 생성하고 Data를 등록한다.
	
	```json
	PUT /member
	{
		"name": "조지워싱턴",
		"class" : "미합중국"
	}
	```
	즉, 기존에 같은 Data가 있던 말던, 몇 번을 보내던, PUT으로 보내기만 하면
  
	```json
	HTTP/1.1 200 OK
	{ “id”: 3, “name”: “조지워싱턴”, “class”: "미합중국" }
	```
	항상 위와 같은 값만 나올 것이다.(이게 PUT은 Idempotent하다는 뜻이다.)
	
+ PATCH
	PATCH는 POST와 PUT을 섞은 것 같은 느낌이다.
	1. Data가 들어왔을 때 식별자가 존재하면 해당 리소스를 업데이트(부분 update)한다.
	2. Data가 들어왔을 때 식별자가 존재하지 않으면 **Exception**이 터진다.

따라서 리소스의 "수정"면에서는 PATCH 메서드를 이용하는 것이 가장 바람직해보인다.



참조 : [https://stackoverflow.com/questions/31089221/what-is-the-difference-between-put-post-and-patch]