
1.web.xml中新增如如下代码
```xml
<context-param>
  <param-name>javax.servlet.jsp.jstl.fmt.localizationContext
  </param-name>
  <param-value>local.messages</param-value>
</context-param>
```


2.local.message代表

代表你需要在src/main/resource目录下有local目录，并且该目录有以message_zh_CN.proeprties
或者message_en_US.properites开头的文件
这些文件就是国际化用到的

国际化文件样式如下
```
username=Jay
username=周杰伦
```

3.在页面使用
```xml
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt"%>
<fmt:message key="username"></fmt:message>
```
