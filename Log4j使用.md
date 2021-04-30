### SpringBoot中Log4j使用



#### 日志信息的优先级从高到低有ERROR、WARN、 INFO、DEBUG



教程1（还有log4j2的使用，以及将日志存在数据库中）：https://blog.csdn.net/zzq900503/article/details/87629782

教程2（Log4j的详细教程）：https://www.jianshu.com/p/c6c543e4975e

 教程3（Log4j在Springboot中的简单使用）：https://my.oschina.net/duo8523/blog/1535685

教程2和3结合起来，比较实用，教程1太多了，而且排版不好

#### 使用流程：

##### 1.在pom文件中手动消除默认的logging ，再添加Log4j依赖

因为高版本的Springboot本身只支持Log4j2，而且SpringBoot中有默认配置的日志框架，所以要把这个框架消去，再引入Log4j的依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <!--排去 -->
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!--引入Log4j-->
 <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>

        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
```

##### 2.定义log4j.properties配置文件

可以在配置文件中指定日志输出位置，格式化日志输出等

自定义打印：

Log4J采用类似C语言中的printf函数的打印格式格式化日志信息，打印参数如下：

- %m 输出代码中指定的消息
- %p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL
- %r 输出自应用启动到输出该log信息耗费的毫秒数
- %c 输出所属的类目，通常就是所在类的全名
- %t 输出产生该日志事件的线程名
- %n 输出一个回车换行符，Windows平台为“rn”，Unix平台为“n”
- %d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921
- %l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10)

```properties
# LOG4J配置
log4j.rootCategory=INFO,stdout,file
# 控制台输出
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %5p %c{1}:%L - %t==>%m%n
```







##### 3.在application.properties中指定log4j.properties的位置

```properties
logging.config=classpath:log4j.properties
```

##### 4.中需要用的的地方使用

使用方式，在需要用到的类中生成logger类，然后调用其方法

```java
private static Logger logger = Logger.getLogger(VelocityDemoApplication.class);

public class VelocityDemoApplication {
    private static Logger logger = Logger.getLogger(VelocityDemoApplication.class);
    public static void main(String[] args) {
        logger.info("spring boot App is starting.......");
        SpringApplication.run(VelocityDemoApplication.class, args);
        logger.info("spring boot App is running........");
    }

}
```

##### 5.与数据库结合使用

1.在log4j.properties中配置输出路径（数据库）

2.在数据库中建表

3.绑定信息，因为logger.info(message)中只可以传一个对象，所以我们可以使用Gson将对象转为json存在数据库中，后面可以通过json转bean对象进行操作

```properties
###数据库输出
log4j.appender.jdbc=org.apache.log4j.jdbc.JDBCAppender
log4j.appender.jdbc.driver=com.mysql.jdbc.Driver
log4j.appender.jdbc.URL=jdbc:mysql://127.0.0.1:3306/test?characterEncoding=utf8&useSSL=true
log4j.appender.jdbc.user=root
log4j.appender.jdbc.password=root
log4j.appender.jdbc.sql=insert into log_icecoldmonitor(level,category,thread,time,location,note) values('%p','%c','%t','%d{yyyy-MM-dd HH:mm:ss:SSS}','%l','%m')
```



```sql
CREATE TABLE `log_icecoldmonitor` (
  `Id` int(11) NOT NULL AUTO_INCREMENT,
  `level` varchar(255) NOT NULL DEFAULT '' COMMENT '优先级',
  `category` varchar(255) NOT NULL DEFAULT '' COMMENT '类目',
  `thread` varchar(255) NOT NULL DEFAULT '' COMMENT '进程',
  `time` varchar(30) NOT NULL DEFAULT '' COMMENT '时间',
  `location` varchar(255) NOT NULL DEFAULT '' COMMENT '位置',
  `note` text COMMENT '日志信息',
  PRIMARY KEY (`Id`)
)
```



```java
@Controller
public class UserController {
    private static Logger logger=Logger.getLogger(UserController.class);
    @RequestMapping("/hello")
    @ResponseBody
    public Object hello(){
        People people=new People("qh","/hello");
        Gson gson=new Gson();
        logger.info(gson.toJson(people));
        return "hello";
    }
}
```