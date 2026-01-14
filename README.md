# Sample Spring Boot Application

This repository contains a minimal Java Spring Boot application scaffold.

## Files created

- [pom.xml](pom.xml)
- [src/main/java/com/example/demo/DemoApplication.java](src/main/java/com/example/demo/DemoApplication.java)
- [src/main/java/com/example/demo/controller/HelloController.java](src/main/java/com/example/demo/controller/HelloController.java)
- [src/main/resources/application.properties](src/main/resources/application.properties)

## Build and run

Build and run the app with Maven:

```bash
mvn -q spring-boot:run
```

Or build a jar and run it:

```bash
mvn -q package
java -jar target/demo-0.0.1-SNAPSHOT.jar
```

Then open http://localhost:8080/api/hello to see the JSON greeting.

