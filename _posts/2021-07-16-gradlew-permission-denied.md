---
title: "./gradlew : Permission Denied"
toc: false
tags:
- EC2
- Putty
- Linux
category: AWS
---

./gradlew : Permission Denied


 ![error](/assets/images/9/error.PNG)
 
프로젝트 배포과정에서 테스트용으로 ./gradlew test를 실행했는데

위와같은 에러가 발생


 ![권한부여](/assets/images/9/prog.PNG)
 
누가봐도 권한이 없는 문제니 chmod +x gradlew로 권한을 부여한 후

다시 실행하니

![실행](/assets/images/9/result.PNG)

 테스트 성공
