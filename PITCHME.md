
# Configuration 

### twelve-factor app preview

# III. Config

+++

### Store config in the environment

* An app’s config is everything that is likely to vary between deploys (staging, production, developer environments, etc). 
* This includes:
  - Resource handles to the database, Memcached, and other backing services
  - Credentials to external services such as Amazon S3 or Twitter
  - Per-deploy values such as the canonical hostname for the deploy

+++

## Rules

* Apps sometimes store config as constants in the code. 
* This is a violation of twelve-factor, which requires strict separation of config from code. 
* Config varies substantially across deploys, code does not.

+++

### External configuration by file

- Can be injected through a property file
- Prior to Spring Boot the config file had to be referenced

```java
@PropertySource("file:/path/to/simple.properties")
public class Application {
```

```java
    @Value("${spring.application.name}")
    private String appName;
```
+++

### External configuration by file

- Properties can be set with a default value, which will be picked in case property is not found or set
- In case no default is defined and the property can't be found the app won't start

```java
    @Value("${spring.application.name: Default Name}")
    private String appName;
```
```
2017-05-03 13:55:26.342 [..] Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'messageRestController': Injection of autowired dependencies failed; nested exception is java.lang.IllegalArgumentException: Could not resolve placeholder 'spring.application.name' in value "${spring.application.name}"
```
+++

### External configuration by file with Spring Boot

- Spring Boot will read the properties in `src/main/resources/application.properties` by default
- Spring Boot allows you to use yaml files instead of properties
- Both examples will work

```yaml
application.name=a-bootiful-client
```
```yaml
application:
    name: a-bootiful-client
```
+++

### External configuration by file with Spring Boot- POJO

```java
@Component
@ConfigurationProperties("application")
class ApplicationProperties{

	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

}
```

+++

### External configuration by file with Spring Boot- POJO Invocation

```java
@RestController
class MessageRestController {

    @Autowired
    private ApplicationProperties ap;

    @RequestMapping("/appNameViaPOJO")
    String appNameViaPOJO() {
        return this.ap.getName();
    }
}
```

+++

# Remember Actuator?

+++

Enable

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
			<scope>runtime</scope>
		</dependency>
```

+++

Enable all endpoints  

```properties
management.endpoints.web.exposure.include=*
```

+++

Enable specific endpoints  

```properties
management.endpoints.web.exposure.include=configprops,mappings,env
```

+++

Point the browser to: http://localhost:8080/actuator/env

```json
"application.name": {
  "value": "test",
  "origin": "class path resource [application.properties]:4:18"
}
```

+++

### Profiles

+++

- different configuration profiles

+++

- use filename

appliction-postgres.properties
```properties
spring.jpa.hibernate.ddl-auto=update
spring.datasource.url= jdbc:postgresql://localhost:5432/mydb 
spring.datasource.username=matthias
spring.datasource.password=password
```
+++

- use filename

appliction-h2.properties
```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2
spring.datasource.url=jdbc:h2:mem:testdb 
spring.datasource.username=sa
spring.datasource.password=
```

- in same file (only in yaml)

```yaml
spring:
  profiles:
    active: test
`---
spring:
  profiles: test
application:
  name: test-app
`---
spring:
  profiles: dev
application:
  name: dev-app
```




