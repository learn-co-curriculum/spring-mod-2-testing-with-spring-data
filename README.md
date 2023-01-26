# Code-Along: Testing with Spring Data

## Learning Goals

- Expand upon the application to provide an example for testing with Spring
  Data.
- Learn how to unit test, integration test, and acceptance test with Spring
  Data.

## Introduction

In the last few lessons, we reviewed testing and how we can test more
functionality together in integration testing and acceptance testing in Spring.

But our Spring application we were working with was pretty small compared to
other applications we have written where it is attached to a database. Let us
talk about testing with Spring Data now! Before we do so, we'll need to expand
our `spring-testing-demo` project.

## Model

Sticking with the cat theme, let's model a cat rescue! For clarification, we
will model a cat rescue as in the place, like an animal shelter for cats
rather than an action or event of rescuing a cat. Consider the following ER
diagram:

![cat-ER-diagram](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/cat-rescue-er-diagram.png)

We'll model a one-to-many relationship in this lesson to closer resemble your
Spring project that you have been working on. In this relationship, notice that
a rescue can have many cats but a cat can only belong to one rescue.

## Code-Along: Expand on the Spring Testing Demo Project

We'll start expanding our `spring-testing-demo` project we have been working
with in this section by creating the database in pgAdmin4. We'll call this
database "testing_demo":

![create-testing-demo-db](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/create-testing-demo-db.png)

Once the database has been created, open up the Query Tool and copy in the
following to define the database schema:

```postgresql
DROP TABLE IF EXISTS rescue;
DROP TABLE IF EXISTS cat;

CREATE TABLE rescue(
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL,
  address TEXT,
  city TEXT,
  state TEXT,
  website_url TEXT NOT NULL,
  phone_number TEXT NOT NULL
);

CREATE TABLE cat(
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL,
  breed TEXT NOT NULL,
  age INTEGER,
  rescue_id INTEGER NOT NULL,
  CONSTRAINT rescue_id FOREIGN KEY (rescue_id) REFERENCES rescue(id)
    ON DELETE CASCADE
);
```

Now let's open up the `spring-testing-demo` project again and add some
dependencies to our project. Open the `pom.xml` file to add the Spring Data JPA
dependency, the Model Mapper dependency, and the PostgreSQL Driver dependency:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.6</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <groupId>com.example</groupId>
  <artifactId>spring-testing-demo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>spring-testing-demo</name>
  <description>spring-testing-demo</description>
  <properties>
    <java.version>11</java.version>
  </properties>
  <dependencies>
    <!-- Add the Spring Data JPA dependency -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Add the ModelMapper dependency -->
    <dependency>
      <groupId>org.modelmapper</groupId>
      <artifactId>modelmapper</artifactId>
      <version>3.1.1</version>
    </dependency>
    <!-- Add the PostgreSQL Driver dependency -->
    <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.24</version>
      <scope>provided</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <version>${project.parent.version}</version>
      </plugin>
    </plugins>
  </build>

</project>
```

Once the dependencies have been added to the `pom.xml`, click the little
Maven icon in the upper-right hand corner to reload the changes:

![load-maven-changes](https://curriculum-content.s3.amazonaws.com/spring-mod-2/authentication/load-maven-changes.png)

Now we'll need to add some packages and classes to our application. Go ahead and
create a `repository` and `entity` package with a `RescueRepository`,
`CatRepository`, `Rescue`, and `Cat` classes in their respective classes. We
will also want to create the following classes:

- `RescueController`
- `CatController`
- `RescueDTO`
- `CatDTO`
- `RescueService`
- `CatService`

The following should be the new project structure:

```text
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── springtestingdemo
    │   │               ├── SpringTestingDemoApplication.java
    │   │               ├── controller
    │   │               │  ├──  CatController.java
    │   │               │  ├──  DemoController.java
    │   │               │  └──  RescueController.java    
    │   │               ├── dto
    │   │               │  ├── CatDTO.java    
    │   │               │  ├── CatFactDTO.java
    │   │               │  └── RescueDTO.java
    │   │               ├── entity
    │   │               │  ├── Cat.java    
    │   │               │  └── Rescue.java  
    │   │               ├── repository
    │   │               │  ├── CatRepository.java    
    │   │               │  └── RescueRepository.java                
    │   │               └── service
    │   │                   ├── CatService.java    
    │   │                   ├── CatFactService.java
    │   │                   └── RepositoryService.java    
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── org
                └── example
                    └── springtestingdemo
                        ├── SpringTestingDemoApplicationTests.java
                        ├──controller
                        │  ├──DemoControllerAcceptanceTest.java
                        │  ├──DemoControllerIntegrationTest.java
                        │  └──DemoControllerUnitTest.java
                        └── service
                            └── CatFactServiceIntegrationTest.java
```

Add the following code to each class (at this point, it should be review):

### application.properties

```properties
spring.datasource.url= jdbc:postgresql://localhost:5432/testing_demo
spring.datasource.username= postgres
spring.datasource.password=postgres
spring.datasource.driver-class-name=org.postgresql.Driver

# Hibernate ddl auto (create, create-drop, validate, update)
spring.jpa.hibernate.ddl-auto=create
spring.jpa.properties.hibernate.dialect= org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.globally_quoted_identifiers=true
```

### Cat.java

```java
package com.example.springtestingdemo.entity;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import javax.persistence.*;

@Entity
@Getter
@Setter
@NoArgsConstructor
@Table
public class Cat {

    @Id
    @GeneratedValue
    private int id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String breed;

    private int age;

    @ManyToOne
    @JoinColumn(name = "rescue_id")
    private Rescue rescue;
}
```

## Rescue.java

```java
package com.example.springtestingdemo.entity;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
@NoArgsConstructor
@Table
public class Rescue {

    @Id
    @GeneratedValue
    private int id;

    @Column(nullable = false)
    private String name;

    private String address;

    private String city;

    private String state;

    @Column(nullable = false)
    private String websiteUrl;

    @Column(nullable = false)
    private String phoneNumber;

    @OneToMany(mappedBy = "rescue")
    private List<Cat> cats = new ArrayList<>();
}
```

### CatDTO.java

```java
package com.example.springtestingdemo.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.Data;

@Data
public class CatDTO {

    private String name;

    private String breed;

    private int age;

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    private int rescueId;

    @JsonProperty(access = JsonProperty.Access.READ_ONLY)
    private String rescueName;

}
```

### RescueDTO.java

```java
package com.example.springtestingdemo.dto;

import lombok.Data;

@Data
public class RescueDTO {
    private String name;
    private String address;
    private String city;
    private String state;
    private String websiteUrl;
    private String phoneNumber;
}
```

### CatRepository.java

```java
package com.example.springtestingdemo.repository;

import com.example.springtestingdemo.entity.Cat;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface CatRepository extends CrudRepository<Cat, Integer> {
}
```

### RescueRepository.java

```java
package com.example.springtestingdemo.repository;

import com.example.springtestingdemo.entity.Rescue;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface RescueRepository extends CrudRepository<Rescue, Integer> {

    Optional<Rescue> findRescueByName(String name);
}
```

### SpringTestingDemoApplication.java

```java
package com.example.springtestingdemo;

import org.modelmapper.ModelMapper;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class SpringTestingDemoApplication {

   public static void main(String[] args) {
       SpringApplication.run(SpringTestingDemoApplication.class, args);
    }

    // Add a RestTemplate bean
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }

}
```

### CatService.java

```java
package com.example.springtestingdemo.service;

import com.example.springtestingdemo.dto.CatDTO;
import com.example.springtestingdemo.entity.Cat;
import com.example.springtestingdemo.repository.CatRepository;
import lombok.extern.slf4j.Slf4j;
import org.modelmapper.ModelMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.Optional;

@Service
@Slf4j
public class CatService {

    private final ModelMapper modelMapper;
    private final CatRepository catRepository;

    @Autowired
    public CatService(ModelMapper modelMapper, CatRepository catRepository) {
        this.modelMapper = modelMapper;
        this.catRepository = catRepository;
    }

    public String saveCat(CatDTO cat) {
        Cat catEntity = modelMapper.map(cat, Cat.class);
        catRepository.save(catEntity);
        String status = String.format("The cat, %s, has been added!", cat.getName());
        log.info(status);
        return status;
    }

    public CatDTO getCat(Integer id) {
        Optional<Cat> optionalCat = catRepository.findById(id);
        Cat catEntity = optionalCat.orElseThrow();
        log.info(String.format("Retrieved cat with ID %d from the database.", id));
        return modelMapper.map(catEntity, CatDTO.class);
    }
}
```

### RescueService.java

```java
package com.example.springtestingdemo.service;

import com.example.springtestingdemo.dto.RescueDTO;
import com.example.springtestingdemo.entity.Rescue;
import com.example.springtestingdemo.repository.RescueRepository;
import lombok.extern.slf4j.Slf4j;
import org.modelmapper.ModelMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.Optional;

@Service
@Slf4j
public class RescueService {

    private final ModelMapper modelMapper;
    private final RescueRepository rescueRepository;

    @Autowired
    public RescueService(ModelMapper modelMapper, RescueRepository rescueRepository) {
        this.modelMapper = modelMapper;
        this.rescueRepository = rescueRepository;
    }

    public String addRescue(RescueDTO rescue) {
        Rescue rescueEntity = modelMapper.map(rescue, Rescue.class);
        rescueRepository.save(rescueEntity);
        String status = String.format("The rescue, %s, has been added!", rescue.getName());
        log.info(status);
        return status;
    }

    public RescueDTO getRescue(String name) {
        Optional<Rescue> optionalRescue = rescueRepository.findRescueByName(name);
        Rescue rescue = optionalRescue.orElseThrow();
        log.info(String.format("Retrieved %s from the database", name));
        return modelMapper.map(rescue, RescueDTO.class);
    }
}
```

### CatController.java

```java
package com.example.springtestingdemo.controller;

import com.example.springtestingdemo.dto.CatDTO;
import com.example.springtestingdemo.service.CatService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/cat")
public class CatController {

    private final CatService catService;

    @Autowired
    public CatController(CatService catService) {
        this.catService = catService;
    }

    @PostMapping
    public String saveCat(@RequestBody CatDTO cat) {
        return catService.saveCat(cat);
    }

    @GetMapping("/{id}")
    public CatDTO getCat(@PathVariable Integer id) {
        return catService.getCat(id);
    }
}
```

### RescueController.java

```java
package com.example.springtestingdemo.controller;

import com.example.springtestingdemo.dto.RescueDTO;
import com.example.springtestingdemo.service.RescueService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/rescue")
public class RescueController {

    private final RescueService rescueService;

    @Autowired
    public RescueController(RescueService rescueService) {
        this.rescueService = rescueService;
    }

    @PostMapping
    public String addRescue(@RequestBody RescueDTO rescue) {
        return rescueService.addRescue(rescue);
    }

    @GetMapping("/{name}")
    public RescueDTO getRescue(@PathVariable String name) {
        return rescueService.getRescue(name);
    }
}
```

## Testing the Application

Whew! That was a lot of code we just added. Before we get into unit testing
again, let us make sure our application is working as expected and to refresh
our memories on manual testing. For this, we'll use our friend, Postman, again.

Start up the application and open up Postman.

First, we will need to add a rescue to the data source before we can do anything
else. In the request URL, enter "http://localhost:8080/rescue" and make sure
that the request type is POST. We will also want to check the "Headers" tab to
ensure that "Content-Type:application/json" is included. In the body tab, click
the "raw" radio button and select "JSON" from the dropdown. Let's add the
following rescue:

```json
{
  "name": "Happy-Cats",
  "address": "100 Main Street",
  "city": "Manitou Springs",
  "state": "Colorado",
  "websiteUrl": "https://happycatshaven.org/",
  "phoneNumber": "555-123-4567"
}
```

Click "Send" to send the request to our API. Ensure the result looks like the
screenshot below:

![postman-add-rescue](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/postman-add-rescue.png)

We can even check pgAdmin4 to make sure that the rescue has been persisted:

![pgadmin-select-all-rescue](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/pgadmin-select-all-rescue.png)

Now let's test the GET method in the `RescueController`. In Postman, change the
request type to GET and the request URL to: 
http://localhost:8080/rescue/Happy-Cats". Then click "Send" and ensure the
response looks like this:

![postman-get-rescue](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/postman-get-rescue.png)

Great! Now that we have a rescue entity we can add some cats!

Change the request type to POST and the request URL to:
"http://localhost:8080/cat". In the body tab, we'll add the following cat:

```json
{
  "name": "Silky",
  "breed": "Domestic Short Hair",
  "age": 1,
  "rescueId": 1
}
```

Click "Send" to save the cat to the database:

![postman-add-cat](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/postman-add-cat.png)

Let's look to see if we persisted the cat to the database in pgAdmin4:

![pgadmin-select-all-cat](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/pgadmin-select-all-cat.png)

We'll test to make sure we can retrieve the cat now from the database. In
Postman, change the request type to GET and the request URL to:
"http://localhost:8080/cat/2". Then click "Send" and ensure the response looks
like this:

![postman-get-cat](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/postman-get-cat.png)

## Unit Testing and Spring Data

When it comes to writing unit tests with Spring Data, we still want to remove
the dependency of Spring and the database connection from the unit tests as much
as possible. We'll make use of the Mockito framework though to help us mock up
service objects.

Let's write the unit test for the `CatController` together. In the
`CatController`, right-click in the blank space within the class --> Choose
"Generate..." --> Then "Test...". We'll name this test class,
`CatControllerUnitTest`:

```java
package com.example.springtestingdemo.controller;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class CatControllerUnitTest {
    
    @BeforeEach
    void setUp() {
    }

    @Test
    void saveCat() {
    }

    @Test
    void getCat() {
    }
}
```

We'll write the test cases in a similar way to how we wrote them using Mockito.
To set up these tests, let's figure out what we will need:

- A `CatController` instance to test the controller methods.
- A `CatService` mock to instantiate the `CatController`.
- A `CatDTO` to test the posting and retrieving methods.

```java
package com.example.springtestingdemo.controller;

import com.example.springtestingdemo.dto.CatDTO;
import com.example.springtestingdemo.service.CatService;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.when;

class CatControllerUnitTest {

    private CatController catController;
    private CatService catService;
    private CatDTO testCat;

    @BeforeEach
    void setUp() {
        catService = Mockito.mock(CatService.class);
        catController = new CatController(catService);

        testCat = new CatDTO();
        testCat.setName("Grumpy Cat");
        testCat.setBreed("DSH");
        testCat.setAge(7);
        testCat.setRescueId(1);
    }

    @Test
    void saveCat() {
    }

    @Test
    void getCat() {
    }
}
```

Now let's write the `saveCat()` test method. The `saveCat()` method returns a
`String` object, so we'll need to make sure our mock `catService` also returns
a `String`. We'll also pass in the `testCat` to save:

```java
package com.example.springtestingdemo.controller;

import com.example.springtestingdemo.dto.CatDTO;
import com.example.springtestingdemo.service.CatService;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.when;

class CatControllerUnitTest {

    private CatController catController;
    private CatService catService;
    private CatDTO testCat;

    @BeforeEach
    void setUp() {
        catService = Mockito.mock(CatService.class);
        catController = new CatController(catService);

        testCat = new CatDTO();
        testCat.setName("Grumpy Cat");
        testCat.setBreed("DSH");
        testCat.setAge(7);
        testCat.setRescueId(1);
    }

    @Test
    void saveCat() {
        String success = "The cat, Grumpy Cat, has been added!";
        when(catService.saveCat(testCat)).thenReturn(success);
        assertEquals(success, catController.saveCat(testCat));
    }

    @Test
    void getCat() {
    }
}
```

Let's write the test `getCat()` method now! The `getCat()` method returns a
`CatDTO` object, so we'll have it return the `testCat` that we created in the
setup for ease:

```java
package com.example.springtestingdemo.controller;

import com.example.springtestingdemo.dto.CatDTO;
import com.example.springtestingdemo.service.CatService;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.when;

class CatControllerUnitTest {

    private CatController catController;
    private CatService catService;
    private CatDTO testCat;

    @BeforeEach
    void setUp() {
        catService = Mockito.mock(CatService.class);
        catController = new CatController(catService);

        testCat = new CatDTO();
        testCat.setName("Grumpy Cat");
        testCat.setBreed("DSH");
        testCat.setAge(7);
        testCat.setRescueId(1);
    }

    @Test
    void saveCat() {
        String success = "The cat, Grumpy Cat, has been added!";
        when(catService.saveCat(testCat)).thenReturn(success);
        assertEquals(success, catController.saveCat(testCat));
    }

    @Test
    void getCat() {
        when(catService.getCat(1)).thenReturn(testCat);
        assertEquals(testCat, catController.getCat(1));
    }
}
```

And that's really it! Simple enough, right? Run the unit test and ensure it
passes.

Go ahead and try to write the unit test class for the `RescueController` on your
own! Name the unit test class:`RescueControllerUnitTest`. Once you've written
the unit test, try running it and making sure that it passes.

## Integration Testing and Spring Data

The integration testing will mostly work the same for the controller classes.
But in this lesson, let's consider this: testing the repository's interaction
with the database.

When testing, we don't want to connect our tests to the database we are using
in production, as that could have consequences that we do not want to deal with.
Instead, we'll connect our tests to an in-memory database, like an H2 database.
This will self-contain the persistence testing. To learn more about H2
databases, see [H2 Database Engine](https://www.h2database.com/html/main.html).

To use the H2 database, we'll need to add this dependency to our `pom.xml` file:

```xml
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
```

Make sure to add the dependency within the `<dependencies>` section. Also
remember to click the Maven icon in the upper-right to reload the changes.

Let's create an `application-test.properties` file. In the test directory,
create the `resources` directory and then an `application-test.properties` file:

```text
    └── test
        ├── java
        │   └── org
        │        └── example
        │             └── springtestingdemo
        │                   ├── SpringTestingDemoApplicationTests.java
        │                   └──controller
        │                       ├──CatControllerUnitTest.java
        │                       ├──DemoControllerAcceptanceTest.java
        │                       ├──DemoControllerIntegrationTest.java
        │                       ├──DemoControllerUnitTest.java
        │                       └──RescueControllerUnitTest.java                     
        └── resources
             └── application-test.properties        
```

Add the following content to the `application-test.properties` file:

```properties
spring.datasource.url = jdbc:h2:mem:test
spring.datasource.driverClassName=org.h2.Driver
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.H2Dialect

spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=create-drop
```

Note: If we wanted to, we could have connected this to a test PostgreSQL
database instead by adding those properties here, just like we would in the
`application.properties` file.

Now we'll generate an integration test for the `CatRepository`. In the
`CatRepository`, right-click in the blank space within the interface --> Choose
"Generate..." --> Then "Test...". We'll name this test class,
`CatRepositoryIntegrationTest`:

```java
package com.example.springtestingdemo.repository;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;

import static org.junit.jupiter.api.Assertions.*;

class CatRepositoryIntegrationTest {

    @BeforeEach
    void setUp() {
    }

    @AfterEach
    void tearDown() {
    }
}
```

Now for this test, we want to actually test whether the data is being persisted
and can be retrieved correctly. We'll need to connect it to a test database as
well. Let's add the following annotations to the test class definition:

```java
package com.example.springtestingdemo.repository;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.TestPropertySource;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@TestPropertySource(locations = "classpath:application-test.properties")
class CatRepositoryIntegrationTest {

    @BeforeEach
    void setUp() {
    }

    @AfterEach
    void tearDown() {
    }
}
```

The `@DataJpaTest` annotation will disable full auto-configuration and apply
only the configurations necessary and relevant to JPA tests. This annotation
also sets up:

- Hibernate and Spring Data.
- Turns on SQL logging.
- Performs an `@EntityScan` to scan for any entities.

We want to make use of the properties we have defined in our test properties
file we just created. So we'll go ahead and add two more annotations to the
class definition: `@AutoConfigureTestDatabase` and `@TestPropertySource`.

The `@AutoConfigureTestDatabase` annotation can be applied so we can specify
whether or not we want a database to be auto-configured or not. Since we want
to use the properties in our test properties file, we'll add
`(replace - AutoConfigureTestDatabase.Replace.NONE)` to the annotation.

Then we can specify we want the test to use a certain set of properties with the
`@TestPropertySource` annotation. We can then provide it a location of the
properties file we want this test class to use by specifying
`(locations = "classpath:application-test.properties)`.

Now let's think of some of the objects we will need to test the `CatRepository`.
We know we will need an instance of a `CatRepository` and a `Cat` entity for
testing:

```java
package com.example.springtestingdemo.repository;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.TestPropertySource;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@TestPropertySource(locations = "classpath:application-test.properties")
class CatRepositoryIntegrationTest {

    @Autowired
    private CatRepository catRepository;

    private Cat testCat;

    @BeforeEach
    void setUp() {
    }

    @AfterEach
    void tearDown() {
    }
}
```

Let's do a little setup to create the `testCat` object:

```java
package com.example.springtestingdemo.repository;

import com.example.springtestingdemo.entity.Cat;
import com.example.springtestingdemo.entity.Rescue;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.TestPropertySource;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@TestPropertySource(locations = "classpath:application-test.properties")
class CatRepositoryIntegrationTest {

    @Autowired
    private CatRepository catRepository;

    private Cat testCat;

    @BeforeEach
    void setUp() {

        // Create a rescue to test with
        Rescue testRescue = new Rescue();
        testRescue.setName("Humane-Society");
        testRescue.setAddress("42 Wallaby Way");
        testRescue.setCity("Morristown");
        testRescue.setState("AZ");
        testRescue.setWebsiteUrl("www.humanesociety.com");
        testRescue.setPhoneNumber("555-123-4567");

        // Create a cat to test with
        testCat = new Cat();
        testCat.setName("Grumpy Cat");
        testCat.setBreed("DSH");
        testCat.setAge(7);
        testCat.setRescue(testRescue);
    }

    @AfterEach
    void tearDown() {
    }
}
```

Now that we have our `Cat` entity ready to persist, let's add a test method
called `testSave()` to ensure our `Cat` can be persisted properly to a data
source.

```java
package com.example.springtestingdemo.repository;

import com.example.springtestingdemo.entity.Cat;
import com.example.springtestingdemo.entity.Rescue;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.TestPropertySource;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@TestPropertySource(locations = "classpath:application-test.properties")
class CatRepositoryIntegrationTest {

  @Autowired
  private CatRepository catRepository;

  private Cat testCat;

  @BeforeEach
  void setUp() {

    // Create a rescue to test with
    Rescue testRescue = new Rescue();
    testRescue.setName("Humane-Society");
    testRescue.setAddress("42 Wallaby Way");
    testRescue.setCity("Morristown");
    testRescue.setState("AZ");
    testRescue.setWebsiteUrl("www.humanesociety.com");
    testRescue.setPhoneNumber("555-123-4567");

    // Create a cat to test with
    testCat = new Cat();
    testCat.setName("Grumpy Cat");
    testCat.setBreed("DSH");
    testCat.setAge(7);
    testCat.setRescue(testRescue);
  }

  @AfterEach
  void tearDown() {
  }


  @Test
  void save() {
    assertEquals(testCat, catRepository.save(testCat));
  }
}
```

Remember, the `save()` method will return a `Cat` entity in this case, so we can
easily perform an `assertEquals()` within this test method.

Now let's write a method to test the retrieving of a `Cat` entity with a test
method called `testFindById()`:

```java
package com.example.springtestingdemo.repository;

import com.example.springtestingdemo.entity.Cat;
import com.example.springtestingdemo.entity.Rescue;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.TestPropertySource;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@TestPropertySource(locations = "classpath:application-test.properties")
class CatRepositoryIntegrationTest {

    @Autowired
    private CatRepository catRepository;

    private Cat testCat;

    @BeforeEach
    void setUp() {

        // Create a rescue to test with
        Rescue testRescue = new Rescue();
        testRescue.setName("Humane-Society");
        testRescue.setAddress("42 Wallaby Way");
        testRescue.setCity("Morristown");
        testRescue.setState("AZ");
        testRescue.setWebsiteUrl("www.humanesociety.com");
        testRescue.setPhoneNumber("555-123-4567");

        // Create a cat to test with
        testCat = new Cat();
        testCat.setName("Grumpy Cat");
        testCat.setBreed("DSH");
        testCat.setAge(7);
        testCat.setRescue(testRescue);
    }

    @AfterEach
    void tearDown() {
    }


    @Test
    void save() {
        assertEquals(testCat, catRepository.save(testCat));
    }

    @Test
    void findById() {

        // First save the cat in the repository
        catRepository.save(testCat);

        // Now test to see if we can retrieve the cat by its ID
        Optional<Cat> optionalCat = catRepository.findById(1);
        assertTrue(optionalCat.isPresent());
        assertEquals(testCat, optionalCat.get());

    }
}
```

Run the `CatRepositoryIntegrationTest` and make sure that our two test methods
pass!

One last thing we should do here is cleanup after our tests. Just to be sure, we
will delete the `testCat` from the data source. This way we know the database
table is empty before we try to save our `Cat` object:

```java
package com.example.springtestingdemo.repository;

import com.example.springtestingdemo.entity.Cat;
import com.example.springtestingdemo.entity.Rescue;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.TestPropertySource;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@TestPropertySource(locations = "classpath:application-test.properties")
class CatRepositoryIntegrationTest {

    @Autowired
    private CatRepository catRepository;

    private Cat testCat;

    @BeforeEach
    void setUp() {

        // Create a rescue to test with
        Rescue testRescue = new Rescue();
        testRescue.setName("Humane-Society");
        testRescue.setAddress("42 Wallaby Way");
        testRescue.setCity("Morristown");
        testRescue.setState("AZ");
        testRescue.setWebsiteUrl("www.humanesociety.com");
        testRescue.setPhoneNumber("555-123-4567");

        // Create a cat to test with
        testCat = new Cat();
        testCat.setName("Grumpy Cat");
        testCat.setBreed("DSH");
        testCat.setAge(7);
        testCat.setRescue(testRescue);
    }

    @AfterEach
    void tearDown() {
        catRepository.delete(testCat);
    }


    @Test
    void save() {
        assertEquals(testCat, catRepository.save(testCat));
    }

    @Test
    void findById() {

        // First save the cat in the repository
        catRepository.save(testCat);

        // Now test to see if we can retrieve the cat by its ID
        Optional<Cat> optionalCat = catRepository.findById(1);
        assertTrue(optionalCat.isPresent());
        assertEquals(testCat, optionalCat.get());

    }
}
```

Run the integration test again and ensure it passes!

On your own, try to write the integration test class for the `RescueRepository`!
Name the test class `RescueRepositoryIntegrationTest`.

## Acceptance Testing and Spring Data

Moving onto acceptance testing. In our acceptance tests with Spring Data, we
want to test all of our endpoints in our cat rescue model by initializing the
entire Spring Framework. We'll go ahead and create a test class called
`CatRescueAcceptanceTest` in our `com.example.springtestingdemo` package. For
this particular test, we will _not_ generate the test like we did before since
this test will encompass testing the `CatController` and the `RescueController`.

```java
package com.example.springtestingdemo;

class CatRescueAcceptanceTest {

}
```

Since we want to imitate a real-world scenario, we'll want to make use of the
`@SpringBootTest` and `@AutoConfigureMockMvc` annotations again to initialize
the entire Spring Framework while also configuring auto-configuration of
`MockMvc`. We will also annotate the class with the `@TestPropertySource`
again to specify that we want to use an in-memory database for testing purposes.

```java
package com.example.springtestingdemo;

import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.TestPropertySource;

@SpringBootTest
@AutoConfigureMockMvc
@TestPropertySource(locations = "classpath:application-test-properties")
class CatRescueAcceptanceTest {

}
```

Like before in our other acceptance test, we'll autowire a `MockMvc` bean in
our test class:

```java
package com.example.springtestingdemo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.TestPropertySource;
import org.springframework.test.web.servlet.MockMvc;

@SpringBootTest
@AutoConfigureMockMvc
@TestPropertySource(locations = "classpath:application-test-properties")
class CatRescueAcceptanceTest {
    
    @Autowired
    private MockMvc mockMvc;

}
```

Now let us think about what else we might need in these tests. We'll need a 
rescue to persist to the database along with a cat. So let's create define a
`RescueDTO` and a `CatDTO` object to create within a `setUp()` method:

```java
package com.example.springtestingdemo;

import com.example.springtestingdemo.dto.CatDTO;
import com.example.springtestingdemo.dto.RescueDTO;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.TestPropertySource;
import org.springframework.test.web.servlet.MockMvc;

@SpringBootTest
@AutoConfigureMockMvc
@TestPropertySource(locations = "classpath:application-test.properties")
class CatRescueAcceptanceTest {

    @Autowired
    private MockMvc mockMvc;

    private RescueDTO testRescue;
    private CatDTO testCat;

    @BeforeEach
    void setUp() {

        testRescue = new RescueDTO();
        testRescue.setName("Humane-Society");
        testRescue.setAddress("42 Wallaby Way");
        testRescue.setCity("Morristown");
        testRescue.setState("AZ");
        testRescue.setWebsiteUrl("www.humanesociety.com");
        testRescue.setPhoneNumber("555-123-4567");

        testCat = new CatDTO();
        testCat.setName("Grumpy Cat");
        testCat.setBreed("DSH");
        testCat.setAge(7);
        testCat.setRescueName(testRescue.getName());
    }
}
```

Before we start writing the test methods, let us pause and consider how we want
to test. We're working with a database and no mock controllers or services. We
also want to consider the relationship between our entities. Remember, a rescue
can have many cats, but a cat will only belong to one rescue. This means, before
we save any cats to the database, we _need_ to have a rescue in the database.

Something we are allowed to do in JUnit5 is order how we want our tests to run.
Normally, tests can run in any order, but we want to control the order a little
bit so we don't run a test method that saves a cat before adding a rescue to the
data source. To do this, we can use the `@TestMethodOrder` and the `@Order`
annotations. The `@TestMethodOrder` is an annotation applied at the class level
and the `@Order` annotation is applied at the method level with an order number
assigned to each method:

```java
package com.example.springtestingdemo;

import com.example.springtestingdemo.dto.CatDTO;
import com.example.springtestingdemo.dto.RescueDTO;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.TestPropertySource;
import org.springframework.test.web.servlet.MockMvc;


@SpringBootTest
@AutoConfigureMockMvc
@TestPropertySource(locations = "classpath:application-test.properties")
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class CatRescueAcceptanceTest {

    @Autowired
    private MockMvc mockMvc;

    private RescueDTO testRescue;
    private CatDTO testCat;

    @BeforeEach
    void setUp() {

        testRescue = new RescueDTO();
        testRescue.setName("Humane-Society");
        testRescue.setAddress("42 Wallaby Way");
        testRescue.setCity("Morristown");
        testRescue.setState("AZ");
        testRescue.setWebsiteUrl("www.humanesociety.com");
        testRescue.setPhoneNumber("555-123-4567");

        testCat = new CatDTO();
        testCat.setName("Grumpy Cat");
        testCat.setBreed("DSH");
        testCat.setAge(7);
        testCat.setRescueName(testRescue.getName());
    }

    @Test
    @Order(1)    // Run this test first
    void addRescue() {
    }

    @Test
    @Order(2)    // run this test second
    void getRescue() {
    }

    @Test
    @Order(3)    // run this test third
    void saveCat() {
    }

    @Test
    @Order(4)    // run this test fourth
    void getCat() {
    }
}
```

As we can see above, we will have the tests run in the order of adding a rescue,
retrieving a rescue, saving a cat, and then retrieving a cat from the data
source.

Now that we have the order defined, let's go ahead and write the first test
method, `addRescue()`:

```java
package com.example.springtestingdemo;

import com.example.springtestingdemo.dto.CatDTO;
import com.example.springtestingdemo.dto.RescueDTO;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.TestPropertySource;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;

import static org.junit.jupiter.api.Assertions.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
@TestPropertySource(locations = "classpath:application-test.properties")
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class CatRescueAcceptanceTest {

    @Autowired
    private MockMvc mockMvc;

    private RescueDTO testRescue;
    private CatDTO testCat;

    @BeforeEach
    void setUp() {

        testRescue = new RescueDTO();
        testRescue.setName("Humane-Society");
        testRescue.setAddress("42 Wallaby Way");
        testRescue.setCity("Morristown");
        testRescue.setState("AZ");
        testRescue.setWebsiteUrl("www.humanesociety.com");
        testRescue.setPhoneNumber("555-123-4567");

        testCat = new CatDTO();
        testCat.setName("Grumpy Cat");
        testCat.setBreed("DSH");
        testCat.setAge(7);
        testCat.setRescueName(testRescue.getName());
    }

    @Test
    @Order(1)    // Run this test first
    void addRescue() throws Exception {

        // Serialize the testRescue into a JSON as a String
        ObjectMapper objectMapper = new ObjectMapper();
        String rescueJson = objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(testRescue);

        MvcResult response =
                mockMvc.perform(post("/rescue").contentType(MediaType.APPLICATION_JSON).content(rescueJson))
                        .andDo(print())
                        .andExpect(status().isOk())
                        .andReturn();
        String status = response.getResponse().getContentAsString();
        assertNotNull(status);
        assertEquals("The rescue, Humane-Society, has been added!", status);
    }

    @Test
    @Order(2)    // run this test second
    void getRescue() {
    }

    @Test
    @Order(3)    // run this test third
    void saveCat() {
    }

    @Test
    @Order(4)    // run this test fourth
    void getCat() {
    }
}
```

Let's break down what is happening in the `addRescue()` test method:

- When we are POSTing data to the database and hitting the endpoint `/rescue`,
  we are expecting to get a `RescueDTO` in JSON format. Think about when we
  would use Postman for testing, we always write the request body in JSON
  format. So we'll need to do the same here to imitate a real-world scenario.
  If we recall from the "JSON" lesson a few modules back, we can use something
  called an `ObjectMapper` to write our data out as a JSON, which is exactly
  what we are doing in the line:
  `String rescueJson = objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(testRescue);`.
- Since we are doing a POST instead of a GET, we'll use the `post()` method and
  pass in the appropriate URL: `"/rescue`.
  - Along with this, we will need to specify the contentType,
    `MediaType.APPLICATION_JSON`, which we may recall setting up in Postman
    when testing POST request types.
  - Once we specified the CRUD request type as POST and the content type, we
    can pass in the content in JSON form.
- We will then proceed as we had in the integration tests where we assert the
  status `String` object that is returned.

Run the test and ensure that everything works before moving onto writing the
test code for the `getRescue()` test method.

We will also use the `ObjectMapper` now in both of our test methods thus far.
So, we can move that up to the `setUp()` method instead to instantiate. Consider
the following code changes:


```java
package com.example.springtestingdemo;

import com.example.springtestingdemo.dto.CatDTO;
import com.example.springtestingdemo.dto.RescueDTO;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.TestPropertySource;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;

import static org.junit.jupiter.api.Assertions.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
@TestPropertySource(locations = "classpath:application-test.properties")
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class CatRescueAcceptanceTest {

  @Autowired
  private MockMvc mockMvc;

  private RescueDTO testRescue;
  private CatDTO testCat;

  private ObjectMapper objectMapper;

  @BeforeEach
  void setUp() {

    testRescue = new RescueDTO();
    testRescue.setName("Humane-Society");
    testRescue.setAddress("42 Wallaby Way");
    testRescue.setCity("Morristown");
    testRescue.setState("AZ");
    testRescue.setWebsiteUrl("www.humanesociety.com");
    testRescue.setPhoneNumber("555-123-4567");

    testCat = new CatDTO();
    testCat.setName("Grumpy Cat");
    testCat.setBreed("DSH");
    testCat.setAge(7);
    testCat.setRescueName(testRescue.getName());

    objectMapper = new ObjectMapper();

  }

  @Test
  @Order(1)    // Run this test first
  void addRescue() throws Exception {

    // Serialize the testRescue into a JSON as a String
    String rescueJson = objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(testRescue);

    MvcResult response =
            mockMvc.perform(post("/rescue").contentType(MediaType.APPLICATION_JSON).content(rescueJson))
                    .andDo(print())
                    .andExpect(status().isOk())
                    .andReturn();
    String status = response.getResponse().getContentAsString();
    assertNotNull(status);
    assertEquals("The rescue, Humane-Society, has been added!", status);
  }

  @Test
  @Order(2)    // run this test second
  void getRescue() throws Exception {

    MvcResult response = mockMvc.perform(get("/rescue/Humane-Society"))
            .andDo(print())
            .andExpect(status().isOk())
            .andReturn();

    String rescueJson = response.getResponse().getContentAsString();
    assertNotNull(rescueJson);
    RescueDTO rescueDTO = objectMapper.readValue(rescueJson, RescueDTO.class);
    assertEquals(testRescue, rescueDTO);
  }

  @Test
  @Order(3)    // run this test third
  void saveCat() {
  }

  @Test
  @Order(4)    // run this test fourth
  void getCat() {
  }
}
```

In the `getRescue()` test method, we will get a JSON back in response. This
JSON should just be the serialized version of the `testRescue` that we created
above. We can use the `objectMapper` again to convert the JSON to a `RescueDTO`
for us to assert and compare.

Run the tests again and ensure that both of these test methods pass!

Now we'll move onto writing our last two test methods:

```java
package com.example.springtestingdemo;

import com.example.springtestingdemo.dto.CatDTO;
import com.example.springtestingdemo.dto.RescueDTO;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.TestPropertySource;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;

import static org.junit.jupiter.api.Assertions.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
@TestPropertySource(locations = "classpath:application-test.properties")
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class CatRescueAcceptanceTest {

    @Autowired
    private MockMvc mockMvc;

    private RescueDTO testRescue;
    private CatDTO testCat;

    private ObjectMapper objectMapper;

    @BeforeEach
    void setUp() {

        testRescue = new RescueDTO();
        testRescue.setName("Humane-Society");
        testRescue.setAddress("42 Wallaby Way");
        testRescue.setCity("Morristown");
        testRescue.setState("AZ");
        testRescue.setWebsiteUrl("www.humanesociety.com");
        testRescue.setPhoneNumber("555-123-4567");

        testCat = new CatDTO();
        testCat.setName("Grumpy Cat");
        testCat.setBreed("DSH");
        testCat.setAge(7);
        testCat.setRescueName(testRescue.getName());

        objectMapper = new ObjectMapper();

    }

    @Test
    @Order(1)    // Run this test first
    void addRescue() throws Exception {

        // Serialize the testRescue into a JSON as a String
        String rescueJson = objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(testRescue);

        MvcResult response =
                mockMvc.perform(post("/rescue").contentType(MediaType.APPLICATION_JSON).content(rescueJson))
                        .andDo(print())
                        .andExpect(status().isOk())
                        .andReturn();
        String status = response.getResponse().getContentAsString();
        assertNotNull(status);
        assertEquals("The rescue, Humane-Society, has been added!", status);
    }

    @Test
    @Order(2)    // run this test second
    void getRescue() throws Exception {

        MvcResult response = mockMvc.perform(get("/rescue/Humane-Society"))
                .andDo(print())
                .andExpect(status().isOk())
                .andReturn();

        String rescueJson = response.getResponse().getContentAsString();
        assertNotNull(rescueJson);
        RescueDTO rescueDTO = objectMapper.readValue(rescueJson, RescueDTO.class);
        assertEquals(testRescue, rescueDTO);
    }

    @Test
    @Order(3)    // run this test third
    void saveCat() throws Exception{

        String catJson = "{\n" +
                "  \"name\" : \"Grumpy Cat\",\n" +
                "  \"breed\" : \"DSH\",\n" +
                "  \"age\" : 7,\n" +
                "  \"rescueId\" : 1\n" +
                "}";

        MvcResult response =
                mockMvc.perform(post("/cat").contentType(MediaType.APPLICATION_JSON).content(catJson))
                        .andDo(print())
                        .andExpect(status().isOk())
                        .andReturn();
        String status = response.getResponse().getContentAsString();
        assertNotNull(status);
        assertEquals("The cat, Grumpy Cat, has been added!", status);
    }

    @Test
    @Order(4)    // run this test fourth
    void getCat() throws Exception {

        MvcResult response = mockMvc.perform(get("/cat/2"))
                .andDo(print())
                .andExpect(status().isOk())
                .andReturn();

        String catJson = response.getResponse().getContentAsString();
        assertNotNull(catJson);
        CatDTO catDTO = objectMapper.readValue(catJson, CatDTO.class);
        catDTO.setRescueName(testRescue.getName());
        assertEquals(testCat, catDTO);
    }
}
```

Consider the following points regarding the last two methods:

- Remember in our `CatDTO` class, we have the following lines of code:

  ```java
    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    private int rescueId;

    @JsonProperty(access = JsonProperty.Access.READ_ONLY)
    private String rescueName;
  ```
  
  Due to the JSON properties we have set in this class, if we use the
  `objectMapper` to write out the `testCat` object, it will _not_ write out a
  `rescueId`. Since we _want_ this to be true in a real-world scenario, we do
  not want to change the code in the `CatDTO` class. Instead, we'll write out
  the JSON as a `String` in a more manual way in the `saveCat()` test method.
- Once the cat is persisted, it should have an ID value of 2 due to the
  `@GeneratedValue` annotation used in the `Cat` entity class.
- When we retrieve the `catJson` in the `getCat()` test method, we will get back
  a JSON like this:

  ```json
  {
    "name": "Grumpy Cat",
    "breed": "DSH",
    "age": 7,
    "rescueName": "Humane-Society"
  }
  ```
  
  When we read this in with the `objectMapper`, the `rescueName` will not be
  written to, per the JSON properties in the class. Again, since we do not want
  to change these properties, as this is appropriate in a real-life scenario, we
  will set the `rescueName` in the following line and then compare the `testCat`
  and `catDTO` objects.

## Check the Project Structure

```text
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── springtestingdemo
    │   │               ├── SpringTestingDemoApplication.java
    │   │               ├── controller
    │   │               │  ├──  CatController.java
    │   │               │  ├──  DemoController.java
    │   │               │  └──  RescueController.java    
    │   │               ├── dto
    │   │               │  ├── CatDTO.java    
    │   │               │  ├── CatFactDTO.java
    │   │               │  └── RescueDTO.java
    │   │               ├── entity
    │   │               │  ├── Cat.java    
    │   │               │  └── Rescue.java  
    │   │               ├── repository
    │   │               │  ├── CatRepository.java    
    │   │               │  └── RescueRepository.java                
    │   │               └── service
    │   │                   ├── CatService.java    
    │   │                   ├── CatFactService.java
    │   │                   └── RepositoryService.java    
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        ├── java
        │   └── org
        │        └── example
        │             └── springtestingdemo
        │                   ├── CatRescueAcceptanceTest.java
        │                   ├── SpringTestingDemoApplicationTests.java
        │                   ├──controller
        │                   │  ├──CatControllerIntegrationTest.java
        │                   │  ├──CatControllerUnitTest.java
        │                   │  ├──DemoControllerAcceptanceTest.java
        │                   │  ├──DemoControllerIntegrationTest.java
        │                   │  ├──DemoControllerUnitTest.java
        │                   │  ├──RescueControllerIntegrationTest.java
        │                   │  └──RescueControllerUnitTest.java
        │                   └──repository
        │                        ├──CatRepositoryIntegrationTest.java
        │                        └──RescueRepositoryIntegrationTest.java                        
        └── resources
             └── application-test.properties        
```

Note: The `CatControllerIntegrationTest` and the
`RescueControllerIntegrationTest` were not covered in this lesson; but we could
create these in a similar fashion to how we created the previous controller
integration tests. The `RescueControllerUnitTest` and the
`RescueRepositoryIntegrationTest` are classes for you to try and write on your
own!

## Conclusion

Let's remember:

- **Unit testing** is a specific kind of testing that focuses on validating the
  functionality of a single component of the system. It tests a small, specific
  piece of functionality.
- **Integration testing** is a type of testing that looks at multiple components
  of a program and tests how well these modules may work together.
- **Acceptance testing** ensures that all the layers of an API are tested and
  functions the way it is expected by the end users.

As we can see, the integration tests and acceptance test can be a little more
complicated when we add Spring Data into the mix. In this lessson, we learned
how to handle the persisting of data to an in-memory database in both the
integration tests and the acceptance test. We also saw how similar the unit
tests were compared to when Spring Data was not a factor.

## Resources

- [DataJpaTest Annotation Documentation](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/orm/jpa/DataJpaTest.html)
- [AutoConfigureTestDatabase Annotation Documentation](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/jdbc/AutoConfigureTestDatabase.html)
- [TestPropertySource Annotation Documentation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/TestPropertySource.html)
- [TestMethodOrder Annotation Documentation](https://junit.org/junit5/docs/5.5.0/api/org/junit/jupiter/api/TestMethodOrder.html)
- [Order Annotation Documentation](https://junit.org/junit5/docs/5.5.0/api/org/junit/jupiter/api/Order.html)
