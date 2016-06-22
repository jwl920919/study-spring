# Spring Boot Depoly 방법

### Maven Build을 통한 Package

정상적으로 배포하기 위해서는 Maven을 통해 

* EClipse에서 실행

  Run As > Maven Build > Goal = package 입력 후 Run
  
  Output Path = {Project Path}/target/{Project Name}.jar


* CLI에서 실행

  mvn package
  
