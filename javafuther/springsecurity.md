### Springsecurity

#### 简介

#### springsecurity认证原理与实战

+ 为什么bcrypt每次加密都不一样还能匹配成功

##### 两种认证方式

##### 表单认证

+ 自定义登录页面

+ 基于数据库实现登录

+ 密码加密

+ 获取当前用户

  + SecurityContextHolder 保留系统当前的安全上下文SecurityContext，其中就包括当前使用系统的用户的信息

  + SecurityContext 安全上下文,获取当前经过身份验证的主体或身份验证请求令牌

  + 代码实现

    ```java
    /**
    * 获取当前登录用户
    * @return 
    */
        @RequestMapping("/loginUser")
        @ResponseBody
        public UserDetails getCurrentUser() {
            UserDetails userDetails = (UserDetails)SecurityContextHolder.getContext().getAuthentication().getPrincipal();
          return userDetails;
    }
    ```

  + 其他方式

    ```java
    /**
    * 获取当前登录用户
    *
    * @return */
        @RequestMapping("/loginUser1")
        @ResponseBody
        public UserDetails getCurrentUser() {
            UserDetails userDetails = (UserDetails)SecurityContextHolder.getContext().getAuthentication().getPrincipal();
            return userDetails;
    }
    /**
    * 获取当前登录用户 *
    * @return
    */
        @RequestMapping("/loginUser2")
        @ResponseBody
        public UserDetails getCurrentUser(Authentication authentication) {
            UserDetails userDetails = (UserDetails) authentication.getPrincipal();
            return userDetails;
        }
    /**
    * 获取当前登录用户 *
    * @return
    */
        @RequestMapping("/loginUser3")
        @ResponseBody
        public UserDetails getCurrentUser(@AuthenticationPrincipal UserDetails
    userDetails) {
            return userDetails;
    }
    ```

    

+ 记住我

  + 过滤链加上配置
  + 持久化token
  + 防止记住我生成的cookie被窃取可以在比较重要的接口中设置权限认证，来自记住我的session禁止请求该接口

+ 自定义登录成功和失败处理

+ 退出登录

##### 图形验证码认证

##### session管理

##### csrf防护机制

##### 跨域与CORS

#### springsecurity授权原理与实战

##### 授权简介

##### springSecurity授权

##### 	基于页面端标签的权限控制

