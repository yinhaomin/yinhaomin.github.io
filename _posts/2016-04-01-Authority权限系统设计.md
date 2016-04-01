---
layout: post
title: 权限系统设计
comments: true
keywords: 权限,权限系统,系统设计
---

在Filter中

```
  @Override
  publicvoiddoFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletResponse httpServletResponse = (HttpServletResponse) response;
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        if (httpServletRequest.getPathInfo().startsWith("/api/hui/inner/coupon/list")) {
            chain.doFilter(request, response);
        } else {
            CasClient casClient = new CasClient(httpServletRequest, httpServletResponse, casInfo);
            try {
                CasCheckResponse casCheckResponse = casClient.validateST();
                if (casCheckResponse == null) {
                    // unexpected response from uc
                    HttpResponseUtils.notLoginResponse(httpServletResponse);
                } else {
                    httpServletRequest.setAttribute(UCConstant.UC_USER_ATTRIBUTE_KEY,
                            casCheckResponse.getUcid());
                    httpServletRequest.setAttribute(UCConstant.UC_USERNAME_ATTRIBUTE_KEY,
                            casCheckResponse.getUsername());
                    httpServletRequest.setAttribute(UCConstant.UC_USER_AUTH_ATTRIBUTE_KEY, manager.findAuthIdsByUcId
                            (casCheckResponse.getUcid()));
                    chain.doFilter(request, response);
                }
            } catch (NotLoginException e) {
                HttpResponseUtils.notLoginResponse(httpServletResponse);
            } catch (McpackException e) {
                HttpResponseUtils.notLoginResponse(httpServletResponse);
            } catch (IOException e) {
                return;
            }
        }
  }
```

在AOP中
```
  @Around(value = "@annotation(casAuth)")
  public Object checkAuth(ProceedingJoinPoint joinPoint, CASAuth casAuth) throws Throwable {
        // check user attribute
        List<Long> authList = AuthUtils.getCurrentAuths();
        if (authList != null) {
            if (authList.contains(casAuth.authId())) {
                return joinPoint.proceed();
            } else {
                BaseResponse response = new BaseResponse(UCConstant.NOT_AUTH_STATUS, UCConstant.NOT_AUTH_MSG);
                return response;
            }
        } else {
            BaseResponse response = new BaseResponse(UCConstant.NOT_LOGIN_STATUS, UCConstant.NOT_LOGIN_MSG);
            return response;
        }
  }
AuthUtils.java

  publicstaticList<Long> getCurrentAuths() {
        HttpServletRequest httpServletRequest =
                ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        List<Long> auths = (List<Long>) httpServletRequest.getAttribute(UCConstant.UC_USER_AUTH_ATTRIBUTE_KEY);
        return auths;
  }
```
在web.xml中配置
```
<listener>
<listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
</listener>
```
问题QA
登陆账户是如何管理的，是不是每个UC账户都有登陆的权限?
是的，新的账户没有权限的话，登陆进去后停在一张啥都没有的页面
 
权限的分表
分为权限表，角色表，角色与权限的对照表，用户表，用户与角色的对照表
一个用户对应多个角色的时候，其权限是怎样的?
目前是将所有的权限做与操作
 
通过/hui-web-1.0/api/hui/inner/session?castk=6700fgd7f0c8ef980d159
获取到了用户的所有的权限
 
疑问
看到了从
CASStatusAOP.checkAuth(ProceedingJoinPoint joinPoint, CASAuth casAuth) throws Throwable
里面使用了前端传过来的权限的表做用户对一个Action是否有权限的判断，感觉这个表可以被前端更改的，是不是使用新读取出来的表比较合理些。
现在是在UCFilter里面加上了authlist，不是前端传输的
 
com.baidu.hui.web.filter.UCFilter过滤器过了权限
调用了UC的sdk实现是否登陆的判断
 
权限树在数据库中按照
Root->最大Tab->子Tab->子功能生成
Tab是否可见
 
前端是如何将authList告诉后端的？这个在header里面没有找到。
不是后端告诉的
 
Annotation的使用
注解
@Retention –什么时候使用该注解
@Target? –注解用于什么地方
@casAuth注解了方法的使用的地方
 
如何将不同的item分发给不同的user，怎么筛选出来的?
Item表有assign_id作为标示字段
 
这个API是干嘛用的? 哪里会调用?
/api/hui/inner/auth/user/verify
 
API role/add验证了Id和user name
那么uc那边，uc的名称会改变吗?如果改变的话，这里不久不对应了吗?
 
在分配Item的时候，没有验证user是否有优惠Tab权限?
这个pm可以接受，但是是一个可以优化的地方
 
 
帮助信息
Web.xml的加载过程
此段试图说明在
CASStatusAOP. checkAuth中调用了
```
  List<Long> authList = AuthUtils.getCurrentAuths();
    publicstaticList<Long> getCurrentAuths() {
        HttpServletRequest httpServletRequest =
                ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        List<Long> auths = (List<Long>) httpServletRequest.getAttribute(UCConstant.UC_USER_AUTH_ATTRIBUTE_KEY);
        return auths;
  }
```
这段代码中的RequestContextHolder的使用
 
启动Web项目的时候，容器包括(JBoss、Tomcat等)首先会读取项目web.xml配置文件的配置，当这一步没问题时，程序才启动起来.
1. 启动WEB项目时，容器首先会去它的配置文件web.xml读取两个节点:
<listener></listener>和<context-param></context-param>
2. 然后容器创建ServletContext（application）,这个容器的所有部分都共享这个上下文.
3. 容器以<context-param></context-param>的name作为键，value作为值，将其转化为键值对，存入ServletContext
4. 容器创建<listener></listener>中的类实例，根据配置的class类路径<listener-class>来创建监听，在监听中会有contextInitialized(ServletContextEvent args)初始化方法，启动Web应用时，系统调用Listener的该方法，在这个方法中获得：
ServletContext application =ServletContextEvent.getServletContext();
context-param的值= application.getInitParameter("context-param的键");
 
总的来说，web.xml的加载顺序是: <context-param>-> <listener> -> <filter> -> <servlet>
