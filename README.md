[![Build status](https://travis-ci.org/sdeleuze/spring-kotlin.svg?branch=master)](https://travis-ci.org/sdeleuze/spring-kotlin)

`spring-kotlin` is a repository that contains Kotlin extensions, documentation,
and any helpful assets that can help to build Spring + Kotlin applications. 

This project has been created after [this discussion](https://github.com/spring-projects/spring-boot/issues/5537) on Spring Boot issue tracker.

You will find  more information about using Spring Boot + Kotlin in these 2 blog posts:
 - [Developing Spring Boot applications with Kotlin](https://spring.io/blog/2016/02/15/developing-spring-boot-applications-with-kotlin)
 - [A Geospatial Messenger with Kotlin, Spring Boot and PostgreSQL](https://spring.io/blog/2016/03/20/a-geospatial-messenger-with-kotlin-spring-boot-and-postgresql)

## Using Spring Kotlin

To use Spring Kotlin extensions, add the following dependency in your `build.gradle` file, with `x.y.z` replaced by the latest release available [here](https://bintray.com/sdeleuze/maven/spring-kotlin/):

```gradle
repositories {
  maven { url 'https://dl.bintray.com/sdeleuze/maven' }
}

dependencies {
  compile 'org.springframework.kotlin:spring-kotlin:x.y.z'
}
```

## Spring + Kotlin FAQ

###What is the simplest way to start a Spring Boot + Kotlin application?

Go to http://start.spring.io/#!language=kotlin, add your dependencies, choose Gradle or Maven and click on "Generate Project" ;-)

###Where this "Configuration problem: @Configuration class 'Foo' may not be final" error message come from?

Spring applications are using proxies to deal with most `@Component` beans like `@Configuration`, `@Service`or `@Repository`. JDK dynamic proxies are used when possible (your bean should implement an interface, and `proxyTargetClass` should be set to `false`, which is the default) while CGLIB proxies are used with classes that do not expose methods with interfaces, and/or when `proxyTargetClass` is set to `true`.

Unlike Java, Kotlin [have made the choice to have a `final` by default behavior](https://discuss.kotlinlang.org/t/classes-final-by-default/166), with `open` keyword used at class or method level to allow extending or overriding them. As a consequence, with CGLIB proxies you need to explicitely specify `open` at class and method level since they require to extend/override the class/methods for technical reasons. Kotlin team is working on a solution that will allow you to use CGLIB proxies without having to use the `open` keyword, see [KT-12149](https://youtrack.jetbrains.com/issue/KT-12149) for more details.

Meanwhile, current best practice is:
 - Add `open` modifier at class level for `@Configuration` class and at `@Bean` methods level since they require CGLIB proxies
 - Use JDK dynamic proxies for other `@Component` beans like `@Service` and `@Repository` by implementing interfaces and making sure `proxyTargetClass=false`. Be aware that Spring Boot [set `proxyTargetClass=true` by default when JDBC is used](https://github.com/spring-projects/spring-boot/commit/58d660d10d7abb5fe2ea502b6c538714bede62ea#diff-3f2cf0894a5a46136680f76234aeee28R41), to use JDK dynamic proxies instead you just have to declare a `PersistenceExceptionTranslationPostProcessor` `@Bean` like [here](https://github.com/sdeleuze/geospatial-messenger/blob/master/src/main/kotlin/io/spring/messenger/Application.kt#L34).

###Why annotation array attributes require `arrayOf()` for single value unlike in Java?

That's a known Kotlin 1.0 issue quite annoying with Spring annotations like `@RequestMapping(method = arrayOf(RequestMethod.GET))`, that will be fixed in Kotlin 1.1, see issue [KT-11235](https://youtrack.jetbrains.com/issue/KT-11235) for more details. As a workaround, as of Spring Framework 4.3 / Spring Boot 1.4 you can use annotations aliases like `@GetMapping`, `@PostMapping`, etc.

###Why `@Value("${thing}")` does not work with Kotlin?

That's because `$` is used for string interpolation in Kotlin. Workarounds are:
 - Escaping `$`: `@Value("\${some.property}")`
 - Configure `PropertySourcesPlaceholderConfigurer` to use another `PlaceholderPrefix`
 - Use `@ConfigurationProperties` instead

See [this Stackoverflow response](http://stackoverflow.com/questions/33821043/spring-boot-change-property-placeholder-signifier/33883230#33883230) for more details.

###What is the recommanded way to inject dependencies in a Spring Kotlin application

Try [to favor contructor injection](http://olivergierke.de/2013/11/why-field-injection-is-evil/). As of Spring Framework 4.3 / Spring Boot 1.4 you just have to write `class MessageController(val repository: MessageService)` and Spring will automatically autowire the constructor (there should be a single constructor with no default values). For previous versions, you need to write `class MessageController @Autowired constructor(val repository: MessageService)`.

If you really need to use field injection, use `lateinit var`:

```kotlin
@Component
class YourBean {

    @Autowired
    lateinit var mongoTemplate: MongoTemplate

    @Autowired
    lateinit var solrClient: SolrClient
}
```

###How can I get Jackson working correctly with Kotlin classes?
You should register [Jackson Kotlin module](https://github.com/FasterXML/jackson-module-kotlin). As of Spring Framework 4.3 / Spring Boot 1.4, you just have to add Maven or Gradle dependency and it will be registered automatically. With previous version, you will have to register it manually in one of your `@Configuration` classes:

```kotlin
@SpringBootApplication
open class Application {

    @Bean
    open fun objectMapperBuilder(): Jackson2ObjectMapperBuilder =
        Jackson2ObjectMapperBuilder().modulesToInstall(KotlinModule())
    
    // ...
````

### What are other Kotlin features or issues to follow?

- [Support Kotlin nullable information in Spring MVC parameters](https://jira.spring.io/browse/SPR-14165)
- [Parent issue for Spring Framework support](https://youtrack.jetbrains.com/issue/KT-6380)
- [Callable reference with expression on the left hand side](https://youtrack.jetbrains.com/issue/KT-6947)
- [Infer type parameter to upper bound when no bounds can be inferred from arguments](https://youtrack.jetbrains.com/issue/KT-11658)
- [Spring: package name(s) in `AnnotationConfigApplicationContext()` call are not resolved](https://youtrack.jetbrains.com/issue/KT-11658)
- [Member vs SAM conversion with more specific signature](https://youtrack.jetbrains.com/issue/KT-11128) (target : 1.1)

## Contribute
Before we accept a non-trivial patch or pull request we will need you to sign the
[contributor's agreement](https://support.springsource.com/spring_committer_signup).
Signing the contributor's agreement does not grant anyone commit rights to the main
repository, but it does mean that we can accept your contributions, and you will get an
author credit if we do.  Active contributors might be asked to join the core team, and
given the ability to merge pull requests. Use `Spring Framework` in the project field
and `Sébastien Deleuze` in the project lead field when you complete the form.

## License
Spring Kotlin extensions is Open Source software released under the
[Apache 2.0 license](http://www.apache.org/licenses/LICENSE-2.0.html).

