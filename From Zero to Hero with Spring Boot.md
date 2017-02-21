>Spring Boot lets you pair-program with the Spring team.  
Josh Long, @starbuxman

## 1. Introduction to Spring Boot
  * Single point of focus (as opposed to large collection of spring-* projects)
  * A tool for getting started very quickly with Spring
  * Common non-functional requirements for a "real" application
  * Exposes a lot of useful features by default
  * Gets out of the way quickly if you want to change defaults
  + An opportunity for Spring to be opinionated
---

## 2. ಠ_ಠ Spring Boot is NOT
+ A prototyping tool
+ Only for embedded container apps
+ Sub-par Spring experience
+ For Spring beginners only
---

## 3.Installation
+ Requirements:
  * JDK6+
  * Maven 3.2+ / Gradle 1.12+
+ Tools:
  * Spring Tool Suite (STS) - IntelliJ IDEA - Netbeans
  * Spring CLI (from https://start.spring.io or gvm)
---

## 4. Getting Started Really Quickly

```java
@RestController
public class HomeController {
  @Value("${conference.name:jsug}")
  private String conference;

  @RequestMapping("/")
  public String home() {
    return "Hello " + conference;
  }
}
```
---

## 5. First integration test
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = DemoApplication.class)
@WebIntegrationTest(randomPort = true)
public class HomeControllerIntegrationTest {
  @Value("${local.server.port}")
  private int port;

  @Test
  public void runAndInvokeHome() {
    String url = "http://localhost:" + port + "/";
    String body = new RestTemplate()
      .getForObject(url, String.class);
    assertThat(body, is("Hello jsug"));
  }
}
```
---

## 6. Add Spring Data JPA

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <scope>runtime</scope>
</dependency>
```
---

## 7. Add a Speaker entity
```java
@Entity
public class Speaker {
  @GeneratedValue
  @Id
  private Long id;
  private String firstName;
  private String lastName;
  private String twitter;
  @Column(columnDefinition = "TEXT")
  private String bio;
  public Speaker(String firstName, String lastName, String twitter) {
  // initialize attributes
  }
  // getters and setters
}
```
---

## 8. Add a Speaker Repository
```java
public interface SpeakerRepository extends CrudRepository<Speaker, Long> {
  Speaker findByTwitter(String twitter);
  Collection<Speaker> findByLastName(String lastName);
}
```

## 9. Test that repository
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = DemoApplication.class)
public class SpeakerRepositoryTest {
  @Autowired
  private SpeakerRepository speakerRepository;

  @Test
  public void testFindByTwitter() throws Exception {
  Speaker stephane = speakerRepository.save(
  new Speaker("Stephane", "Nicoll", "snicoll"));
  assertThat(speakerRepository.findByTwitter("snicoll")
     .getId(),
  is(stephane.getId()));
  }
}
```
---
## 10. Add Spring Data REST
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```
---
## 11. RESTful repository
```java
public interface SpeakerRepository extends CrudRepository<Speaker, Long> {
  @RestResource(path = "by-twitter")
  Speaker findByTwitter(@Param("id") String twitter);
  Collection<Speaker> findByLastName(@Param("name") String lastName);
}
```
---

## 12. Add Spring Security
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
---
## 13. Spring Security config
```java
@Configuration
public class SecurityConfig extends GlobalAuthenticationConfigurerAdapter {
  @Override
  public void init(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
      .withUser("hero").password("hero").roles("HERO", "USER")
      .and()
      .withUser("user").password("user").roles("USER");
  }
}
```
---

## 14. Fix our test
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = DemoApplication.class)
@WebIntegrationTest(randomPort = true)
public class HomeControllerIntegrationTest {
  @Value("${local.server.port}")
  private int port;

  @Test
  public void runAndInvokeHome() {
    String url = "http://localhost:" + port + "/";
    String body = new TestRestTemplate("user", "user")
    .getForObject(url, String.class);
    assertThat(body, is("Hello jsug"));
  }
}
```
---

## 15. Add Spring Boot actuator
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
---

## 16. Add a health indicator
```java
@Bean
public HealthIndicator jsugHealthIndicator() {
  return () -> {
    if (new Random().nextBoolean()) {
       return Health.up().build();
    }
    else {
      return Health.down().withDetail("Boooo",42).build();
    }
  };
}
```
---

## 17. Add remote shell
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-remote-shell</artifactId>
</dependency>
```

## 18. Add a speaker controller
```java
@Controller
public class SpeakerController {
  private final SpeakerRepository speakerRepository;

  @Autowired
  public SpeakerController(SpeakerRepository speakerRepository) {
    this.speakerRepository = speakerRepository;
  }

  @RequestMapping("/ui/speakers/{id}")
  public String show(@PathVariable Long id, ModelMap model) {
    Speaker speaker = speakerRepository.findOne(id);

    if (speaker == null) {
      throw new SpeakerNotFoundException();
    }

    model.put("speaker", speaker);
    return "speakers/show";
  }

  @ResponseStatus(HttpStatus.NOT_FOUND)
  public static class SpeakerNotFoundException extends RuntimeException {
  }
}
```
---
## 19. Create speaker show view
```html
<html xmlns:th="http://www.thymeleaf.org">
  <head>
    <title th:text="${speaker.firstName} + ' ' + ${speaker.lastName}">View speaker</title>
  </head>
  <body>
    <div class="profile">
      <h1 class="name" th:text="${speaker.firstName} + ' ' + ${speaker.lastName}">Stephane Nicoll</h1>
      <div>
        <p th:text="${speaker.bio}">Sample Biography.</p>
      </div>
    <div>
      <a th:href="'http://twitter.com/' + ${speaker.twitter}" class="twitter" th:text="'@' + $
      {speaker.twitter}">@snicoll</a>
    </div>
    </div>
  </body>
</html>
```
---

## 20. Best experience with PaaS
```
$mvn package  
$ cf push jsug -p target/jsug-0.0.1-SNAPSHOT.jar  
$ cf scale jsug -i 4  
```

+ Spring Boot features get a lot done for 12factor.net
+ PaaS friendly: fast startup, devops oriented

---

## 21. Starter POMs
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
```groovy
compile("org.springframework.boot:spring-boot-starter-web")
```

+ standard POM / gradle files: define dependencies
+ many available: batch, integration, web, ampq…
+ starter data-jpa = spring-data-jpa + hibernate

---
## 22. Writing your own starter
+ Add support for X in Boot with a PR+
+ Distribute a client lib in your company
+ Standardize usage of component X in a platform
+ [your use case here]

---
## 23. New auto-config project
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-autoconfigure</artifactId>
</dependency>
```
+ Create a new hello-service-auto-configuration project
+ Only one mandatory dependency
+ Should contain specific dependencies and auto-configuration classes
---

# 24. Add custom service interface
```java
public interface HelloService {
  String sayHello();
}
```

+ This is the part that we’re trying to auto-configure
+ In a typical use case, this interface comes from a 3rd party lib

## 25. Create a default implementation
```java
public class ConsoleHelloService implements HelloService {
  @Override
  public String sayHello() {
    System.out.println("Hello Console");
  }
}
```
+ This default implementation will be shipped with the auto-config project but
should not be used if the application provides one.
---

## 26. Auto-configuration example
```java
@Configuration
@ConditionalOnClass(HelloService.class)
public class HelloServiceAutoConfiguration {
  @ConditionalOnMissingBean
  @Bean
  public HelloService helloService() {
    return new ConsoleHelloService();
  }
}
```

## 27. Declare the auto-configuration
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
demo.hello.HelloServiceAutoConfiguration
```

```java
// one can order AutoConfigurations
// with those annotations
@AutoConfigureBefore
@AutoConfigureAfter
```
---

## 28. Add a dependency to our auto-configuration
```xml
<dependency>
  <groupId>org.test.jsug</groupId>
  <artifactId>hello-service-auto-configuration</artifactId>
  <version>...</version>
</dependency>
```

+ Add our auto-config project as a dependency in our main project
---

## 29. Invoke our service when the application starts
```java
@Component
public class Startup implements CommandLineRunner {
  @Autowired private HelloService helloService;
  @Override
  public void run(String... args) throws Exception {
    helloService.sayHello();
  }
}
```
+ A Hello Service implementation is used on startup
+ Running this will use the default implementation

---

## 30. Override default (auto-configured) implementation
```java
@Bean
public HelloService helloService() {
  return () ->
    LoggerFactory.getLogger(DemoApplication.class)
    .info("Hello from logs");
}
```
+ Add a @Bean definition in our DemoApplication class
+ We provide our own implementation, so the default one won’t be created

---

## 31. Add type-safe properties
```java
@ConfigurationProperties("hello")
public class HelloProperties {
  private String prefix = "Hello ";
  private String target;
  // getters and setters
}
```
---

## 32. Enable properties support
```java
@EnableConfigurationProperties(HelloProperties.class)
@Configuration
@ConditionalOnClass(HelloService.class)
public class HelloServiceAutoConfiguration {}
```
---

## 33. Use type-safe properties
```java
public class ConsoleHelloService implements HelloService {
  @Autowired
  private HelloProperties properties;

  @Override
  public void run(String... strings) throws Exception {
    System.out.println(properties.getPrefix() + " " +
    properties.getTarget());
  }
}
```
---

## 34. Generate properties meta-data
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency>
```
+ annotation processor that generates meta-data
+ uses Javadoc to build keys descriptions
+ default values detected
+ manual declaration allowed

---

## 35. Document properties
```java
@ConfigurationProperties("hello")
  public class HelloProperties {
  /**
  * Prefix of welcome message.
  */
  private String prefix = "Hello ";
  /**
  * Target of welcome message.
  */
  private String target;
  // getters and setters
}
```
---
 
