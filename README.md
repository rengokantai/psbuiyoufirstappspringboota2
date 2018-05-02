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




