### springbootLifecycle

#### 启动时

+ 初始依赖

  ```pom
  <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>2.5.2</version>
      </parent>
  
  <dependencies>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-configuration-processor</artifactId>
              <optional>true</optional>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
          </dependency>
  </dependencies>
  ```

  

+ 实现接口ApplicationRunner

  ```java
  @Component
  public class SpringbootInit implements ApplicationRunner {
      @Override
      public void run(ApplicationArguments args) throws Exception {
          System.out.println("启动执行...");
      }
  }
  ```

+ 实现接口CommandLineRunner

  ```java
  @Component
  public class MyCommandLineRunner1 implements CommandLineRunner{
  	@Override
  	public void run(String... args) throws Exception {
  		System.out.println("启动执行...");
  	}	
  }
  ```

+ 也可以在启动类上直接实现接口（如果是小型项目对层次要求不是很清晰的话）

  ```java
  @SpringBootApplication
  public class ServerBootstrapApplication implements CommandLineRunner {
  
  
      public static void main(String[] args) {
          SpringApplication.run(ServerBootstrapApplication.class, args);
      }
  
      @Override
      public void run(String... args) throws Exception {
          System.out.println("启动执行...");
      }
  }
  ```

#### 销毁时

+ 实现接口DisposableBean

  ```java
  import org.springframework.beans.factory.DisposableBean;
  import org.springframework.stereotype.Component;
  
  
  @Component
  public class MyDisposableBean implements DisposableBean{
   
  	@Override
  	public void destroy() throws Exception {
  		System.out.println("结束");
  		
  	}
   
  }
  ```

  