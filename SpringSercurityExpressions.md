1. Giới thiệu  
- trong bài viết này chúng ta cùng tìm hiểu về Sercurity trong SrpingBoot  
- Spring Security là một trong những core feature quan trọng của Spring Framework,
 nó giúp chúng ta phân quyền và xác thực người dùng trước khi cho phép họ truy cập vào các tài nguyên của chúng ta.
2. Maven Deppendencies  
để sử dụng Spring Sercurity ta cần thêm deppendencies spring security web vào file `pom.xml` trong dự án của mình  
```java
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-web</artifactId>
  <version>5.7.3</version>
</dependency>
```
version hiện tại trong bài viết này là 5.7. Bạn kiểm tra vesion mới nhất [tại đây](https://search.maven.org/search?q=a:spring-security-web) .  
lưu ý : trong bài viết này chúng ta sẽ chỉ dùng Spring Sercurity , trong thực tế một web hoàn chỉnh sẽ cần thêm spring-core và spring-context 
3. Cấu hình   
- Đầu tiên chúng ta sẽ kế thừa lớp WebSecurityConfigurerAdapter (Class WebSecurityConfigurerAdapter là một abstract class implement interface 
WebSecurityConfigurer định nghĩa các cấu hình mặc định cần thiết cho Spring Security
. Chúng ta cần sử dụng class này với annotation @EnableWebSecurity để enable hỗ trợ security cho ứng dụng web của chúng ta.)  
```java 
@Configuration
@EnableAutoConfiguration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityJavaConfig extends WebSecurityConfigurerAdapter {
    ...
}
```

- @EnableGlobalMethodSecurity : prePostEnabled => Cho phép phân quyền sử dụng annotation@PreAuthorize/@PostAuthorize - annotation của spring    
- @EnableAutoConfiguration là một annotation cho phép chúng ta cấu hình tự động. Điều đó có nghĩa là SpringBoot sẽ tìm kiếm
các bean tự động cấu hình trên classpath và tự động áp dụng chúng. lưu ý rằng @EnableAutoConfiguration luôn đi cùng @Configuration  
- Cầu hình XML :  
```java 
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans ...>
    <global-method-security pre-post-annotations="enabled"/>
</beans:beans>
```
4. Web Security Expressions  
- các biểu thức bảo mật trong Spring Sercurity :  
  - hasRole, hasAnyRole  
  - hasAuthority, hasAnyAuthority  
  - permitAll, denyAll  
  - isAnonymous, isRememberMe, isAuthenticated, isFullyAuthenticated  
  - principal, authentication  
  - hasPermission  

  4.1. hasRole, hasAnyRole  
  - dùng để phân quyền cho các URL và các method trong chương trình   
 ```java
  @Override
  protected void configure(final HttpSecurity http) throws Exception {
    ...
    .antMatchers("/auth/admin/*").hasRole("ADMIN")
    .antMatchers("/auth/*").hasAnyRole("ADMIN","USER")
    ...
  }
 ```
 trong ví dụ trên :   
- để truy cập vào các URL bắt đầu bằng `/auth` thì cần phải có quyền của ADMIN hoặc UER   
- đối với các URL bắt đầu bằng `/auth/admin` thì bắt buộc phải có quyền của ADMIN   
cấu hình XML có dạng như sau :   

```java
<http>
    <intercept-url pattern="/auth/admin/*" access="hasRole('ADMIN')"/>
    <intercept-url pattern="/auth/*" access="hasAnyRole('ADMIN','USER')"/>
</http>
```

  4.2. hasAuthority, hasAnyAuthority  
  -với Spring Security 4, tiền tố ‘ROLE_‘ được thêm tự động (nếu chưa có), hasAuthority (‘ROLE_ADMIN ') tương tự như hasRole (‘ ADMIN') vì tiền tố ‘ROLE_‘ được thêm tự động.  
 - phân quyền tài khoản :   
 ```java 
  @Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
      .withUser("user1").password(encoder().encode("user1Pass"))
      .authorities("USER")
      .and().withUser("admin").password(encoder().encode("adminPass"))
      .authorities("ADMIN");
}
  ```
  
  - quyền truy cập :  
  ```java 
  @Override
protected void configure(final HttpSecurity http) throws Exception {
    ...
    .antMatchers("/auth/admin/*").hasAuthority("ADMIN")
    .antMatchers("/auth/*").hasAnyAuthority("ADMIN", "USER")
    ...
}
```

- nếu bạn sử dụng Spring 5 thì cần thêm bean sau để băm password :  
```java 
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

- cấu hình XML :  
```java 
<authentication-manager>
    <authentication-provider>
        <user-service>
            <user name="user1" password="user1Pass" authorities="ROLE_USER"/>
            <user name="admin" password="adminPass" authorities="ROLE_ADMIN"/>
        </user-service>
    </authentication-provider>
</authentication-manager>
<bean name="passwordEncoder" 
  class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/>
  ```
  ```java 
  <http>
    <intercept-url pattern="/auth/admin/*" access="hasAuthority('ADMIN')"/>
    <intercept-url pattern="/auth/*" access="hasAnyAuthority('ADMIN','USER')"/>
</http>
```

  4.3. permitAll, denyAll  
  - hai annotations giúp chúng ta từ chối hoặc cho phép truy cập vào URL trong chương trình của mình  
  ```java 
  ...
.antMatchers("/*").permitAll()
...
```
-ở ví dụ trên đối với các URL bắt đầu `/` thì không cần quyền , user nào cũng có thể truy cập vào được (cả ẩn danh và đã đăng nhập hoặc chưa)  
```java 
...
.antMatchers("/*").denyAll()
...

```
- trong ví dụ trên ngược lại với `permitAll` thì `denyAll` sẽ từ chối quyền truy cập với tất cả uer khi vào URL bắt đầu với `/`  
-cấu hình XML :  
```java 
<http auto-config="true" use-expressions="true">
    <intercept-url access="permitAll" pattern="/*" /> <!-- Choose only one -->
    <intercept-url access="denyAll" pattern="/*" /> <!-- Choose only one -->
</http>
```
4.4. isAnonymous, isRememberMe, isAuthenticated, isFullyAuthenticated  
-dùng để kiểm tra trạng thái đăng nhập   
```java 
...
.antMatchers("/*").anonymous()
...
```
-> đối với với người dùng chưa đăng nhập sẽ truy cập được vào URL bắt đầu với `/`  
cấu hình XML :  
```java 
<http>
    <intercept-url pattern="/*" access="isAnonymous()"/>
</http>
```

- ngược lại nếu bạn muốn user phải đăng nhập mới có quyền truy cập thì sẽ dùng `authenticated`:  
 ```java 
...
.antMatchers("/*").authenticated()
...
```
cấu hình XML :  
```java 
<http>
    <intercept-url pattern="/*" access="isAuthenticated()"/>
</http>
```
- thông qua cookie Spring cung cấp `rememberMe` để ghi nhớ đăng nhập của user   
```java 
...
.antMatchers("/*").rememberMe()
...
```

cấu hình XML :  
```java 
<http>
    <intercept-url pattern="*" access="isRememberMe()"/>
</http>
```
- `fullyAuthenticated` sẽ yêu cầu xác thực  người dùng ngay cả khi vẫn còn đăng nhập ( được dùng khi user muốn thay đổi thông tin tài khoản, thông tin thanh toán ...)  
``` ...
.antMatchers("/*").fullyAuthenticated()
...
```
cấu hình XML :  
```java 
<http>
    <intercept-url pattern="*" access="isFullyAuthenticated()"/>
</http>
```
  4.5. principal, authentication  
  - lấy ra thông tin user đang đăng nhập trong SecurityContext ta đã lưu   
  4.6. hasPermission APIs  
 -  `hasPermission` là cầu nối obj với hệ thống ACL của Spring Security, giúp ta cấp quyền cho các 
  ( Access Control List (ACL) xác định quyền cho người dùng / vai trò cụ thể trên một domain object. )  
  
 - ví dụ :  Trong UserService mình có 2 phương thức lấy thông tin và xóa user, bạn có thể thấy trên mỗi method mình sẽ yêu cầu user hiện tại cần có quyền READ hoặc EDIT trên domain USER hay ko.  
 
  ```java 

public interface UserService {
    @PreAuthorize("hasPermission('USER', 'READ')")
    UserRegisterDto getUser(String userId);
    
    @PreAuthorize("hasPermission('USER', 'EDIT')")
    void deleteUser(String userName);
}
```
  - Chúng ta cũng cần nhớ cấu hình rõ ràng PermissionEvaluator trong ngữ cảnh ứng dụng của chúng ta, trong đó customInterfaceImplementation sẽ là lớp triển khai PermissionEvaluator:  
  ```java 
  <global-method-security pre-post-annotations="enabled">
    <expression-handler ref="expressionHandler"/>
</global-method-security>

<bean id="expressionHandler"
    class="org.springframework.security.access.expression
      .method.DefaultMethodSecurityExpressionHandler">
    <property name="permissionEvaluator" ref="customInterfaceImplementation"/>
</bean>
```

- Tất nhiên, chúng ta cũng có thể làm điều này với cấu hình Java:  
```java 
@Override
protected MethodSecurityExpressionHandler expressionHandler() {
    DefaultMethodSecurityExpressionHandler expressionHandler = 
      new DefaultMethodSecurityExpressionHandler();
    expressionHandler.setPermissionEvaluator(new CustomInterfaceImplementation());
    return expressionHandler;
}
```

5. Tổng kết   
Bài viết này là một giới thiệu toàn diện và hướng dẫn về các Biểu thức Spring Security .  

[link bài viết](https://www.baeldung.com/spring-security-expressions)  





