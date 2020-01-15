# Vue + springboot 前后端分离部署

## 一、vue-cli3.0以上打包部署

1. 在根目录新建vue.config.js文件，配置如下：

   （配置参考：https://cli.vuejs.org/zh/config/#pages）

```js
module.exports = {
    /* 部署应用包的基本URL */
    /* baseUrl 从 Vue CLI 3.3 起已弃用 ，请使用publicPath */
    //  baseUrl: process.env.NODE_ENV === "production" ? "./" : "./",
    publicPath: process.env.NODE_ENV === "production" ? "./" : "./",
    /* 生产环境构建文件的目录 defalut: dist */
    outputDir: "dist",
    /* 放置生成的静态文件目录（js css img） */
    assetsDir: "static",
    /* 指定生成的index.html 输出路径 相对 default: index.html */
    indexPath: "index.html",
    /* 指定生成文件名中包含hash default: true */
    filenameHashing: true,
    /* 是否保存时 lint 代码 */
    lintOnSave: process.env.NODE_ENV === "production",
    /* 是否使用编译器 default: false */
    runtimeCompiler: true,
    /* 默认babel-loader会忽略node_modules中的文件，你想显示的话在这个选项中列出来 */
    // transpileDependencies: [],
    /* 生产环境的source map */
    productionSourceMap: true,
    // crossorigin: "",
    integrity: false,
    configureWebpack: {
        resolve: {
            alias: {
                'assets': '@/assets',
                'components': '@/components',
                'view': '@/view',
            }
        }
    },
    // css相关配置
    css: {
        // 是否使用css分离插件 ExtractTextPlugin
        extract: true,
        // 开启 CSS source maps?
        sourceMap: false,
        // css预设器配置项
        loaderOptions: {},
        // 启用 CSS modules for all css / pre-processor files.
        modules: false
    },
    devServer: {
        port: 8080,
        host: "0.0.0.0",
        https: false,
        // 自动启动浏览器
        open: false,
        proxy: {
            "/api": {
                //代理路径 例如 https://baidu.com
                target: "https://baidu.com",
                // 将主机标头的原点更改为目标URL
                changeOrigin: true,
                ws: true,
                pathRewrite: {
                    "^/api": ""
                }
            }
        }
    }
};
```

2. 运行npm run build，根目录出现dist文件夹。

3. 将dist文件夹拷贝到tomcat的webapps目录下，打开tomcat即可访问。

4. 访问路径为：

```js
// localhost:[tomcat端口号]/[打包后文件夹名]/[路由]
localhost:8080/dist/index.html
localhost:8080/dist/shopping
```

## 二、vue打包之后遇到的问题

* 如果用到了路由，路由不起作用，可以将路由mode:"history"去掉。

## 三、springboot打包部署

1. 配置的jdk版本要一致，此处以jdk1.8为例。

2. pom文件中打包war：

   ```xml
   <packaging>war</packaging>
   ```

3. 导入依赖：

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-tomcat</artifactId>
       <scope>provided</scope>
   </dependency>
   
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
       <exclusions>
           <exclusion>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-tomcat</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   ```

   ```xml
   <plugin>
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-war-plugin</artifactId>
   </plugin>
   ```

4. 修改Application

   ```java
   @ServletComponentScan
   @SpringBootApplication
   public class App extends SpringBootServletInitializer {
       public static void main(String[] args) {
           System.out.println("Hello World!");
           SpringApplication.run(App.class, args);
       }
   
       @Override
       protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
           return builder.sources(App.class);
       }
   }
   ```

5. 打包，可以运行：

   ```shell
   mvn clean package
   ```
   
   也可以点击ide上，
   
   > File -> project Structure -> Artifacts -> +号 
   >
   > -> Web Application:Exploded -> From Modules -> 选择项目
   >
   > -> +号 -> Web Application:Archive -> For '项目' 
   >
   > -> 修改(项目:war)中的Name(为打包后的文件名) -> ok
   
   > Build -> Build Artififacts -> All Artififacts -> Build
   
6. 将（项目:war）文件拷贝到tomcat的webapps目录下即可。
   
7. 访问路径为：

   ```java
   // localhost:[tomcat端口号]/[打包后文件夹名]/[路由]
   localhost:8080/demo/user/1
   ```

## 三、springboot打包之后遇到的问题

* 注：在springboot打包之后，不能使用字符流去读取资源文件，需要使用字节流，不然取不到数据。

  ```java
  BufferedReader reader = null;
  String content = null;
  
  reader = new BufferedReader(new InputStreamReader(getClass().getClassLoader().getResourceAsStream("static/test.json"), "UTF-8"));
  
  content = IOUtils.toString(reader);
  ```

## 四、跨域

```java
package com.demo.backend.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;

@Configuration
public class CorsConfig extends WebMvcConfigurationSupport {
    private CorsConfiguration buildConfig() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.setAllowCredentials(true);
        return corsConfiguration;
    }

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", buildConfig());
        return new CorsFilter(source);
    }
}
```

