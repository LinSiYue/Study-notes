# Token校验，拦截器+redis存储

## 一、编写拦截器

```java
package com.sy.example.core.interceptor;

import com.sy.example.core.service.TokenService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class TokenInterceptor implements HandlerInterceptor {

    @Autowired
    private TokenService tokenService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        try {
            return tokenService.checkToken(request.getHeader("name"), request.getHeader("token"));
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 请求处理之后进行调用，但是在视图被渲染之前（Controller方法调用之后）
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
        // System.out.println("执行了TestInterceptor的postHandle方法");
    }

    /**
     * 在整个请求结束之后被调用，也就是在DispatcherServlet 渲染了对应的视图之后执行（主要是用于进行资源清理工作）
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        // System.out.println("执行了TestInterceptor的afterCompletion方法");
    }

}
```

## 二、编写拦截器配置类

* 注意在注册拦截器的时候要用注入，不然用new创建对象时，会导致调用redisUtil的注入不成功，出现空指针异常。

```java
package com.sy.example.core.config;

import com.sy.example.core.interceptor.TokenInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class TokenConfig implements WebMvcConfigurer {

    @Autowired
    private TokenInterceptor tokenInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        InterceptorRegistration registration = registry.addInterceptor(tokenInterceptor);
        // InterceptorRegistration registration = registry.addInterceptor(new TokenInterceptor());
        registration.addPathPatterns("/**");
        registration.excludePathPatterns(
                "/user/login",
                "/user/logout"
        );
    }

}
```