在分布式集群的项目中 , 由于不同服务之间的session是不存在共享的关系 , 所以用Cookie携带一个令牌 , 传递到服务端 ,服务端判断令牌是否存在  , 如果存在就去Redis中找该令牌对应的那条数据 , 如果没有找到就去数据库中查询然后保存到缓存里面 , 所以无论是Cookie还是Redis的缓存 , 一定是具有一定时间的有效性的 , 当过了有效性 ,cookie一旦不存在 , 用户即重新登录 



代码实现 : 

服务:

```java
    /**
     * 通过token来获取redis缓存中的数据
     *
     * @param token
     * @return user对象
     */
    @Override
    public UserInfo getToken(String token) {
        //判断令牌是否为空
        if (token == null) {
            return null;
        }
        //获取Redis中的缓存数据
        String userInfo = template.opsForValue().get(token);
        if (userInfo != null) {
            //重新设置Redis的缓存时间
            template.expire(token, RedisKeys.USER_INFO_LOGIN.getCodeTime(), TimeUnit.SECONDS);
            return JSON.parseObject(userInfo, UserInfo.class);
        }
        return null;
    }
```

拦截器

```java
  /**
     * 拦截器 用于拦截是否有登录
     *
     * @param request   请求
     * @param response  响应
     * @param handler   数据体
     * @return          true存在登录   false不存在登录   
     * @throws Exception 异常
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //判断是否是动态资源          //静态资源时不一样的实现类
        if (handler instanceof HandlerMethod) {
            HandlerMethod method = (HandlerMethod) handler;
            if (method.hasMethodAnnotation(RequireLogin.class)) {
                String token = CookieUtil.getCookie(request, response);
                if (token == null) {
                    response.sendRedirect("/login.html");
                    return false;
                }
                UserInfo userInfo = userInfoRedisService.getToken(token);
                if (userInfo == null) {
                    response.sendRedirect("/login.html");
                    return false;
                }
                request.getSession().setAttribute("userInfo", userInfo);
                return true;
            }
        }
        return true;
    }
```

