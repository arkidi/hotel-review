# Trip

spring-boot를 이용한 REST App 개발

## 1. 환경 설정
* build.gradle 파일 작성
* directory 생성
* Controller, Application 작성
* Run the application
* ```$ ./gradle build && java -jar build/libs/trip-0.1.0.jar```

## 2. 모니터링 기능 추가
* build.gradle에 ```compile("org.springframework.boot:spring-boot-starter-actuator:0.5.0.M4")``` 의존성 추가
* 아래 url과 같은 기능이 추가됨

```
	/env
	/health
	/beans
	/info
	/metrics
	/trace
	/dump
	/shutdown
```

# 문서 요약
## 1. Spring Boot
[원문](http://projects.spring.io/spring-boot/docs/spring-boot/README.html)

### 1.1 SpringApplication

SpringApplication은 main() 메소드를 통해 spring application을 편리하게 기동할 수 있게 한다. 대부분의 경우 SpringApplication.run 메소드로 위임만 하면 된다.

```
public static void main(String[] args) {
    SpringApplication.run(MySpringConfiguration.class, args);
}
```

### 1.2 Customizing SpringApplication

아래와 같이 설정하여 배너를 끌 수도 있다.

```
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setShowBanner(false);
    app.run(args);
}
```

### 1.3 Accessing command line properties

디폴트로 SpringApplication은 커맨드 라인 옵션 아규먼트(e. --server.port=9000)를 PropertySource로 변환하여 spring Environment에 최고 우선순위를 부여하여 추가한다.
Environment(System Property와 OS 환경 변수도 포함)의 Property들은 ```@Value``` 태그를 통해 spring 컴포넌트에 주입된다.

```
import org.springframework.stereotype.*
import org.springframework.beans.factory.annotation.*

@Component
public class MyBean {
    @Value("${name}")
    private String name; 
    // Running 'java -jar myapp.jar --name=Spring' will set this to "Spring"
}
```

### 1.4 CommandLineRunner beans

아래와 같은 행위를 수행하길 원한다면 CommandLineRunner 인터페이스를 구현하면 된다.

	- raw command line argument에 접근
	- SpringApplication이 실행된 후 어떤 코드를 수행
	
```
import org.springframework.boot.*
import org.springframework.stereotype.*

@Component
public class MyBean implements CommandLineRunner {

    public void run(String... args) {
        // Do something...
    }

}
```

둘 이상의 CommandLineRunner 구현체가 특정 순서로 호출되도록 하기 위해서는 ```org.springframework.core.Ordered interface```를 구현하거나 ```org.springframework.core.annotation.Order``` 어노테이션을 사용하면 된다.

### 1.5 Application Exit

spring의 lifecycle callback(```DisposableBean```이나 ```@PreDestroy```)을 이용해서 SpringApplication에 shutdown hook을 등록할 수 있다.
또 어플리케이션 종료시 특정 종료 코드를 반환하기 위해 ```org.springframework.boot.ExitCodeGenerator``` 인터페이스를 구현할 수도 있다.

### 1.6 Externalized Configuration

SpringApplication은 클래스 패스의 루트에서 ```application.properties```를 읽어서 spring Environment에 추가한다.

```application.properties```를 찾는 순서는 아래와 같다.

1. classpath root
2. current directory
3. classpath /config package
4. /config subdir of the current directory

profile에 종속적인 설정 파일도 사용 가능: ```application-{profile}.properties```

application.properties의 이름 변경 가능: ```$ java -jar myproject.jar --spring.config.name=myproject```

### 1.7 Setting the Default Spring Profile

profile을 사용하여 `@Profile`로 표시된 `@Component`만 로딩되도록 할 수 있다.

`spring.profiles.active` Environment를 사용. 이 환경 변수는 

1. application.properties에 설정 - `spring.profiles.active=dev,hsqldb` - 할 수도 있고
2. command line에 설정 - `--spring.profiles.active=dev,hsqldb` - 할 수도 있다.

### 1.8 Application Context Initializers

spring에서 ApplicationContextInitializer 인터페이스를 구현함으로써 ApplicationContext가 사용되기 전에 커스터마이즈할 수 있다. SpringApplication와 함께 ApplicationContextInitializer를 사용하려면 addInitializers 메소드를 사용하면 된다.

Environment property(`context.initializer.classes`)에 컴마로 분리된 클래스 이름 목록을 제공하거나 SpringFactoriesLoader 메커니즘을 사용하여 initializer들을 지정할 수 있다.

### 1.9 Embedded Servlet Container Support

ApplicationContext의 새로운 타입인 EmbeddedWebApplicationContext은 등록된 EmbeddedServletContainerFactory를 검색하여 컨테이너를 구동하는 WebApplicationContext이다. spring boot에는 TomcatEmbeddedServletContainerFactory와 JettyEmbeddedServletContainerFactory가 있다.
이를 통해 모든 표준적인 spring 개념들을 사용할 수 있다.

```
@Configuration
public class MyConfiguration {

    @Value("${tomcatport:8080}")
    private int port; 

    @Bean
    public EmbeddedServletContainerFactory servletContainer() {
        return new TomcatEmbeddedServletContainerFactory(this.port);
    }

}
```

### 1.10 Customizing Servlet Containers

AbstractEmbeddedServletContainerFactory는 톰캣, 제티용 Srvlet Container 팩토리의 공통 부모 클래스이다. 이를 통해 web.xml에서 설정하던 사항들을 programmatic하게 설정할 수 있다.

```
@Bean
public EmbeddedServletContainerFactory servletContainer() {
    TomcatEmbeddedServletContainerFactory factory = new TomcatEmbeddedServletContainerFactory();
    factory.setPort(9000);
    factory.setSessionTimeout(10, TimeUnit.MINUTES);
    factory.addErrorPages(new ErrorPage(HttpStatus.404, "/notfound.html");
    return factory;
}
```

### 1.11 Properteis 대신 YAML 사용

[SnakeYAML](http://code.google.com/p/snakeyaml/)이 클래스에 패스에 있으면 YAML 사용 가능

### 1.12 External EmbeddedServletContainerFactory configuration

EmbeddedServletContainerFactory를 설정할 수 있는 @ConfigurationProperties 클래스 ServerProperties를 제공

ServerProperties를 통해 아래와 같은 항목을 설정 가능

* server.port: defaults to 8080
* server.address: defaults to local address
* server.context_path: defaults to '/'

만일 톰캣을 사용한다면 아래와 같은 추가 설정이 가능

* The Tomcat access log pattern: server.tomcat.accessLogPattern
* The remote IP and protocol headers: server.tomcat.protocolHeader, server.tomcat.remoteIpHeader
* The Tomcat base directory: server.tomcat.basedir

### 1.13 Customizing Logging

commons-logging을 사용한다. 그리고 구현체는 변경 가능하고, java util logging, log4j, LogBack에 대한 디폴트 설정을 제공한다.

클래스 패스에 적절한 라이브러리를 포함함으로써 로깅이 활성화되고 클래스 패스 루트나 `logging.config` Environment에 정의된 패스에 설정 파일을 제공하여 커스터마이징할 수 있다.

로깅 시스템에 따라 다음과 같은 설정 파일이 로딩된다.

| Logging System | Customization |
| ------------- |-------------|
| Logback | right-aligned |
| Log4j | centered      |
| JDK | are neat      |
