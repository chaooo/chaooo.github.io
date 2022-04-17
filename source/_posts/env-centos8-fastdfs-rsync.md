---
title: 「环境配置」记一次生产事故引发的FastDFS图片迁移
date: 2021-05-28 22:11:23
tags: [环境配置, CentOS, FastDFS]
categories: 环境配置
---

- **事件经过**：由于前端同学不小心把上传图片服务器地址写死了测试域名（指向测试服务器），然后项目上到正式环境，一段时间后，发现用户发布商品时的商品详情富文本中的图片全部指向测试图片服务器域名，然后图片又太多了，手动逐条修复数据不太现实。<!-- more -->

- **解决思路**：
    + 从数据库中查询商品详情富文本，分析出所有测试域名图片的`Url`地址；
    + 通过`Url`下载图片到本地服务器并保持图片存放路径与图片文件名和原本一致；
    + 通过`rsync`远程同步命令同步到正式服务器；
    + 修改数据库中商品图片`Url`地址指向正式服务器域名(因为路径和文件名与测试服务器一致，只需替换域名即可)。

1. 引入`pom.xml`依赖

``` xml
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.11.2</version>
</dependency>
```

2. 提供出临时接口以备调用同步图片

``` java
/**
 * 提供出临时接口以备调用修复
 */
@PostMapping("/product/img/repair")
public void repairProductImg() {
    // 获取带有测试域名图片的富文本列表
    List<String> infoList = productDao.getProductInfo();
    // 下载图片到本地服务器
    infoList.forEach(RepairImgUtil::saveProductImg);
}
```

``` sql
<!-- 这里测试图片服务器域名为：img-test.abc.com -->
<select id="getProductInfo" resultType="java.lang.String">
      select detail from productInfo where detail like '%img-test.abc.com%'
</select>
```

``` java
@Slf4j
public class RepairImgUtil {
    /**
     * 保存图片到本地服务器
     * @param textBody 富文本
     */
    public static void saveProductImg(String textBody) {
        // 解析富文本
        Element doc = Jsoup.parseBodyFragment(textBody).body();
        Elements images = doc.select("img[src]");
        List<String> srcList = new ArrayList<>();
        for (Element element : images) {
            String imgUrl = element.attr("src");
            // 筛选测试服务器的图片路径
            if (imgUrl.indexOf("img-test.abc.com") > 0) {
                srcList.add(imgUrl);
            }
        }
        // 本地存放路径
        String basePath = "/apps/fdfs/storage/data/";
        srcList.forEach(img -> {
            try {
                // 下载图片，并根据路径规律保持原有路径
                RepairImgUtil.downloadImage(img, img.substring(45), basePath+img.substring(39, 45));
            } catch (Exception e) {
                log.error("try-catch:",e);
            }
        });
    }

    /**
     * 下载图片
     *
     * @param urlString 图片链接
     * @param filename  图片名称
     * @param savePath  保存路径
     */
    private static void downloadImage(String urlString, String filename, String savePath) throws Exception {
        // 构造URL打开连接，并设置输入流与缓冲
        URL url = new URL(urlString);
        URLConnection con = url.openConnection();
        con.setConnectTimeout(5*1000);
        InputStream is = con.getInputStream();
        byte[] bs = new byte[1024];
        // 读取到的数据长度
        int len;
        // 输出的文件流
        File sf=new File(savePath);
        if(!sf.exists()){
           sf.mkdirs();
        }
        OutputStream os = new FileOutputStream(sf.getPath()+"/"+filename);
        // 开始读取
        while ((len = is.read(bs)) != -1) {
            os.write(bs, 0, len);
        }
        os.close();
        is.close();
        // 打印图片链接
        log.info(urlString);
    }
}
```

3. 调用接口后，图片会先备份到本地服务器，通过rsync远程同步命令同步到正式服务器

``` bash
# 登录远程服务器，进入图片服务器目录
cd /apps/fdfs
# 远程登陆下载图片的服务器并同步数据
rsync -avz myuser@119.29.36.15:/apps/fdfs/storage  ./

# 根据提示，确认连接输入 yes，输入本地服务器（下载图片的服务器）用户密码后开始同步

The authenticity of host '119.29.36.15 (119.29.36.15)' can't be established.
ECDSA key fingerprint is SHA256:5m4KgPF0QgBO1xE7Tz1RT7U/tfCue+QBE/t4zEDEDJQ.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '119.29.36.15' (ECDSA) to the list of known hosts.
myuser@119.29.36.15's password: 
receiving incremental file list
storage/
storage/data/
storage/data/03/
storage/data/03/08/
storage/data/03/08/Cmgy61-BU3GANDoCAAGOnj-x-ws715.jpg
storage/data/03/08/Cmgy61-BU8SAeTvqAAGOnj-x-ws868.jpg
storage/data/03/08/Cmgy61-BVn-AdXoKAA2Msh3VVsk076.jpg
...
storage/data/03/3C/Cmgy62CwSTOARHdWABLV4kLKua4925.png

sent 15,786 bytes  received 422,224,711 bytes  10,425,691.28 bytes/sec
total size is 448,804,964  speedup is 1.06
```

4. 把测试域名换成正式域名访问图片成功！最后修改数据库商品详情：
``` sql
update productInfo set detail = REPLACE(detail, 'img-test.abc.com%'', 'img.abc.com%'') where detail like like '%img-test.abc.com%'
```

