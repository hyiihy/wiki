# <center>Spring Session</center>

## Sessio共享方案  
- **粘性Session**
- **服务器Session复制**
- **Session持久化到数据库**
- **Terracotta？**
- **使用memcached、redis**

## Spring Session redis
单主机时，Session一般保存在Tomcat等Servlet容器的内存中。  
集群时Session共享可使用Tomcat等提供的Session共享功能。
而Spring Session可以不依赖Tomcat，直接在Web层实现Session共享。

## 操作
1. Maven加入依赖
- web.xml中配置Filter，HttpSession的实现被Spring Session替换，操作HttpSession等同于操作redis中的数据。
- Spring中配置redis属性等
- 实际项目中，往往通过Controller层切面将Session中的信息放到ThreadLocal中。
