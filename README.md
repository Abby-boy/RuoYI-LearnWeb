<p align="center" >
<span style="background-color: white ;display: inline-block"> 
<img alt="logo" src="./src/assets/logo/logo.png">
</span>
</p>
<h1 align="center" style="margin: 30px 0 30px; font-weight: bold;">若依springboot开发框架前端</h1>


## 项目需求

|    内容    |                        要求                         |
| :--------: | :-------------------------------------------------: |
|   redis    |                        >=5.0                        |
| RuoYI-Vue  | git clone https://gitee.com/y_project/RuoYi-Vue.git |
|    Vue     |                        vue3                         |
|    JDK     |                        >=1.8                        |
|   Mysql    |                        >=5.7                        |
|   Nodes    |                        >=16                         |
|   Maven    |                        >=3.6                        |
| 阿里云OSS  |                   创建一个Bucket                    |
| springboot |             Spring Boot Version: 2.5.15             |


<h2>部分代码分析</h2>

```
/**
 * 通用上传请求（单个）
 */
@PostMapping("/upload")
public AjaxResult uploadFile(MultipartFile file) throws Exception
{
    try
    {
        // 记录上传文件的详细信息
        log.info("Uploading file: {}, size: {}", file.getOriginalFilename(), file.getSize());
        // 上传文件路径
        //String filePath = RuoYiConfig.getUploadPath();
        // 上传并返回新文件名称
        //String fileName = FileUploadUtils.upload(filePath, file);
        //String url = serverConfig.getUrl() + fileName;

        // 指定OSS 保存文件路径 2024/12/3
        String objectName = LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy/MM/dd")) + "/";
        // 上传并返回新文件名称
        FileInfo fileInfo = fileStorageService.of(file)
                .setPath(objectName) //保存到相对路径下，为了方便管理，不需要可以不写
                .upload();
        log.info("File uploaded successfully: {}, URL: {}", fileInfo.getFilename(), fileInfo.getUrl());
        AjaxResult ajax = AjaxResult.success();
        ajax.put("url", fileInfo.getUrl());
        ajax.put("fileName", fileInfo.getUrl());
        ajax.put("newFileName", fileInfo.getUrl());
        ajax.put("originalFilename", file.getOriginalFilename());
        return ajax;
    }
    catch (Exception e)
    {
        log.error("Failed to upload file: {}", file.getOriginalFilename(), e);
        return AjaxResult.error(e.getMessage());
    }
}



```

<h3>关于阿里云OSS 文件上传的增强问题</h3>
引入坐标问题
![阿里云OSS](./image/阿里云文件上传功能.png)
这里的坐标一个是阿里云OSS使用的坐标，另一个是关于不同存储桶方便引入的问题<a href="https://x-file-storage.xuyanwu.cn/#/" title="点击这里访问目标网站">访问目标网站</a>
dbk-admin的application.yml爆红问题
![阿里云OSS](./image/阿里云OSS.png)
这部分yml文件爆红没有问题，相反如果该了功能就不能使用了，亲测了好几个AI工具修复但是都不能用，还有上述关于文件上传的功能代码，可以根据最新的代码使用，只需要按照要求即可。

<h2>关于接口文档的说明</h2>

接口文档的两个坐标

```
<!-- knife4j -->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
<!--引入Knife4j的官方start包,该指南选择Spring Boot版本<3.0,开发者需要注意-->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    <version>4.4.0</version>
</dependency>
```

说明：

![swagger技术](./image/swagger.png)

  第一个技术介于 swagger  和 knife4j 但是现在 swagger 接口文档技术基本不用，不要问为什么，单纯的 swagger 接口文档页面巨丑，下面两个坐标都是脱胎于 swagger 技术，第二个技术需要在 application.yml 文件配置一下:

```
# springdoc-openapi项目配置
springdoc:
  swagger-ui:
    path: /swagger-ui.html
    tags-sorter: alpha
    operations-sorter: alpha
  api-docs:
    path: /v3/api-docs
  group-configs:
    - group: 'default'
      paths-to-match: '/**'
      packages-to-scan: com.atguigu.cloud.controller  //替换自己的，其他地方可以不动
# knife4j的增强配置，不需要增强可以不配
knife4j:
  enable: true
  setting:
    language: zh_cn
```

## 业务介绍

![流程](./image/流程.png)

  黑马课程讲解时用的springboot,可以更具上图流程分解功能板块为springcloud项目，其实板块大致就是这四个板块。

 随着售货机平台方运营售货机设备数量及点位数量越来越多，为了降低成本加快运营和运维效率将有限的资源迅速变现，平台方将系统做了切分，其中客户端包含三个：面向维修人员的运维客户端，主要用来跟踪和解决维修人员的日常工作和及时处理设备故障；运营客户端，主要用来提高日常运营效率，减少缺货设备，提高设备的盈利能力；C端用户客户端，面向C端用户的客户端，提高用户体验，缩短用户购物流程。

为了更好的针对不同B端用户的使用习惯，平台端切分为两个端：平台管理端，主要为了管理设备、货道、商品、工单及运营、运维数据。合作商后台：针对有大量点位但是没有运营能力的用户提供数据接入和销售分成的管理。
