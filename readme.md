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


### what is Beans ?
- There are 6 types of bean scopes, but the two most important (and most used) are:
- Singleton (default)
    1. Only one object (bean) is created for the entire Spring application context.
    2. Every time you @Autowired that bean, Spring gives you the same instance.
    3. Good for stateless services (e.g., Service classes, Repositories).
    4. because singleton is default
    ```java
        // this is by default beam.....
        @Service
        public class UserService {
            public String getUser() {
                return "Sharad";
            }
        }

        @RestController
        public class TestController {
            @Autowired
            private UserService userService1;

            @Autowired
            private UserService userService2;

            @GetMapping("/check")
            public String checkBeans() {
                return (userService1 == userService2) ? "Same Object" : "Different Objects";
            }
        }
    ```
- Prototype
    1. Every time you request the bean, Spring creates a new object.
    2. Useful for stateful beans (like objects with temporary data, calculators, etc.).
    ```java
        // this will create the prototype beam.....
        @Component
        @Scope("prototype")
        public class RandomNumberBean {
            private int number;

            public RandomNumberBean() {
                this.number = (int)(Math.random() * 1000);
            }

            public int getNumber() {
                return number;
            }
        }

        @RestController
        public class TestController {
            @Autowired
            private RandomNumberBean bean1;

            @Autowired
            private RandomNumberBean bean2;

            @GetMapping("/check")
            public String checkBeans() {
                return bean1.getNumber() + " - " + bean2.getNumber();
            }
        }
    ```
- Other Scopes (used in Web apps)
- - request → one bean per HTTP request.
- - session → one bean per HTTP session (per user).
- - application → one bean per servlet context.
- - websocket → one bean per WebSocket session.


### Practical Example (Using @Configuration + @Bean)
- Imagine you have a service that depends on a third-party class (not under your control, so you can’t annotate it with @Service).
- Use when you need beans for classes you can’t annotate with @Component/@Service.
- @Configuration = bean factory class.
- @Bean = method that defines a bean.
```java
    // third-party class (we can’t modify it)
    public class EmailClient {
        private String host;

        public EmailClient(String host) {
            this.host = host;
        }

        public String sendEmail(String to) {
            return "Email sent to " + to + " via " + host;
        }
    }

    // Configure it as a Spring bean manually:
    @Configuration
        public class AppConfig {

        // now the object of this has been created....
        @Bean
        public EmailClient emailClient() {
            return new EmailClient("smtp.gmail.com");
        }
    }

    @Service
    public class NotificationService {
        private final EmailClient emailClient;

        @Autowired
        public NotificationService(EmailClient emailClient) {
            this.emailClient = emailClient;
        }

        public String notifyUser(String user) {
            return emailClient.sendEmail(user);
        }
    }

    @RestController
    public class AppController {

        @Autowired
        private NotificationService notificationService;

        @GetMapping("/notify/{user}")
        public String notify(@PathVariable String user) {
            return notificationService.notifyUser(user);
        }
    }
```


### @Bean vs @Component
- @Component → put directly on class (Spring scans automatically).
- @Bean → used when:
1. You want to configure third-party classes.
2. You need to customize how a bean is created.
- Bean Names
- By default, bean name = method name.
- You can override:
- Then inject it with @Qualifier("customEmailClient").
```java
@Bean(name = "customEmailClient")
public EmailClient emailClient() {
    return new EmailClient("smtp.mailtrap.io");
}
```


### Spring Data JPA & Databases
- What is JPA?
- JPA (Java Persistence API) = Specification for ORM (Object Relational Mapping).
- ORM = maps Java classes (objects) to database tables.
- mplementation: Hibernate (most popular, used by default in Spring Boot).
- Example:
- A User class in Java = users table in DB.
- User object → row in DB.

- Connecting to MySQL/Postgres
- application.properties:
```bash
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

- spring.jpa.hibernate.ddl-auto
1. create → creates tables from entities (drops existing ones).
2. update → updates schema automatically (most used in dev).
3. validate → only validates schema.
4. none → no action.

- Entity Class (@Entity)
- Represents a table in DB.
- Each object = row in table.
- Equivalent to a Mongoose Schema in Node.js.
```java
import jakarta.persistence.*;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // auto increment
    private Long id;

    private String name;
    private String email;

    // getters & setters
    // for getting and setting the values in the database....
}
```

- Repository (DAO layer)
- Instead of writing SQL manually, Spring provides JpaRepository interface.
- JpaRepository<User, Long> → manages User entity with Long as ID type.
- Provides built-in CRUD methods:
1. findAll()
2. findById(id)
3. save(user)
4. deleteById(id)
- Similar to Mongoose’s User.find(), User.create().
```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {}
```