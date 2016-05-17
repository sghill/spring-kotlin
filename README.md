# spring-kotlin
Spring Kotlin extensions is a repository that contains Kotlin extensions, documentation,
and any helpful assets that can help to build Spring + Kotlin applications. 

This project has been created after [this discussion](https://github.com/spring-projects/spring-boot/issues/5537) on Spring Boot issue tracker.

You will find  more information about using Spring Boot + Kotlin in these 2 blog posts:
 - [Developing Spring Boot applications with Kotlin](https://spring.io/blog/2016/02/15/developing-spring-boot-applications-with-kotlin)
 - [A Geospatial Messenger with Kotlin, Spring Boot and PostgreSQL](https://spring.io/blog/2016/03/20/a-geospatial-messenger-with-kotlin-spring-boot-and-postgresql)

## Spring + Kotlin FAQ

###What is this strange "Configuration problem: @Configuration class 'Application' may not be final" error message?

Spring applications are using proxies to deal with most `@Component` beans like `@Configuration`, `@Service`or `@Repository`. JDK dynamic proxies are used when possible (your bean should implement an interface, and `proxyTargetClass` should be set to `false`, which is the default) while CGLIB proxies are used with class that do not expose methods with interfaces, and/or when `proxyTargetClass` is set to `true`.

Unlike Java, Kotlin [have made the choice to have a `final` by default behavior](https://discuss.kotlinlang.org/t/classes-final-by-default/166), with `open` keyword used at class or function level to allow extending or overriding. This difference is an issue with CGLIB proxies, since they require to extend/override the class/methods for technical reasons. There is some discussions on issue [KT-12149](https://youtrack.jetbrains.com/issue/KT-12149) and [here](https://github.com/spring-projects/spring-boot/issues/5537#issuecomment-218005683) to see if we can find a solution but there is no perfect solution yet. JDK dynamic proxy worls perfectly well with Kotlin, but can't be used everywhere.

So current best practice is:
 - Use `open` at class level for `@Configuration` class and at `@Bean` method level since they require CGLIb proxies
 - Use JDK dynamic proxies for other `@Component` beans like `@Service` and `@Repository` by implementing interfaces and making sure `proxyTargetClass=false`. Be aware that Spring Boot [set `proxyTargetClass=true` by default when JDBC is used](https://github.com/spring-projects/spring-boot/commit/58d660d10d7abb5fe2ea502b6c538714bede62ea#diff-3f2cf0894a5a46136680f76234aeee28R41), to use JDK dynamic proxies instead you just have to declare a `PersistenceExceptionTranslationPostProcessor` `@Bean` like [here](https://github.com/sdeleuze/geospatial-messenger/blob/master/src/main/kotlin/io/spring/messenger/Application.kt#L34).

###Why annotation array attribute value require `arrayOf()` for single value unlike in Java?

That's a known Kotlin 1.0 issue quite annoying with Spring annotation like `@RequestMapping(method = arrayOf(RequestMethod.GET))`, that will be fixed in Kotlin 1.1, see issue [KT-11235](https://youtrack.jetbrains.com/issue/KT-11235) for more details. As a workaround, as of Spring Framework 4.3 / Spring Boot 1.4 (not released yet) you can use annotations aliases like `@GetMapping`, `@PostMapping`, etc.

###Why `@Value("${thing}")` does not work with Kotlin?

That's because `$` is used for string interpolation in Kotlin. Workarounds are:
 - Escaping `$`: `@Value("\${some.property}")`
 - Configure `PropertySourcesPlaceholderConfigurer` to use another `PlaceholderPrefix`
 - Use `@ConfigurationProperties` instead

See [this Stackoverflow response](http://stackoverflow.com/questions/33821043/spring-boot-change-property-placeholder-signifier/33883230#33883230) for more details.

## Sign the Contributor License Agreement
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

