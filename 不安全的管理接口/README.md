# 不安全的管理接口

## Springboot-Actuator

Actuator endpoints let you monitor and interact with your application. 
Spring Boot includes a number of built-in endpoints and lets you add your own. 
For example, the health endpoint provides basic application health information. 
Some of them contains sensitive info such as :

执行器终端可以让你监控应用程序并进行交互。Spring Boot包含许多端点，你也可以添加你自己的。比如说，运行状态端提供基本的应用程序的运行状态信息。其中会包含一些敏感信息，例如:

- `/trace` (默认情况下，目录下包含最近一百个HTTP请求以及包头)
- `/env` (当前环境属性)
- `/heapdump` (从应用程序使用的JVM中构建并返回堆dump文件). 

这些端点在 Springboot 1.X中是默认开启的. 但是在 Springboot 2.x 中只默认开启了 `/health` 和 `/info` 。


## References

* [Springboot - Official Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html)

