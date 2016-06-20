## Annotation 장단점

***장점***
- 코드의 가독성 증대
- 복잡한 XML Schema를 파악하지 않아됨
- 별도의 Parser를 적용하지 않고 활용 가능

***단점***
- XML 파일 등에 설정하던 것에 비해 지역화 됨
- 리플렉션을 통해 해당 클래스를 분석해야 하는 오버헤드 발생
- Module이나 Application에 쓰는 메타 데이터를 설정할 수 없음


## Annotation 종류

***@Override***
- 상위 클래스의 메소드 재정의.

***@Deprecated***
- 특정 클래스나 인터페이스, 메소드, 필드 등이 더 이상 사용되지 말아야한다는 것을 경고

***@Required***
- 스프링 빈의 프로퍼티에 정의된 빈으로 설정할 때(setter) 사용되며 이 프로퍼티가 필수임을 알려줌. 
- 해당 프로퍼티의 setter 메소드 선언구에 설정. 
- 어떤 클래스를 쓰려고 할때 '@Required' 로 명시하면, xml에 해당 프로퍼티가 정의되어 있는지를 검사해서 없으면 예외 발생시킴

***@Autowired***
- 타입을 기준으로 의존관계를 자동으로 설정한다. 자동 와이어링 기능을 제공하며, 필드, 생성자, 메소드(일반,세터)에 적용할 수 있다. 가장 유연하고 강력한 기능을 가짐.
- xml 문서의 양을 대폭 줄여줄 수 있다.
- 프로퍼티나 생성자 파라미터를 지정하지 않고 미리 정해진 규칙을 이용해 자동으로 DI 설정을 컨테이너에 추가
- 자동으로 의존 관계가 맺어져야 할 빈을 찾아서 연결

***@Qualifier***
- Autowired 와 같이 사용하여, 타입이외의 정보를 추가해서 자동 연결될 빈 객체의 수식어를 값으로 제공

***@Resource***
- Autowired 가 타입을 기준으로 의존관계를 자동으로 설정한다면, Resource는 이름과 일치되는 스프링 빈을 자동으로 검색해서 적용

***@Configuration***
- 자바에서 빈을 생성하고자 할 때, 빈 설정 메타 정보를 담고 있는 자바에 선언
- ApplicationContext 객체 생성 시, 생성자의 인자 값으로 xml 파일이 아닌 (해당 클래스명).class가 적용된다.

***@Profile***
- 구동 환경을 세분화하여 서비스를 관리 (ex. 개발(dev), 테스트(test), 운영(prod) ...)
- DB 접속 계정 및 옵션, 리소스, 로그 관리 정책 등을 Profile 단위로 구분하여 관리할 수 있다.
- Profile 설정 - ex) 운영체제의 환경 변수 (RHEL/CentOS)
```
$ vi $HOME/.bashrc
export SPRING_PROFILES_ACTIVE=dev
$ $HOME/.bashrc
```
- 설정된 Profile 확인
```java
import org.springframework.core.env.Environment;

@Component
public class AnyBean {

    @Autowired
    private Environment environment;

    public String[] getActiveProfiles() {
        return environment.getActiveProfiles();
    }
}
```
- Profile 단위 application.properties 작성

/src/main/resources/application-dev.properties
```
...
logging.level.org.springframework=DEBUG
...
```
/src/main/resources/application-prod.properties
```
...
logging.level.org.springframework=INFO
...
```
- application.properties 설정 값 가져오기
```
import org.springframework.beans.factory.annotation.Value;

@Component
public class AnyBean {

    @Value("${logging.level.org.springframework:INFO")
    private String loggingLevel;

    public String getLoggingLevel() {
        return loggingLevel;
    }
}
```
- Profile 별로 선별적으로 Bean 등록하기
```
import org.springframework.stereotype.Component;

@Component
@Profile({"dev","test"})
public class AnyComponent {
    ...
}
```
Bean을 정의할 수 있는 @Configuration 클래스에도 적용됨
```
package com.jsonobject.example;

import org.springframework.context.annotation.Configuration;

@Configuration
@Profile({"dev","test"})
public class AnyConfiguration {
    ...
}
```

## Stereo type Annotation

***@Component***
- 일반적인 스프링 빈으로 등록. 
- 클래스이름을 빈의 id로 사용되게 된다. 단 첫글자는 소문자로 전환.
- 이름을 지정하고 싶으면 @Component("name") 이런 식으로 지정 할 수 있음.
- @Scope("prototype")을 밑에 줄에 추가하여, 스코프를 지정할 수 있음.
- 스캐닝해서 빈으로 등록하는 방법은 두 가지로, 
  자바단에서 
```java
@Component
public class EmployeeDAOImpl implements EmployeeDAO {
    ...
}
```
  ***또는,***
```java
  ApplicationContext context = new AnnotationConfigApplicationContext("spring.study");
```
  또는 xml 파일내에서, 
```xml
 <context:component-scan base-package="com.howtodoinjava.demo.service" />
 <context:component-scan base-package="com.howtodoinjava.demo.dao" />
 <context:component-scan base-package="com.howtodoinjava.demo.controller" />
```
로 스캐닝 시킬 수 있음.

***@Controller***
- Spring MVC의 Controller bean 객체로 자동 등록된다
- Spring Controller, Servlet을 상속할 필요가 없다

***@Repository***
- DAO와 같이 데이터 접근을 위한 저장소 객체로 등록된다.
- @Repository는 일반적으로 DAO에 사용되며, DB Excpetion을 DataAccessException으로 변환한다.
```java
@Repository("bbs.boardDAO")
public class BoardDAO {
    private SqlSession sqlSession;
    
    public int insertBoard(Board dto) throws Exception {
        ...
    }
}
```
```java
public class BoardServiceImpl implements BoardService {
    @Resource(name="bbs.boardDAO")
    private BoardDAO dao;
 
    public int insertBoard(Board dto){}
}
```

***@Service***
- 비지니스 로직을 구현한 서비스 객체로 등록된다.
- Service를 적용한 Class는 비지니스 로직이 들어가는 Service로 등록이 된다.
- Controller에 있는 @Autowired는 @Service("xxxService")에 등록된 xxxService와 변수명이 같아야 한다.
- Service에 있는 @Autowire는 @Repository("xxxDao")에 등록된 xxxDao와 변수명이 같아야 한다.
```java
@Service("helloService")
public class HelloServiceImpl implements HelloService {
    @Autowired
    private HelloDao helloDao;
 
    public void hello() {
        System.out.println("HelloServiceImpl :: hello()");
        helloDao.selectHello();
    }
}
```
helloDao.selectHello();와 같이 @Autowired를 이용한 객체를 이용하여 Dao 객체를 호출한다.
```java
@Service("test2.testService") 
//괄호 속 문자열은 식별자를 의미한다.
//괄호를 생략할 경우 클래스명 그대로 사용한다. 
//따라서 ,같은 클래스명이 존재 할 시 같은 식별자가 생성되기때문에 에러가 발생한다.
public class TestService {
    public String result(int num1, int num2, String oper) {
        String str = null;
        
        if (oper.equals("+")) {
            //...
            return str;
        }
    }
}
```
@Reduce로 연결
```java
@Resource(name="test2.testService") 
//name에 필요한 것은 @Service("test2.testService") <- 여기서 괄호 속 문자열, 즉 식별자
 
private TestService service; 
//TestService service = new TestService(); 라고 하는것과 같은 식
    
@RequestMapping(value="/test2/oper.action", method={RequestMethod.GET})
public String form() throws Exception {
    return "test2/write";
}
```


## Advise Annotation

***@Aspect***
- 설정 파일에 Advice, Pointcut 등의 설정을 하지 않고, 자동으로 Advice 적용
- Spring 2부터 지원
- <aop:aspectj-autoproxy> 태그를 설정 파일에 추가
> @Aspect 어노테이션이 적용된 빈을 Aspect로 사용
> @Aspect 어노테이션이 적용된 클래스를 로딩
> 클래스에 명시된 Advice, Pointcut 정보를 이용
> 알맞은 빈 객체에 Advice를 적용
- XML Tag:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation ="...
             http://www.springframework.org/schema/aop
             http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">
 
    <aop:aspectj-autoproxy/>
 
    <bean id="someService"
              class="some.package.SomeServiceImpl"/>
 
    . . .
 
</beans>
```
- Java Code:
```Java
@Aspect
public class SomeAspect{
 
    @Before("execution(public !void get*(..))")
    public void someBeforeAdviceMethod(JoinPoint joinPoint) {
        // 대상 객체 메서드 실행전 수행할 공통기능
    }
 
    @AfterReturning(pointcut = "execution(public !void get*(..))", returnint = "returnValFromTargetObjectMethod")
    public void someAfterReturningAdviceMethod(JoinPoint joinPoint, Object returnValFromTargetObjectMethod) {
        // 대상 객체 메서드 정상 실행후 수행할 공통기능
    }
 
    @AfterThrowing(pointcut = "execution(public !void get*(..))", throwing = "ex")
    public void someAfterThrowingAdviceMethod(JoinPoint joinPoint, Throwable ex) {
        // 대상 객체 메서드 실행중 Exception 발생 시 수행할 공통기능
    }
 
    @After("execution(public !void get*(..))")
    public void someAfterAdviceMethod(JoinPoint joinPoint) {
        // 대상 객체 메서드 정상/Exception 발생여부와 상관없이 실행 후 수행할 공통기능
    }
 
    @Around("execution(public !void get*(..))")
    publi void someAroundAdviceMethod(ProceedingJoinPoint joinPoint) {
        // 대상 객체 메서드 실행 전 실행될 공통 기능         
        . . .    
        try{         
            // 대상 객체 메서드 실행
            Object returnValFromTargetObjectMethod = joinPoint.proceed();
              
            // 대상 객체 메서드 실행 후 실행될 공통 기능  
            . . .        
        }catch(Throwable e){         
            // 대상 객체 메서드 실행 중 Exception 발생 시 실행될 공통 기능   
            . . .        
        }finally{        
            // 대상 객체 메서드 정상 실행 여부와 상관없이 실행될 공통 기능    
            . . .        
        }
    }
}
```

***@Before***
- 조인 포인트를 실행하기 전에 어떤 처리를 할 수 있음
- 스프링에서 조인포인트는 항상 메소드 호출이기 때문에 해당 메소드를 처리 하기 전에 전처리 과정을 수행하는데 사용
- 호출하는 메소드에 넘겨주는 인자들을 비롯한 타겟 객체에도 접근할 수 있지만 메소드 실행 자체를 제어할 수는 없음
```xml
<aop:before />
```

***@AfterReturning***
- 조인 포인트의 매소드가 실행을 끝내고 값을 반환했을 때 실행
- 해당 메소드에 넘겨주는 인자와 타겟 객체, 그리고 반환값에 접근
- After Returning 어드바이스를 적용하기 전에 이미 메소드는 실행을 마쳤기 때문에 메소드 호출 자체를 제어할 수는 없음
```xml
<aop:after-returning />
```

***@AfterThrowing***
- Exception이 발생하여 조인포인트가 빠져나갈 때 수행
- try-catch 블록의 catch블록과 비슷
- 메소드가 호출된 후에 실행되는데 단 해당 메소드가 예외를 던졌을 경우에만 실행
- 특정 예외만 잡아 메소드에 넘겨준 인자와 타겟 객체에 접근내도록 설정할 수 있으며, 해당 예외를 발생
```xml
<aop:after-throwing />
```

***@After***
- 조인 포인트를 빠져나가는 (정상적이거나 예외적인 반환) 방법에 상관없이 수행
- try-catch-finally 에서 finally 블록과 비슷
```xml
<aop:after />
```

***@Around***
- 조인포인트 전후에 수행
- 메소드 실행 전과 후에 어떤 작업을 수행할 수 있으며, 어느 시점에 메소드를 호출 할지 혹은 실행하지 않을지 제어할 수 있음
- 필요한 경우 해당 메소드 대신 별도의 로직을 제공
```xml
<aop:around />
```

***@RequestMapping***
- 요청에 대해 어떤 Controller 어떤 메소드가 처리할지를 맵핑하기 위한 어노테이션
- url을 class 또는 method와 mapping 시켜주는 역활을 한다.
- GET 또는 POST 방식등의 옵션을 줄 수 있다.
- Method가 실행된 후, return 페이지가 정의되어 있지 않으면, @RequestMapping["/url"]에 설정된 url로 다시 돌아간다.

***@RequestMapping***
- Controller 메소드 파라미터에는 @PathVariable을 이용해 URI 템플릿 중에서 어떤 파라미터를 가져올지 지정할 수 있다.
```java
@RequestMapping("/user/view/{id}")
public String view(@PathVariable("id") int id) {
 ...
}
```

***@RequestParam***
- Controller 메소드의 파라미터와 웹요청 파라미터와 맵핑하기 위한 Annotation
- 단일 HTTP 요청 파라미터를 메소드 파라미터에 넣어주는 Annotation
- 스프링의 내장 변환기가 다룰 수 있는 모든 타입을 지원한다
- key=value 형태로 화면에서 넘어오는 파라미터를 매핑 된 메소드의 파라미터로 지정해 준다.
```java
public String view(@RequestParam("id") int id, @RequestParam("name") String name) {
...
}
```

***@ModelAttribute***
- Controller 메소드의 파라미터나 리턴값을 Model 객체와 바인딩하기 위한 Annotation
- 화면의 from 속성으로 넘어온 model을 매핑된 메소드의 파라미터로 지정해준다.
- 도메인 오브젝트나 DTO의 프로퍼티에 요청 파라미터를 바인딩해 한번에 받는다.
- Command Object라고 부르기도 한다.
```java
@RequestMapping("/usr/search")
public String search(@ModelAttribute UserSearch userSearch) {
 List<User> list = userService.search(userSearch);
 model.addAttribute("userList", list);
 ...
}
```

***@SessionAttributes***
- Model 객체를 세션에 저장하고 사용하기 위한 어노테이션
- 세션상에서 model의 정보를 유지하고 싶을 경우 사용한다.
```java
@Controller
@SessionAttributes("blog")
public class BlogController {
    ...
    @RequestMapping("/createBlog")
    public ModelMap createBlogHandler() {
        blog = new Blog();
        blog.setRegDate(new Date());
        return new ModelMap(blog);
    }
    ...
}
```

***@RequestPart***
- Multipart 요청의 경우, 웹요청 파라미터와 맵핑가능한 어노테이션(egov 3.0, Spring 3.1.x부터 추가)

***@CommandMap***
- Controller메소드의 파라미터를 Map형태로 받을 때 웹요청 파라미터와 맵핑하기 위한 어노테이션(egov 3.0부터 추가)

***@ControllerAdvice***
- Controller를 보조하는 어노테이션으로 Controller에서 쓰이는 공통기능들을 모듈화하여 전역으로 쓰기 위한 어노테이션(egov 3.0, Spring 3.2.X부터 추가)

