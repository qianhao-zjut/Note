### apache common常用工具类

#### 一.commons-beanutils：使用反射配置beans

教程：https://blog.csdn.net/qq_36838191/article/details/81297113

二.commons-codec：设置编码和解码以及摘要计算

教程：https://blog.csdn.net/jianggujin/article/details/51149133

三.commons-lang：常用工具集合

教程：https://blog.csdn.net/hebtu666/article/details/84580192

四.commons-io

教程：https://blog.csdn.net/langgufu314/article/details/84721816



### 定时任务：cron

教程：https://blog.csdn.net/sunnyzyq/article/details/98597252

示例：

```java
@Component
@Configuration
@EnableScheduling
public class TaskConfigration {
    private int time=0;
    @Scheduled(cron = "*/5 * * * * *")
    private void getTime(){
        time++;
        String s=""+time+new Date();
        System.out.println("Time==>"+s);
    }
}
```

