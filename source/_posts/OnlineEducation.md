---
title: 在线教育平台
date: 2024-05-27 19:04:00
tags: [Java, Vue]
categories:
  - [项目]
thumbnail: "https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/Home.jpg"
excerpt: "本篇文章用于总结在线教育平台整个项目的开发过程，以及项目的功能和技术栈"
---

# 序言

由于在开发该系统时，并未有书写开发文档的经验，而仅仅是记录了每一天的开发日志，但开发日志多达 21 篇，因此把所有开发日志放在博客当中是影响读者观感的，因此本篇文章仅仅是作为总结，如果有具体需要，请联系我，或者访问我的 GitHub 仓库。

# 项目简介

在线教育平台采用 B2C 模式，Spring Cloud 搭建整个微服务架构，后台采用 Spring Boot+MySQL+MyBatis-Plus+Redis，并且结合 Vue 前端框架，采用 Nuxt 服务端渲染技术来优化前端页面，运用阿里云视频点播技术。在管理系统的后台中，运用 Spring Security 进行用户认证和授权，以确保对不同用户权限的细致划分。在用户的登录系统方面，则采纳了手机验证码注册和登录方式，并运用 JWT 生成 Token 以实现便捷的单点登录。此外，用户通过微信支付来进行课程购买。

# 技术栈

## 后端

- Spring Boot
- Spring Cloud
- MySQL
- MyBatis-Plus
- Redis
- Spring Security
- EasyExcel

## 前端

- Vue
- Nuxt
- ElementUI
- Axios
- ECharts

# 后台管理系统

在线教育平台后台管理系统的前端使用的是 vue-admin-template 模板
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/BackendLogin.jpg)

## 讲师管理

对讲师进行增删改查操作，后端集成了阿里云 OSS，用于讲师头像的上传。
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/AdminTeachers.png)
**开发中值得一提的：**  
vue-router 导航切换 时，如果两个路由都渲染同个组件，组件会重（chong）用,  
组件的生命周期钩子（created）不会再被调用, 使得组件的一些数据无法根据 path 的改变得到更新  
因此：

1. 我们可以在 watch 中监听路由的变化，当路由变化时，重新调用 created 中的内容；
2. 在 init 方法中我们判断路由的变化，如果是修改路由，则从 api 获取表单数据。  
   如果是新增路由，则重新初始化表单数据

```JavaScript
watch: { // 监听
    $route(to, from) { // 路由变化方式，路由发生变化，方法就会执行
      this.init()
    }
  },
  created() { // 页面渲染之前执行
    this.init()
  },
  methods: {
    init() {
      // 判断路径是否有id值
      if (this.$route.params && this.$route.params.id) {
        // 从路径获取id值
        const id = this.$route.params.id
        // 调用根据id查询的方法
        this.getInfo(id)
      } else { // 路径没有id值，做添加
        // 清空表单
        this.teacher = {}
      }
    },
```

## 课程分类管理

前端上传课程 Excel 表格，后端通过 EasyExcel 来处理表格并将其持久化存储于数据库中。
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/CourseExcel.png)
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/CourseCategory.png)

## 课程管理

可以查看课程详细信息并管理课程，如果是发布课程需要进行三个步骤，分别是“填写课程基本信息”、“创建课程大纲”、“最终发布”，需要按照该执行顺序去操作才能完整发布课程。
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/CourseAdmin.png)
**值得一提的是课程视频上传的实现**

1. 引入依赖  
   引入依赖存在问题
   ![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/DependencyProblem.png)
   mvn 需要配置环境变量，这样才能在命令行中使用 mvn 命令
   ![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/MavenPath.png)
   上传视频  
   参考官网压缩包里面的 sample 示例代码改造
   ![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/ExampleCode.png)

```java
public static void main(String[] args) {
        String accessKeyId = "";
        String accessKeySecret = "";

        String title = "6 - How Does Project Submission Work - upload by sdk";  // 上传之后文件名称
        String fileName = "E:\\6 - What If I Want to Move Faster.mp4";   // 本地文件路径和名称

        // 上传视频的方法
        UploadVideoRequest request = new UploadVideoRequest(accessKeyId, accessKeySecret, title, fileName);
        /* 可指定分片上传时每个分片的大小，默认为2M字节 */
        request.setPartSize(2 * 1024 * 1024L);
        /* 可指定分片上传时的并发线程数，默认为1，(注：该配置会占用服务器CPU资源，需根据服务器情况指定）*/
        request.setTaskNum(1);
        UploadVideoImpl uploader = new UploadVideoImpl();
        UploadVideoResponse response = uploader.uploadVideo(request);

        if (response.isSuccess()) {
            System.out.print("VideoId=" + response.getVideoId() + "\n");
        } else {
            /* 如果设置回调URL无效，不影响视频上传，可以返回VideoId同时会返回错误码。其他情况上传失败时，VideoId为空，此时需要根据返回错误码分析具体错误原因 */
            System.out.print("VideoId=" + response.getVideoId() + "\n");
            System.out.print("ErrorCode=" + response.getCode() + "\n");
            System.out.print("ErrorMessage=" + response.getMessage() + "\n");
        }
    }
```

2. 配置文件

```yml
# 服务端口
server:
  port: 8082

spring:
  application:
    # 服务名
    name: service-vod
  profiles:
    # 环境设置：dev、test、prod
    active: dev
  servlet:
    multipart:
      # 最大上传单个文件大小：默认1M
      max-file-size: 1024MB
      # 最大总上传的数据大小：默认10MB
      max-request-size: 1024MB

# 阿里云 vod
# 不同的服务器，地址不同
aliyun:
  vod:
    file:
      keyid:
      keysecret:
```

3. VodApplication

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@ComponentScan(basePackages = {"com.invictusqiu"})
public class VodApplication {

    public static void main(String[] args) {
        SpringApplication.run(VodApplication.class, args);
    }
}
```

4. 工具类  
   常量读取工具类，读取配置文件的内容

```java
@Component
public class ConstantVodUtils implements InitializingBean {

    @Value("${aliyun.vod.file.keyid}")
    private String keyid;

    @Value("${aliyun.vod.file.keysecret}")
    private String keysecret;

    // 定义公开常量
    public static String ACCESS_KEY_ID;
    public static String ACCESS_KEY_SECRET;

    @Override
    public void afterPropertiesSet() throws Exception {
        ACCESS_KEY_ID = keyid;
        ACCESS_KEY_SECRET = keysecret;
    }
}
```

5. 控制器

```java
@RestController
@RequestMapping("/eduvod/video")
@CrossOrigin
public class VodController {

    @Autowired
    private VodService vodService;

    // 上传视频到阿里云
    @PostMapping("uploadAlyVideo")
    public Result uploadAlyVideo(MultipartFile file) {
        // 返回上传视频id
        String videoId = vodService.uploadVideoAly(file);
        return Result.ok().data("videoId",videoId);
    }
}
```

6. 服务实现类

```java
@Service
public class VodServiceImpl implements VodService {

    // 上传视频到阿里云（采用流式上传接口）
    @Override
    public String uploadVideoAly(MultipartFile file) {
        try {
            // accessKeyId, accessKeySecret
            // fileName: 上传文件原始名称
            // 01.03.09.mp4
            String fileName = file.getOriginalFilename();

            // title: 上传之后显示名称
            // 去除最后一个.
            String title = fileName.substring(0, fileName.lastIndexOf("."));

            // inputStream: 上传文件输入流
            InputStream inputStream = file.getInputStream();

            UploadStreamRequest request = new UploadStreamRequest(ConstantVodUtils.ACCESS_KEY_ID, ConstantVodUtils.ACCESS_KEY_SECRET, title, fileName, inputStream);

            UploadVideoImpl uploader = new UploadVideoImpl();
            UploadStreamResponse response = uploader.uploadStream(request);

            String videoId = null;
            if (response.isSuccess()) {
                videoId = response.getVideoId();
            } else { //如果设置回调URL无效，不影响视频上传，可以返回VideoId同时会返回错误码。其他情况上传失败时，VideoId为空，此时需要根据返回错误码分析具体错误原因
                videoId = response.getVideoId();
            }
            return videoId;
        } catch(Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

7. 前端
   chapter.vue

```html
<el-form-item label="上传视频">
  <el-upload
    :on-success="handleVodUploadSuccess"
    :on-remove="handleVodRemove"
    :before-remove="beforeVodRemove"
    :on-exceed="handleUploadExceed"
    :file-list="fileList"
    :action="BASE_API + '/eduvod/video/uploadAlyVideo'"
    :limit="1"
    class="upload-demo"
  >
    <el-button size="small" type="primary">上传视频</el-button>
    <el-tooltip placement="right-end">
      <div slot="content">
        最大支持1G，<br />
        支持3GP、ASF、AVI、DAT、DV、FLV、F4V、<br />
        GIF、M2T、M4V、MJ2、MJPEG、MKV、MOV、MP4、<br />
        MPE、MPG、MPEG、MTS、OGG、QT、RM、RMVB、<br />
        SWF、TS、VOB、WMV、WEBM 等视频格式上传
      </div>
      <i class="el-icon-question" />
    </el-tooltip>
  </el-upload>
</el-form-item>
```

```JavaScript
fileList: [], // 上传视频的列表
BASE_API: process.env.BASE_API // 接口API地址

// 成功回调
handleVodUploadSuccess(response, file, fileList) {
  this.video.videoSourceId = response.data.videoId
},
// 视图上传多于一个视频
handleUploadExceed() {
  this.$message.warning('想要重新上传视频，请先删除已上传的视频')
},
```

8. nginx 配置

```conf
location ~ /eduvod/ {
    proxy_pass http://localhost:8082;
}
```

配置 nginx 上传文件大小，否则上传时会有 413 (Request Entity Too Large) 异常  
打开 nginx 主配置文件 nginx.conf，找到 http{}，添加

```conf
client_max_body_size 1024m;
```

9. 如果数据库没有视频名称  
   修改前端

```JavaScript
// 上传视频成功调用的方法
handleVodUploadSuccess(response, file, fileList) {
  // 上传视频id赋值
  this.video.videoSourceId = response.data.videoId
  // 上传视频名称赋值
  this.video.videoOriginalName = file.name
},
```

## 统计分析

统计分析页面，前端页面使用 Echarts 组件库实现图表展示，用户可以选择指定日期范围生成统计数据，包括范围内的用户登录数和注册数，以及课程播放数等数据。
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/StatisticalAnalysis.png)
该模块使用了 Feign 远程调用  
比如调用接口 UcenterClient

```java
@Component
@FeignClient("service-ucenter")
public interface UcenterClient {

    // 查询某一天注册人数
    @GetMapping("/educenter/member/countRegister/{day}")
    public Result countRegister(@PathVariable("day") String day);
}
```

StatisticsDailyServiceImpl 服务实现类

```java
@Autowired
private UcenterClient ucenterClient;

// 统计某一天注册人数，生成统计数据
@Override
public void registerCount(String day) {

    // 添加记录之前删除表相同日期的数据
    QueryWrapper<StatisticsDaily> wrapper = new QueryWrapper<>();
    wrapper.eq("date_calculated", day);
    baseMapper.delete(wrapper);

    // 远程调用得到某一天注册人数
    Result registerResult = ucenterClient.countRegister(day);
    Integer countRegister = (Integer)registerResult.getData().get("countRegister");

    // 把获取数据添加数据库，统计分析表里面
    StatisticsDaily sta = new StatisticsDaily();
    sta.setRegisterNum(countRegister); //注册人数
    sta.setDateCalculated(day); //统计日期

    sta.setVideoViewNum(RandomUtils.nextInt(100,200));
    sta.setLoginNum(RandomUtils.nextInt(100,200));
    sta.setCourseNum(RandomUtils.nextInt(100,200));
    baseMapper.insert(sta);
}
```

除此之外，启用定时任务实现每天统计  
启动类添加注释

```java
@EnableScheduling //定时任务注解
```

创建 ScheduleTask 类

```java
@Component
public class ScheduleTask {

    @Autowired
    private StatisticsDailyService staService;

    /* 定时器测试
 0/5 * * * * ?表示每隔5秒执行一次这个方法
    @Scheduled(cron = "0/5 * * * * ?")
    public void task1() {
        System.out.println("********************task1执行了...");
    }
*/

    // 在每天凌晨1点，把前一天的数据进行数据查询添加
    @Scheduled(cron = "0 0 1 * * ?")
    public void task2() {
        staService.registerCount(DateUtil.formatDate(DateUtil.addDays(new Date(),-1)));
    }
}
```

# 前台用户系统

## 前端框架

Nuxt.js 是一个基于 Vue.js 的轻量级应用框架,可用来创建服务端渲染 (SSR) 应用,也可充当静态站点引擎生成静态站点应用,具有优雅的代码结构分层和热加载等特性。  
[官方网站](https://zh.nuxtjs.org/)  
幻灯片插件：vue-awesome-swiper

## 首页

展示轮播图、热门课程等信息，然后对用户展示网站幻灯片、热门课程、名师等内容，为了提高访问速度使用了 Redis 缓存首页数据。
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/HomePage.png)

## 注册和登录

注册功能需要用户通过填写昵称、手机号，然后接收验证码的方式进行注册。如果使用手机号码注册，系统会通过阿里云短信服务向该用户发送短信验证码，后端保存该验证码来和用户输入的验证码进行比对。如果用户是以扫描微信二维码的方式进行注册，后端接收到该请求后会将页面重定向至二维码页面，扫码之后获得微信官方返回的临时票据，使用票据可以获得该用户微信账号的访问凭证和唯一标识，然后请求微信官方的接口地址得到该用户的账号信息，并将其持久化存储于数据库中，实现微信扫码注册功能。
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/RegisterPage.png)
值得一提的是使用 Redis 解决验证码有效时间问题

```java
// springboot整合的Redis模板对象
@Autowired
private RedisTemplate<String,String> redisTemplate;

// 发送短信的方法
@GetMapping("send/{phone}")
public Result sendMsm(@PathVariable String phone) {
    // 1.从redis获取验证码，如果获取到直接返回
    String code = redisTemplate.opsForValue().get(phone);
    if (!StringUtils.isEmpty(code)) {
        return Result.ok();
    }

    // 2.如果redis获取不到，进行阿里云发送
    // 生成随机值，传递阿里云进行发送
    code = RandomUtil.getFourBitRandom();
    Map<String,Object> param = new HashMap<>();
    param.put("code",code);
    // 调用service发送短信的方法
    boolean isSend = msmService.send(param,phone);
    if (isSend) {
        // 发送成功，把发送成功验证码放到redis里面
        // 设置有效时间
        redisTemplate.opsForValue().set(phone,code,5, TimeUnit.MINUTES);
        return Result.ok();
    } else {
        return Result.error().message("短信发送失败");
    }
}
```

## 课程列表

课程列表，展示上架课程，对不同种类的课程进行了分类，可以按照销量、发布时间、售价来对课程列表进行排序。
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/CourseSchedule.png)
后端处理条件分页

```java
// 1.条件查询带分页查询课程
@Override
public Map<String, Object> getCourseFrontList(Page<EduCourse> pageCourse, CourseFrontVo courseFrontVo) {

    QueryWrapper<EduCourse> wrapper = new QueryWrapper<>();
    // 判断条件值是否为空，不为空拼接
    if (!StringUtils.isEmpty(courseFrontVo.getSubjectParentId())) { //一级分类
        wrapper.eq("subject_parent_id", courseFrontVo.getSubjectParentId());
    }
    if (!StringUtils.isEmpty(courseFrontVo.getSubjectId())) {   //二级分类
        wrapper.eq("subject_id",courseFrontVo.getSubjectId());
    }
    if (!StringUtils.isEmpty(courseFrontVo.getBuyCountSort())) {   //关注度
        wrapper.orderByDesc("buy_count");
    }
    if (!StringUtils.isEmpty(courseFrontVo.getGmtCreateSort())) {   //最新
        wrapper.orderByDesc("gmt_create");
    }
    if (!StringUtils.isEmpty(courseFrontVo.getPriceSort())) {   //价格
        wrapper.orderByDesc("price");
    }
    // 只获取发布状态的课程
    wrapper.eq("status","Normal");
    baseMapper.selectPage(pageCourse,wrapper);

    List<EduCourse> records = pageCourse.getRecords();
    long current = pageCourse.getCurrent();
    long pages = pageCourse.getPages();
    long size = pageCourse.getSize();
    long total = pageCourse.getTotal();
    boolean hasNext = pageCourse.hasNext();
    boolean hasPrevious = pageCourse.hasPrevious();

    // 把分页数据获取出来，放到map集合
    Map<String, Object> map = new HashMap<>();
    map.put("items", records);
    map.put("current", current);
    map.put("pages", pages);
    map.put("size", size);
    map.put("total", total);
    map.put("hasNext", hasNext);
    map.put("hasPrevious", hasPrevious);

    // map返回
    return map;
}
```

## 课程详情

课程详情页，包含课程基本信息、分类、讲师等内容，课程分为免费和付费，如果是付费课程，那么前端的“立即观看”按钮会变为“立即购买”按钮，并且在该页面用户可以发表对该课程的评论。
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/CourseDetail.png)

## 视频播放

![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/VodPlayer.png)

### 获取播放地址

[参考文档](https://help.aliyun.com/document_detail/61064.html)  
前面的 03-使用服务端 SDK 介绍了如何获取非加密视频的播放地址。直接使用 03 节的例子获取加密视频播放地址会返回如下错误信息  
Currently only the AliyunVoDEncryption stream exists, you must use the Aliyun player to play or set the value of ResultType to Multiple.  
目前只有 AliyunVoDEncryption 流存在，您必须使用 Aliyun player 来播放或将 ResultType 的值设置为 Multiple。  
因此在 testGetPlayInfo 测试方法中添加 ResultType 参数，并设置为 true

```java
privateParams.put("ResultType", "Multiple");
```

此种方式获取的视频文件不能直接播放，必须使用阿里云播放器播放

### 视频播放器

[参考文档](https://help.aliyun.com/document_detail/61109.html)  
**视频播放器介绍**  
阿里云播放器 SDK（ApsaraVideo Player SDK）是阿里视频服务的重要一环，除了支持点播和直播的基础播放功能外，深度融合视频云业务，如支持视频的加密播放、安全下载、清晰度切换、直播答题等业务场景，为用户提供简单、快速、安全、稳定的视频播放服务。

**集成视频播放器**  
[参考文档](https://help.aliyun.com/document_detail/51991.html)  
参考 【播放器简单使用说明】一节  
引入脚本文件和 css 文件

```html
<link
  rel="stylesheet"
  href="https://g.alicdn.com/de/prismplayer/2.8.1/skins/default/aliplayer-min.css"
/>
<script
  charset="utf-8"
  type="text/javascript"
  src="https://g.alicdn.com/de/prismplayer/2.8.1/aliplayer-min.js"
></script>
```

初始化视频播放器

```html
<body>
  <div class="prism-player" id="J_prismPlayer"></div>
  <script>
    var player = new Aliplayer(
      {
        id: "J_prismPlayer",
        width: "100%",
        autoplay: false,
        cover: "http://liveroom-img.oss-cn-qingdao.aliyuncs.com/logo.png",
        //播放配置
      },
      function (player) {
        console.log("播放器创建好了。");
      }
    );
  </script>
</body>
```

**1. 播放地址播放**  
在 Aliplayer 的配置参数中添加如下属性

```JavaScript
//播放方式一：支持播放地址播放,此播放优先级最高，此种方式不能播放加密视频
source: '你的视频播放地址',
```

启动浏览器运行，测试视频的播放

**2. 播放凭证播放（推荐）**  
阿里云播放器支持通过播放凭证自动换取播放地址进行播放，接入方式更为简单，且安全性更高。播放凭证默认时效为 100 秒（最大为 3000 秒），只能用于获取指定视频的播放地址，不能混用或重复使用。如果凭证过期则无法获取播放地址，需要重新获取凭证。

```JavaScript
encryptType:'1',//如果播放加密视频，则需设置encryptType=1，非加密视频无需设置此项
vid : '视频id',
playauth : '视频授权码',
```

注意：播放凭证有过期时间，默认值：100 秒 。取值范围：100~3000。  
设置播放凭证的有效期  
在获取播放凭证的测试用例中添加如下代码

```JavaScript
request.setAuthInfoTimeout(200L);
```

[在线配置参考](https://player.alicdn.com/aliplayer/setting/setting.html)

### 后端获取播放凭证

**播放组件相关文档：**  
[集成文档](https://help.aliyun.com/document_detail/51991.html?spm=a2c4g.11186623.2.39.478e192b8VSdEn)  
[在线配置](https://player.alicdn.com/aliplayer/setting/setting.html)  
[功能展示](https://player.alicdn.com/aliplayer/presentation/index.html)

## 整合阿里云视频播放器

### 后端

修改 VideoVo

```java
public class VideoVo {

    private String id;

    private String title;

    private String videoSourceId;   //视频id
}
```

VodController

```java
// 根据视频id获取视频凭证
@GetMapping("getPlayAuth/{id}")
public Result getPlayAuth(@PathVariable String id) {
    try {
        // 创建初始化对象
        DefaultAcsClient client =
                InitVodClient.initVodClient(ConstantVodUtils.ACCESS_KEY_ID,ConstantVodUtils.ACCESS_KEY_SECRET);
        // 创建获取凭证request和response对象
        GetVideoPlayAuthRequest request = new GetVideoPlayAuthRequest();
        // 向request设置视频id
        request.setVideoId(id);
        // 调用方法得到凭证
        GetVideoPlayAuthResponse response = client.getAcsResponse(request);
        String playAuth = response.getPlayAuth();
        return Result.ok().data("playAuth",playAuth);
    } catch (Exception e) {
        throw new EduException(20001,"获取凭证失败");
    }
}
```

### 前端

api  
vod.js

```JavaScript
import request from '@/utils/request'

export default {
  getPlayAuth(vid) {
    return request({
      url: `/eduvod/video/getPlayAuth/${vid}`,
      method: 'get'
    })
  }
}
```

创建新的 layouts  
video.vue

```html
<template>
  <div class="guli-player">
    <div class="head">
      <a href="#" title="在线教育">
        <img class="logo" src="~/assets/img/logo.png" lt="在线教育" />
      </a>
    </div>
    <div class="body">
      <div class="content">
        <nuxt />
      </div>
    </div>
  </div>
</template>
<script>
  export default {};
</script>
<style>
  html,
  body {
    height: 100%;
  }
</style>

<style scoped>
  .head {
    height: 50px;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
  }

  .head .logo {
    height: 50px;
    margin-left: 10px;
  }

  .body {
    position: absolute;
    top: 50px;
    left: 0;
    right: 0;
    bottom: 0;
    overflow: hidden;
  }
</style>
```

\_id.vue  
点击小节携带视频 id 跳转

```html
<a :href="'/player/'+video.videoSourceId" title target="_blank"></a>
```

新建 Page/player/\_vid.vue

```html
<template>
  <div>
    <!-- 阿里云视频播放器样式 -->
    <link
      rel="stylesheet"
      href="https://g.alicdn.com/de/prismplayer/2.8.1/skins/default/aliplayer-min.css"
    />

    <!-- 定义播放器dom -->
    <div id="J_prismPlayer" class="prism-player" />
  </div>
</template>
<script
  charset="utf-8"
  type="text/javascript"
  src="https://g.alicdn.com/de/prismplayer/2.8.1/aliplayer-min.js"
/>
<script>
  import vod from "@/api/vod";
  export default {
    layout: "video", // 使用video布局
    asyncData({ params, error }) {
      return vod.getPlayAuth(params.vid).then((response) => {
        return {
          playAuth: response.data.playAuth,
          vid: params.vid,
        };
      });
    },
    mounted() {
      new Aliplayer(
        {
          id: "J_prismPlayer",
          vid: this.vid, // 视频id
          playauth: this.playAuth, // 播放凭证
          // encryptType: '1', // 如果播放加密视频，则需设置encryptType=1，非加密视频无需设置此项
          width: "100%",
          height: "500px",
        },
        function (player) {
          console.log("播放器创建成功");
        }
      );
    },
  };
</script>
```

排错

> 先看看播放器的 js 有没有引入
> 摁下 F12，在网络中（network）查看，如果没有可以尝试在 nuxt.config.js 文件中的 head 中添加。  
> 不要删除原\_vid.vue 中的

````JavaScript
<script charset="utf-8" type="text/javascript" src="https://g.alicdn.com/de/prismplayer/2.8.1/aliplayer-min.js"/>
```html
把它放到`<template></template>`标签外
```JavaScript
head: {
  script: [{ src: 'https://g.alicdn.com/de/prismplayer/2.8.1/aliplayer-min.js' }],
}
````

## 名师列表

得到所有讲师信息，显示所有名师的头像、名称、简介内容。
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/TeacherList.png)

## 讲师详情

在名师列表页可以选择不同讲师的卡片，通过携带讲师 id 请求后端接口来查询该讲师的信息和所授课程，页面中展示了名师的详细信息和所授课程。
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/TeacherDetail.png)

## 订单模块

课程支付，用户只有登录后才能购买对应课程。购买会生成课程订单和微信支付的二维码，在此支付期间每隔 3 秒会查询支付状态，只有扫码成功后才更新数据库中该订单的支付状态，一旦查询支付状态为“已支付”才能为用户开通课程观看权限。
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/onlineEducation/OrderPay.png)
服务实现代码

```java
@Autowired
private OrderService orderService;

// 生成微信支付二维码接口
@Override
public Map createNative(String orderNo) {
    try {
        // 1.根据订单号查询订单信息
        QueryWrapper<Order> wrapper = new QueryWrapper<>();
        wrapper.eq("order_no",orderNo);
        Order order = orderService.getOne(wrapper);

        // 2.使用map设置生成二维码需要的参数
        Map m = new HashMap();
        m.put("appid", "wx74862e0dfcf69954");
        m.put("mch_id", "1558950191");
        m.put("nonce_str", WXPayUtil.generateNonceStr());
        m.put("body", order.getCourseTitle());
        m.put("out_trade_no", orderNo);
        m.put("total_fee", order.getTotalFee().multiply(new BigDecimal("100")).longValue()+"");
        m.put("spbill_create_ip", "127.0.0.1");
        m.put("notify_url", "http://guli.shop/api/order/weixinPay/weixinNotify\n");
        m.put("trade_type", "NATIVE");

        // 3.发送httpClient请求，传递参数xml格式，微信支付提供的固定地址
        HttpClient client = new HttpClient("https://api.mch.weixin.qq.com/pay/unifiedorder");
        // 设置xml格式的参数
        client.setXmlParam(WXPayUtil.generateSignedXml(m,"T6m9iK73b0kn9g5v426MKfHQH7X8rKwb"));
        client.setHttps(true);
        // 执行请求发送
        client.post();

        // 4.得到发送请求返回的结果
        // 返回内容，是使用xml格式返回
        String xml = client.getContent();

        // 把xml格式转换map集合，把map集合返回
        Map<String,String> resultMap = WXPayUtil.xmlToMap(xml);

        //最终返回数据的封装
        Map map = new HashMap();
        map.put("out_trade_no", orderNo);
        map.put("course_id", order.getCourseId());
        map.put("total_fee", order.getTotalFee());
        map.put("result_code", resultMap.get("result_code"));   // 返回二维码操作状态码
        map.put("code_url", resultMap.get("code_url")); //二维码地址

        return map;

    } catch (Exception e) {
        throw new EduException(20001,"生成二维码失败");
    }
}
```

# 项目仓库

[在线教育平台](https://github.com/QiuHaimengjing/online-education-platform)
