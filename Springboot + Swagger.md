# Springboot + Swagger

## 一、添加依赖

```xml
		<properties>
            <swagger.version>2.9.2</swagger.version>
    	</properties>
		<!--Swagger-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>${swagger.version}</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>${swagger.version}</version>
        </dependency>
```

## 二、swagger配置文件

* 注意：.apis填写的路径是需要显示的接口所在包

```java
package com.sy.example.core.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket helloApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                // 方法需要有ApiOperation注解才能生存接口文档
                .apis(RequestHandlerSelectors.basePackage("com.sy.example"))
                // 路径使用any风格
                .paths(PathSelectors.any())
                .build()
                // 接口文档的基本信息
                .apiInfo(apiInfo());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                // 文档标题
                .title("API 文档")
                // 文档描述
                .description("https://github.com/einverne/thrift-swift-demo/tree/master/spring-boot-demo")
                .termsOfServiceUrl("https://github.com/einverne/thrift-swift-demo/tree/master/spring-boot-demo")
                .version("v1")
                .build();
    }
}
```

## 三、使用

* 注意：在使用的时候，返回值要有ApiModel的类型时，ApiModel注解的实体类才会显示。

```java
@RestController
@Api(tags = {"User Login"},description = "user login api")
@RequestMapping("/user")
public class LoginController {

  @Autowired private UserLoginService userLoginService;

  @PostMapping("/login")
  @ApiOperation(value = "login", notes = "user login by nickName and passWord")
  public ResponseEntity<Object> login(String nickName, String passWord) {

    if (StringUtils.isEmpty(nickName) || StringUtils.isEmpty(passWord)) {
      return new ResponseEntity<>("Input is null.", HttpStatus.BAD_REQUEST);
    } else {
      UserDTO userDTO = new UserDTO();
      userDTO = userLoginService.login(nickName, passWord);

      if (userDTO != null) {
        return new ResponseEntity<>(userDTO, HttpStatus.OK);
      } else {
        return new ResponseEntity<>("NickName or Password is mistake.", HttpStatus.BAD_REQUEST);
      }
    }
  }

  @GetMapping("/getInfo")
  @ApiOperation(value = "get user Info", notes = "get user info")
  public ResponseEntity<User> getInfo(String nickName, String passWord) {
    return new ResponseEntity<>(new User(nickName, passWord, "1234"), HttpStatus.OK);
  }
}
```

```java
@Entity
@Table(name = "t_user")
@Setter
@Getter
@ApiModel(description="User")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @ApiModelProperty(value = "userID")
    private int id;

    @Column(length = 32)
    @ApiModelProperty(value = "userNickName")
    private String nickName;

    @Column(length = 16)
    @ApiModelProperty(value = "userPassWord")
    private String passWord;

    @Column(length = 11)
    @ApiModelProperty(value = "userPhone")
    private String phone;

    public User() {
    }

    public User(String nickName, String passWord, String phone) {
        this.nickName = nickName;
        this.passWord = passWord;
        this.phone = phone;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", nickName='" + nickName + '\'' +
                ", passWord='" + passWord + '\'' +
                ", phone='" + phone + '\'' +
                '}';
    }

}
```

![1578899589961](C:\Users\LINRE\AppData\Roaming\Typora\typora-user-images\1578899589961.png)

