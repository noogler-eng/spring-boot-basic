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


### 3. application.properties (application.yml)
- it is like a Spring Boot is like your .env file in Node.js/Express.
- It is used to configure your application without hardcoding values in code.
- Spring Boot automatically loads it at startup.
- Common Use Cases
- - Database connection settings
- - Server port
- - Logging levels
- - Custom app values (like API keys, names)
```bash
    spring.datasource.url=jdbc:mysql://localhost:3306/mydb
    spring.datasource.username=root
    spring.datasource.password=secret
    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.show-sql=true

    // changing the server port using this
    server.port=8081

    // You can define your own values:
    app.name=sharad
    app.version=1.0.4
```

```java
    @RestController
    public class AppController {
        @Value("${app.name}")
        private String appName;

        @Value("${app.version}")
        private String version;

        @GetMapping("/info")
        public String getAppInfo() {
            return appName + " - v" + version;
        }
    }
```

- Profiles (dev, test, prod)
- You can create environment-specific files:
- application-dev.properties
- application-test.properties
- application-prod.properties
```bash
    java -jar app.jar --spring.profiles.active=dev
```
- Similar to having .env.development and .env.production in Node.js.
```bash
    # application-dev.properties
    server.port=8081

    # application-prod.properties
    server.port=80
```


### 4. Dependency Injection (DI) & Inversion of Control (IoC).
- The Problem (Without DI)
- Imagine in Node.js you do this:
```js
    // we create an object from class
    const userService = new UserService();
    // making an another class with same object
    // OrderService depends on UserService.
    // You are the one creating objects (new UserService()).
    // If tomorrow UserService changes, you need to update everywhere.
    // This is called Tight Coupling.
    const orderService = new OrderService(userService);
```

- Inversion of Control (IoC)
- Inversion of Control = Don’t create objects yourself, let the framework do it.
- You controlling object creation,
- The Spring IoC Container (like a manager) creates and manages them for you.
- Dependency Injection = Give the required objects (dependencies) to a class automatically.
```java
    UserService userService = new UserService();
    OrderService orderService = new OrderService(userService);

    // instead od this we can write this
    @Service
    public class OrderService {
        @Autowired
        private UserService userService;
    }
    // Spring will:
    // Create a UserService object (a bean).
    // Inject it into OrderService automatically.
```

#### Types of Dependency Injection in Spring
- Field Injection (most common, but less recommended for testing)
```java
@Service
public class OrderService {
    @Autowired
    private UserService userService;
}
```

- Constructor Injection (recommended way)
```java
@Service
public class OrderService {
    private final UserService userService;

    @Autowired
    public OrderService(UserService userService) {
        this.userService = userService;
    }
}
```

- Setter Injection
```java
@Service
public class OrderService {
    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}
```

### project flow 
```java
// Creates UserService bean.
// Injects it into OrderService.
// Injects OrderService into AppController.
// You never used new keyword — Spring handled it.
@Service
public class UserService {
    public String getUserName() {
        return "Sharad";
    }
}

@Service
public class OrderService {
    private final UserService userService;

    @Autowired
    public OrderService(UserService userService) {
        this.userService = userService;
    }

    public String createOrder() {
        return "Order created for user: " + userService.getUserName();
    }
}

@RestController
public class AppController {
    private final OrderService orderService;

    @Autowired
    public AppController(OrderService orderService) {
        this.orderService = orderService;
    }

    @GetMapping("/order")
    public String placeOrder() {
        return orderService.createOrder();
    }
}
```