# api-boot (Microservice)
Developed a microservice using SpringBoot. It has 3 layers: controller, service and repository. Let's have a look at the code.
### Application.java
This is the main class that runs the application.
```
@SpringBootApplication
@Import({ WebConfig.class, SwaggerConfig.class })
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```
Here, just annotating the class with **@SpringBootApplication** makes it a spring boot application.
**SwaggerConfig** class contains the swagger configuration. It is **optional** and not required to develop a microservice.

In SpringBoot, you don't need to create separate bean definitions for configuring your database with JPA or allowing CORS. It comes embedded with SpringBoot. You just have to set some properties.
### resources/application.properties
```
server.port=9000
server.context-path=/api
endpoints.cors.allowed-origins=*
endpoints.sensitive=false

#DataSource Properties
spring.datasource.url=jdbc:mysql://localhost:3306/spring_db?useSSL=false
spring.datasource.username=root
spring.datasource.password=1234
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true
```
Here you can prefix your URL, assign a specific port, setup database connections with predefined properties of SpringBoot.

Now, let's have a look at 3 layers.

### controller/UserController.java
```
@RestController
@RequestMapping(value = URI.USERS)
@Api(tags = "users")
public class UserController {

	private UserService service;

	public UserController(UserService service) {
		this.service = service;
	}

	@RequestMapping(method = RequestMethod.GET)
	@ApiOperation(value = "Find All Users", notes = "Returns a list of users in the app")
	@ApiResponses(value = { @ApiResponse(code = 200, message = "Success"),
			@ApiResponse(code = 500, message = "Internal Server Error"), })
	public List<User> findAll() {
		return service.findAll();
	}
  ```
  **@RestController** includes @Controller as well as @ResponseBody. It means it is the controller which redirects request to specific method and response body indicates that a method return value should be bound to web response body.
  **@ApiOperation** and **@ApiResponses** are swagger configuration and are optional. This method is to find all users.
  We have injected UserService in this which we will see next.
  
  ### service/UserServiceImpl.java
  ```
  @Service
public class UserServiceImpl implements UserService {

	private UserRepository repository;

	public UserServiceImpl(UserRepository repository) {
		this.repository = repository;
	}

	@Override
	@Transactional(readOnly = true)
	public List<User> findAll() {
		return repository.findAll();
	}
  ```
  This is the service layer. Whenever controller gets a request, it passes it to one of the service layer methods. In this case, it passes it to findAll method of service layer.
  We put all the business logic here. e.g. if values are null or not found. We put **@Transactional** at service layer because we want the whole request to be done in a single unit.
  This layer calls repository to get the data which is next.
  
  ### repository/UserRepository.java
  ```
  public interface UserRepository extends Repository<User, String> {

	public List<User> findAll();

	public Optional<User> findOne(String id);

	public Optional<User> findByEmail(String email);

	public User save(User user); //update and insert

	public void delete(User user);
}
```
We have used Spring Data here. So, we don't need to provide implementation of methods. We just have to extend spring data repository and rest is taken care by spring data. Though, we have to follow a specific naming convention by spring data for naming our methods.
The mechanism strips the prefixes from method and starts parsing the rest of it.
e.g. **findByEmail**. Here the first **By** indicates the start of actual criteria which means we want to search by the **email**.

These are the main components of the application.
