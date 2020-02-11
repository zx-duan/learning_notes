## json参数的加密解密问题

1.如何对请求的json参数进行加密解密的步骤



使用**RequestBodyAdvice**接口来为controller传入的参数进行

使用步骤

创建一个类实现 RequestBodyAdvice 接口 , 重写内部的四个方法 , 在类上面使用控制器增强的的注解 `@controllerAdvice`

**代码**

```java
@ControllerAdvice
public class test implements RequestBodyAdvice {
    @Override
    public boolean supports(MethodParameter methodParameter, Type type, Class<? extends HttpMessageConverter<?>> aClass) {
        return false;
    }

    @Override
    public HttpInputMessage beforeBodyRead(HttpInputMessage httpInputMessage, MethodParameter methodParameter, Type type, Class<? extends HttpMessageConverter<?>> aClass) throws IOException {
        return null;
    }

    @Override
    public Object afterBodyRead(Object o, HttpInputMessage httpInputMessage, MethodParameter methodParameter, Type type, Class<? extends HttpMessageConverter<?>> aClass) {
        return null;
    }

    @Override
    public Object handleEmptyBody(Object o, HttpInputMessage httpInputMessage, MethodParameter methodParameter, Type type, Class<? extends HttpMessageConverter<?>> aClass) {
        return null;
    }
}


```

