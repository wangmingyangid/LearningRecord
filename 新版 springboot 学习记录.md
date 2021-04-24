**雷丰阳新版springboot学习记录** 

## 1. hello world

### 1.1 创建maven工程

### 1.2 添加依赖

~~~xml
	<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
    </parent>

    <!--引入相关场景的依赖，如web-->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
~~~

###1.3 创建主程序

~~~java
/**
 * @author wmy
 * @create 2021-04-18 20:25
 *
 * 主程序类
 * @SpringBootApplication: 告诉spring boot 这是一个spring boot应用
 */

@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class,args);
    }
}
~~~

### 1.4 编写业务

~~~java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String handler01() {
        return "spring boot 02";
    }
}
~~~

### 1.5 测试

直接运行 main 方法

### 1.6 简化配置

application.properties

~~~xml
server.port=8888
~~~

### 1.7 简化部署

~~~xml
<!--为了可以打成可执行的 jar 包-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
~~~

把项目打成 jar 包，直接在目标服务器运行即可。

![打包图示](D:\E\学习记录\springboot2学习图片\打包图示.jpg)

## 2. 了解自动配置原理

### 2.1 spring boot特点

#### 2.1.1 依赖管理

* 父项目做依赖管理

~~~xml
依赖管理
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
 </parent>

--------------------------------------------------------

spring-boot-starter-parent 的父项目
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.4.RELEASE</version>
 </parent>

他里面声明了很多依赖的版本号，所以在子项目中无需声明版本号，自动仲裁。
<properties>
    <activemq.version>5.15.13</activemq.version>
    <antlr2.version>2.7.7</antlr2.version>
    <appengine-sdk.version>1.9.82</appengine-sdk.version>
    <artemis.version>2.12.0</artemis.version>
    <aspectj.version>1.9.6</aspectj.version>
    <mysql.version>8.0.21</mysql.version>
  	.
  	.
  	.
 </properties>
~~~

* 开发导入 starter 场景启动器

~~~xml
1. 需要什么场景，就引入相关的场景启动器 spring-boot-starter-*
2. 只要引入starter，这个场景的所有需要的依赖都会自动引入
3. SpringBoot 所有支持的场景网址
https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter
4. *-spring-boot-starter：这是第三方为我们提供的简化开发的场景启动器
5. 所有的场景启动器最底层的依赖
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <version>2.3.4.RELEASE</version>
      <scope>compile</scope>
    </dependency>
~~~

* 无需关注版本号，自动版本仲裁

~~~xml
1. 引入依赖默认都可以不写版本号
2. spring-boot-dependencies 中没有定义的版本，要写版本号
~~~

* 可以修改版本号

~~~xml
步骤：
1.查看 spring-boot-dependencies里面规定当前依赖的版本用的 key 
2.在当前项目里重写配置
 <properties>
   <mysql.version>5.1.43</mysql.version>
 </properties>
~~~

#### 2.1.2 自动配置（以引入 web 场景为例）

* 自动配好Tomcat

  * 引入Tomcat依赖 

  ~~~java
  <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <version>2.3.4.RELEASE</version>
        <scope>compile</scope>
    </dependency>
  ~~~

  * 配置Tomcat

* 自动配好SpringMVC

  * 引入SpringMVC全套组件

  ~~~java
     <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.2.9.RELEASE</version>
        <scope>compile</scope>
      </dependency>
  ~~~

  * 自动配好SpringMVC常用组件（功能）

* 自动配好Web常见功能，如：字符编码问题、文件上传、视图解析

  * SpringBoot帮我们配置好了所有web开发的常见场景

  ~~~java
     <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.2.9.RELEASE</version>
        <scope>compile</scope>
      </dependency>
  ~~~

* 默认的包结构

  * 主程序所在包及其下面的所有子包里面的组件都会被默认扫描出来

  * 无需以前的包扫描配置

  * 可以通过如下方式改变扫描路径

    1. @SpringBootApplication(scanBasePackages = "org.wmy")

    2. @ComponentScan指定扫描路径

       ~~~java
       @SpringBootApplication
       等同于
       @SpringBootConfiguration
       @EnableAutoConfiguration
       @ComponentScan()
       可在主程序类上用上面3个注解代替@SpringBootApplication注解，然后通过@ComponentScan("org.wmy") 指定扫描路径
       ~~~

* 各种配置拥有默认值

  * 配置文件的值最终会绑定到某个类上，这个类会在容器中创建对象

* 按需加载所有自动配置项

  * 引入哪些场景这个场景的自动配置才会开启

  * SpringBoot所有的自动配置功能都在

    ~~~java
    场景启动器依赖 --> spring-boot-starter --> 依赖 spring-boot-autoconfigure
    <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-autoconfigure</artifactId>
          <version>2.3.4.RELEASE</version>
          <scope>compile</scope>
    </dependency>
    ~~~

## 3. 容器功能

### 3.1 组件添加

#### 3.1.1 @Configuration

Full模式和Lite模式最佳实战

* 配置类组件之间无依赖关系用Lite模式加速容器启动过程，减少判断
* 配置类组件之间有依赖关系，用Full模式（方法会被调用得到之前的单实例组件）

~~~java
/**
 *  1. 配置类里面使用 @Bean 标注在方法上给容器注册组件，默认是单实例的
 *  2. 配置类本身就是组件
 *  3. proxyBeanMethods：代理bean方法，默认是true
 *      Full（重量级模式）:proxyBeanMethods = true
 *      Lite（轻量级模式）:proxyBeanMethods = false
 *
 */
@Configuration(proxyBeanMethods = true) //告诉 SpringBoot 这是一个配置类 == 配置文件
public class MyConfig {

    @Bean//给容器中添加组件；以方法名作为组件的id；返回类型就是组件类型；返回的值就是组件组件在容器中的实例
    //@Bean("person")，通过这种方式改变组件id
    public Person person01(){
        return new Person("zhang san",18);
    }
}
=====================================================================================
  
/**
 * 主程序类
 * @SpringBootApplication: 告诉spring boot 这是一个spring boot应用
 */

@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {
        //返回IOC容器
        ConfigurableApplicationContext applicationContext = 	      SpringApplication.run(MainApplication.class, args);
        //查看容器里面的组件
        String[] names = applicationContext.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }
        //验证@bean标签添加的是单实例
        Person bean = applicationContext.getBean("person01", Person.class);
        Person bean1 = applicationContext.getBean("person01", Person.class);
        System.out.println(bean == bean1);//true
        //验证配置类也是组件
        MyConfig myConfig = applicationContext.getBean(MyConfig.class);
      	//org.wmy.boot.config.MyConfig$$EnhancerBySpringCGLIB$$89028ac@53765e1c
        System.out.println(myConfig);

        //验证 proxyBeanMethods
        //如果 proxyBeanMethods = true ，组件在容器中是单实例
        Person person = myConfig.person01();
        Person person1 = myConfig.person01();
        System.out.println(person == person1);//true
    }
}

~~~

#### 3.1.2 @Bean、@Component、@Controller、@Service、@Repository

#### 3.1.3 @ComponentScan、@Import

```text
@import({User.class})，该注解标注在给容器中导入组件的类上，会把相关的类对象导入容器，组件id是类的全名
```

#### 3.1.4 @Conditional

条件装配：满足Conditional指定的条件，则进行组件注入

![条件注解体系图](D:\E\学习记录\springboot2学习图片\条件注解体系图.jpg)

~~~java
   //当容器中存在组件id 为 pet 的组件时，User组件才会被添加
    @Bean
    @ConditionalOnBean(name = "pet")
    public User user(){
        User user = new User("tom", 99);
        user.setPet(pet());
        return user;
    }
    @Bean("my_pet")
    public Pet pet(){
        return new Pet();
    }
~~~

### 3.2 原生配置文件引入

@ImportResource("classpath:beans.xml") 导入spring的配置文件；标注在配置类上

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  	<!--类中必须要有getter和setter方法-->
    <bean id="haha" class="org.wmy.boot.bean.Person">
        <property name="name" value="tom"></property>
        <property name="age" value="18"></property>
    </bean>
</beans>
~~~

~~~java
@ImportResource("classpath:beans.xml")
@Configuration
public class MyConfig {}
~~~

### 3.3配置绑定 

####1.  @Component + @ConfigurationProperties

~~~java
/**
 *
 * 只有容器中的组件，才能拥有spring boot 提供的强大功能（比如，配置绑定）
 */

@Component
@ConfigurationProperties(prefix = "car")
public class Car {
    private String brand;
    private int price;
    .
    .
}

============
配置文件中的值
car.brand=BYD
car.price=333
~~~

####2. @EnableConfigurationProperties + @ConfigurationProperties 

@EnableConfigurationProperties(Car.class) 的作用（该注解标注在配置类上）

* 开启Car配置绑定功能
* 把Car这个组件自动注册到容器中

~~~java
@Configuration
@EnableConfigurationProperties(Car.class)
public class MyConfig {}

============================================
  
@ConfigurationProperties(prefix = "car")
public class Car {
    private String brand;
    private int price;
    .
    .
}
~~~

## 4 自动配置原理入门

### 4.1 引导加载自动配置类

~~~java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {}
~~~

#### 1. @SpringBootConfiguration 

~~~java
@Configuration
public @interface SpringBootConfiguration {}
~~~

代表当前主程序也是一个配置类

####2.  @ComponentScan（）  

指定扫描哪些组件

####3.  @EnableAutoConfiguration  

~~~java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {}
~~~

* @AutoConfigurationPackage

~~~java 
@Import({Registrar.class}) //给容器中导入一个 Registrar 组件
public @interface AutoConfigurationPackage {}

//利用Registrar 给容器中导入一系列的组件
将指定的一个包下的所有组件导入进来（主配置类所在的包）
~~~

~~~java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
        
        public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
            AutoConfigurationPackages.register(registry, (String[])(new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames().toArray(new String[0]));
        }

~~~

![自动导入包图](D:\E\学习记录\springboot2学习图片\自动导入包图.jpg)

* @Import({AutoConfigurationImportSelector.class})

~~~java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
            AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
            return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
        }
    }
~~~

**利用 this.getAutoConfigurationEntry(annotationMetadata); 给容器中批量导入一些组件** 

~~~java
 protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        } else {
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            configurations = this.removeDuplicates(configurations);
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.getConfigurationClassFilter().filter(configurations);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
        }
    }
~~~

![配置的组件图示](D:\E\学习记录\springboot2学习图片\配置的组件图示.jpg)

**利用 this.getCandidateConfigurations(annotationMetadata, attributes); 获取到所有需要导入到容器中的配置类** 

~~~java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
~~~

~~~java
 public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
        String factoryTypeName = factoryType.getName();
        return (List)loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
    }
===========================================
利用 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) 得到所有的组件
从"META-INF/spring.factories"位置加载类加载一个文件
	默认扫描我们当前系统里所有的 META-INF/spring.factories 位置的文件
	spring-boot-autoconfigure-2.3.4.RELEASE.jar 包里面也有 META-INF/spring.factories
~~~

![自动配置类](D:\E\学习记录\springboot2学习图片\自动配置类.jpg)

### 4.2 按需开启自动配置项

虽然我们127个场景的所有自动配置启动的时候默认全部加载；但是最终会按需加载（根据条件装配）

### 4.3 定制化修改自动配置

~~~java
@Bean
@ConditionalOnBean({MultipartResolver.class})
@ConditionalOnMissingBean(name = {"multipartResolver"})
public MultipartResolver multipartResolver(MultipartResolver resolver) {
  return resolver;
}
这段代码的意思时，当容器中有MultipartResolver类型的组件，并且该组件的id不是multipartResolver时，就从容器中找到MultipartResolver类型的组件，把它返回出去（此时该组件的id就变成了multipartResolver）
目的：防止有些用户配置的文件解析器不符合规范。
说明：给@Bean注解标注的方法传入对象参数时，这个参数的值就会从容器中找。
~~~

**SpringBoot 默认会在底层配置好所有的组件，但是如果用户自己配置了，它就不再配置** （@ConditionalOnMissingBean）

~~~java
@Bean
@ConditionalOnMissingBean
public CharacterEncodingFilter characterEncodingFilter() {
  CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
  filter.setEncoding(this.properties.getCharset().name());
  filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.web.serv			let.server.Encoding.Type.REQUEST));
  filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.web.ser			vlet.server.Encoding.Type.RESPONSE));
  return filter;
}
~~~

总结：

* SpringBoot 先加载所有的自动配置类 xxxAutoConfiguration
* 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值。（从xxxProperties里面取值，xxxProperties和配置文件进行了绑定）
* 生效的配置类就会给容器中装配很多组件
* 只要容器中有这些组件，相当于这些功能就有了
* 定制化配置
  * 用户直接用添加组件的方式进行替换
  * 用户修改组件的的配置文件值

### 4.4 最佳实践

#### 1. 引入场景依赖

https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter

#### 2. 查看自动配置了哪些（选做）

* 自己分析相关场景下的自动配置包
* 配置文件debug = true 开启自动配置报告。控制台输出 Positive matches(生效)，Negative matches（不生效）

#### 3. 是否需要修改

* 参考文档修改配置项
  * https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#common-application-properties
  * 自己分析，xxxProperties绑定了配置文件的哪些
* 自定义加入加入或者替换组件
  * @Bean、@Component
* 自定义器 xxxCustomizer（待补充）

## 5 配置文件

### 5.1 YAML

#### 5.1.1 基本语法

* key：value （kv之间有空格）

* 大小写敏感

* 使用缩进表示层级关系

* 缩进不允许使用Tab，只允许空格

* 缩进的空格数不重要，只要相同层级的元素左对齐即可

* ‘#’ 表示注释

* 字符串不需加引号，如果加单引号（‘’）表示字符串会被转义，加双引号（""）表示字符串不会被转义

  ~~~text
  比如单引号会将 \n 作为字符串输出		双引号会将 \n 作为换行输出
  因为 \n 本意就是换行，单引号加上不换行了，所以是被转义了
  ~~~

  

#### 5.1.2 数据类型

* 字面量：单个的、不可再分的值。比如，date、boolean、string、number、null

~~~yaml
k: v
~~~

* 对象：键值对的集合。比如，object、map、hash

~~~yaml
#行内写法
k: {k1:v1,k2:v2,k3:v3}
#或者
k:
  k1: v1
  k2: v2
  k3: v3
~~~

* 数组：一组按次序排列的值。比如：array、list、queue、set

~~~yaml
#行内写法
k: [v1,v2,v3]
#或者
k:
  - v1
  - v2
  - v3
~~~

#### 5.1.3 示例

~~~java
@Data
public class Person {
	
	private String userName;
	private Boolean boss;
	private Date birth;
	private Integer age;
	private Pet pet;
	private String[] interests;
	private List<String> animal;
	private Map<String, Object> score;
	private Set<Double> salarys;
	private Map<String, List<Pet>> allPets;
}

@Data
public class Pet {
	private String name;
	private Double weight;
}
~~~

~~~yaml
# yaml表示以上对象
person:
  userName: zhangsan
  boss: false
  birth: 2019/12/12 20:12:33
  age: 18
  pet: 
    name: tomcat
    weight: 23.4
  interests: [篮球,游泳]
  animal: 
    - jerry
    - mario
  score:
    english: 
      first: 30
      second: 40
      third: 50
    math: [131,140,148]
    chinese: {first: 128,second: 136}
  salarys: [3999,4999.98,5999.99]
  allPets:
    sick:
      - {name: tom}
      - {name: jerry,weight: 47}
    health: [{name: mario,weight: 47}]
~~~

### 5.2 配置提示

自定义的类和配置文件绑定一般没有提示

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>

<--/>
 <build>
   <plugins>
     <plugin>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-maven-plugin</artifactId>
       <configuration>
          <!--为了打包的时候排处掉，因为该包只能开发有关-->
         <excludes>
           <exclude>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-configuration-processor</artifactId>
           </exclude>
         </excludes>
       </configuration>
     </plugin>
   </plugins>
 </build>
~~~

## 6 Web开发

### 6.1 SpringMVC自动配置概览

Spring Boot provides auto-configuration for Spring MVC that works well with most applications.

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

  内容协商视图解析器和BeanName视图解析器

- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content))).

  静态资源（包括webjars）

- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.

  自动注册 `Converter，GenericConverter，Formatter `

- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-message-converters)).

  支持 `HttpMessageConverters` （后来我们配合内容协商理解原理）

- Static `index.html` support.

- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-web-binding-initializer)).

  自动使用 `ConfigurableWebBindingInitializer` ，（DataBinder负责将请求数据绑定到JavaBean上）

If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring/docs/5.3.6/reference/html/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`.

If you want to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, and still keep the Spring Boot MVC customizations, you can declare a bean of type `WebMvcRegistrations` and use it to provide custom instances of those components.

If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`, or alternatively add your own `@Configuration`-annotated `DelegatingWebMvcConfiguration` as described in the Javadoc of `@EnableWebMvc`.

### 6.2 简单功能分析

#### 6.2.1 静态资源访问

##### 1. 静态资源目录

静态资源（图片，css，js）放在类路径下： called `/static` (or `/public` or `/resources` or `/META-INF/resources`  

访问方式 ： 当前项目根路径/ + 静态资源名 

访问的原理：By default, resources are mapped on `/**`, but you can tune that with the `spring.mvc.static-path-pattern` property. For instance, relocating all resources to `/resources/**` can be achieved as follows:

~~~yaml
# 改变访问静态资源的路径
spring:
  mvc:
    static-path-pattern: "/resources/**"
# 原来 static 下有bug.jpg的图片，可以 ip:port/bug.jpg 来访问；现在需要 ip:port/resources/bug.jpg
~~~

请求进来，先去找Controller看能不能处理。不能处理的所有请求**又都交给静态资源处理器** （映射/**，就是映射所有）。静态资源也找不到则响应404页面。

**改变静态资源目录：** 

You can also customize the static resource locations by using the `spring.web.resources.static-locations` property.

#### 6.2.2 欢迎页支持

Spring Boot supports both static and templated welcome pages. It first looks for an `index.html` file in the configured static content locations. If one is not found, it then looks for an `index` template. If either is found, it is automatically used as the welcome page of the application.

* 静态资源路径下放  index.html（注意，可以配置静态资源路径，但是不可以配置静态资源的访问前缀。否则导致 index.html 不能被默认访问。）

  ~~~yaml
  spring:
  #  mvc:
  #    static-path-pattern: /res/**   这个会导致welcome page功能失效

    resources:
      static-locations: [classpath:/haha/]
  ~~~

* 编写controller能处理/index 请求（待补充，没测试成功，因为没有添加模板引擎）

#### 6.2.3 自定义Favicon

favicon.ico 放在静态资源目录下即可

~~~yaml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致 Favicon 功能失效
~~~

#### 6.2.4  静态资源配置原理

* SpringBoot 启动默认加载xxxAutoConfiguration类
* SpringMVC功能的自动配置类是WebMvcAutoConfiguration（生效）

~~~java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {}
~~~

该类里有WebMvcAutoConfigurationAdapter 内部类

~~~java
@Configuration(proxyBeanMethods = false)
@Import(EnableWebMvcConfiguration.class)
/*
配置文件的相关属性和相关配置类进行了绑定。WebMvcProperties匹配spring.mvc前缀的配置、ResourceProperties匹配spring.resources前的配置
*/
@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
@Order(0)
public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {}
~~~

##### 1. 配置类只有一个有参构造器

~~~java

//有参构造器所有参数的值都会从容器中确定
//ResourceProperties resourceProperties；获取和spring.resources绑定的所有的值的对象
//WebMvcProperties mvcProperties 获取和spring.mvc绑定的所有的值的对象
//ListableBeanFactory beanFactory Spring的beanFactory
//HttpMessageConverters 找到所有的HttpMessageConverters
//ResourceHandlerRegistrationCustomizer 找到资源处理器的自定义器。(这个是和静态资源相关的)
//DispatcherServletPath  
//ServletRegistrationBean   给应用注册Servlet、Filter....

public WebMvcAutoConfigurationAdapter(ResourceProperties resourceProperties, WebMvcProperties mvcProperties,
				ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider,
				ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
				ObjectProvider<DispatcherServletPath> dispatcherServletPath,
				ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
			this.resourceProperties = resourceProperties;
			this.mvcProperties = mvcProperties;
			this.beanFactory = beanFactory;
			this.messageConvertersProvider = messageConvertersProvider;
			this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
			this.dispatcherServletPath = dispatcherServletPath;
			this.servletRegistrations = servletRegistrations;
		}
~~~

##### 2 资源处理的默认规则（资源就是指静态资源，我的理解）

该方法在WebMvcAutoConfigurationAdapter类里，注册资源处理器。

~~~java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
   if (!this.resourceProperties.isAddMappings()) {
       //spring.resources.add-mappings=false 禁用掉资源映射
      logger.debug("Default resource handling disabled");
      return;
   }
    //获得缓存有效期
   Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
   CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
   if (!registry.hasMappingForPattern("/webjars/**")) {
       // 请求/webjars/**路径下的资源时，去classpath:/META-INF/resources/webjars/下找
      customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
            .addResourceLocations("classpath:/META-INF/resources/webjars/")
            .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
   }
    //可以在配置文件修改
   String staticPathPattern = this.mvcProperties.getStaticPathPattern();
   if (!registry.hasMappingForPattern(staticPathPattern)) {
       //请求 staticPathPattern（默认是/**）路径下的资源时，
       //去this.resourceProperties.getStaticLocations()找(静态资源位置)
      customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
            .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
            .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
   }
}
~~~



~~~java
ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
	//默认的静态资源位置
	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };
	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/].
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
}
~~~

##### 3 欢迎页的规则

在WebMvcAutoConfiguration 内部配置类EnableWebMvcConfiguration中

~~~java
//  HandlerMapping：处理器映射。保存了每一个Handler能处理哪些请求。  
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
      FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
   WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
         new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
         this.mvcProperties.getStaticPathPattern());
   welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
   welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
   return welcomePageHandlerMapping;
}
~~~

~~~java
//WelcomePageHandlerMapping的构造函数
WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
			ApplicationContext applicationContext, Optional<Resource> welcomePage, String staticPathPattern) {
		if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {
			// //要用欢迎页功能，必须是/**
            logger.info("Adding welcome page: " + welcomePage.get());
			setRootViewName("forward:index.html");
		}
		else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
			 // 调用Controller  /index
            logger.info("Adding welcome page template: index");
			setRootViewName("index");
		}
	}
~~~

### 6.3 请求参数处理

#### 6.3.1 请求映射

##### 1. rest使用与原理

* Rest风格支持（*使用 **HTTP**请求方式动词来表示对资源的操作*）
  * 以前：/getUser 获取用户	/deleteUser 删除用户	/editUser 修改用户	/saveUser 保存用户
  * 现在：/user    GET--获取用户      DELETE--删除用户      PUT--修改用户      POST--保存用户
* **表单提交开启rest 风格**，核心是HiddenHttpMethodFilter
  * 用法：表单method = post，隐藏域 _method = put/delete
  * SpringBoot中开启 （spring.mvc.hiddenmethod.filter = true）

~~~java
   //可用@GetMapping替代
	@RequestMapping(value = "/user",method = RequestMethod.GET)
    public String getUser(){
        return "GET-张三";
    }
	
	//可用@PostMapping替代
    @RequestMapping(value = "/user",method = RequestMethod.POST)
    public String saveUser(){
        return "POST-张三";
    }

	//可用@PutMapping替代
    @RequestMapping(value = "/user",method = RequestMethod.PUT)
    public String putUser(){
        return "PUT-张三";
    }
	//可用@DeleteMapping替代
    @RequestMapping(value = "/user",method = RequestMethod.DELETE)
    public String deleteUser(){
        return "DELETE-张三";
    }


	//表单开启rest风格才配置，否则不用配置
	@Bean
	@ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
	@ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false)
	public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
		return new OrderedHiddenHttpMethodFilter();
	}


//自定义filter
    @Bean
    public HiddenHttpMethodFilter hiddenHttpMethodFilter(){
        HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
        //通过这种方式可以自定义隐藏域（比如替换隐藏域 _method）
        methodFilter.setMethodParam("_m");
        return methodFilter;
    }
~~~

* 使用客户端工具，如PostMan直接发送Put、delete等方式请求，无需Filter。

* HiddenHttpMethodFilter原理 

  ~~~java
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
      HttpServletRequest requestToUse = request;
      //原生请求是 post 方式
      if ("POST".equals(request.getMethod()) && request.getAttribute("javax.servlet.error.exception") == null) {
          //this.methodParam 的值是"_method"
          String paramValue = request.getParameter(this.methodParam);
          if (StringUtils.hasLength(paramValue)) {
              String method = paramValue.toUpperCase(Locale.ENGLISH);
              //ALLOWED_METHODS是个集合，里面有put/delete/patch
              if (ALLOWED_METHODS.contains(method)) {
                  //对原请求进行了包装，改变了原来的请求方式
                  requestToUse = new HiddenHttpMethodFilter.HttpMethodRequestWrapper(request, method);
              }
          }
      }
  	//继续执行其它的拦截器
      filterChain.doFilter((ServletRequest)requestToUse, response);
  }
  ===========
  原生request（post），利用包装模式requesWrapper重写了getMethod方法，重写后返回的是传入的值。
  过滤器链放行的时候用wrapper。以后的方法调用getMethod是调用requesWrapper的。
  
  ~~~

##### 2. 请求映射原理 

<img src="D:\E\学习记录\springboot2学习图片\dispatchServlet继承体系.jpg" alt="dispatchServlet继承体系"  />

* HttpServlet 中有 doGet、doPost 等方法，在 FrameWorkServlet 中被重写了，在重写的方法中都调用了
  processRequest(request, response);
* processRequest 方法内部调用 doService(request, response);
* doService 方法在 FrameWorkServlet 是抽象方法，所以该方法被 DispatcherServlet 进行了重写；
* 在DispatcherServlet的 doService 方法中调用了 doDispatch(request, response) 方法。

~~~tex
doXXX -->processRequest-->doService-->doDispatch
所有的请求最后都交由 doDispatch 方法进行处理
~~~

[SpringMVC 的请求流程图](https://www.jianshu.com/p/91a2d0a1e45a)

<img src="D:\E\学习记录\springboot2学习图片\SpringMVC请求流程..png" alt="SpringMVC请求流程." style="zoom: 80%;" />

~~~java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}
~~~

==mappedHandler = getHandler(processedRequest);== 

~~~java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
            //遍历处理器映射，找到能处理请求的handler
			for (HandlerMapping mapping : this.handlerMappings) {
				HandlerExecutionChain handler = mapping.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}
~~~

<img src="D:\E\学习记录\springboot2学习图片\处理器映射.jpg" alt="处理器映射" style="zoom: 50%;" />

**RequestMappingHandlerMapping**：保存了所有标注@RequestMapping 注解和handler的映射规则。

![RequestMappingHandlerMapping保存的映射](D:\E\学习记录\springboot2学习图片\RequestMappingHandlerMapping保存的映射.jpg)

总结：所有的请求映射都在HandlerMapping中

* SpringBoot自动配置欢迎页的 WelcomePageHandlerMapping 。访问 /能访问到index.html；
* SpringBoot自动配置了默认 的 RequestMappingHandlerMapping
* 请求进来，挨个尝试所有的HandlerMapping看是否能处理请求信息。
  * 如果能处理就找到这个请求对应的handler
  * 如果不能就是下一个 HandlerMapping
* 我们需要一些自定义的映射处理，我们也可以自己给容器中放**HandlerMapping**。自定义 **HandlerMapping**

```
WebMvcAutoConfiguration 中给容器中放了以下默认的处理器映射
```

![WebMVC自动配置类给容器中放了默认的处理器映射](D:\E\学习记录\springboot2学习图片\WebMVC自动配置类给容器中放了默认的处理器映射.jpg)

#### 6.3.2 普通参数与基本注解

##### 1. 注解

@PathVariable、@RequestHeader、@ModelAttribute、@RequestParam、@MatrixVariable、@CookieValue、@RequestBody

~~~java
@RestController
public class ParameterTestController {

    //  car/2/owner/zhangsan?age=18&inters=basketball&inters=game
    @GetMapping("/car/{id}/owner/{username}")
    public Map<String,Object> getCar(@PathVariable("id") Integer id,
                                     @PathVariable("username") String name,
                                     @PathVariable Map<String,String> pv,
                                     @RequestHeader("User-Agent") String userAgent,
                                     @RequestHeader Map<String,String> header,
                                     @RequestParam("age") Integer age,
                                     @RequestParam("inters") List<String> inters,
                                     @RequestParam Map<String,String> params,
                                     @CookieValue("_ga") String _ga,
                                     @CookieValue("_ga") Cookie cookie){

        Map<String,Object> map = new HashMap<>();

//        map.put("id",id);
//        map.put("name",name);
//        map.put("pv",pv);
//        map.put("userAgent",userAgent);
//        map.put("headers",header);
        map.put("age",age);
        map.put("inters",inters);
        map.put("params",params);
        map.put("_ga",_ga);
        System.out.println(cookie.getName()+"===>"+cookie.getValue());
        return map;
    }


    //@RequestBody 会封装请求体里的内容为指定对象
    @PostMapping("/save")
    public Map postMethod(@RequestBody String content){
        Map<String,Object> map = new HashMap<>();
        map.put("content",content);
        return map;
    }


    //1、语法： 请求路径：/cars/sell;low=34;brand=byd,audi,yd
    //2、SpringBoot默认是禁用了矩阵变量的功能
    //      手动开启：对于路径的处理依靠 UrlPathHelper 进行解析。该类里有个属性
    //              removeSemicolonContent（移除分号内容）来控制是否支持矩阵变量
    //3、矩阵变量必须有url路径变量才能被解析
    @GetMapping("/cars/{path}")
    public Map carsSell(@MatrixVariable("low") Integer low,
                        @MatrixVariable("brand") List<String> brand,
                        @PathVariable("path") String path){
        Map<String,Object> map = new HashMap<>();

        map.put("low",low);
        map.put("brand",brand);
        map.put("path",path);
        return map;
    }

    
    // /boss/1;age=20/2;age=10
    @GetMapping("/boss/{bossId}/{empId}")
    public Map boss(@MatrixVariable(value = "age",pathVar = "bossId") Integer bossAge,
                    @MatrixVariable(value = "age",pathVar = "empId") Integer empAge){
        Map<String,Object> map = new HashMap<>();

        map.put("bossAge",bossAge);
        map.put("empAge",empAge);
        return map;

    }

}
~~~

~~~java
//public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer
//中的默认方法
@Override
@SuppressWarnings("deprecation")
public void configurePathMatch(PathMatchConfigurer configurer) {
    configurer.setUseSuffixPatternMatch(this.mvcProperties.getPathmatch().isUseSuffixPattern());
    configurer.setUseRegisteredSuffixPatternMatch(
        this.mvcProperties.getPathmatch().isUseRegisteredSuffixPattern());
    this.dispatcherServletPath.ifAvailable((dispatcherPath) -> {
        String servletUrlMapping = dispatcherPath.getServletUrlMapping();
        if (servletUrlMapping.equals("/") && singleDispatcherServlet()) {
            UrlPathHelper urlPathHelper = new UrlPathHelper();
            urlPathHelper.setAlwaysUseFullPath(true);
            configurer.setUrlPathHelper(urlPathHelper);
        }
    });
}
==========================================================================

//在配置类中，添加组件就可以修改 SpringBoot 的默认配置
//我们的配置和SpringBoot的默认配置，都生效了，只不过先加载的是SpringBoot的，我们对其进行了覆盖
@Bean
public WebMvcConfigurer webMvcConfigurer(){
    return new WebMvcConfigurer() {
        @Override
        public void configurePathMatch(PathMatchConfigurer configurer) {
            UrlPathHelper urlPathHelper = new UrlPathHelper();
            urlPathHelper.setRemoveSemicolonContent(false);
            configurer.setUrlPathHelper(urlPathHelper);
        }
    };
}
~~~



#### 6.3.3 POJO 封装过程

#### 6.3.4 参数处理原理

* HandlerMapping 中找到能处理请求的Hanlder (就是哪个Controller的哪个方法)
* 为当前Handler 找一个适配器HandlerAdapter
* 适配器执行目标方法并确定方法参数的每一个值

##### 1. HandlerAdapter

~~~java
// Determine handler adapter for the current request.
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
~~~

~~~java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
    if (this.handlerAdapters != null) {
        for (HandlerAdapter adapter : this.handlerAdapters) {
            if (adapter.supports(handler)) {
                return adapter;
            }
        }
    }
    throw new ServletException("No adapter for handler [" + handler +
                               "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
}
~~~

<img src="D:\E\学习记录\springboot2学习图片\HandlerAdapter.jpg" alt="HandlerAdapter"  />

0--支持方法上标注@RequestMapping的

1--支持函数式编程的

##### 2. 执行目标方法

~~~java
// Actually invoke the handler.
//doDispatch()方法中执行该方法
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
~~~

~~~java
//RequestMappingHandlerAdapter 中执行该方法
mav = invokeHandlerMethod(request, response, handlerMethod);
~~~

~~~java
//在 invokeHandlerMethod 方法中给invocableMethod 放置参数解析器和返回值解析器
if (this.argumentResolvers != null) {
    invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
}
if (this.returnValueHandlers != null) {
    invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
}
~~~

~~~java
//放完参数解析器和返回值解析器后，在invokeHandlerMethod 中执行下面的方法
invocableMethod.invokeAndHandle(webRequest, mavContainer);
~~~

~~~java
//ServletInvocableHandlerMethod 类
public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer,
                            Object... providedArgs) throws Exception {

    //真正的执行我们写的controller 方法
    Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
    .
    .
}
~~~

~~~java
//InvocableHandlerMethod 类
public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {
    //确定每一个参数的值（见5）
    Object[] args = this.getMethodArgumentValues(request, mavContainer, providedArgs);
    if (this.logger.isTraceEnabled()) {
        this.logger.trace("Arguments: " + Arrays.toString(args));
    }
	//确定完参数，调用我们编写的Controller 方法
    return this.doInvoke(args);
}
~~~

##### 3. 参数解析器（HandlerMethodArgumentResolver）

确定将要执行的目标方法的每一个参数的值是什么;

SpringMVC目标方法能写多少种参数类型。取决于参数解析器。

<img src="D:\E\学习记录\springboot2学习图片\参数值解析器.jpg" alt="参数值解析器"  />

~~~java
public interface HandlerMethodArgumentResolver {
    boolean supportsParameter(MethodParameter var1);

    @Nullable
    Object resolveArgument(MethodParameter var1, @Nullable ModelAndViewContainer var2, NativeWebRequest var3, @Nullable WebDataBinderFactory var4) throws Exception;
}

~~~

* 当前解析器是否支持解析这种参数

* 支持就调用 resolveArgument

##### 4. 返回值处理器

![返回值处理器](D:\E\学习记录\springboot2学习图片\返回值处理器.jpg)

##### 5. 如何确定目标方法每一个参数的返回值

~~~java
protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {
    MethodParameter[] parameters = this.getMethodParameters();
    if (ObjectUtils.isEmpty(parameters)) {
        return EMPTY_ARGS;
    } else {
        Object[] args = new Object[parameters.length];

        for(int i = 0; i < parameters.length; ++i) {
            MethodParameter parameter = parameters[i];
            parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
            args[i] = findProvidedArgument(parameter, providedArgs);
            if (args[i] == null) {
                if (!this.resolvers.supportsParameter(parameter)) {
                    throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
                }

                try {
                    args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
                } catch (Exception var10) {
                    if (this.logger.isDebugEnabled()) {
                        String exMsg = var10.getMessage();
                        if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
                            this.logger.debug(formatArgumentError(parameter, exMsg));
                        }
                    }

                    throw var10;
                }
            }
        }

        return args;
    }
}
~~~

###### 5.1 挨个判断所有参数解析器哪个支持解析这个参数

~~~java
//this.resolvers.supportsParameter(parameter)
private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
    HandlerMethodArgumentResolver result = (HandlerMethodArgumentResolver)this.argumentResolverCache.get(parameter);
    if (result == null) {
        Iterator var3 = this.argumentResolvers.iterator();
		//循环遍历参数解析器，找到支持该参数的解析器
        while(var3.hasNext()) {
            HandlerMethodArgumentResolver resolver = (HandlerMethodArgumentResolver)var3.next();
            if (resolver.supportsParameter(parameter)) {
                result = resolver;
                this.argumentResolverCache.put(parameter, resolver);
                break;
            }
        }
    }

    return result;
}
~~~

###### 5. 2 解析这个参数的值

~~~java
//确定好解析器，来到这行代码 args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
//在该方法中根据确定的解析器，调用解析器的resolveArgument 方法

~~~



### 6.4 响应数据与内容协商

### 6.5 视图解析与模板引擎

### 6.6 拦截器

### 6.7 跨域

### 6.8 异常处理

### 6.9 原生组件注入

### 6.10 嵌入式Web容器

### 6.11 定制化原理