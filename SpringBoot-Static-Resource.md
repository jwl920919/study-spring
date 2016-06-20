# Spring Boot에서 Static Resource 서비스

Spring Boot로 제작한 애플리케이션을 빌드하면 1개의 .jar 파일이 생성되며 일반적인 Java 애플리케이션과 완전히 동일한 방법으로 구동된다. 
빌드시 .html, .css, .js와 같은 정적 리소스(Static Resource) 또한 .jar 파일 안에 같이 패키징된다. 
만약 개발 과정에서 프론트엔드 개발자가 이러한 정적 리소스 파일 수정에 따른 재배포를 요청할 경우 재빌드에 따른 번거로운 상황이 발생한다. 
Spring Boot는 이런 경우에 재빌드 없이 손쉽게 대응할 수 있는 방법을 제공한다.

## 기본 설정된 정적 리소스 경로 적용하기

Spring Boot 프로젝트에서 별도의 커스터마이징이 없을 경우 정적 리소스의 위치는 우선순위대로 아래와 같다.

- 첫째는, 프로젝트 상의 /src/main/META-INF/resources 디렉토리이다. 
  이 경로에 포함된 파일은 .jar 파일 안에 패키징된다. 
  /src/main/META-INF/resources/attachments/example.txt 경로에 파일이 위치할 경우 HTTP 상의 요청 주소는 /attachments/example.txt가 된다.

- 둘째는, java -jar {jar} 명령을 실행한 시점의 현재 디렉토리에 서브로 존재하는 public 디렉토리이다. 
  앞서 첫번째 방법과 다르게 .jar 파일 외부에 정적 리소스가 위치한다.
  따라서 정적 리소스 수정시 재빌드가 필요없는 장점이 있다. 
  /{currentWorkingDirectory}/public/attachments/example.txt 경로에 파일이 위치할 경우, HTTP 상의 요청 주소는 전자와 동일하게 /attachments/example.txt가 된다. 
  현재 디렉토리라는 기준은 애매할 수 있으므로 절대 경로를 정하고 싶다면 java -jar {jar} -cp {absoluteDirectory}와 같이 JVM 옵션을 명시하면 된다.

## 가장 확실한 방법, 절대 경로 명시하기

가장 확실한 방법은 정적 리소스가 서비스될 절대 경로를 명시하는 것이다.
Windows 개발 환경과 RHEL/CentOS 운영 환경으로 이원화하여 해당 프로파일에 적합한 파일 시스템 경로의 위치를 달리 설정하였다. 
/assets/jquery.js 요청의 경우 개발 서버에서는 C:\assets\jquery.js, 운영 서버에서는 /opt/assets/jquery.js 경로를 서비스한다.

- application-dev.properties
```
static.resource.location=file:///C:/assets/
```

- application-prod.properties
```
static.resource.location=file:/opt/assets/
```

- StaticResourceConfig.java
```java
package com.jsonobject.example;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
public class StaticResourceConfig extends WebMvcConfigurerAdapter {

    @Value("${static.resource.location}")
    private String staticResouceLocation;

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/assets/**").addResourceLocations(staticResouceLocation);
    }
}
```

## 결론
사실 대부분의 대용량 트래픽을 처리해야 하는 기업용 애플리케이션은 HTTP 요청의 앞 단계에서 Apache, NGINX와 같은 웹 서버를 이용하여 정적 리소스와 애플리케이션이 처리할 요청을 자동으로 구분하여 처리하기 때문에 앞서 설명한 방법은 사용하지 않는다.
하지만 소규모 및 기업 내부용 애플리케이션 구현시 위 설명한 방법은 매우 효과적이기 때문에 추천한다.
