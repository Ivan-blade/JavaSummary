### javaweb

#### springboot获取当前request和session

+ java-futher-study

  + Stage2-module2
    + Projectwork

#### springboot在启动时和销毁时执行默认操作

+ java-futher
  + springbootLifecycle

#### springboot指定编译版本

+ 可以通过properties配置添加pom配置

  ```xml
  <properties>
      <java.version>11</java.version>
      <maven.compiler.source>11</maven.compiler.source>
      <maven.compiler.target>11</maven.compiler.target>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  <!--    <java.version>节点是自己定义的表示java版本。-->
  <!--    <maven.compiler.source>表示源码使用的java版本。-->
  <!--    <maven.compiler.target>表示编译时使用的java版本。-->
  <!--    <project.build.sourceEncoding>表示项目使用的编码，一般设置为UTF-8-->
  ```


+ 通过maven插件配置

  ```xml
  <build>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.1</version>
        <configuration>
            <source>1.8</source> <!-- 源代码使用的开发版本 -->
            <target>1.8</target> <!-- 需要生成的目标class文件的编译版本 -->
            <!-- 一般而言，target与source是保持一致的，但是，有时候为了让程序能在其他版本的jdk中运行(对于低版本目标jdk，源代码中需要没有使用低版本jdk中不支持的语法)，会存在target不同于source的情况 -->
  
            <!-- 这下面的是可选项 -->
            <meminitial>128m</meminitial>
            <maxmem>512m</maxmem>
            <fork>true</fork> <!-- fork is enable,用于明确表示编译版本配置的可用 -->
            <compilerVersion>1.3</compilerVersion>
  
            <!-- 这个选项用来传递编译器自身不包含但是却支持的参数选项 -->
            <compilerArgument>-verbose -bootclasspath ${java.home}\lib\rt.jar</compilerArgument>
  
        </configuration>
    </plugin>
   </build>
  ```

  

#### 线程池使用

+ 实现runnable

  ```java
  public class ThreadTask implements Runnable{
  
      public ThreadTask() {}
      
      @Override
      public void run() {
          dealMessage();
      }
      
  	 public void dealMessage() {
  	 System.out.println("hello world");
  	 }
  }
  ```

+ 创建线程池

  ```java
  public class Application {
  
      private ThreadPoolExecutor executor = new ThreadPoolExecutor(8, 16, 500, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());
  
      public void test() {
  
          while (true) {
              ThreadTask threadTask = new ThreadTask();
              executor.execute(threadTask);
          }
  
      }
  
      public static void main(String[] args) {
  
          Application application = new Application();
          application.test();
      }
  
  }
  
  ```

  

#### 本地缓存

+ 引入依赖

```xml
<dependency>
   <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>25.1-jre</version>
</dependency>
```
```java
// 缓存过滤重复简称映射,过期时间5天
  private Cache<String, String> abbrMappingCache = CacheBuilder.newBuilder().maximumSize(100000).expireAfterWrite(5, TimeUnit.DAYS).build();
// 当hashmap用

String key = "diykey";
String cacheValue = abbrMappingCache.getIfPresent(key);
// 如果存在进行操作
if (cacheValue == null) {
  // 写死value，提供查询的返回值，这边缓存查重只要key就可以
  abbrMappingCache.put(key,"exist");
}
```

#### 将静态参数注入spring容器

```java
@Component
public class LoadProperties{
  
    public static String NO;
    private static IOptionService optionService;
    
    @Value("${test.no}")
    private void setNO(String NO){
      LoadProperties.NO = NO;
    }  
  
    @Autowired
    public void setOptionService(IOptionService optionService) {
        Commons.optionService = optionService;
    }
}
```

