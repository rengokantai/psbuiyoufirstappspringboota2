# psbuiyoufirstappspringboota2
## 2. Creating the Spring Boot Project
### 8  Demo: Using the API Endpoint
RestController, tells all payload communication should use json by default
```
@RestController
@RequestMapping("/api/v1/bikes")
public class BikeController {
  @GetMapping
  public List<Bike> list(){
    List<Bike> bikes = new ArrayList<>();
    return bikes;
  }
  @PostMapping
  @ResponseStatus(HttpStatus.OK)
  public void create(@RequestBody Bike bike){
  
  }
  
  @GetMapping("/{id}")
  public Bike get(@PathVariable("id") long id){
    return new Bike();
  }
}
```

## 3. Adding a Persistence Layer
### 4 Demo: Adding Persistence Dependencies
add three dependencies
```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>org.xerial</groupId>
  <artifactId>sqlite-jdbc</artifactId>
</dependency>
<dependency>
  <groupId>com.zsoltfabok</groupId>
  <artifactId>sqlite-dialect</artifactId>
  <version>1.0</version>
</dependency>
```

### 5 Configuring Persistent Properties
application.properties
```
spring.jpa.database-platform=org.hibernate.dialect.SQLiteDialect
spring.jpa.hibernate.ddl-auto=none
spring.jpa.show-sql=true


spring.datasource.url=jdbc:sqlite.bike.db
spring.datasource.username=
spring.datasource.password=
spring.datasource.driver-class-name=org.sqlite.JDBC
```


### 6 Creating a JPA Entity
```
import javax.persistence.Entity;
@Entity
public class Bike {
  @Id
  @GenerateValue(strategy=GenerationType.AUTO)
  private Long id;
}
```

### 7 Creating a JpaRepository
```
import org.springframework.data.jpa.repository.JpaRepository;

public interface BikeRepository extends JpaRepository<Bike,Long>{
  
}
```
in BikesController.java
```
@AutoWired
private BikeRepository bikeRepository;
@GetMapping
public List<Bike> list(){
  return bikeRepository.findAll();
}
@GetMapping("/{id}")
public Bike get(@PathVariable("id") long id){
  return bikeRepository.getOne(id);
}
```

### Customizing JSON with Jackson
```
@JsonIgnoreProperties({"hibernateLazyInitializer","handler"})
```

```
@JsonFormat(shape=JsonFormat.Shape.STRING,pattern="MM-dd-yyyy")
```

## 4. Creating the Angular Project
### Generating
```
ng new pj --routing
```

### Setting Up a Development Proxy
in ng project root, add
```proxy.conf.json```
content
```
{
  "/server":{
    "target":"http://localhost:8080",
    "secure":false,
    "changeOrigin":true,
    "logLevel":"debug",
    "pathRewrite":{
      "^/server":""
    }
  }
}
```
package.json
```
"start":"ng serve --proxy-config proxy.conf.json"
```

### Generating an Angular Service
```
const httpOptions {
  headers: new HttpHeaders({'Content-Type':'application/json'})
}
```
## 5. Finishing the Angular Screens
### 2 Finishing the API Service Calls
```
createBikeRegistration(bike){
  let body=JSON.stringify(bike);
  return this.http.post('/server/api/v1/bikes',body,httpOptions);
}
```


## 6. Securing the Application
### 4 Adding Security Dependencies
```
<dependency>
  <groupId>com.auth0</groupId>
  <artifactId>auth0-spring-security-api</artifactId>
  <version>1.0.0-rc.3</version>
</dependency>
```

add
```
auth0.issuer:https://dan.auth0.com/
auth0.apiAudience:http://localhost:8080
```

### 5 Setting up Spring Security Configuration
```
import com.auth0.spring.security.api.JwtWebSecurityConfigurer;

@EnableWebSecurity
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter{
  @Value(value="${auth0.apiAudience}")
  private String apiAudience;
  @Value(value="${auth0.issuer}")
  private String issuer;
  
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    JwtWebSecurityConfigurer.forRS256(spiAudience,issuer).configure(http)
    .authorizeRequests()
    .antMatchers(HttpMethod.POST,"/api/v1/bikes").permitAll()
    .antMatchers(HttpMethod.GET,"/api/v1/bikes").hasAuthority("view:registrations")
    .antMatchers(HttpMethod.GET,"/api/v1/bikes/**").hasAuthority("view:registration")
    .anyRequest().authenticated();
  
  }
}
```

### Configuring an Auth0 client
```
npm install --save auth0-js
```


### Creating an Authorization Service
```
import { Injectable } from '@angular/core';
import { Router } from '@angular/router';
import 'rxjs/add/operator/filter';
import * as auth0 from 'auth0-js';

@Injectable()
export class AuthService {
  auth0 = new auth0.WebAuth({
    clientID:
    domain:
    responseType:
    audience:
    redirectUri:
    scope:
  })
  
  public login():void{
    this.auth0.authorize();
  }
  
  public handleAuthentication(): void{
    this.auth0.parseHash((err,authResult)=>{
      if(authResult&& authResult.accessToken && authResult.idToken){
        window.location.hash='';
        this.setSession(authResult);
        this.router.navigate(['/admin']);
      } else if(err){
        this.router.navigate(['/admin']);
      }
    });
  }
  
  private setSession(authResult):void{
    const expiresAt = JSON.stringify((authResult.expiresIn*1000)+new Date().getTime());
    localStorage.setItem('access_token',authResult.accessToken);
  
  }
  
  public logout():void{
    this.router.navigate(['/']);
  }
  public isAuthenticated(): boolean {
    const expiresAt = JSON.parse(localStorage.getItem('expires_at'));
    return new Date().getTime()<expiresAt;
  }
  
  
  
  
  
}




```



