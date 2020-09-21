# Eclipse
[TOC]

## 插件

在eclipse目录下新建 links 和 plugins_my 目录
插件放到 plugins_my 下

vim.link放到links下. 里面写上插件的目录
```
path=D:\\Program\\eclipse\\jee-neon\\eclipse\\plugins_my\\vrapper
```


## Maven
### Cannot change version of project facet Dynamic Web Module to 3.0?

1. 修改org.eclipse.wst.common.project.facet.core.xml version为3.0
```
<installed facet="jst.web" version="3.0"/>
```

2. Maven -> Update Project

### "javax.servlet.http.HttpServlet" was not found on the Java Build Path

```

```




