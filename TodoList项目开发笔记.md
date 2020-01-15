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

   * 报错

     ```
     Failed to load resource: the server responded with a status of 404 ()
     :8087/#/dashboard:1
     
     Access to XMLHttpRequest at 'http://localhost:8086/sockjs-node/info?t=1579069561892' from origin 'http://localhost:8087' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
     ```

     ![1579069595190](C:\Users\LINRE\AppData\Roaming\Typora\typora-user-images\1579069595190.png)

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

