# Notes for Spring framework

## Spring basics
-  the objects in an application have `dependencies` on each other
- `Core Container`: spring-core, spring-beans, spring-context, etc
- dependency management:
  - manage physical resources in file systems
  - assemble libraries (jar) onto **classpath** at runtime
  - transitive dependencies
  - the "build" process

## Getting Started with Maven
- Maven Central: the default repository that Maven queries
- archetype: a Maven project template; `mvn archetype:generate`
- groupId: unique identifier of the organization or at least a domain name
- artifactId: the name of the jar without version

```
mvn -B archetype:generate \
    -DgroupId=com.amazonaws.chozhang \
    -DartifactId=ChozhangAWSCodeSnippets

```
- `mvn compile`: compiled classes were placed in ${basedir}/target/classes
- `mvn compile -e`
- `mvn package`: create a JAR
- `mvn install`: install the artifact JAR file in local repository ${user.home}/.m2/repository
- SNAPSHORT version: the 'development' version before the final 'release' version
  - a 'release' version (any version value without the -SNAPSHOT) is unchanging
- plugin: will be automatically downloaded and used like a dependency
- `${basedir}/src/main/resources`: files here are packaged in JAR with the exact same structure **starting at the base of the JAR**
  - `src/main/resources/META-INF`
- pom reference: XML elements.property
  - ${project.name}, ${project.version}, ${project.build.resource.directory}
- `mvn process-resources`
  - resources are copied and filtered under target/classes
- `<resources><resource><directory>configuration</directory></resource></resources>`
  - **add reference to external file**
  - can now placeholder reference of property from external file
- applications.properties
- `<properties></properties>`
- `<build><sourceDirectory>src</sourceDirectory></build>`: specify src path
- `jar -tf target/ChozhangAWSCodeSnippets-1.0-SNAPSHOT.jar`
- Spring's mandatory logging dependency: Jakarta Commons Logging API (commons-logging)
- log4j:
  - provide a configuration file `log4j.properties/.xml` in the root of the classpath
  - [https://logging.apache.org/log4j/2.x/manual/configuration.html]
  - `coral.spring.LoggingHelper`

## The IoC container
- IoC = dependency injection (DI); objects define their dependencies/fields through:
  - constructor arguments
  - factory method arguments
  - (setter on the instance)
- the container **injects** dependencies prior to creating the bean
  - inverse the process of POJO's self-instantiation by direct construction (`new`) of classes
- **bean**: objects of application and managed by IoC container
- `ApplicationContext`: the container instantiates, configures, and assembles objects:
  - XML bean definitions
  - Java annotations: @Component, @Controller, etc
  - @Bean methods in Java-based @Configuration classes

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
        <property name="..." ref="..."/>
    </bean>

    <!-- more bean definitions go here -->
</beans>
```

- `bean id` attribute: a string to identify individual bean definition
- `bean class`: the type of the bean; **full qualified classname**
- `property name`: the name of the JavaBean property
- `bean ref`: the name of another bean definition
- **This linkage between id and ref expresses the dependency between objects.**
- `<import resource=""/>`: load bean definitions from another file
  - each individual XML config file represents a logical layer or module
  - all location paths are relative to the definition file
- `T getBean(String name, Class<T> requiredType)`: retrieve instances of beans
  - but should never get called
  - should declare dependency on a specific bean through metadata (e.g. @Autowiring)

## Beans naming and instantiating
- Java convention for **instance field names** when naming beans
- inner class names: use `binary` name for static nested class
  - `com.example.Foo$Bar`
- `bean class` attribute is internally a `Class` property
- instantiation
   - the container creates the bean calling constructor reflectively by `class`
   - the container invokes a static factory method to create the bean

- beans-factory: class-static-factory-method
```xml
<bean id="beanId"
      class="path.to.Factory"
      factory-method="factoryMethodName"/>
```

- beans-factory: instance-factory-method
  - existing bean + `factory-bean` + `factory-method` to create a new bean
  - the bean can be in current or parent container
- one factory class **can hold more than one factory method**

```xml
<beans>
    <!-- the factory bean, which contains a method called createInstance() -->
    <bean id="beanFactory" class="path.to.Factory">
        <!-- inject any dependencies required by this factory bean -->
    </bean>

    <!-- the bean to be created via the factory bean -->
    <bean id="clientService"
          factory-bean="beanFactory"
          factory-method="createClientServiceInstance"/>
</beans>
```

## Dependency Injection
- Constructor-based DI: the container invoking a constructor with a number of args
  - each argument resolution matching occurs using the argument’s type
  - the order of args defined in bean definition is the order applied to the constructor
- `constructor-arg`: **inject constructor or method args** needed to create an object
  - **static factory method args**: supplied via `<constructor-arg/>` elements
- `constructor-arg type`: the type of constructor/method argument
- `constructor-arg index`: the index of constructor/method argument
  - resolves when a constructor has two args of the same type
- `constructor-arg name`: the constructor parameter name for value disambiguation
- `property name`: setters are declared to match against the properties in the XML
  - results in a `setXXX(...) call` on the property field
- Spring **converts text inside the \<value/> element into `java.util.Properties`**
  - **favor nested <value/> element over value="..." attribute**
- generally advocates constructor injection as **immutable objects**
  - also ensure that required dependencies are **not null**
- `constructor-arg/property value=""`: define inline values
- `constructor-arg/property ref=""`
  - **initialized on demand before the property is set**
  - regardless of whether in the same XML file
  - the value of `bean` attribute may be `id` or `name` of target bean
- **inner bean**: \<bean/> element inside the \<property/> or <constructor-arg/>
  - in place of ref???
  - does not require a defined id or name
  - always anonymous and always created with the outer bean
  - not possible to inject inner beans into beans other than enclosing bean

- `<property|constructor-arg><list><value>...</value></list>`: the single para is a list
- `<property|constructor-arg><set><value>...</value></set>`:   the single para is a set
- `<property|constructor-arg><map><entry key="" value=""/></map>`: single para is a map
- `<property|constructor-arg><props><prop key="">...<prop/></props>`: java.util.Properties

```xml
<!-- results in a setJavaProperty(java.util.Properties) call -->
<property name="javaProperty">
    <props>
        <prop key="key">value</prop>
    </props>
</property>
```
- `property value=""`: treats empty arguments for properties as empty Strings
- `<null/>` element: set Java `null` value
- `bean depends-on`: explicitly force one or more beans to be initialized before the bean
- `bean lazy-init="true"`:
  - tells IoC container to create a bean instance when first requested
  - when a lazy-initialized bean is a dependency of a singleton bean (non lazy-initialized)
  - the ApplicationContext creates the lazy-initialized bean at startup
  - because it must satisfy the singleton’s dependencies
- `bean autowire="byName"`: Spring looks for a bean with same name as the property-field
  - explicit dependency in property/constructor-arg overrides autowiring
  - cannot autowire primitives, String, Class, or array of simple properties

- Dependency resolution process
  - `ApplicationContext` is created and initialized with configuration metadata
  - each property/constructor-arg is a definition of value, or a reference to another bean
  - each property/constructor-arg is converted to actual type
    - Spring can convert from string to built-in int, long, String, boolean...
  - the container injects those dependencies
  - beans that are singleton-scoped and pre-instantiated (the default) are created
  - o.w., the bean is created only when it is requested.

## Autowiring
- Spring container can autowire relationships between collaborating beans
- XML-based: rely on angle-bracket declarations
- annotation-based: rely on bytecode metadata for wiring up components
- Annotation injection is performed before XML injection
  - thus the latter will override
- `@Required`: applies to bean **property-setter methods only**
  - the affected bean property must be populated at configuration time
  - through explicit property value in definition or through autowiring
  - still put assertions in bean class or init method to enforce non null
- `@Autowired`: tells Spring where an injection needs to occur
  - [https://www.tutorialspoint.com/spring/spring_autowired_annotation.htm]
  - [https://stackoverflow.com/a/19419296]
- `@Inject` can replace `@Autowired`
- `@Autowired|@Inject on constructor`
  - **can get rid of \<constructor-arg>**
  - **auto inject constructor args** when creating the bean
  - multiple non-required constructors can be annotated as candidate
- `@Autowried|@Inject on property-setter`
  - **can get rid of \<property>**
  - **auto inject property field** by set prefix and annotation
  - **byType** autowiring
- `@Autowired|@Inject on fields`
  - **can get rid of \<property> + setter and \<constructor-arg>**
- `@Autowired|@Inject on both field and constructor`
  - **auto inject both the field and the constructor arguments**

```java
public class MovieRecommender {
    private final CustomerPreferenceDao customerPreferenceDao;

    @Inject  // @Autowired
    private MovieCatalog movieCatalog;

    @Inject  // @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }
}
```

- `@Autowired on array or typed collections`
  - fill in all beans of a particular type
  - the Map values will contain all beans of the expected type
  - the Map keys will contain the corresponding bean names
- `@Autowired(required=true)`: favor over `@Required`

## Scope, Lifecycle and Inheritance
- singleton: exactly one instance of the object by bean definition
  - this same shared instance is injected into each dependent
  - default scope in Spring
- prototype: for stateful bean
- `bean init-method`
  - allows a bean to perform init work after all properties are set
  - **void no-argument signature**
  - `@Bean(initMethod = "init")`
- `bean destroy-method`
  - allows a bean to get callback when container is destroyed
- `bean parent`: indicate child bean and specify parent bean
- a child bean inherits definition from parent but can also override
- mark bean as abstract="true" if parent bean does not specify a class
  - abstract bean cannot be instantiated

## @Qualifier and @Resource
- `<bean class=""><qualifier value="..."></bean>`
  - bean with sub-element qualifier
  - wire the bean with constructor argument qualified with same value
- `@Qualifier("targetBean")`: **narrow** matching of bean
  - combine with `@Autowired`
- `f(@Qualifier("") args)`: can be applied to constructor/method arg
- **bean name is considered a default qualifier value**
- `@Autowired`: select candidate beans **by type**, with optional qualifier
  - apply to fields, constructors, and multi-argument methods
  - allow for narrowing through qualifier annotations at the parameter level
  - stick with qualifiers if injection target is a constructor or a multi-argument method.
- `@Autowired|@Inject private List<Store<Integer>> s;`
  - generic serves as a qualifier
  - inject all `Store` beans as long as they have an <Integer> generic
  - all matching beans according to the qualifiers are injected into collection
- `@Resource(name="")`:
  - **identify a specific target component by its unique name**
  - supported only for fields and bean property-setter methods with single argument
  - but single-arg property-setter is common for JavaBean pattern
- `@Resource`: derive default name from field name or setter method

```java
@Service
public class SimpleMovieLister {
    @Resource(name = "computeTypeToMaxFleetCapacity")
    private Map<String, Integer> computeTypeToMaxFleetCapacity;
    @Bean(name= "computeTypeToMaxFleetCapacity")
    public Map<String, Integer> getComputeTypeToMaxFleetCapacity() {
        /*...*/
    }

    private MovieFinder movieFinder;
    @Resource  // takes the bean property name "movieFinder" injected
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

- `@PostConstruct`: call the method upon initialization

## Component scanning
- implicitly detect candidate components by scanning classpath
  - remove bean registration through XML
  - can use annotations to select which classes will get registered

```xml
<beans>
    <context:component-scan base-package="path.to.package"/>
</beans>

```

- `@ComponentScan(basePackages = "path.to.package")`
- Annotation injection is performed before XML injection
  - thus the latter will override
- `@Component`: a generic stereotype for any Spring-managed component
- `@Repository`: special @Component, a marker for DAO
- `@Service`: special @Component for service layer
- `@Controller`: special @Component for presentation layer
- meta-annotation: an annotation applied to another one
- `<context:annotation-config />`
  - **annotations just annotate; needs to activate for registered beans**
  - but by default no "targets" of beans got registered
- `<context:component-scan base-package="path.to.package"/>`
  - scans packages to find + register beans within the application context
  - implicitly enables \<context:annotation-config>
- Difference between annotation-config and component-scan
  - [https://stackoverflow.com/a/7456501]
- `@ComponentScan includeFilters = `
  - `<context:include-filter type="regex" expression=".*Repository"/>`
  - custom include filters
  - Component, etc, are the only detected components
- filter type:
  - annotation: an annotation to be present at type level in target components.
  - regex + expression

```xml
<beans>
    <context:component-scan base-package="path.to.package">
        <context:include-filter type="regex"
                expression=".*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```

- `@Service("name")`: naming auto-detected components
  - by default un-capitalized non-qualified class name
  - `<context:component-scan name-generator="path.to.Class"/>`
- `@Component @Qualifier("...")`
  - provide qualifier md with type-level annotations on candidate class

## JSR330: @Inject, @Named
- `@Inject` == `@Autowired`
  - at the field level, method level and constructor-argument level
- `@Named("")` == `@Qualifier`
  - naming the dependency as string-based qualifier
  - inject the named bean after @Inject
- `@Named` == `@Component`
  - can also be used as @Component without specifying a name

## @Bean and @Configuration
- `@Bean`
  - method-level; **same as \<bean/> element**
  - most used inside `@Configuration` and @Component beans
- `@Configuration`
  - class-level; purpose as **source of bean definitions**
  - allow calling other @Bean methods in same class as **inter-bean injection**

```java
@Configuration
public class ServiceConfig {
    // <bean id="myService" class="path.to.MyServiceImpl"/>
    @Bean
    public MyService myService() { return new MyServiceImpl(); }
    
    // inter-bean injection;
    // foo receives a reference to bar via constructor injection
    @Bean
    public Foo foo() { return new Foo(bar()); }
    @Bean
    public Bar bar() { return new Bar(); }
}
```

- **@Configuration classes are ultimately just beans in container**
- The ServiceConfig above would be equivalent to the following Spring <beans/> XML:

```xml
<beans>
    <bean id="myService" class="path.to.MyServiceImpl"/>
</beans>
```

- **lite-mote bean**
  - @Bean methods are declared within classes not annotated with @Configuration
  - e.g., declared in @Component or even POJO
  - should not invoke another @Bean method; no more inter-bean injection
  - only using @Bean methods within @Configuration classes is recommended 
- ApplicationContext
  - the @Configuration class itself is registered as a bean definition
  - all declared @Bean methods within the class are also bean definitions
  - DI metadata @Inject or @Autowired are used within the registered classes
  - use @Configuration class as input => completely XML free
- @Bean methods usage
  - id: **by default bean id is same as method name**
  - class: **register bean type/class same as return value's type**
  - args: method parameters are **the dependencies** to build that bean
  - can have qualifier by `@Qualifier` == `<bean><qualifier/></bean>`
  - can have arbitrary number of parameters as dependencies
  - resolved by type first and if duplicates are found, then by name
  - **BZ: no need to annotate @Inject on @Bean methods**

```java
@Configuration
public class AppConfig {
    // identical to constructor-based dependency injection
    // bean id: transferService; class: TransferService
    // constructor-args: accountRepository; byType then byName
    // Todo: no need to annotate @Inject on @Bean methods??
    @Bean
    public TransferService transferService(@Qualifier("...") AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

- `@Bean(name = "...")`: override method name as bean's id/name
- `@Scope("prototype")`: a single bean can have any number of object instances
- `@Import(...class)` == `<import resource="...xml">`
  - allow loading @Bean definitions from another config class
  - @Import({A.class, B.class})
- `@Inject` still auto-wires across @Configuration classes
  - [https://dzone.com/articles/spring-configuration-and]
- XML-centric use of @Configuration classes

## BZ: Unsatisfied dependency expressed through constructor argument with index 2 of type [java.lang.String]: Ambiguous constructor argument types
### Define bean

    **@Configuration**
    @Bean
    public String getQualifier(@Value("${domain}.${realm}") String qualifier){
        System.out.println(qualifier);
        return qualifier;
    }
        or
    <bean id="qualifier" class="java.lang.String">
      <constructor-arg value="${domain}.${realm}"/>
    </bean>

### Autowire bean

    @javax.inject.Inject
    @org.springframework.beans.factory.annotation.Value("${domain}.${realm}")
    private String qualifier;

### Autowire bean on final field

    @RequiredConstructorArgs
        +
    <constructor-arg ref="qualifier"/>
    <constructor-arg value="${domain}.${realm}" type="java.lang.String"/>
        +
    @NonNull
    private final String qualifier;

### @Autowired VS final
- http://stackoverflow.com/questions/34580033
- Having @Autowired and final on a field are contradictory
  - final: variable has one and only one value, and it's initialized at construction time
  - Inject: Spring will construct the object, leaving the filed as null; later use relflection to initialize the bean
- **use constructor injection to autowire final fields**


