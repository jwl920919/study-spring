# External Properties 정리

### Properties Class


* @PropertySource : Properties Source 지정  

  - 먼저 정의된 순서가 우선 순위가 높다.

  - ignoreResourceNotFound : 파일이 존재하지 않을 경우 무시
  
  - 외부 설정 파일이 지정되지 않을 경우, jar 내부 설정 파일 순으로 지정 가능.

  - (참고) classpath:~ - jar 파일 기준
  
  - (참고) file:~ - 외부 파일 경로 

  
example:
```java
@Component
@PropertySources({
	@PropertySource(value="file:app.properties", ignoreResourceNotFound=true),
	@PropertySource("classpath:application.properties")
})
@ConfigurationProperties(prefix = "system")
public class SystemProperties {

	private String name;
	private String version;
	...
}

```

* Jar 실행 (외부 Properties 지정)
```
java -jar swt-manager-0.0.1.jar --spring.config.location="file:app.properties"
```
