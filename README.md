# Spring 3.0 upgrade

This is an overview of Spring Boot 3.0 upgrade.
This video is quite useful:
https://www.youtube.com/watch?v=TR254zh-f3c

### Main points:

- Java 17
- Jakarta EE
- Observability
- Native Images using Graal VM

This blog gives an overview of Spring Boot 3.0 going GA:
https://spring.io/blog/2022/11/24/spring-boot-3-0-goes-ga

These release notes are useful:
https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Release-Notes

We use start.spring.io to generate a new project:
![image](https://user-images.githubusercontent.com/27693622/233359070-6bcaa88a-2f42-4629-ba08-89acf78580af.png)

The main starting point is that we have Java 17. This is quite useful on Java 17:
https://www.youtube.com/watch?v=U14IA5XiX1I

This list of the Java history is also useful:
https://openjdk.org/jeps

This table is quite useful for the differences between Java 11 and Java 17:

| Java 11                                      | Java 17                                    | description                                                                                                                       | JEP link                     |
|----------------------------------------------|--------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|------------------------------|
| No text blocks                               | Text blocks                                | Tuple quotes. For whitespace put quotes to where you want the whitespace to be ignored.                                           | https://openjdk.org/jeps/378 |
| Switch Expressions space and colon           | Switch with expressions via arrow operator | No need for break statements, switch can return result using yield keyword                                                        | https://openjdk.org/jeps/361 |
| Pattern matching not available               | Pattern matching available                 | We can now refer to a new pattern variable in the instanceof check and then use that variable within the if block scope           | https://openjdk.org/jeps/394 |
| We cannot generate sealed classes in java 11 | We can generate sealed classes in java 17  | Specify what can inherit the interface or abstract class using the permits keyword. We can also use non-sealed to open extensions | https://openjdk.org/jeps/409 |
| Records are not available                    | Records are available                      | Reduce boiler plate code for getters setters, constructors for transparent modelling of data. Records are implicitly final.       | https://openjdk.org/jeps/395 |

This table is useful for Runtime improvements:

| Java 11                       | Java 17                        | description                                                                                                              | JEP link                                            |
|-------------------------------|--------------------------------|--------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------|
| No support for CDS updates    | Support for CDS                | Class Data Sharing - shared memory when running on the same host. Reduce start up time and memory footprint between JVMs | https://openjdk.org/jeps/378                        |
| GC garbage collection         | Support for Z GC               | Redesign of garbage collection. Much more scalable and less latency (-XX:+UseZGC -Xmx<size> -Xlog:gc)                    | https://openjdk.org/jeps/377                        |
| Default NullPointerExceptions | Helpful NullPointer Exceptions | More context for NullPointerExceptions - reduces the need for debugging                                                  | https://openjdk.org/jeps/358                        |
| No AArch64 support            | Aarch64 support                | Arm support for better performance                                                                                       | https://openjdk.org/jeps/391                        |
| Slower start up               | Better general performance     | Spring Boot Petclinic 2.4.5 startup time improved by 2 seconds                                                           | https://github.com/spring-projects/spring-petclinic |


There are some deprecations for Java 17:
- Security Manager (JEP 411)
- Applet API (JEP 398)

Removals:
- Nashorn (Javascript Engine) - (JEP 372)
- CMS Garbage Collector - (JEP 363)

Other changes:
- Strongly encapsulated JDK Internals - continued from start of module system
  - no relax on --illegal-access=permit
  - Some internal APIs remain available e.g. sun.misc.Unsafe
  - Can still declare --add-opens for reflective access

Nice list of new methods:
https://docs.oracle.com/en/java/javase/17/docs/api/new-list.html



When we run the application, the logs show that we are using Tomcat 10.1.7:

```bash
2023-04-20T12:55:47.549+01:00  INFO 12058 --- [           main] c.tomspencerlondon.blog.BlogApplication  : Starting BlogApplication using Java 19.0.2 with PID 12058 (/home/tom/Projects/SpringBoot3Upgrade/blog/target/classes started by tom in /home/tom/Projects/SpringBoot3Upgrade/blog)
2023-04-20T12:55:47.558+01:00  INFO 12058 --- [           main] c.tomspencerlondon.blog.BlogApplication  : No active profile set, falling back to 1 default profile: "default"
2023-04-20T12:55:48.702+01:00  INFO 12058 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
2023-04-20T12:55:48.720+01:00  INFO 12058 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 10 ms. Found 0 JPA repository interfaces.
2023-04-20T12:55:49.287+01:00  INFO 12058 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-04-20T12:55:49.301+01:00  INFO 12058 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-04-20T12:55:49.302+01:00  INFO 12058 --- [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.7]
2023-04-20T12:55:49.400+01:00  INFO 12058 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-04-20T12:55:49.400+01:00  INFO 12058 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1609 ms
2023-04-20T12:55:49.619+01:00  INFO 12058 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2023-04-20T12:55:49.786+01:00  INFO 12058 --- [           main] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Added connection conn0: url=jdbc:h2:mem:c76b5e83-8637-4ddc-b45b-93bca7aa912b user=SA
2023-04-20T12:55:49.788+01:00  INFO 12058 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2023-04-20T12:55:49.830+01:00  INFO 12058 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
2023-04-20T12:55:49.888+01:00  INFO 12058 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 6.1.7.Final
2023-04-20T12:55:50.249+01:00  INFO 12058 --- [           main] SQL dialect                              : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
2023-04-20T12:55:50.461+01:00  INFO 12058 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
2023-04-20T12:55:50.473+01:00  INFO 12058 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2023-04-20T12:55:50.517+01:00  WARN 12058 --- [           main] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2023-04-20T12:55:50.890+01:00  INFO 12058 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2023-04-20T12:55:50.897+01:00  INFO 12058 --- [           main] c.tomspencerlondon.blog.BlogApplication  : Started BlogApplication in 4.234 seconds (process running for 5.634)
```
