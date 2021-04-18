# 前言
> 做项目不是为了敲代码，而是为了学知识，学原理，不深入去理解底层原理的话就是普通的 CRUD 工程师。
> 项目中涉及的比较重要的内容可以查看 Wiki 页面，或者 [awesome-architect](https://raymond-zhao.top/campus-interview/#/)。
>
> 现在文档中的知识点还比较有限，内容还在持续完善中。以后将会逐渐深入源码，分析学习运行原理与机制。

微服务架构电商系统，主要分为三个阶段。

- 第一阶段：[分布式基础篇-全栈开发](#分布式基础篇-全栈开发)
  - 快速地开发一个前后端分离的电商系统
  - Spring Boot + Spring Cloud + Vue + Docker + MyBatis Plus
- 第二阶段：[分布式高级-微服务架构](#分布式高级篇-微服务架构)，打通分布式开发中的所有技术栈。
  - ElasticSearch
  - Redis 基本使用与 Lua 脚本
  - Redisson 分布式锁
  - 性能压测模拟
  - Nginx 反向代理、动静分离、负载均衡
  - 多线程与异步
  - 单点登录与社交登录
  - RabbitMQ 消息队列
  - Nacos 服务注册、发现、配置中心
  - 分布式事务与 Seata
  - 秒杀系统设计
  - 定时任务与分布式调度
  - Sentinel 服务容错
  - Sleuth & Zipkin 链路追踪
  - 实现一整套的微服务整合，包括秒杀，结算，库存...
- 第三阶段：[高可用集群-架构师提升](#高可用集群篇-架构师提升)
  - 搭建 Kubernetes 集群，实现全流程 DevOps。
  - 搭建 MySQL 集群，Redis 集群，RabbitMQ 集群，ElasticSearch 集群。

![谷粒商城-微服务架构图](https://tva1.sinaimg.cn/large/007S8ZIlly1geblwvpadsj31f10u07dn.jpg)

- [x] 《分布式基础篇-全栈开发》
- [x] 《分布式高级篇-微服务架构》
- [ ] 《高可用集群篇-架构师提升》
- [ ] 完善系统功能
  - [ ] 完善用户评论、收藏、物流
  - [x] 系统自动生成了 `Apache Shiro` 权限控制
  - [ ] 增加卖家角色及相关功能
  - [ ] 增加推荐子系统
  - [ ] 增加数据仓库与数据挖掘

## 项目重启日志

因为准备秋招耽误了许久，现在重新开始，重新启动项目时遇到许多麻烦，下面记录一下大致步骤，避免以后忘记。

系统为 `macOS BigSur`，包管理工具为 `Homebrew`。

- 使用 `MySQL` 建立相应数据库，并运行相应 `sql` 文件，数据库账号密码为 `root/root`。
- 安装  `mall-commons` 到本地 `Maven` 仓库。

```shell
# 进入到 mall-commons 所在目录
$ mvn clean
$ mvn install
```

- 添加静态文件到 `nginx` 服务器中，并且启动 `nginx` 服务器。在渲染商城首页三级菜单时，如果没有正确显示，需要注意 `catalogLoader.js` 这个文件中的请求地址。
  - 现在包括 `cart、index、item、login、member、order、register、search` 几个静态资源目录。

```shell
$ brew install nginx
$ brew info nginx
# 静态文件目录 /usr/local/var/www/static
```

- 下载并启动 `Redis` 服务器

```shell
$ brew install redis
$ redis-server /usr/local/etc/redis.conf
```

- [nacos server](https://nacos.io/zh-cn/docs/quick-start.html) 用于服务注册与发现，或者下载 [Release 版本](https://github.com/alibaba/nacos/releases)。

```shell
# 启动命令 ( standalone 代表着单机模式运行，非集群模式)
$ sh startup.sh -m standalone
```

- 下载并启动 [Zipkin](https://github.com/openzipkin/zipkin)。

```shell
$ curl -sSL https://zipkin.io/quickstart.sh | bash -s
$ java -jar zipkin.jar
```

- 下载并启动 `Elasticsearch`，建立相应的索引 `product`。

```shell
# 如果内存不是特别大的话，最好设置一下 ES 启动时的虚拟机参数，在 mac 中 Brew 安装目录是
# /usr/local/Cellar/elasticsearch/7.10.0/libexec/config/jvm.options
```

- 添加与 `ElasticSearch` 版本对应的 `ik` 分词器，并将其放入 `ES ` 目录下的 `plugins` 目录下。下载链接 [elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik/releases)。

```shell
# mac 终端打开
$ open /usr/local/Cellar/elasticsearch/7.10.0/libexec/plugins
```

- 下载并启动 `RabbitMQ`，在此选择客户端。


```shell
$ brew cask install rabbitmq
```

- 阿里云 `OSS` 对象存储配置信息（用于图片存储）、短信配置，在启动 `mall-thirdparty` 模块时需要配置 `endpoint` 。

> 完成上面几个步骤之后，项目启动正常。

# 分布式基础篇-全栈开发

## 基础环境

### 不少于一台 PC

- 建议使用类 Linux 系统

### CentOS 虚拟机

- 购买阿里云服务器
- 本地虚拟机

### Docker 环境

- [安装 Docker](https://docs.docker.com/engine/install/)

- 安装 MySQL

```shell
$ docker pull mysql:5.7
$ docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7

$ docker ps
```

- 配置MySQL编码


```mysql
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
```

- 重启 MySQL

```shell
$ docker restart mysql
$ docker exec -it mysql /bin/bash
```

- 安装 Redis

```shell
$ docker pull redis
$ mkdir -p /mydata/redis/conf
$ touch /mydata/redis/conf/redis.conf
$ docker run -p 6379:6379 --name redis -v /mydata/redis/data:/data \
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis redis-server /etc/redis/redis.conf
$ docker ps
$ docker run --restart=always # 随机自启
$ docker update --restart=always <CONTAINER ID> # 随机自启
$ docker exec -it redis redis-cli
```

- 增加 Redis 持久化

```shell
$ vi /mydata/redis/conf/redis.conf
# appendonly yes
$ docker restart redis
```

- 免费的 mac/windows redis 客户端：[AnotherRedisDesktopManager - GitHub](https://github.com/qishibo/AnotherRedisDesktopManager/releases)

## 开发环境

### 微服务模块

- 项目基础功能模块: 
  - 商品模块：`mall-product`
  - 库存模块：`mall-ware`
  - 会员模块：`mall-member`
  - 优惠模块：`mall-coupon`
  - 检索模块：`mall-order`
  - 秒杀模块：`mall-seckill`
  - 单点登录：`xxl-sso`
- 公共依赖: `mall-commons`
- 后台管理: `renren-fast`
- 网关模块：`mall-gateway`
- 授权服务：`mall-auth-server`
- 第三方应用：`mall-third-party`
  - 短信服务
  - OSS 对象存储
- 配置文件：`config-file`
- SQL 文件：`sql`

### 初始化数据库

- 利用 SQL 文件建库建表

### 逆向工程

- [人人开源后台管理 - 码云](https://gitee.com/renrenio/renren-fast)

- [人人开源前台 Vue - 码云](https://gitee.com/renrenio/renren-fast-vue)

- [人人开源代码生成器 - 码云](https://gitee.com/renrenio/renren-generator)

### Maven

- `settings.xml`

```xml
<!-- 阿里云镜像 -->
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus Aliyun</name>
    <url>https://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云Google仓库</name>
    <url>https://maven.aliyun.com/repository/google</url>
</mirror>
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云Apache仓库</name>
    <url>https://maven.aliyun.com/repository/apache-snapshots</url>
</mirror>
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云Spring仓库</name>
    <url>https://maven.aliyun.com/repository/spring</url>
</mirror>
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云Spring插件仓库</name>
    <url>https://maven.aliyun.com/repository/spring-plugin</url>
</mirror>
```

```xml
<!-- 编译环境 -->
<profile>
    <id>jdk-1.8</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```

### Node.js

```shell
$ npm config set registry http://registry.npm.taobao.org/
$ npm config get registry
$ npm config set registry https://registry.npmjs.org/
```

- 大坑 node-sass

```
Module build failed: Error: Node Sass does not yet support your current environment: OS X 64-bit with Unsupported runtime (72)
```

解决办法：

```shell
$ npm uninstall node-sass
$ npm i node-sass --sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
$ npm run dev # 此时可成功
```

## 生成基本CRUD代码

- 使用 `renren-generator` 模块
- 修改配置文件中数据库连接信息
- 运行项目，进入 Web 界面，生成基础增删改查及 `Vue` 模板文件

## Spring Cloud Alibaba

- [Spring Cloud Alibaba - GitHub](https://github.com/alibaba/spring-cloud-alibaba)
- `Spring Cloud Alibaba Nacos`: 注册中心(服务发现/注册)，配置中心(动态配置管理)
- `Spring Cloud Alibaba Sentinel`: 服务容错(限流、降级、熔断)
- `Spring Cloud Alibaba Seata`: 分布式事务解决方案
- `Spring Cloud OpenFeign`: 声明式 `HTTP` 客户端，远程服务调用。
- `Spring Cloud Ribbon`: 负载均衡
- `Spring Cloud Sleuth`: 调用链路监控追踪

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### Nacos 作为服务注册与发现中心

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

```properties
spring.application.name: mall-coupon # 微服务名
spring.cloud.nacos.discovery.server-addr: localhost:8848 # 注册地址
```

```java
// 主启动类nacos注解
@EnableDiscoveryClient
```
nacos server的url:127.0.0.1:8848
nacos/nacos 登录

使用版本：
springboot：2.4.4
springcloud：2020.0.2
springcloudalibaba：2.2.1.RELEASE

启动错误：
No Feign Client for loadBalancing defined. Did you forget to include spring-cloud-starter-loadbalanc

解决办法：common里加入下面依赖，成功启动
	<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.cloud</groupId>
                    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
            <version>2.2.1.RELEASE</version>
        </dependency>


### OpenFeign 作为服务通信组件 声明式远程调用，http请求

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
// 编写接口
@FeignClient("mall-coupon") // 微服务名
public interface CouponFeign {
    @GetMapping("/coupon/coupon/member/list")  // 全限定路径
    R memberList();
}

// 主启动类 basePackages 可加可不加
@EnableFeignClients(basePackages = "edu.dlut.catmall.member.feign")
```

### Nacos 作为配置中心

```xml
1，引入依赖
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

2，配置中心设置
```properties
# bootstrap.properties 启动优先级高于
spring.application.name=mall-coupon
spring.cloud.nacos.config.server-addr=localhost:8848
```

```java
// Controller 动态刷新
@RefreshScope
```

application.properties的 内容
coupon.user.name=wangchunyu
coupon.user.age=36

动态获取配置
public class CouponController {
    @Value("${coupon.user.name}")
    private String name;
    @Value("${coupon.user.age}")
    private Integer age;
    @RequestMapping("/test")
    public R test(){
        return R.ok().put("name",name).put("age",age);
    }
3，在配置中心中添加数据集，应用名.properties
在`nacos`配置中心添加配置文件 `gulimall-coupon.properties`

在Controller里追加注解，动态发布
@RefreshScope
public class CouponController

如果读取配置失败，pom添加
     <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
     </dependency>

4，优先使用配置中心的配置。


- 命名空间：配置隔离，默认新增的所有都在public空间
  1，开发，测试，生产环境，利用命名空间来做环境隔离
  	spring.cloud.nacos.config.namespace=a2b3c641-562a-44db-856a-0268365352c0
  2，每一个微服务来建立命名空间	
- 配置集；所有配置的集合
- 配置集ID；类似文件名，应用名.properties
- 配置分组:默认所有分组为DEFAULT_GROUP
  1111,618,1212,dev,test,prop
  	spring.cloud.nacos.config.group=1111

3,同时加载多个配置集
 任何配置文件都可以放在配置中心，并且优先使用配置中心的配置
 

bootstrap.properties里下面的配置

spring.application.name=gulimall-coupon
spring.cloud.nacos.config.server-addr=127.0.0.1:8848
spring.cloud.nacos.config.namespace=e2579108-10e5-44a2-9776-46f2b6429771
spring.cloud.nacos.config.group=dev

spring.cloud.nacos.config.extension-configs[0].data-id=datasource.yml
spring.cloud.nacos.config.extension-configs[0].group=dev
spring.cloud.nacos.config.extension-configs[0].refresh=true

spring.cloud.nacos.config.extension-configs[1].data-id=mybatis.yml
spring.cloud.nacos.config.extension-configs[1].group=dev
spring.cloud.nacos.config.extension-configs[1].refresh=true

spring.cloud.nacos.config.extension-configs[2].data-id=other.yml
spring.cloud.nacos.config.extension-configs[2].group=dev
spring.cloud.nacos.config.extension-configs[2].refresh=true


### Spring Cloud Gateway 作为网关

各种网关的性能比较
Spring Cloud Gateway  32213.38
zuul	20800.13
linked  28050.76
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```
启动时database的错误，声明中exclude = {DataSourceAutoConfiguration.class}

@EnableDiscoveryClient
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class GulimallGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GulimallGatewayApplication.class, args);    }

}

application.yml的配置
spring:
  cloud:
    gateway:
      routes:
        - id: test_route
          uri: https://www.baidu.com
          predicates:
            - Query=url,baidu

        - id: qq_route
          uri: https://www.qq.com
          predicates:
            - Query=url,qq

##前端基础 技术栈简
-vscode
-ES6
-Node.js
-Vue
-Babel
-Webpack

前端后端技术栈类比
前端			后端
javascript 		 java
es6,7,8,9   		jdk8,9,10,11
框架			框架	
JQuery,Vue,React	spring,springMVC
工具			工具
webstorm,vscode  	idea,eclipse
项目构建		      项目构建
webpack,gulp		maven,gradle
依赖管理		      依赖管理
npm			maven

![image](https://user-images.githubusercontent.com/81153158/115147926-346d8d00-a098-11eb-9b70-7d6989724882.png)
29、前端基础 ES6 let&const
《ECMAScript 6 入门教程》是一本开源的 JavaScript 语言教程，全面介绍 ECMAScript 6 新引入的语法特性。
因此，ECMAScript 和 JavaScript 的关系是，前者是后者的规格，后者是前者的一种实现（另外的 ECMAScript 方言还有 JScript 和 ActionScript）。日常场合，这两个词是可以互换的。

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>
        let a =123;
        // let a = 10;
        var b = 1;
        console.log(a);
        for (let i = 0; i < 10; i++) {
            console.log(i);
        }
        // console.log(i);

        // var 的情况
        console.log(foo); // 输出undefined
        var foo = 2;

        // let 的情况
        let bar = 2; 
        console.log(bar); // 报错ReferenceError
        const PI = 3.1415;
        PI // 3.1415
        // PI = 3;  
        // TypeError: Assignment to constant variable.
        const foo = {};

        // 为 foo 添加一个属性，可以成功
        foo.prop = 123;
        foo.prop // 123

        // 将 foo 指向另一个对象，就会报错
        foo = {}; // TypeError: "foo" is read-only    
    </script>   
</body>
</html>

30、前端基础 ES6 解构&字符串

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>
        let arr=[1,2,3];
        let [a,b,c]=arr
        console.log(a,b,c)

        const Person = {
        name: '张三',
        //等同于birth: birth
        age: 24,
        language:['java','php','c++']
        // 等同于hello: function ()...
        // hello() { console.log('我的名字是', this.name); }
        };
        const {name:abc,age,language}=Person
        console.log(abc,age,language)

        let s = 'Hello world!';

        // includes()：返回布尔值，表示是否找到了参数字符串。
        // startsWith()：返回布尔值，表示参数字符串是否在原字符串的头部。
        // endsWith()：返回布尔值，表示参数字符串是否在原字符串的尾部    
        console.log(s.startsWith('Hello')) // true
        console.log(s.endsWith('!')) // true
        console.log(s.includes('o')) // true
        
        // 模板字符串
        let ss = `
            <ul>
                <li>first</li>
                <li>second</li>
            </ul>`
            console.log(ss)
        //变量和表达式可以插入

        function fun(){
            return "这是一个函数"
        }    
        let info = `我是${abc},今年${age}了，我想说${fun()}`
        console.log(info);
    </script>
</body>
</html>

31、前端基础 ES6 箭头函数
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>

        //函数参数的默认值
        //ES6 之前，不能直接为函数的参数指定默认值，只能采用变通的方法。
        // function log(x, y) {
        // y = y || 'World';
        // console.log(x, y);
        // }

        // console.log('Hello') // Hello World
        // console.log('Hello', 'China') // Hello China
        // console.log('Hello', '') // Hello World

        //ES6 允许为函数的参数设置默认值，即直接写在参数定义的后面
        function log(x, y = 'World') {
            console.log(x, y);
        }

        console.log('Hello') // Hello World
        console.log('Hello', 'China') // Hello China
        console.log('Hello', '') // Hello

        function Point(x = 0, y = 0) {
            this.x = x;
            this.y = y;
        }

        const p = new Point(2,5);
        console.log(p) // { x: 2, y: 5 }
        //2.不定参数
        //ES6 引入 rest 参数（形式为...变量名），用于获取函数的多余参数，这样就不需要使用arguments对象了。
        //rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中。
        function add(...values) {
            let sum = 0;
            for (var val of values) {
                sum += val;
            }
            return sum;
        }

        console.log(add(20, 50,1,2,3,4,5)) // 10
        //3.箭头函数 ES6 允许使用“箭头”（=>）定义函数。
        var f = v => v;
        // 等同于
        var f = function (v) {
            return v;
        };

        var sum2 = (a,b)=> a+b;
        console.log(sum2(10,20))    

        var sum3 = (a,b)=> {
            c=a+b
            return a+c;
        }
        console.log(sum3(10,20))  

        const Person = {
        name: '张三',
        //等同于birth: birth
        age: 24,
        language:['java','php','c++']
        // 等同于hello: function ()...
        // hello() { console.log('我的名字是', this.name); }
        };
        function hello(person){
            console.log("hello" + person.name)  
        }
        //箭头表达式加解构函数
        var hello2=({name})=>console.log("hello" + name)  
        hello2(Person)

    </script>
</body>
</html>	

32、前端基础 ES6 对象优化
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>
        const Person = {
        name: '张三',
        //等同于birth: birth
        age: 24,
        language:['java','php','c++']
        // 等同于hello: function ()...
        // hello() { console.log('我的名字是', this.name); }
        };
        console.log(Object.keys(Person))
        console.log(Object.values(Person))
        console.log(Object.entries(Person))
        const target = {a:1}
        const source1 = {b:2}
        const source2 = {c:3}
        Object.assign(target,source1,source2)
        console.log(target)
        //声明对象简写
        const age=23
        const name="zhangsan"
        const person2 = {name,age}
        console.log(person2)
        let person3 ={
            name: '张三',
            eat: function(food){
                console.log(this.name+"在吃"+food)
            },
            eat2:food=>console.log(person3.name+"在吃"+food),
            eat3(food){
                console.log(this.name+"在吃"+food)
            }

        }
        person3.eat("apple")
        person3.eat2("bnana")
        person3.eat3("pean")
        //对象拓展运算符
        //1、拷贝对象，深拷贝
        let obj = {name: '张三',age:16};
        let someone = {...obj};
        console.log(someone)

        //2、合并对象
        let age1 = {name: '张三',age:16};
        let age2 = {name1: '李四',age2:23};
        let a1 = {name1: 'wangwu',age2:30};
        a1 = {...age1,...age2};
        console.log(a1)

    </script>
</body>
</html>

33、前端基础 ES6 map、reduce
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>
        //map() 接收一个函数，将原数组中的所有元素用这个函数处理后放入新数组返回
         let arr =['1','-11', '15','3'];
        // arr=arr.map((item)=>{
        //     return item*2
        // });
        arr=arr.map(item=>item*2);
        console.log(arr)
        //reduce为数组中的每一个元素依次执行回调函数，不包括数组中被删除或从未被赋值的元素 
        //array.reduce(function(total, currentValue, currentIndex, arr), initialValue)
        //1.previousValue 上一次调用回调函数返回的值
        //2.currentValue 数组中当前被处理的元素
        //3.index 当前元素在数组中的索引
        //4.array 调用reduce的数组
        let res =arr.reduce((a,b)=>{
            console.log("上一次处理后:"+a);
            console.log("正在处理的:"+b);
            return a+b
        },100)
        console.log(res)

    </script>
</body>
</html>

34、前端基础 ES6 promise异步编排
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
</head>

<body>
    <script>
        // 1、查处当前用户信息
        // 2、按照当前用户的id 查处他的课程
        // 3、按照当前课程id查出得分
        // $.ajax({
        //     url: "mock/user.json",
        //     success(data) {
        //         console.log("查询用户:", data);
        //         $.ajax({
        //             url: `mock/user_corse_${data.id}.json`,
        //             success(data) {
        //                 console.log("查询到课程:", data);
        //                 $.ajax({
        //                     url: `mock/corse_score_${data.id}.json`,
        //                     success(data) {
        //                         console.log("查询到分数了:" + data);
        //                     },
        //                     error(error) {
        //                         console.log("出现异常:" + error);
        //                     }
        //                 })
        //             },
        //             error(error) {
        //                 console.log("出现异常:" + error);
        //             }
        //         })
        //     },
        //     error(error) {
        //         console.log("出现异常:" + error);
        //     }
        // })

        //1、Promise 可以封装异步操作
        // let p = new Promise((resolve, reject) => {
        //     // 1、异步操作
        //     $.ajax({
        //         url: "mock/user.json",
        //         success: function (data) {
        //             console.log("查询用户成功:", data);
        //             resolve(data);
        //         },
        //         error: function (err) {
        //             reject(err);
        //         }
        //     })
        // })

        // p.then((obj) => {
        //     //1、Promise 可以封装异步操作
        //     // 1、异步操作
        //     new Promise((resolve, reject) => {
        //         $.ajax({
        //             url: `mock/user_corse_${obj.id}.json`,
        //             success: function (data) {
        //                 console.log("查询到课程成功:", data);
        //                 resolve(data);
        //             },
        //             error: function (err) {
        //                 reject(err);
        //             }
        //         })
        //     }).then((data) => {
        //         console.log("上一步的结果:", data);
        //         $.ajax({
        //             url: `mock/corse_score_${data.id}.json`,
        //             success: function (data) {
        //                 console.log("查询到分数了成功:" + data);
        //             },
        //             error: function (err) {
        //             }
        //         })
        //     })
        // })

        //抽取一个公共的方法
        function get(url, data) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    url: url,
                    data: data,
                    success: function (data) {
                        resolve(data);
                    },
                    error: function (err) {
                        reject(err)
                    }
                })
            });
        }

        // 测试
        get("mock/user.json")
            .then((data) => {
                console.log("用户数据查询成功", data)
                return get(`mock/user_corse_${data.id}.json`);
            })
            .then((data) => {
                console.log("课程数据查询成功", data)
                return get(`mock/corse_score_${data.id}.json`);
            })
            .then((data) => {
                console.log("课程数据查询成功", data)
            })
            .catch((err) => {
                console.log("出现异常", err)
            })
    </script>
</body>
</html>

35、前端基础 ES6 模块化
hello.js
export const util ={
    sum(a,b){
        return a + b;    
    }  
}

// export {util}

user.js
var name = "jack"
var age = 21
function add(a,b){
    return a+b;
}

export {name,age,add}

import util from "./hello.js"
import {name,age,add} from "./user.js"

util.sum(1,2);
console.log(name);
add(1,3);

36、前端基础 Vue 介绍&HelloWorld
MVVM思想

M：model 包括数据和一些基本操作
V：view 视图，页面渲染结果
VM：View-model，模型与视图间的双向操作（无需开发人员干涉）
视图和数据通过VM绑定起来，model里有变化会自动地通过Directives填写到视view中，视图表单中添加了内容也会自动地通过DOM Listeners保存到模型中。








## 业务功能

### 三级菜单

- 下次再遇到需要生成菜单的业务逻辑，这个基本上就可以直接拿来使用了。


### 业务逻辑层

```mysql
CREATE TABLE `pms_category` (
  `cat_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '分类id',
  `name` char(50) DEFAULT NULL COMMENT '分类名称',
  `parent_cid` bigint(20) DEFAULT NULL COMMENT '父分类id',
  `cat_level` int(11) DEFAULT NULL COMMENT '层级',
  `show_status` tinyint(4) DEFAULT NULL COMMENT '是否显示[0-不显示，1显示]',
  `sort` int(11) DEFAULT NULL COMMENT '排序',
  `icon` char(255) DEFAULT NULL COMMENT '图标地址',
  `product_unit` char(50) DEFAULT NULL COMMENT '计量单位',
  `product_count` int(11) DEFAULT NULL COMMENT '商品数量',
  PRIMARY KEY (`cat_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1433 DEFAULT CHARSET=utf8mb4 COMMENT='商品三级分类';
```

```java
@Data
@TableName("pms_category")
public class CategoryEntity implements Serializable {
	private static final long serialVersionUID = 1L;

	@TableId
	private Long catId;

	private String name;

	private Long parentCid;

	private Integer catLevel;

	private Integer showStatus;

	private Integer sort;

	private String icon;

	private String productUnit;

	private Integer productCount;

	@TableField(exist = false)
	private List<CategoryEntity> children;

}
```

```java
public List<CategoryEntity> listWithTree() {
    // 这个类继承了 ServiceImpl
    // 1. 查出所有分类列表
    List<CategoryEntity> entities = baseMapper.selectList(null); // 传入 null 代表查询所有

    // 2. 组装成树形结构
    List<CategoryEntity> levelMenu = entities.stream()
        .filter(categoryEntity -> categoryEntity.getParentCid() == 0)
        .map(menu -> {
            menu.setChildren(getChildren(menu, entities));
            return menu;
        }).sorted((m1, m2) -> m1.getSort() == null ? 0 : m1.getSort() - (m2.getSort() == null ? 0 : m2.getSort())).collect(Collectors.toList());
    return levelMenu;
}

/**
* 递归查找所有菜单的子菜单
*
* @param root
* @param all
* @return
*/
private List<CategoryEntity> getChildren(CategoryEntity root, List<CategoryEntity> all) {
    List<CategoryEntity> children = all.stream()
        .filter(categoryEntity -> categoryEntity.getParentCid() == root.getCatId())
        .map(categoryEntity -> {
            categoryEntity.setChildren(getChildren(categoryEntity, all));
            return categoryEntity;
        })
        .sorted((m1, m2) -> m1.getSort() == null ? 0 : m1.getSort() - (m2.getSort() == null ? 0 : m2.getSort()))
        .collect(Collectors.toList());
    return children;
}
```

### 跨域问题

- 本系统解决方案：后端统一配置。
- 除此之外，还要把`renren-fast`自带的跨域配置给关闭掉。

```java
@Configuration
public class CORSConfig {

    @Bean
    public CorsWebFilter corsWebFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration corsConfiguration = new CorsConfiguration();

        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.setAllowCredentials(true);

        source.registerCorsConfiguration("/**", corsConfiguration);
        return new CorsWebFilter(source);
    }
}
```

### MyBatis Plus 逻辑删除

- 逻辑上的删除，只是更改状态，并不实际删除。
- 如三级菜单中 `show_status` 字段，逻辑上的删除是指在前端页面进行隐藏，而并不真正删除数据库这条数据。

## 前端代码调整

### 关闭前端权限认证

- 放权`src/util/index.js`

```javascript
export function isAuth (key) {
  // return JSON.parse(sessionStorage.getItem('permissions') || '[]').indexOf(key) !== -1 || false
  return true
}
```

### 关闭 ESLint

- 可选关闭`eslint `，路径`build/webpack.base.conf.js`，实际上是必关的。
- 因为自动生成的代码中并不符合 ESLint 标准，手动调整格式代价太大。

```javascript
const createLintingRule = () => ({
  test: /\.(js|vue)$/,
  loader: 'eslint-loader',
  enforce: 'pre',
  include: [resolve('src'), resolve('test')],
  options: {
    formatter: require('eslint-friendly-formatter'),
    emitWarning: !config.dev.showEslintErrorsInOverlay
  }
})
```

## 阿里云 OSS

- 开通服务，设置子账户，给子账户授权；
- 注意要把账号密码配置在 `nacos-server` 上，不要直接写在项目配置文件中，不然总会收到`GitHub`与阿里云发送的短信提醒（可能泄密）。
- [Spring Cloud Alibaba OSS](https://help.aliyun.com/document_detail/91868.html?spm=a2c4g.11186623.2.15.17706e28EQIQWR#concept-ahk-rfz-2fb)
- [OSS 获取服务器签名](https://help.aliyun.com/document_detail/91868.html?spm=a2c4g.11186623.2.15.57276e2888qoXF#concept-ahk-rfz-2fb)

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alicloud-oss</artifactId>
</dependency>
```

## 数据验证

- [前端表单验证-自定义验证](https://element.eleme.cn/#/zh-CN/component/form)
- 后端`JSR303`校验
  - 添加校验注解  `javax.validation.constraints`，定义自己的校验规则。
  - `Controller @Valid`，校验的`Bean`之后添加`BindingResult`可以获得校验结果。
  - 编写异常处理类，`@RestControllerAdvice`，使用`ExceptionHandler`标注方法可以处理的异常。
- 统一异常处理类
- `JSR`分组校验
  - 创建标记接口`public interface UpdateGroup{},public interface AddGroup{}`
  - 注解分组`@NotBlank(message = "品牌名不能为空", groups = {AddGroup.class, UpdateGroup.class})`
  - `Controller`添加`@Validated({UpdateGroup.class})`
  - 默认没有指定分组的校验注解，在分组校验情况下不生效。
- 自定义注解校验
  - 编写一个自定义的校验注解
  - 编写一个自定义的校验器 `ConstraintValidator`
  - 关联自定义的校验器和自定义的校验注解

> 自定义注解：@ListValue，用作值域，用于验证某个字段取值是否在此值域内。

```java
@Documented
@Constraint(validatedBy = { ListValueConstraintValidator.class})
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
@Retention(RUNTIME)
public @interface ListValue {

    String message() default "{edu.dlut.common.valid.ListValue.message}";

    Class<?>[] groups() default { };

    Class<? extends Payload>[] payload() default { };

    int[] value() default { };

}
```

```java
public class ListValueConstraintValidator implements ConstraintValidator<ListValue,Integer> {

    private Set<Integer> set = new HashSet<>();
    //初始化方法
    @Override
    public void initialize(ListValue constraintAnnotation) {
        int[] value = constraintAnnotation.value();
        for (int val : value)
            set.add(val);
    }

    /**
     *
     * @param value 需要校验的值
     * @param context
     * @return
     */
    @Override
    public boolean isValid(Integer value, ConstraintValidatorContext context) {
        return set.contains(value);
    }
}
```

```properties
# ValidationMessages.properties
edu.dlut.common.valid.ListValue.message=必须提交指定的值
```

```java
@ListValue(value = {0, 1}, groups = {AddGroup.class, UpdateStatusGroup.class})
```

## SKU与SPU

> 这两个名词将会贯穿从此开始到高级篇结束的所有内容。

- `SPU: Standard Product Unit` （标准产品单位）
  -  SPU 是商品**信息聚合的最小单位**，是一组可复用、易检索的标准化信息的集合，该集合描述了一个产品的特性。
  - 通俗点讲，属性值、特性相同的商品就可以称为一个 SPU。
  - 例如：`iPhone 12`就是一个 SPU，与商家，与颜色、款式、套餐都无关。

- `SKU: Stock Keeping Unit`(库存量单位) 
  - SKU 即库存进出计量的单位， 可以是以件、盒、托盘等为单位。
  - SKU 是物理上不可分割的**最小存货单元**。在使用时要根据不同业态，不同管理模式来处理。
  - 在服装、鞋类商品中使用最多最普遍。
  - 例如： `iPhone 12` 的颜色(深空灰等)，存储容量(64GB 256GB)。

# 分布式高级篇-微服务架构

## ElasticSearch

### 前置工作

- 通过 Docker 安装

```shell
$ docker pull elasticsearch:7.4.2 # 存储和检索数据
$ dock pull kibana:7.4.2 # 可视化检索数据
```

- 通过 `Homebrew` 安装

```shell
$ brew tap elastic/tap
$ brew install elastic/tap/elasticsearch-full
$ elasticsearch
$ brew services start elastic/tap/elasticsearch-full # 开机自启 可选
$ brew install kibana/tap/kibana-full
$ kibana
$ brew services start elastic/tap/kibana-full # 开机自启 可选

# ik 分词 
$ /usr/local/var/elasticsearch/plugins/ik/config
```

- `ElasticSearch` 配置与运行参数

```shell
$ mkdir -p /mydata/elasticsearch/config
$ mkdir -p /mydata/elasticsearch/data
$ echo "http.host:0.0.0.0" >> /mydata/elasticsearch/config/elasticsearch.yml
$ docker run --name elasticsearch -p 9200:9200 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms128m -Xmx128m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.4.2
$ chmod -R 777 /mydata/elasticsearch
```

- 运行 `Kibana`

```shell
# 其中IP地址一定要改为自己机器或服务器的IP
$ docker run --name kibana -e ELASTICSEARCH_HOSTS=http://xxx.xx.xx.xxx:9200 -p 5601:5601 -d kibana:7.4.2
```

### 倒排索引(面试重点)

### 官方开发手册

- [ElasticSearch Documentation](https://www.elastic.co/guide/index.html)

#### _cat

```json
GET/_cat/nodes
GET/_cat/health
GET/_cat/master
GET/_cat/indices // 查看所有索引
```

#### 索引文档

```json
// 保存一条数据 保存在哪个索引的哪个类型下 指定用哪一个标识
PUT customer/external/1 // PUT 和 POST 均可 PUT必须带ID，POST可带可不带
{
  "name": "John Snow"
}
```

### 整合 Spring Boot

- [Elasticsearch Clients](https://www.elastic.co/guide/en/elasticsearch/client/index.html)

### Kibana 创建 sku 索引

> 此索引后来会出现问题，下面有修改。

```json
PUT product
{
  "mappings": {
    "properties": {
      "skuId": {
        "type": "long"
      },
      "spuId": {
        "type": "keyword"
      },
      "skuTitle": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "skuPrice": {
        "type": "keyword"
      },
      "skuImg": {
        "type": "keyword",
        "index": false,
        "doc_values": false
      },
      "saleCount": {
        "type": "long"
      },
      "hasStock": {
        "type": "boolean"
      },
      "hotScore": {
        "type": "long"
      },
      "brandId": {
        "type": "long"
      },
      "catalogId": {
        "type": "long"
      },
      "catalogName": {
        "type": "keyword",
        "index": false,
        "doc_values": false
      },
      "brandName": {
        "type": "keyword",
        "index": false,
        "doc_values": false
      },
      "brandImg": {
        "type": "keyword",
        "index": false,
        "doc_values": false
      },
      "attrs": {
        "type": "nested",
        "properties": {
          "attrId": {
            "type": "long"
          },
          "attrName": {
            "type": "keyword",
            "index": false,
            "doc_values": false
          },
          "attrValue": {
            "type": "keyword"
          }
        }
      }
    }
  }
}

// 执行结果
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "product"
}
```
### 安装 nginx 将词库放入nginx，ik分词器向nginx发送请求。
```shell
docker container cp nginx:/etc/nginx .

docker run  -p 80:80 -name nginx \
-v /mydata/nginx/html:/usr/share/nginx/html \
-v /mydata/nginx/logs:/var/log/nginx \
-v /mydata/nginx/conf:/ect/nginx \
-d nginx:1.10
```
## Feign调用流程

> 推荐阅读：
>
> - 总结版
>
> - [详细版：Feign 原理（图解）](https://www.cnblogs.com/crazymakercircle/p/11965726.html)

### Feign 远程调用丢失请求头问题

`Feign`在远程调用之前要构造请求，此时会丢失请求头`headers`，`request`中包含许多拦截器。

在构建新请求的时候需要吧“老请求”中的数据获取并保存传递到新请求中。

```java
@Configuration
public class MallFeignConfig {
    @Bean("requestInterceptor")
    public RequestInterceptor requestInterceptor() {
        return new RequestInterceptor() {
            @Override
            public void apply(RequestTemplate template) {
                ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
                String cookie = requestAttributes.getRequest().getHeader("Cookie");
                template.header("Cookie", cookie);
            }
        };
    }
}
```

### Feign异步编排丢失请求头问题

- 原因：因为`RequestContextHolder`中的`ThreadLocal`只在当前线程可用，线程间独立，而在异步编排时会创建不同的线程执行任务，`ThreadLocal`中的数据将会丢失。
- 解决办法：在异步编排前首先获取`RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();`，然后在异步任务开始前重新设置进去，`RequestContextHolder.setRequestAttributes(requestAttributes);`

## 页面渲染

### 整合 Thymeleaf

- 关闭缓存`spring.thymeleaf.cache=false`
- 静态资源都放在`static`文件夹下就可以按照路径直接访问
- 页面都在`templates`下直接访问
- `SpringBoot`访问项目时会默认寻找`index.html`

## Nginx

### 域名配置

- `Nginx` 代理给网关的时候，**会丢失请求的`host`信息**，手动设置 `proxy_set_header Host $host`。

```nginx
#user  nobody;
worker_processes  1;
#pid        logs/nginx.pid;
events {
    worker_connections  1024;
}
http {
    upstream catmall{
       server 127.0.0.1:8888;
    }
    server {
        listen       80;
        server_name  catmall.com;
        location / {
	        proxy_set_header HOST $host;
	        proxy_pass http://catmall;
            #root   html;
            #index  index.html index.htm;
        }
    }
    include servers/*;
}
```

### 动静分离

> [本项目可能问到的 Nginx 面试题](https://github.com/raymond-zhao/cat-mall/wiki/Nginx)

- 将项目中 `static/` 下的静态资源移动到 `nginx` 服务器中，`mac` 为 `/usr/local/var/www`；

- 替换 `index.html` 中的文件路径；
- 配置 `nginx`；
- 重载配置 `nginx -s reload`。

```
// 在 server 块中添加
location /static/ {
    root /usr/local/var/www;
}
```


## 性能压测

### 基本概念

- HPS(Hits Per Second)：每秒点击次数
- TPS(Transaction Per Second)：每秒处理事务次数
- QPS(Query Per Scond)：每秒查询次数
- 最大响应时间
- 最少响应时间
- 90% 响应时间

### JVM

- `jconsole`
- `jvisualvm` 安装插件 `visualgc`
- [visualvm插件更新地址](https://visualvm.github.io/pluginscenters.html)

### Apache JMeter

- [NON_GUI](https://jmeter.apache.org/usermanual/get-started.html#non_gui)

```
================================================================================
Don't use GUI mode for load testing !, only for Test creation and Test debugging.
For load testing, use CLI Mode (was NON GUI):
   jmeter -n -t [jmx file] -l [results file] -e -o [Path to web report folder]
& increase Java Heap to meet your test requirements:
   Modify current env variable HEAP="-Xms1g -Xmx1g -XX:MaxMetaspaceSize=256m" in the jmeter batch file
Check : https://jmeter.apache.org/usermanual/best-practices.html
================================================================================
```

```shell
# 执行测试计划
$ jmeter -n -t testplan/RedisLock.jmx -l testplan/result/result.txt -e -o testplan/webreport
```

```shell
-n
This specifies JMeter is to run in cli mode
-t
[name of JMX file that contains the Test Plan].
-l
[name of JTL file to log sample results to].
-j
[name of JMeter run log file].
-r
Run the test in the servers specified by the JMeter property "remote_hosts"
-R
[list of remote servers] Run the test in the specified remote servers
-g
[path to CSV file] generate report dashboard only
-e
generate report dashboard after load test
-o
output folder where to generate the report dashboard after load test. Folder must not exist or be empty
The script also lets you specify the optional firewall/proxy server information:
-H
[proxy server hostname or ip address]
-P
[proxy server port]
```

### 压测优化

- JVM
- 索引
- 逻辑优化

## Redis

> 相关问题已整理至 [Wiki 页面]([https://github.com/raymond-zhao/cat-mall/wiki/%E7%BC%93%E5%AD%98](https://github.com/raymond-zhao/cat-mall/wiki/缓存))。

### 基本使用

- 第一个使用场景：缓存商品首页三级菜单


- 遇到的问题：堆外内存(直接内存)溢出 `OutOfDirectMemoryError`
- 分析思路：
  - SpringBoot2.0 之后默认使用 lettuce 作为操作 Redis 的客户端，lettuce 使用 Netty 进行网络通信。
  - lettuce 的 bug 导致 Netty 堆外内存溢出，Netty 如果没有指定对外内存 默认使用 JVM 设置的参数，可以通过 `-Dio.netty.maxDirectMemory` 设置堆外内存。
- 解决方案：
  - 不能仅仅使用 `-Dio.netty.maxDirectMemory` 去调大堆外内存，堆外内存也是有限的；
  - 可以选择升级 lettuce 客户端；
  - 切换使用 jedis 作为客户端；
  - `RedisTemplate` 对 `lettuce` 与 `jedis` 均进行了封装，所以直接使用。

```java
@Override
public Map<String, List<Catelog2VO>> getCatalogJson() {
    // 给缓存中放入JSON字符串，取出JSON字符串还需要逆转为能用的对象类型
    // 1. 加入缓存逻辑， 缓存中存的数据是 JSON 字符串
    String catalogJSON = stringRedisTemplate.opsForValue().get("catalogJSON");
    if (StringUtils.isEmpty(catalogJSON)) {
        // 2 如果缓存未命中 则查询数据库
        Map<String, List<Catelog2VO>> catalogJsonFromDB = getCatalogJsonFromDB();
        // 3 查到的数据再放入缓存 将对象转为JSON放入缓存
        String cache = JSON.toJSONString(catalogJsonFromDB);
        stringRedisTemplate.opsForValue().set("catalogJSON", cache);
        // 4 返回从数据库中查询的数据
        return catalogJsonFromDB;
    }

    Map<String, List<Catelog2VO>> result = JSON.parseObject(catalogJSON, new TypeReference<Map<String, List<Catelog2VO>>>() {});
    
    return result;
}
```

> 关于上面提到的 lettuce Bug，Lettuce 源码。

```java
private static void incrementMemoryCounter(int capacity) {
    if (DIRECT_MEMORY_COUNTER != null) {
        long newUserMemory = DIRECT_MEMORY_COUNTER.addAndGet((long) capacity);
        if (newUsedMemory > DIRECT_MEMORY_LIMIT) {
            DIRECT_MEMORY_COUNTER.addAndGet((long) (-capacity));
            throw new OutOfDirectMemoryError("failed to allocate " + capacity + " byte(s) of direct memory.")
        }
    }
}
```

### 缓存击穿、穿透、雪崩

|   类型   | 描述                                                         | 解决                                                         |
| :------: | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 缓存击穿 | 对于一些设置了过期时间的 key，如果这些 key 可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。如果这个 key 在大量请求同时进来前正好失效，那么所有对这个 key 的数据査询都落到db. | 加锁。大量并发只让一个去查，其他人等待，査到以后释放锁，其他人获取到锁，先查缓存，就会有数据，不用去 db |
| 缓存穿透 | 指查询一个一定不存在的数据，由于缓存是不命中，将去查询数据库，但是数据库也无此记录，我们没有将这次查询的null写入缓存，这将导致这个不存在的数据每次请求都要到存储层去査询，失去了缓存的意义。利用不存在的数据进行攻击，数据库瞬时压力增大，最终导致崩溃 | nul 结果缓存，并加入短暂过期时间                             |
| 缓存雪崩 | 缓存雪崩是指在我们设置缓存时 key 采用了相同的过期时间，导致缓存在某一时刻同时失效，请求全部转发到 DB, DB 瞬时压力过重雪崩。 | 原有的失效时间基础上增加一个随机值，比如 1-5 分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。 |

### 缓存数据一致性

- 双写模式：修改数据后(写到数据库)从数据库再查一遍放入缓存(写到缓存)
  - 脏数据问题：部分脏数据，缓存过期后又能得到最新的正确数据
- 失效模式：修改数据后删除缓存，等待下一次请求到来时再重新查询后放入缓存
- 解决： `canal`
  - 使用 `canal`更新缓存
  - 使用 `canal`解决数据易购
- 本系统的一致性解决方案
  - 为所有缓存数据设置过期时间，数据过期下一次查询触发主动更新。
  - 读写数据的时候，加上分布式的读写锁。(读多写少时几乎无影响)

### 分布式锁

- SpringBoot 所有的组件在容器中默认都是单例的，使用 `synchronized (this)` 可以实现加锁；
- 得到锁之后，应该再去缓存中确定一次，如果没有的话才需要继续查询；
- 假如有 100W 个并发请求，首先得到锁的请求开始查询，此时其他的请求将会排队等待锁；
- 等到获得锁的时候再去执行查询，但是此时有可能前一个加锁的请求已经查询成功并且将结果添加到了缓存中。

> 在每一个微服务中的`synchronized(this)`加锁的对象只是当前实例，但是并未对其他微服务的实例产生影响，即使每个微服务加锁后只允许一个请求，假如有 8 个微服务，仍然会有 8 个线程存在。

#### 锁-时序性问题

- **确认缓存-查询数据库-结果放入缓存** 这三个操作必须当做一个事务来执行，放在同一把锁里面完成。

### Redis 实现分布式锁🔐

## Redisson

> [Redisson-GitHub Wiki](https://github.com/redisson/redisson/wiki/目录)

### 看门狗

```java
RLock lock = redissonClient.getLock("my-lock");
lock.lock();
```

- 阻塞式等待，默认加的锁都是 30s 时间。
- 锁的自动续期，如果业务超长，如果业务运行时间较长，运行期间自动给锁续上新的 30s，不用担心业务时间过长(大于锁的过期时间)导致锁被删掉。
- 加锁的业务只要运行完成就不会给当前锁续期，即使不手动解锁，锁也会在 30s 后自动删除。

```java
lock.lock(10, TimeUnit.SECONDS);
```

- 在锁时间到了以后，不会自动续期。
- 如果我们传递了锁的超时时间，就发送给 redis 执行脚本，进行占锁，默认超时就是我们指定的时间。
- 如果我们未指定锁的超时时间，就使用 30*1000[**Lockwatchdog Timeout 看门狗的默认时间**]
- 只要占锁成功，就会启动一个定时任务【**重新给锁设置过期时间，新的过期时间就是看门狗的默认时间，每隔10s自动续期成30s**】， `internalLockLeaseTime`[看门狗时间/3 = 10s]

## Spring Cache

[Spring Cache Documentation](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/integration.html#cache)

### 常用注解

- `@Cacheable`: Triggers cache population.
- `@CacheEvict`: Triggers cache eviction.
- `@CachePut`: Updates the cache without interfering with the method execution.
- `@Caching`: Regroups multiple cache operations to be applied on a method.
- `@CacheConfig`: Shares some common cache-related settings at class-level.

### 重点类

- `CacheManager`
- `Cache`

### 默认行为(`@Cacheable({"category"})`)

- 如果命中缓存，方法不再被调用。
- `key`默认自动生成`category::SimpleKey []` 
  - 自定义接收SpEL：`@Cacheable(value = {"category"}, key= "'name'")`
  - `@Cacheable(value = {"category"}, key = "#root.method.name")`
- 缓存的`value`的值，默认使用`JDK`序列化机制，将序列化后的数据存到`Redis`
  - 保存为`JSON`格式原理
  - `CacheAutoConfiguration` -> `RedisCacheConfiguration` -> 自动配置了`RedisCacheManager` -> 初始化所有的缓存 -> 每个缓存决定用什么配置 -> 如果`redisCacheConfiguration`有就用已有的，没有就用默认配置 -> 想改缓存配置，只需要给容器中存放一个`RedisCacheConfiguration`即可 -> 就会应用到当前`RedisCacheManager`管理的所有缓存分区中。
- 默认`TTL=-1`
  - `spring.cache.redis.time-to-live=3600000`
- 自定义缓存配置

```java
@Configuration
@EnableCaching
@EnableConfigurationProperties(CacheProperties.class)
public class MyCacheConfig {
    @Bean
    public RedisCacheConfiguration redisCacheConfiguration(CacheProperties cacheProperties) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
        config = config.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()));
        config = config.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

        CacheProperties.Redis redisProperties = cacheProperties.getRedis();

        if (redisProperties.getTimeToLive() != null) {
            config = config.entryTtl(redisProperties.getTimeToLive());
        }
        if (redisProperties.getKeyPrefix() != null) {
            config = config.prefixKeysWith(redisProperties.getKeyPrefix());
        }
        if (!redisProperties.isCacheNullValues()) {
            config = config.disableCachingNullValues();
        }
        if (!redisProperties.isUseKeyPrefix()) {
            config = config.disableKeyPrefix();
        }
        return config;
    }
}
```

### `Spring Cache`总结

- 读模式
  - 缓存穿透：`cache-null-values=true`
  - 缓存击穿：`sync=true`
  - 缓存雪崩：`spring.cache.redis.time-to-live=时间`
- 写模式
  - 读写加锁
  - 引入 `Canal` , 感知到`MySQL`的更新则去更新缓存
  - 读多写多，直接去数据库查询
- 总结
  - 常规数据(度多写少，即时性，一致性要求不高的数据)，完全可以使用`Spring Cache`
  - 特殊数据，特殊设计。

## 检索服务

> 坑：在从首页点击分类名跳转到搜索页时，跳转链接在`catalogLoader.js`中，原静态资源链接为`http://search.gmall.com/`，需要改为自己在  HOST  文件中配置的域名。

### 迁移索引 `mapping`

```json
// 不要直接删除重建 会丢失已上架的商品数据

PUT product
{
  "mappings": {
    "properties": {
      "skuId": {
        "type": "long"
      },
      "spuId": {
        "type": "keyword"
      },
      "skuTitle": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "skuPrice": {
        "type": "keyword"
      },
      "skuImg": {
        "type": "keyword"
      },
      "saleCount": {
        "type": "long"
      },
      "hasStock": {
        "type": "boolean"
      },
      "hotScore": {
        "type": "long"
      },
      "brandId": {
        "type": "long"
      },
      "catalogId": {
        "type": "long"
      },
      "catalogName": {
        "type": "keyword"
      },
      "brandName": {
        "type": "keyword"
      },
      "brandImg": {
        "type": "keyword"
      },
      "attrs": {
        "type": "nested",
        "properties": {
          "attrId": {
            "type": "long"
          },
          "attrName": {
            "type": "keyword"
          },
          "attrValue": {
            "type": "keyword"
          }
        }
      }
    }
  }
}

PUT mall_product

POST _reindex
{
    "source": {
        "index": "product"
    },
    "dest": {
        "index": "mall_product"
    }
}
```

### 构建检索请求与封装检索结果

- `mall-search/src/main/java/包名/service/impl/MallSearchServiceImpl`

## 多线程与异步

- [进程、线程与线程池](https://raymond-zhao.top/2020/07/19/2020-07-19-ProcessAndThread/)

- `CompletableFuture<T>`


### 业务场景 - 商品详情页

- 获取 SKU 基本信息
- 获取 SKU 图片信息
- 获取 SKU 促销信息
- 获取 SPU 所有销售属性
- 获取规格参数组及组下的规格参数
- SPU 详情

### CompletableFuture 主要功能

- 创建异步对象
- 计算完成时回调方法

- `handle` 方法
- 线程串行化

- 两任务组合

- 多任务组合

## 认证服务

### 短信验证

- 接口防刷

### OAuth2.0 之微博登录

- [微博开放平台](https://open.weibo.com/)

- [微博OAuth2.0文档](https://open.weibo.com/wiki/%E6%8E%88%E6%9D%83%E6%9C%BA%E5%88%B6%E8%AF%B4%E6%98%8E)

> 微博登录出现的问题：

- 微博回调域名：`auth.catmall.com`，而不是`catmall.com`；
- `OAuthController`：
  -  `doPost`方法后三个参数数据类型是`Map`，均不能传入``null`，而是传入空的 `Map`；
  - 另外`Map`的顺序有变化，在构建请求条件时应该将`map`传入查询参数`querys`中，而不是请求体`bodys`。

```java
HttpResponse response = HttpUtils.doPost("https://api.weibo.com", "/oauth2/access_token", "post", new HashMap<>(), map, new HashMap<>());
```

## Spring Session

- [Spring Session Documentation](https://docs.spring.io/spring-session/docs/2.3.0.RELEASE/reference/html5/#introduction)

### 对象 JDK 序列化

### Spring Session核心原理

- `@EnableRedisHttpSession`注解中导入了`RedisHttpSessionConfiguration`类
  - 给容器中添加了一个组件：`SessionRepository` -> `RedisOperationsSessionRepository`，利用`Redis`来进行`Session`的增删改查等各种操作。
- `SessionRepositoryFilter`: `Session`存储过滤器，每个请求都必须经过`filter`
  - 创建的时候，自动从容器中获取`SessionRepository`
  - 原始的`request、response`都被包装成`SessionRepositoryRequestWrapper.SessionRepositoryResponseWrapper`
  - 使用装饰者模式进行包装。

## 购物车

- 用户可以在登录状态下将商品加入[在线购物车/用户购物车]
  - 放入`MongoDB`；
  - 放入`MySQL`；
  - 放入`Redis`(采用)，登录以后，会将临时购物车中的数据合并过来。
- 用户可以在未登录状态下将商品加入[离线购物车/游客购物车]
  - 放入`localStorage`；
  - 放入`Cookie`；
  - 放入`WebSQL`；
  - 放入`Redis`(采用)，即使浏览器关闭，临时购物车数据都在。

- 用户可以使用购物车一起结算下单
- 用户可以添加商品到购物车
- 用户可以查询自己购物车
- 用户可以选中购物车中商品
- 用户可以在购物车中修改购买的商品数量
- 用户可以在购物车中删除商品
- 在购物车中展示优惠信息
- 提示购物车商品价格变化

> 京东给每个用户生成一个值类似于 UUID 的`user-key`，有效期一个月，存储在`Cookie`，浏览器保存以后，每次访问都会带上这个`cookie`。
>
> 登录后：`session`有用户信息
>
> 未登录：`cookie`中的`user-key`
>
> 第一次使用时，如果没有临时用户，帮忙创建一个临时用户。

### ThreadLocal 用户身份鉴别

- `public class CartInterceptor implements HandlerInterceptor{}`重写`preHandle, postHandle` ，不用加`@Component`
- 添加`MallWebConfig`

```java
@Configuration
public class MallWebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new CartInterceptor()).addPathPatterns("/**");
    }
}
```

## 消息队列 - RabbitMQ (面试高频)

本系统消息队列工作图

![本系统消息队列工作图](https://tva1.sinaimg.cn/large/007S8ZIlly1ghtsf6mboij30yl0t7afn.jpg)

### 应用场景

- 异步处理
- 应用解耦
- 流量控制(削峰、填谷)

### 类型

- 队列(点对点)
- 订阅

### 执行标准

- JMS
- AMQP

### 安装

- Docker
  - 4369,25672: Erlang 发现 & 集群端口
  - 5672,5671: AMQP端口
  - 15672: Web管理后台端口
  - 1883,8883: MQTT协议端口
  - 61613, 61614: STOMP协议端口

```shell
$ docker pull rabbitmq
$ docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management
```

- 手动下载安装

```shell
$ cd 安装目录
$ # 启用 rabbitmq management插件
$ sudo sbin/rabbitmq-plugins enable rabbitmq_management
$ # 配置环境变量(可选)
$ rabbitmq-server -detached # 后台启动
# 查看状态 浏览器内输入 http://localhost:15672, 默认的用户名密码都是 guest。
$ rabbitmqctl status
$ rabbitmqctl stop # 关闭
```

```
# Setting for RabbitMQ
export RABBIT_HOME=/usr/local/Cellar/rabbitmq/3.8.9_1
export PATH=$PATH:$RABBIT_HOME/sbin
```

- Homebrew 安装

```shell
$ brew install rabbitmq
# OR
$ brew cask install rabbitmq
```

### 发送消息

- `@RabbitListener`
- `@RabbitHandler`

如果是传输对象的话，传输的对象必须实现序列化接口，默认的序列化方式是 JDK 序列化，但是也可以手动指定序列化的方式。

```java
@Configuration
public class MyRabbitConfig {
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

### 可靠投递

### 延时队列

### Dead Letter Exchanges(DLX) - 死信路由

### 消息积压、重复、丢失等问题解决方案

- 消息丢失
  - 消息发送出去，由于网络问题没有抵达服务器
    - 做好容错方法(`try-catch`)，发送消息可能会网络失败，失败后要有重试机制，可记录到系统数据库，采用定期扫描重发的方式。
    - 做好日志记录，每个消息状态是否都被服务器收到都应该被记录
    - 做好定期重发，如果消息没有发送成功，定期去数据库扫描未成功的消息进行重发
  - 消息抵达`Broker`，`Broker`要将消息写入磁盘才算成功，此时`Broker`尚未持久化完成，宕机。
    - `publisher`必须加入确认回调机制，确认成功的消息，修改数据库消息状态
  - 自从`ACK`状态下，消费者收到消息，但没来得及消费便宕机
    - 一定开启手动`ACK`，消息成功才移除，失败或者没来得及处理就`noACK`并重新入队。
- 消息重复
  - 消息消费成功，事务已经提交，`ack`时，机器宕机，导致没有`ack`成功，`Broker`的消息重新由`unack-> ready`，并发送给其他消费者。
  - 消息消费失败，由于重试机制，自动又将消息发送出去。
  - 成功消费，`ack`时宕机，消息由`unack`变为`ready`，`Broker`又重新发送
    - 消费者的业务消费接口应该设计成幂等性的，比如扣库存工作单的状态标志
    - 使用 **防重表(redis, mysql)** ，发送消息每一个都有业务的唯一标识，处理过就不用再处理。
    - `RabbitMQ`的每一个消息都有`redelivered`字段，可以获取消息是否是被重新投递的。
- 消息积压
  - 消费者宕机积压
  - 消费者消费能力不足积压
  - 发送者发送流量太大
    - 上线更多的消费者，进行正常消费。
    - 上线专门的队列消费服务，将消息先批量取出来，记录数据库，离线慢慢处理。

```mysql
create table `mq_message` (
    `message_id` char(32) not null,
    `content` text,
    `to_exchange` varchar(255) default null,
    `routing_key` varchar(255) default null,
    `class_type` varchar(255) default null,
    `message_status` int(1) default '0' comment '0-新建 1-已发送 2-错误抵达 3-已抵达',
    `create_time` datetime default null,
    `update_time` datetime default null,
    primary key (`message_id`)
) engine InnoDB default charset=utf8mb4
```

## 订单服务

### 基本环境搭建

- 配置 `Nginx` 静态资源，网关等。

### 订单流程

下单->创建订单->验证令牌->核算价格->锁定库存

## 接口幂等性

在确认页点击 **提交订单** 时，用户可能不小心点击多次，所以即使用户点击次数大于1次，也应该保证只提交一次。

- 接口幂等性：保证用户对统一操作发起的一次请求或多次请求的结果时一致的。

### 应用情况

- 用户多次点击按钮
- 用户页面回退后再次提交
- 微服务相互调用，由于网络问题导致请求失败，触发`feign`重试机制
- 其他业务情况

### 幂等性解决方案

- Token机制
  - `Redis Lua` 脚本
- 各种锁机制
  - 数据库悲观锁、乐观锁
  - 业务层分布式锁
- 各种唯一性约束
  - 数据库唯一性约束
  - `redis set `防重
  - 防重表
  - 全局请求唯一ID

## 分布式事务

![image-20200525153506445](https://tva1.sinaimg.cn/large/007S8ZIlly1gf4qgbev2nj31jx0u049h.jpg)

- CAP 定理
  - C: 一致性，在分布式系统中的所有数据备份，在同一时刻是否有同样的值。
  - A: 可用性，再急群众一部分结点故障后，集群整体是否还能响应客户端的读写请求。
  - P: 分区容错性，大多数分布式系统都分布在多个子网络，每个子网络就叫做一个区，分区容错的意思是，区间通信可能失败。
  - CAP 定理指的是以上三点至多只能同时保证两点，不能三者兼顾，一般来说在分布式系统中 P 不可避免，所以一个系统至多只能满足 CP 或 AP。
- [Raft定理动画](http://thesecretlivesofdata.com/raft/)
- BASE 定理
  - 选择 AP，舍弃实现 C (强一致性)，选择实现弱一致性，保证实现最终一致性。
  - 基本可用
  - 软状态
  - 最终一致性

### 事务传播

- 本地事务失效问题
  - 同一个对象内事务互调默认失败，原因是绕过了代理对象，而事务是通过代理对象来控制的。
- 解决方法
  - 使用代理对象来调用事务方法，引入`spring-boot-starter-aop`，`aop`又引入了`aspectj`
  - `@EnableAspectJAutoProxy(exposeProxy = true)`，开启`aspectj`动态代理功能，如果不开启的话，默认使用的是`JDKProxy`，开启后以后创建对象采用`aspectj`动态代理(即使没有接口也可以创建代理对象, JDKProxy要求被代理的对象有接口定义)
  - 本类事务互相调用此时可以实现`AopContext.currentProxy`

### 解决方案

- 2PC(2 phase commit, 二阶段提交)模式
- 柔性事务-TCC事务补偿性方案
  - 刚性事务：遵循ACID
  - 柔性事务：遵循BASE
- 柔性事务-最大努力通知型方案
- 柔性事务-可靠消息+最终一致性(异步确保型)

## Seata

```mysql
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

## 支付宝支付

- [支付宝开放平台](https://open.alipay.com/platform/home.htm)

### 加密与解密

- 对称加密
- 非对称加密
- 公钥、私钥、签名、验签

### 沙箱环境

[文档-沙箱环境](https://opendocs.alipay.com/open/200/105311)

[RSA生成器](https://opendocs.alipay.com/open/291/106097/)

### 内网穿透

### 支付宝异步通知

## 秒杀业务

![秒杀(高并发)系统关注的问题](https://tva1.sinaimg.cn/large/007S8ZIlly1ghtt8gm4nuj30l50ff413.jpg)

秒杀业务具有瞬间高并发的特点，必须要做限流+异步+缓存(页面静态化)+独立部署

限流方式:

- 前端限流: 一些高并发的网站直接在前端页面开始限流。
- `nginx` 限流: 直接负载部分请求到错误的静态页面，令牌算法，漏斗算法。
- 网关限流: 限流的过滤器
- 代码中使用分布式信号量
- `RabbitMQ` 限流，保证发挥所有服务器的性能。

### CRON表达式

### 定时与异步

- 定时任务
  - `@EnableScheduling`开启定时任务
  - `@Scheduled`开启一个定时任务
- 异步任务
  - `@EnableAsync`开启异步任务
  - `@Async` 标注在需要异步执行的方法上

### 幂等性保证

- 随机码

> 查看总的代码行，包括添加了多少行，删除了多少行，现在总共多少行。

```shell
$ git log  --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -
```

## Sentinel流控熔断降级

[Sentinel Wiki - 中文](https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D)

| Sentinel       | Hystrix                                        |                               |
| -------------- | ---------------------------------------------- | ----------------------------- |
| 隔离策略       | 信号量隔离                                     | 线程池隔离/信号量隔离         |
| 熔断降级策略   | 基于响应时间或失败比率                         | 基于失败比率                  |
| 实时指标实现   | 滑动窗口                                       | 滑动窗口（基于 RxJava）       |
| 规则配置       | 支持多种数据源                                 | 支持多种数据源                |
| 扩展性         | 多个扩展点                                     | 插件的形式                    |
| 基于注解的支持 | 支持                                           | 支持                          |
| 限流           | 基于 QPS，支持基于调用关系的限流               | 有限的支持                    |
| 流量整形       | 支持慢启动、匀速器模式                         | 不支持                        |
| 系统负载保护   | 支持                                           | 不支持                        |
| 控制台         | 开箱即用，可配置规则、查看秒级监控、机器发现等 | 不完善                        |
| 常见框架的适配 | Servlet、Spring Cloud、Dubbo、gRPC 等          | Servlet、Spring Cloud Netflix |

## Sleuth + Zipkin 链路追踪

[Zipkin](https://zipkin.io/pages/quickstart.html)

```shell
# docker
$ docker run -d -p 9411:9411 openzipkin/zipkin

# java
$ curl -sSL https://zipkin.io/quickstart.sh | bash -s
$ java -jar zipkin.jar
```

# 高可用集群篇-架构师提升

## Kubernetes

[Kubernetes-中文文档](https://kubernetes.io/zh/)

### 环境准备

- 进入三个虚拟机，开启 `root` 的密码访问权限。

```shell
$ vargrant ssh xxxxx
$ su root # password vargrant
$ vi /etc/ssh/sshd_config
# 修改 PasswordAuthentication yes
$ service sshd restart
# 所有虚拟机设置为 4 core 4G
```

- 设置`linux`环境(三个结点都要执行)

```shell
# 关闭防火墙
$ systemctl stop firewalld
$ systemctl disable firewalld
# 关闭 selinux
$ sed -i 's/enforcing/disabled/' /etc/selinux/config
# 关闭内存交换
$ swapoff -a # 临时
$ sed -ri 's/.*swap.*/#&/' /etc/fstab # 永久
$ free -g # 验证 swap 必须为 0
```

- 添加主机名与 IP 映射

```shell
$ vi /etc/hosts
# 前边为网卡地址 后边为集群结点名
xxxxxxx  k8s-node1
xxxxxxx  k8s-node2
xxxxxxx  k8s-node3
```

- 将桥接的 `IPv4` 流量传递到 `iptables` 链

```shell
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### 安装docker

- 卸载系统之前到docker

```shell
$ sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```

- 安装 Docker-CE

```shell
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# 设置 docker repo 到 yum 位置
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 安装 docker docker-cli
$ sudo yum install -y docker-ce docker-ce-cli containerd.io
```

- docker 加速

```shell
$ sudo mkdir -p /etc/docker
$ sudo tee /etc/docker/daemon.json << -'EOF'
{
	"registry-mirrors": [阿里云是个不错的选择]
}
EOF
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
$ sudo systemctl enable docker # 开机自启
```

## MySQL 集群

### 集群类型

- 主从式
  - 主从复制，同步方式。
  - 主从调度，控制方式。
- 分片式
  - 数据分片存储，片区之间备份。
- 选主式
  - 出现容灾时选主，调度时选主。

### Docker 安装 MySQL - 一主两从

> 感觉哪里少了东西

```shell
$ docker run -p 3306:3306 --name mysql-master \
-v /mydata/mysql/master/log:/var/log/mysql \
-v /mydata/mysql/master/data:/var/lib/mysql \
-v /mydata/mysql/master/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7

$ docker run -p 3306:3306 --name mysql-slave-1 \
-v /mydata/mysql/slave/log:/var/log/mysql \
-v /mydata/mysql/slave/data:/var/lib/mysql \
-v /mydata/mysql/slave/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7

$ docker run -p 3306:3306 --name mysql-slave-2 \
-v /mydata/mysql/slave/log:/var/log/mysql \
-v /mydata/mysql/slave/data:/var/lib/mysql \
-v /mydata/mysql/slave/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7
```

```shell
$ vi /mydata/mysql/master/conf/my.cnf
```

```
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve

# 主从的这个配置仅在于 id 的不同
server_id=1
log-bin=mysql-bin
read-only=0
binlog-do-db=catmall_ums
binlog-do-db=catmall_pms
binlog-do-db=catmall_oms
binlog-do-db=catmall_sms
binlog-do-db=catmall_wms
binlog-do-db=catmall_admin

replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema
```

- 为 master 授权用户来同步数据

```shell
# 进入 master 容器
$ docker exec -it mysql /bin/bash
$ mysql -uroot -p
mysql> grant all priviledges on *.* to 'root'@'%' identified by 'root' with grant option;
mysql> flush priviledges;
mysql> GRANT REPLICATION SLAVE ON *.* to 'backup'@'%' identified by '123456';
show master status
```

- 设置主库连接

```shell
change master to master_host='xxxxxxx', matser_user='backup', master_password='123456', master_log_file='mysql-bin.000001', master_log_pos=0, master_port=3307;
# 启动主从同步
start slave
# 查看从库状态
show slave status
```

### [Sharding-Sphere](https://shardingsphere.apache.org/document/legacy/4.x/document/cn/overview/)

### Redis集群 - 三主三从

```shell
for port in $(seq 7001 7006) \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port ${port}
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip xxxxxx
cluster-announce-port ${port}
cluster-announce-bus-port ${port}
appendonly yes
EOF
docker run -p ${port}:${port} -p 1${port}:1${port} --name redis-${port} \
-v /mydata/redis/node-${port}/data:data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
-d redis:5.0.7 redis-server /etc/redis/redis.conf
done
```

```shell
$ docker stop ${docker ps -a | grep redis-700 | awk '{print $1}'}
$ docker rm ${docker ps -a | grep redis-700 | awk '{print $1}'}
```

- 使用 Redis 建立集群

```shell
$ docker exec -it redis-7001 bash
$ redis-cli --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 --cluster-replicas 1
```

## ElasticSearch集群

## RabbitMQ集群

## Kubernetes部署

## 流水线
