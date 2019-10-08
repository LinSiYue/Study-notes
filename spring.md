# Spring学习笔记

## 一、springboot注解

### 1、配置类注解

* @SpringBootApplication：<u>启动类配置</u>，等同于：@Configuration ，@EnableAutoConfiguration 和 @ComponentScan 三个配置。
* @EnableAutoConfiguration：尝试根据你添加的jar依赖<u>自动配置</u>你的Spring应用。例如，如果你的classpath下存在HSQLDB，并且你没有手动配置任何数据库连接beans，那么我们将自动配置一个内存型（in-memory）数据库”。你可以将@EnableAutoConfiguration或者@SpringBootApplication注解添加到一个@Configuration类上来选择自动配置。如果发现应用了你不想要的特定自动配置类，你可以使用@EnableAutoConfiguration注解的排除属性来禁用它们。
* @ComponentScan：表示将该类自动发现<u>扫描组件</u>。个人理解相当于，如果扫描到有@Component、@Controller、@Service等这些注解的类，并注册为Bean，可以自动收集所有的Spring组件，包括@Configuration类。经常使用@ComponentScan注解搜索beans，并结合@Autowired注解导入。如果没有配置的话，Spring Boot会扫描启动类所在包下以及子包下的使用了@Service、@Repository等注解的类。

- @Configuration：相当于传统的xml<u>配置</u>文件，如果有些第三方库需要用到xml文件，建议仍然通过@Configuration类作为项目的配置主类——可以使用@ImportResource注解加载xml配置文件。
- @Component：泛指<u>组件</u>，当组件不好归类的时候，我们可以使用这个注解进行标注。

### 2、springMvc相关注解

* @ResponseBody：表示该方法的返回结果直接写入HTTP response body中，一般在异步获取数据时使用，用于构建RESTful的api。在使用@RequestMapping后，返回值通常解析为跳转路径，加上@esponsebody后返回结果不会被解析为跳转路径，而是直接写入HTTP response body中。比如异步获取<u>json数据</u>，加上@Responsebody后，会直接返回json数据。该注解一般会配合@RequestMapping一起使用。

* @Controller：用于定义<u>控制器类</u>，一般这个注解在类中，通常方法需要配合注解@RequestMapping。

* @RestController：用于<u>标注控制层</u>组件，@ResponseBody和@Controller的合集。

* @RequestMapping：@RequestMapping(“/path”)表示该控制器处理所有“/path”的UR L请求。RequestMapping是一个用来处理<u>请求的地址映射</u>的注解，可用于类或方法上。

  用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。该注解有六个属性：
  params:指定request中必须包含某些参数值是，才让该方法处理。
  headers:指定request中必须包含某些指定的header值，才能让该方法处理请求。
  value:指定请求的实际地址，指定的地址可以是URI Template 模式。
  method:指定请求的method类型， GET、POST、PUT、DELETE等。
  consumes:指定处理请求的提交内容类型（Content-Type），如application/json,text/html。
  produces:指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回。

* @RequestParam：用在方法的参数前面。<u>获取请求中对应变量名的参数</u>。

* @PathVariable：<u>获取路径变量</u>，根据url中传递的变量名称匹配。

* @Service：一般用于<u>修饰service层</u>的组件。

* @Repository：使用@Repository注解可以确保DAO或者repositories提供异常转译，这个<u>注解修饰的DAO或者repositories类会被ComponetScan发现并配置</u>，同时也不需要为它们提供XML配置项。

* @Bean：用@Bean标注方法等价于XML中配置的<u>bean</u>，相当于XML中的,放在方法的上面，而不是类，意思是产生一个bean,并交给spring管理。

* @Import：用来<u>导入</u>其他配置类。

* @ImportResource：用来<u>加载xml配置</u>文件。

* @Autowired：<u>自动导入依赖</u>的bean。byType方式。把配置好的Bean拿来用，完成属性、方法的组装，它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。当加上（required=false）时，就算找不到bean也不报错。
* @Qualifier：当有多个同一类型的Bean时，可以用@Qualifier(“name”)来指定。与@Autowired配合使用。<u>@Qualifier限定描述符除了能根据名字进行注入</u>，但能进行更细粒度的控制如何选择候选者。
* @Resource(name=”name”,type=”type”)：没有括号内内容的话，默认byName。<u>通过属性匹配对应的bean。</u>
* @Inject：<u>等价于默认的@Autowired，只是没有required属性</u>。

* @Value：<u>注入Spring boot application.properties配置的属性的值</u>。

### 3、异常相关注解

* @ControllerAdvice：包含@Component。可以被扫描到。统一处理异常。

* @ExceptionHandler（Exception.class）：用在方法上面表示<u>遇到这个异常就执行以下方法</u>。

## 二、@Autowired相关知识

对于@Autowired注解，我们利用他进行bean的注入，根据类的类型进行匹配，而他能将子类赋给父类。类被spring工厂收集起来，当注解@Autowired被启用时，会在bean工厂里面找类型相匹配的类，在注入过程中将该类<u>实例创建</u>并注入。这就实现了父类找到子类。
而子类找到父类有super()方法。 