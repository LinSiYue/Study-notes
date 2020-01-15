# TodoList项目开发笔记

## 一、遇到的难题

1. 跨域

   * vue.config

     ```js
     proxy: {
           [process.env.VUE_APP_BASE_API]: {
             // project-backend ip and port
             target: 'http://127.0.0.1:8086',
             // if cross
             changeOrigin: true,
             pathRewrite: {
               ['^' + process.env.VUE_APP_BASE_API]: ''
             }
           }
           // after: require('./mock/mock-server.js') // remove the mock
     }
     ```

   * .env.development

     ```
     // url = target + VUE_APP_BASE_API + api
     VUE_APP_BASE_API = ''
     ```

   * /utils/request.js

     ```js
     const service = axios.create({
       baseURL: process.env.VUE_APP_BASE_API, // url = base url + request url
       // withCredentials: true, // send cookies when cross-domain requests
       timeout: 5000 // request timeout
     })
     ```

   * 如果还使用了mock，在main.js去掉

     ```js
     mockXHR()
     ```

   * 添加api

     ```js
     import request from '@/utils/request'
     
     // form submit
     export function login(data) {
       return request({
         url: '/user/login',
         method: 'post',
         data
       })
     }
     
     export function getInfo(userName, token) {
       return request({
         url: '/user/getInfo',
         method: 'get',
         params: { userName, token }
       })
     }
     ```

2. 报错

     ```
     Failed to load resource: the server responded with a status of 404 ()
     :8087/#/dashboard:1
     
     Access to XMLHttpRequest at 'http://localhost:8086/sockjs-node/info?t=1579069561892' from origin 'http://localhost:8087' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
     ```

     ![1579069595190](https://github.com/LinSiYue/Study-notes/blob/master/img/todolist%E5%BC%80%E5%8F%91%E7%AC%94%E8%AE%B0/sockjs%20error.png?raw=true)

     解决：在vue.config.js中加入public，参考https://www.jianshu.com/p/147083b647ef

     ```js
     proxy: {
         // change xxx-api/login => mock/login
         // detail: https://cli.vuejs.org/config/#devserver-proxy
         [process.env.VUE_APP_BASE_API]: {
             // target: `http://127.0.0.1:${port}/mock`,
             target: 'http://127.0.0.1:8086',
             changeOrigin: true,
             pathRewrite: {
                 ['^' + process.env.VUE_APP_BASE_API]: ''
             }
         }
     },
     public: '127.0.0.1:8087'
     ```

    * vue.config.js配置：

     https://webpack.docschina.org/configuration/dev-server/#src/components/Sidebar/Sidebar.jsx

## 二、初始化数据库

在resources目录下添加文件

* schema.sql

  ```sql
  DROP TABLE IF EXISTS `t_user`;
  CREATE TABLE `t_user`(
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `user_name` VARCHAR(32) DEFAULT NULL ,
    `pass_word` VARCHAR(16) DEFAULT NULL ,
    `phone` VARCHAR(11) DEFAULT NULL ,
    `roles` VARCHAR(11) DEFAULT NULL ,
    `introduction` VARCHAR(32) DEFAULT NULL ,
    `avatar` VARCHAR(255) DEFAULT NULL ,
    PRIMARY KEY (`id`)
  )
  ```

* data.sql

  ```sql
  INSERT INTO t_user VALUES (1, "Gold", "Lin123.", "13452220114", "editor", "I am an editor", "https://wpimg.wallstcn.com/f778738c-e4f8-4870-b634-56703b4acafe.gif")
  
  INSERT INTO t_user VALUES (2, "Reece", "qq123.", "13452220114", "editor", "I am an editor", "https://wpimg.wallstcn.com/f778738c-e4f8-4870-b634-56703b4acafe.gif")
  
  INSERT INTO t_user VALUES (3, "Zhang", "ww123.", "13452220114", "editor", "I am an editor", "https://wpimg.wallstcn.com/f778738c-e4f8-4870-b634-56703b4acafe.gif")
  
  INSERT INTO t_user VALUES (4, "Lin", "zz123.", "13452220114", "editor", "I am an editor", "https://wpimg.wallstcn.com/f778738c-e4f8-4870-b634-56703b4acafe.gif")
  
  INSERT INTO t_user VALUES (5, "Wang", "ss123.", "13452220114", "editor", "I am an editor", "https://wpimg.wallstcn.com/f778738c-e4f8-4870-b634-56703b4acafe.gif")
  
  INSERT INTO t_user VALUES (6, "Tian", "dd123.", "13452220114", "editor", "I am an editor", "https://wpimg.wallstcn.com/f778738c-e4f8-4870-b634-56703b4acafe.gif")
  ```

* 修改配置文件application-dev.yml

  ```yml
  server:
    port: 8086
  spring:
    jpa:
      generate-ddl: false # 修改
      show-sql: true
      hibernate:
        ddl-auto: none # 修改
    datasource:
      continue-on-error: false
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/todolist?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai&zeroDateTimeBehavior=CONVERT_TO_NULL
      username: root
      password: ''
      initialization-mode: ALWAYS # 修改
  ```

  