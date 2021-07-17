---
title: Spring 프로젝트 AWS 배포 ~ Travis CI 자동 배포가이드
category: AWS
tags:
- EC2
- putty
- puttygen
- ppk
- pem
---

# 들어가기 전에

개인적으로 계속 써먹을 것 같기에...

배운거 고대로 정리하는 글입니다.

기본적으로 로컬환경에서 OAuth 구현이 되어있다는 전제하에 진행됩니다.



# EC2

## EC2 생성 및 기본 설정

![EC2들어가기](/assets/images/10/AWS-EC2.PNG)

위에 서비스를 눌러서 EC2로 들어가거나 검색창에 EC2를 검색하면 나온다.



![인스턴스 시작](/assets/images/10/EC2-Instans.PNG)

인스턴스 시작을 눌러 인스턴스 생성

이 뒤는 프리티어라 선택이 제한되어 있기 때문에 그런 부분은 생략한다.



![AMI 선택](/assets/images/10/AMI.PNG)

AMI는 가장 위에 있는 걸로.. 괜히 맨 위에 있는게 아닐거다.



![스토리지 선택](/assets/images/10/storage.PNG)

밑에 잘 보면 최대 30GB까지 가능하다고 한다.

돈과 용량은 다다익선이니 [크기]부분에 30을 넣어준다.



![태그 추가](/assets/images/10/tag.PNG)

내 인스턴스 구별할 수 있는 Name 태그를 추가해준다.



![보안그룹 선택](/assets/images/10/secu.PNG)

보안그룹 = 방화벽.

이미 보안그룹이 있으면 그거 써도 되고 새로 만들 꺼면 위와 같이 설정한다.

참고로 SSH(22포트)는 터미널 접속할 때 쓰는 전용 창구 같은 건데, 

뒤에서 발급받는 pem키가 없으면 접속이 안된다. (그래서 pem키 잊어버리면 접속을 못한다.)

그게 귀찮다고 내IP로 제한하는걸 빼먹고 0.0.0.0/0, ::/0으로 전체 오픈해버리면 

내 AWS 인스턴스는 전 세계 공용 비트코인 채굴 서버가 되어버리니 꼭 설정하자.



![키 선택](/assets/images/10/key.PNG)

쓰던게 있으면 넣어도 되고, 없으면 새로 생성하고 다운받는다.

위에서도 언급했지만 잊어먹으면....GG..

그렇다고 "절대로 안잊어먹게 프로젝트에 넣어서 Github에 올려놔야지 히힛"하면 안된다.



![인스턴스 완성](/assets/images/10/instanscom.PNG)

인스턴스는 완성됐다.

하지만 이 상태로는 인스턴스 껐다 킬때 IP를 새로 발급받게 되니 

매번 IP확인하러 AWS를 켜야하는 상황이 생긴다.

그런 귀찮은 걸 방지하기 위해 탄력적 IP를 할당받자.



![EIP](/assets/images/10/eip.PNG)

왼쪽에 보면 탄력적IP라는 메뉴가 있다.

여기는 프리티어 유저라면 선택지가 하나밖에 없이니 생성 화면은 생략-



![발급받은 EIP](/assets/images/10/eipresult.PNG)

그림에는 지워놔서 안보이는데 여기서 할당된 IPv4주소를 가지고 인스턴스로 돌아가서..



![EIP설정](/assets/images/10/EIPSET.PNG)

요렇게 탄력적 IP주소를 연결하면 된다.

물론 난 이미 만들었기 때문에 해제 메뉴가 보인다.





## EC2 접속

### Putty 설정

접속은 터미널로하는데 보통은 [putty](https://www.putty.org/)를 사용한다.

#### puttygen

설치하고 나면 먼저 Puttygen을 실행한다.

![푸티젠](/assets/images/10/puttygen.PNG)

Conversions -> Import Key에서 아까 EC2만들 때 다운받았던 pem 키를 넣고

![푸티젠2](/assets/images/10/puttygen2.PNG)

Save Private Key 를 클릭하면 ppk 키가 툭 튀어나온다.

putty에선 pem키 대신 ppk를 써서 접속하니 ppk도 안전한 곳에 고이 모셔두도록 하자




#### putty

![푸티](/assets/images/10/putty.PNG)

Host Name은 username@public_ip를 입력하는데 아까 무지성 따라하기로 EC2 생성했으면

username이 ec2-user로 되어있을 것이므로

ec2-user@탄력적IP를 넣어주면 된다.

![푸티set](/assets/images/10/puttyset.PNG)

왼쪽 메뉴에서 Connection - SSH - Auth로 들어가서 Browse클릭 후 

아까 puttygen 에서 만든 ppk를 쑤셔넣은 다음

![푸티save](/assets/images/10/puttysave.PNG)

Session으로 돌아가서 이름 넣고 Save버튼 눌러주면 다음부턴 이딴 설정 안건드려도 

뚝딱 접속 가능하다.

![ec2 접속](/assets/images/10/ec2access.PNG)

접속 잘된다.





## EC2 내부 설정

이제 EC2라는 건물(EC2 인스턴스)도 지었고, 

들어가는 입구(putty)도 만들어 놨으니,

인테리어 작업할 시간이다.

EC2라는 건물에 입주예정이신 스프링부트의 취향을 고려하여 가구를 넣어주자.



### JAVA설치

당연한 소리겠지만 자바를 돌릴려면 자바를 깔아줘야한다.

본인 프로젝트의 자바버전에 맞게 깔아주면 된다.

난 자바 11로 해놔서 자바11을 깔았는데 8쓰는 사람은 8로 깔면된다.



#### JAVA 8

~~~sh
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
~~~



#### JAVA 11

yum에서 설치가능한 JDK는 1.8까지밖에 없기 때문에 JDK11을 깔고 싶으면

Amazon에서 제공하는 Amazon Coretto를 써서 깔아야된다.

~~~sh
# aws coreetto 다운로드
sudo curl -L https://corretto.aws/downloads/latest/amazon-corretto-11-x64-linux-jdk.rpm -o jdk11.rpm

# jdk11 설치
sudo yum localinstall jdk11.rpm
~~~



#### JAVA 설치 후 마무리

설치가 되면 JAVA 버전을 선택해 준다.

~~~sh
sudo /usr/sbin/alternatives --config java
~~~

![java선택](/assets/images/10/javasel.PNG)

원하는 번호 입력하고 엔터누르면 된다.



~~~sh
java --version
~~~

![javaver](/assets/images/10/javaver.PNG)

위 코드로 잘 적용 되었는지 확인해보고



JAVA 11깐 사람이라면

~~~sh
rm -rf jdk11.rpm
~~~

를 입력해서 설치키트를 지워주자

안지워도 상관없는데 지우는게 깔끔하니까..



### 타임존 변경

~~~sh
sudo rm /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
~~~

위 명령어로 시간을 서울로 설정해주고



~~~sh
date
~~~

![date](/assets/images/10/date.PNG)

date를 입력해서 제대로 바뀌었는지 확인해본다.



### 호스트네임 변경

위에 스샷을 보면 알겠지만 난 이미 hostname이 설정되어 있어서

사용자 이름이 [ec2-user@springprac]으로 되어있는 것을 볼 수 있다.

원래는 요상한 ip가 들어가 있기때문에 인스턴스 하나까진 괜찮지만 두 개 세 개 되고나면

내가 대체 무슨 인스턴스에서 작업하는지 모르게되니 hostname을 설정해준다.



~~~sh
sudo vim /etc/sysconfig/network
~~~

우선 설정 파일로 들어간다.

여기선 vim썼는데 vi를 쓰던 nano를 쓰던 아무거나 쓰면된다.

개인적으로 젤 쉬운건 nano인듯..한데 죄다 vim을 쓰니 그냥 vim을 쓰겠다.

![호스트이름 설정](/assets/images/10/hostnameset.PNG)

요로코롬 설정해주고 



~~~sh 
sudo reboot
~~~

으로 재부팅하면....된다고 책에 써있었는데 안되네?

[AWS User Guide](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/set-hostname.html)를 보면 한가지 빼먹었다.

~~~sh
sudo hostnamectl set-hostname [설정할 호스트 이름]
~~~

이것까지 넣어주고 reboot하면 제대로 hostname이 변경된다.



마지막으로 

~~~sh
sudo vim /etc/hosts
~~~

![hosts](/assets/images/10/hosts.PNG)

에다가 hostname을 등록시켜주면 금상첨화





# 참고 자료

[책 저자 github](https://github.com/jojoldu/freelec-springboot2-webservice)

[AWS EC2에 JDK 11 설치](https://pompitzz.github.io/blog/Java/awsEc2InstallJDK11.html#jdk-%E1%84%89%E1%85%A5%E1%86%AF%E1%84%8E%E1%85%B5)
