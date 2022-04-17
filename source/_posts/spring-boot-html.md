---
title: 「Spring」SpringBoot Thymeleaf页面静态化实现
date: 2021-05-11 14:12:42
tags: [后端开发, Spring, Thymeleaf, SpringBoot]
categories: Spring
---

页面静态化是指把动态生成的HTML页面变为静态文件保存，当请求到来，直接访问静态文件，而不需要经过项目服务器的渲染。<!-- more -->

### 1. 配置Nginx代理静态页面
``` conf
location / {
    root D:/temp/static;                  # 自定义静态文件存放根目录
    set $www_temp_path $request_filename; # 设置请求的文件名到临时变量
    if ($uri = '/') {                     # 若为根目录则加上/index.html
        set $www_temp_path $request_filename/index.html;
    }
    if (!-f $www_temp_path) {             # 若请求的文件不存在，就反向代理服务器的渲染
        proxy_pass http://127.0.0.1:8080;
    }
    # 其他配置...
}
```
然后重启`Nginx`。


### 2. Thymeleaf实现手动把模板渲染结果写入到指定位置
`Thymeleaf`手动渲染原理：当与`SpringBoot`结合时，放入`Model`的数据就会被放到上下文(`Context`)中，并且此时模板解析器(`TemplateResolver`)已经创建完成(默认模板存放位置:`templates`，默认模板文件类型:`html`)；然后通过模板引擎(`TemplateEngine`)结合上下文(`Context`)与模板解析器(`TemplateResolver`)，利用内置的语法规则解析，从而输出解析后的文件到指定目录。
``` java
/**
 * 模板引擎处理函数
 * 
 * @param templateName 模板名
 * @param context      上下文
 * @param writer       输出目的地的流
 */
templateEngine.process(templateName, context, writer);
```
- **具体实现**

相关依赖：
``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
</dependency>
```

配置文件：
``` ymal
server:
  port: 8080
# 自定义静态文件存放根目录
myserver:
  destPath: D:/temp/static
```

接口和实现类：
``` java
@RestController
public class GenerateApi {
    @Resource
    private GenerateHtml generateHtml;
    /**
     * 生成静态首页
     */
    @GetMapping("/generate/home")
    public  String generateStaticHome(ServerWebExchange exchange) {
        return generateHtml.generateStaticHome(exchange);
    }
}

public interface GenerateHtml {
    String generateStaticHome(ServerWebExchange exchange);
}

@Slf4j
@Service
public class GenerateHtmlImpl implements GenerateHtml {
    /** 打包时间 */
    @Value("${spring.application.build-time}")
    private String buildTime;
    
    /** 静态文件存放根目录 */
    @Value("${myserver.destPath}")
    private String destPath;
    
    /** 模板引擎 */
    @Resource
    private TemplateEngine templateEngine;
    
    /**
     * 生成静态首页
     */
    @Override
    public String generateStaticHome(ServerWebExchange exchange) {
        // 上下文
        SpringWebFluxContext context = new SpringWebFluxContext(exchange);
        // 设置页面数据
        Map<String, Object> modelMap = new HashMap<>();
        modelMap.put("name", "testGenerateStaticHome");
        modelMap.put("version", buildTime);
        context.setVariables(modelMap);
        // 输出流
        File dest = new File(destPath, "index.html");
        if (dest.exists()) {
            boolean delete = dest.delete();
        }
        try (PrintWriter writer = new PrintWriter(dest, "UTF-8")) {
            // 模板引擎生成html
            templateEngine.process("index", context, writer);
            return "[静态页服务]：生成静态首页成功! ^_^";
        } catch (Exception e) {
            log.error("[静态页服务]：生成静态首页异常!", e);
            return "[静态页服务]：生成静态首页异常!"+e.toString();
        }
    }
}
```

html模板原型：
``` html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Thymeleaf 静态页面</title>
    <link th:href="@{/css/index.css(v=${version})}" rel="stylesheet" type="text/css">
</head>
<body>
	<h1 th:text="'页面名字：' + ${name}"></h1>
	<h1 th:text="'页面版本：' + ${version}"></h1>
</body>
</html>
```

启动测试：浏览器访问：`http://127.0.0.1:8080/generate/home`或编写测试类测试。


### 3. 将项目resources目录下静态资源复制到指定位置
相关依赖：
``` xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.8.0</version>
</dependency>
```

接口和实现类：
``` java
public interface GenerateHtml {
    /**
     * 拷贝静态资源文件
     */
    String generateStaticFiles();
}

@Slf4j
@Service
public class GenerateHtmlImpl implements GenerateHtml {
    /** 静态文件存放根目录 */
    @Value("${myserver.destPath}")
    private String destPath;

    /**
     * 拷贝静态资源文件
     */
    @Override
    public String generateStaticFiles() {
        try {
            copyResourceToFile();
            return "[静态页服务]：拷贝静态资源文件成功! ^_^";
        } catch (Exception e) {
            log.error("[静态页服务]：拷贝静态资源文件异常!", e);
            return "[静态页服务]：拷贝静态资源文件异常!"+e.toString();
        }
    }

    /**
     * 资源清单获取并拷贝
     */
    private void copyResourceToFile() throws IOException {
        // 资源清单获取
        ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        org.springframework.core.io.Resource[] resources = resolver.getResources("static/**");
        for (org.springframework.core.io.Resource resource : resources) {
            String fileName = resource.getFilename();
            assert fileName != null;
            if (fileName.indexOf(".") > 0) {
                InputStream inputStream = null;
                try {
                    inputStream = resource.getInputStream();
                } catch (Exception e) {
                    log.warn(String.format("[%s]获取输入流发生异常!", resource.getURL()));
                    throw new RuntimeException(String.format("[%s]获取输入流发生异常!", resource.getURL()));
                }
                // 分析相对目录
                String tempPath = "";
                String[] urls = resource.getURL().toString().split("/static/");
                if (urls.length >= 2 ){
                    tempPath = urls[urls.length-1];
                } else {
                    throw new RuntimeException("relativeRootPath有误：无法分析相对目录");
                }
                tempPath = tempPath.substring(0, tempPath.length() - fileName.length());
                if (StringUtils.isEmpty(tempPath)) {
                    tempPath = File.separator;
                }
                String filePath = destPath + File.separator + tempPath;
                if (createDir(filePath)) {
                    String destName = filePath + fileName;
                    // 输出流
                    File dest = new File(destName);
                    if (dest.exists()) {
                        boolean delete = dest.delete();
                    }
                    FileUtils.copyInputStreamToFile(inputStream, dest);
                } else {
                    throw new RuntimeException(String.format("创建本地目录[%s]失败！", resource.getURL()));
                }
            }
        }
    }

    /**
     * 创建目录
     */
    private static boolean createDir(String dirName) {
        File dir = new File(dirName);
        if (dir.exists()) {
            return true;
        }
        if (!dirName.endsWith(File.separator)) {
            dirName = dirName + File.separator;
        }
        if (dir.mkdirs()) {
            log.warn("创建目录" + dirName + "成功！");
            return true;
        } else {
            log.warn("创建目录" + dirName + "失败！");
            return false;
        }
    }
}
```

可在项目启动后执行覆盖拷贝操作：
``` java
@Slf4j
@Component
@Order(value = 10)
public class MyCommandLineRunner implements CommandLineRunner {
    @Resource
    private GenerateHtml generateHtml;

    @Override
    public void run(String... args) throws Exception {
          log.info("执行MyCommandLineRunner:拷贝静态文件！");
          // 拷贝静态资源文件
          generateHtml.generateStaticFiles();
    }
}
```

* 重启测试：
发现指定目录生成了静态文件，并且请求速度得到了极大提升。

生成静态文件：
``` html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Thymeleaf 静态页面</title>
    <link href="/css/index.css?v=20210511-075911" rel="stylesheet" type="text/css">
</head>
<body>
    <h1>页面名字：testGenerateStaticHome</h1>
    <h1>页面版本：20210511-075911</h1>
</body>
</html>
```
