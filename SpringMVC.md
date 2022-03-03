# **SpringMVC** 

## 概述

**SpringMVC** 是一种基于 Java 的实现 **MVC 设计模型**的请求驱动类型的轻量级 **Web 框架**，属于**SpringFrameWork** 的后续产品，已经融合在 Spring Web Flow 中。

SpringMVC 已经成为目前最主流的MVC框架之一，并且随着Spring3.0 的发布，全面超越 Struts2，成为最优秀的 MVC 框架。==它通过一套注解，让一个简单的 Java 类成为处理请求的控制器，而无须实现任何接口==。同时它还支持 **RESTful** 编程风格的请求。

<img src="images/image-20220303201310244.png" alt="image-20220303201310244" style="zoom:80%;" />

先回归一下，前面的Web知识，客户端服务端请求响应的逻辑：

- 每个Servlet都有一些共有行为，也就是一般情况下都要执行的。如先接收求参数，封装实体，访问业务层，接收返回结果，指派视图这几个。所以我们就考虑是不是可以把这些共有的行为抽取出来呢？

<img src="images/image-20220303202424095.png" alt="image-20220303202424095"  />

- 下面我们以2个Servlet为例，分析一下，把共有行为抽取出来，这样不就很好了嘛。

![image-20220303203033171](images/image-20220303203033171.png)

- 而前面的共有行为肯定不是我们自己写撒，这是由框架来提供的，并且这个共有行为部分框架被称为**前端控制器**（这也是SpringMVC的核心），这里就实现了前面提到的所有共有行为，还需要加入的其他的特殊行为就是通过我们的POJO（JavaBean）来自己实现



## 快速入门

需求：客户端发起请求，服务器端接收请求，执行逻辑并进行视图跳转。

开发步骤：

① 导入SpringMVC相关坐标

② 配置SpringMVC核心控制器DispathcerServlet

③ 创建Controller类（图中的POJO）和视图页面

④ 使用注解配置Controller类中业务方法的映射地址（到Spring容器里面）

⑤ 配置SpringMVC核心文件 spring-mvc.xml（实现组件扫描功能）。和Spring的配置文件是分开独立的。

⑥ 客户端发起请求测试

![image-20220303203729863](images/image-20220303203729863.png)

