# Spring Boot Depoly 방법

### Maven Build을 통한 Package

* Maven을 통해 package화 해서 jar를 생성

    - Eclipse export를 통해 runable jar 생성 시, 정상 동작되지 않음.


* Mave - eclipse에서 실행

  - Run As > Maven Build > Goal: 'package' 입력 > Run
  
  - Output Path = {Project Path}/target/{Project Name}.jar


* Mave - CLI에서 실행

  - mvn package
  
