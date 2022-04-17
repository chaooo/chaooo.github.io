---
title: 「Spring」SpringBoot 整合 FastDFS
date: 2020-08-02 20:25:23
tags: [后端开发, Spring, SpringBoot, FastDFS]
categories: Spring
---


### 1. 在maven项目pom.xml中添加依赖
``` xml
<dependency>
    <groupId>com.github.tobato</groupId>
    <artifactId>fastdfs-client</artifactId>
    <version>1.27.2</version>
</dependency>
```

<!-- more -->

### 2. 在application.yml中添加配置
``` ymal
#FDFS配置
fdfs:
  so-timeout: 5000 #上传的超时时间
  connect-timeout: 2000 #连接超时时间
  thumb-image:             #缩略图生成参数
    width: 150
    height: 150
  tracker-list:            #TrackerList参数,支持多个
  - 192.168.2.100:22122
```

### 3. 编写FastDFS工具类
``` java
@Component
public class FastDfsUtil {

    private static ThumbImageConfig thumbImageConfig;
    private static FastFileStorageClient fastFileStorageClient;
    private static FdfsWebServer fdfsWebServer;

    public FastDfsUtil(ThumbImageConfig thumbImageConfig, FastFileStorageClient fastFileStorageClient, FdfsWebServer fdfsWebServer) {
        FastDfsUtil.thumbImageConfig = thumbImageConfig;
        FastDfsUtil.fastFileStorageClient = fastFileStorageClient;
        FastDfsUtil.fdfsWebServer = fdfsWebServer;
    }

    /**
     * @param multipartFile 文件对象
     * @return 返回文件地址
     * @author qbanxiaoli
     * @description 上传文件
     */
    @SneakyThrows
    public static String uploadFile(MultipartFile multipartFile) {
        StorePath storePath = fastFileStorageClient.uploadFile(multipartFile.getInputStream(), multipartFile.getSize(), FilenameUtils.getExtension(multipartFile.getOriginalFilename()), null);
        return storePath.getFullPath();
    }
    
    // 其他代码...
}
```

### 4. 编写测试Controller
``` java
@Slf4j
@RestController
public class FileController {
    /**
     * 上传图片并保存
     * @param file
     * @param request
     * @return
     * @throws IOException
     */
    @PostMapping("/upload")
    public Map<String, Object> uploadFile(MultipartFile[] file, HttpServletRequest request) throws IOException {
        Map<String, Object> map = new HashMap<String, Object>();
        if (file != null && file.length > 0) {
            String saveFile = null;
            for (int i = 0; i < file.length; i++) {
                MultipartFile partFile = file[i];
                // 保存文件
                saveFile = FastDfsUtil.uploadFile(partFile);;
                saveFile = "http://192.168.2.100/" + saveFile;
            }
            map.put("data", saveFile);
            map.put("msg", "上传成功");
            log.info(">>>>>>>>>>>>>>>>图片上传成功:" + saveFile);
        } else {
            map.put("msg", "上传失败");
            log.info(">>>>>>>>>>>>>>>>图片上传失败");
        }
        return map;
    }
}
```


### 5. 编写前端页面代码
``` html
<body>
    <div>
        图片上传：
        <input type="file" name="file" value="" id="input-file" accept="image/png,image/jpeg,image/gif,image/jpg">
    </div>
    <div>
        图片回显：
        <img id="upload-file" src="" alt="" width="300">
    </div>
<script charset="utf-8" type="text/javascript" src="/lib/jquery-3.5.1.min.js"></script>
<script charset="utf-8" type="text/javascript">
$('#input-file').on("change", function() {
    var file = this.files[0];
    var formData = new FormData();
    formData.append('file', file);
    
    // 开始上传
    fileUpload("/upload", formData);

    // 测试读取图片(本地显示)
    var fileReader = new FileReader();
    fileReader.readAsDataURL(file);
    fileReader.onloadend = function(fEvent){
        var src = fEvent.target.result;
        console.log(src);
        $("#upload-file").attr("src", src);
    }
});

function fileUpload(url, formData){
    $.ajax({
        url: url,
        type: 'POST',
        cache: false,
        data: formData,
        processData: false,
        contentType: false,
        dataType: "json",
        success: function (res) {
            console.log(res);
        },
        error: function (xhr, type, errorThrown) {
            console.log("照片上传失败")
        }
    });
}
</script>
```


### 6. 浏览器访问页面测试
![](https://oscimg.oschina.net/oscnet/up-83d0d727e478c0a4832f661f79076b4128d.JPEG)


> demo源码地址：[https://gitee.com/chaoo/fastdfs-demo.git](https://gitee.com/chaoo/fastdfs-demo.git)