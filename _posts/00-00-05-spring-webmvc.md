---
categories: spring
date: 2017-09-01 19:30
description: 'spring webmvc'
keywords: spring,mvc
layout: post
status: public
title: spring webmvc
---

[官方文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html#mvc-exceptionhandlers)

### spring webmvc 异常处理

文档中给出两种异常处理办法：实现HandlerExceptionResolver接口，实现方法handleIOException返回一个ModelAndView对象

```
@Component
public class MyExceptionResolver implements HandlerExceptionResolver {

    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        return null;
    }
}
```

另外一种方式：在注解了Controller的类中添加一个异常处理方法，添加注解@ExceptionHandler

```
@Controller
public class SimpleController {

    // @RequestMapping methods omitted ...

    @ExceptionHandler(IOException.class)
    public ResponseEntity<String> handleIOException(IOException ex) {
        // prepare responseEntity
        return responseEntity;
    }

}
```

