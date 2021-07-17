---
category: AWS
tags:
- EC2
- Shell
- Deploy
title: Spring 프로젝트 AWS 배포(3)EC2서버에 배포하기
---

# 배포

여태까지 배포를 위한 환경설정이였고 이제 본격적인 배포다.

설정에서 엄청난 시간을 쓴거 같다..



## 프로젝트 Colne하기

먼저 EC2에 깃헙 코드를 받아오기 위해 git을 설치해준다

putty를 키고 EC2를 접속해 커맨드를 입력한다.

~~~sh
sudo yum install git
~~~



설치가 끝나면 

~~~sh
git --version
~~~

으로 제대로 설치가 되었는지 확인해보자.



~~~sh
mkdir ~/app && mkdir ~/app/step1
~~~

깃 프로젝트를 저장할 디렉토리를 만들어주고



~~~sh
cd ~/app/step1
~~~

그 디렉토리로 들어가서 깃을 클론해주면 된다.



![깃헙clone](/assets/images/12/GIT.PNG)

본인 프로젝트 깃 주소를 복사해서

~~~sh
git clone [복사한 깃 주소]
~~~

로 진행하면된다.



확인은

~~~sh
# clone 확인
cd 프로젝트명
ll

# 코드 정상 실행 확인(test)
./gradle test
~~~

로 할 수 있다. 



아마 ./gradle test를 실행하면

~~~sh
-bash: ./gradlew: Permission denied
~~~

라는 에러가 뜰 텐데 권한이 없다는 뜻이므로

~~~sh
chmod +x ./gradlew
~~~

로 실행 권한을 주면된다.



이것 외의 다른 에러가 뜬다면 테스트를 수정하고 깃에 다시 푸시한다음

~~~sh
git pull
~~~

을 통해 다시 가져오면된다.





## 배포 스크립트

배포는 

+ git을 clone(또는 pull)하고

+ gradle로 프로젝트 테스트&빌드

+ EC2서버에서 프로젝트 실행/재실행

의 단계로 이뤄지는데

이걸 일일이 하는건 불편하니 쉘 스크립드로 순차 실행시킨다.



우선

~~~sh
vim ~/app/step1/deploy.sh
~~~

로 deploy.sh 파일을 생성하고 다음 코드를 입력해준다



~~~sh
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=boardJPA

cd $REPOSITORY/$PROJECT_NAME

echo "> Git Pull"

git pull

echo "> 프로젝트Build 시작"

./gradlew build

echo "> step1 디렉토리로 이동"

cd $REPOSITORY

echo "> Build 파일 복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -f ${PROJECT_NAME}.*.jar)

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
        echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
        echo "> kill -15 $CURRENT_PID"
        kill -15 $CURRENT_PID
        sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &


~~~

코드 설명은 생략.

입력한 스크립트를 실행하려면 실행권한을 줘야하므로

~~~sh
chmod +x ./deploy.sh
~~~

로 권한을 주고

~~~sh
./deploy.sh
~~~

로 실행해본다.



정겨운 Spring 마크를 기대하겠지만 Security설정이 되어있지 않아서

APPLICATION FAILED TO START 에러가 뜰 것이다.





## 외부 SECURITY 파일 등록

로컬 환경에는 application-oauth.properties가 있어서 상관없지만

이 프로퍼티를 깃에 올리는 순간 인증키가 공공재가 되어버리기 때문에 깃에 올리는건 안된다.

(물론 깃에다 프로퍼티 업로드해버리면 위에 배포스크립트만 작성해도 잘 돌아간다.)

따라서 서버에 직접 이 설정을 박아줘야한다.



1. 먼저 app 디렉토리에 properties 파일을 생성해준다

~~~sh
vim /home/ec2-user/app/application-oauth.properties
~~~

그리고 로컬에 적어놓은 application-oauth.properties를 고대로 복붙해준다.



2. 위에서 작성한 쉘스크립트에 이 프로퍼티 파일을 등록해줘야 하기때문에 다시 deploy.sh로 들어가 nohup부분을 수정해준다.

~~~sh
nohup java -jar \
        -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties
                $REPOSITORY/$JAR_NAME 2>&1 &

~~~

![스프링](/assets/images/12/SPRING.PNG)

스프링이 빵끗 웃으며 반겨줄 거다.
