---
title: ServletContextListener使用
tags: ServletContextListener
sidebar: 
  nav: docs-zh
---
  ServletContextListener能够监听ServletContext的生命周期，即监听Web的生命周期；当Servlet容器启动或终止时会触发ServletContextEvent事件，并由ServletContextListener接口的两个方法去处理；
  当Servlet 容器启动Web 应用时调用contextInitialized(ServletContextEvent sce)方法。在调用完该方法之后，容器再对Filter 初始化，并且对那些在Web 应用启动时就需要被初始化的Servlet 进行初始化;当Servlet 容器终止Web 应用时调用contextDestroyed(ServletContextEvent sce)方法。在调用该方法之前，容器会先销毁所有的Servlet 和Filter 过滤器。
  例：容器启动时，获取bean对象，做一些初始化操作；
```
@Override
public void contextInitialized(ServletContextEvent sce) {
    ApplicationContext appContext = WebApplicationContextUtils.getWebApplicationContext(sce.getServletContext());
    if (appContext != null) {
        userMapper = (OrderViewInfoMapper) appContext.getBean("userMapper");
    }
    if (userMapper != null) {
        initUserMap();
    }
}
```