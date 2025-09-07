## Spring Boot

### 1. What is Maven?
- It’s a build automation and dependency management tool for Java projects.
- Think of Maven as the npm + package.json + scripts of the Java world.
- You install dependencies with mvn install.
- You run builds/tests with mvn clean install or mvn test.
- You manage project info in pom.xml (like package.json).
- Defines project details, dependencies, build plugins, etc.

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.myapp</groupId> <!-- like package namespace -->
    <artifactId>demo-app</artifactId> <!-- like project name -->
    <version>1.0.0</version>

    <dependencies>
        <!-- Example: Spring Boot Starter Web (like express) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
</project>
```

- Maven Central Repository = like npm registry
- Maven downloads all your Java libraries (dependencies) from there.
- Lifecycle / Command
1. mvn clean → cleans build folder (like deleting dist/).
2. mvn compile → compiles Java source.
3. mvn test → runs tests (like npm test).
4. mvn package → builds a .jar file (like npm run build).
5. mvn install → installs the built jar to your local repo.


### 2. Common Spring Annotations
- annotations are like special labels we put on classes, methods, 
- or variables to tell Spring what to do with them.
- They reduce the need for long XML configuration files and make the code cleaner.

1. @SpringBootApplication
- Put on the main class.
- Tells Spring Boot: "This is the entry point, start the application here."
- It is a combination of three annotations:
- @Configuration, @EnableAutoConfiguration, @ComponentScan


2. @Component, @Service, @Repository, @Controller
- These mark a class so Spring knows to create an object (bean) of it and manage it
- Differences
- - @Component → generic (any Spring-managed class).
- - @Service → business logic layer.
- - @Repository → database layer (also handles DB exceptions).
- - @Controller → web controller (used with Spring MVC).
```java
@Service
public class UserService {
    // logic here
}
```


3. @Autowired
- Tells Spring: "Please inject (give me) the required bean automatically."
- Used for dependency injection.
```java
@Service
public class OrderService {
    
    @Autowired
    private UserService userService; // Spring will inject UserService here
}
```


4. @RestController
- A shortcut for @Controller + @ResponseBody.
- Used for APIs → directly returns JSON/XML instead of HTML.
```java
@RestController
public class UserController {
    
    @GetMapping("/hello")
    public String sayHello() {
        return "Hello World";  // returns as response body
    }
}
```


5. @RequestMapping / @GetMapping / @PostMapping
- Tell Spring which URL a method should respond to.
- @RequestMapping → general (can be GET, POST, etc.).
- @GetMapping → specifically for GET requests.
- @PostMapping → specifically for POST requests.
```java
    @GetMapping("/users")
    public List<User> getUsers() {
        return userService.getAllUsers();
    }
```


6. @PathVariable and @RequestParam
- @PathVariable → fetches value from the URL path.
- @RequestParam → fetches value from query parameters.
```java
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable int id) { ... }

    @GetMapping("/search")
    public User searchUser(@RequestParam String name) { ... }
```


7. @Configuration & @Bean
- @Configuration → marks a class that defines beans.
- @Bean → tells Spring "create this object and manage it."
```java
    @Configuration
    public class AppConfig {
    
        @Bean
        public UserService userService() {
            return new UserService();
        }
    }
```


8. @Qualifier
- Used with @Autowired when multiple beans of the same type exist.
- Helps specify which one to inject.
```java
    @Autowired
    @Qualifier("fastService")
    private PaymentService paymentService;
```


9. @Value
- Injects values from application.properties file.
```java
    @Value("${app.name}")
    private String appName;
```