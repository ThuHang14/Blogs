
1. Giới thiệu
- Hướng dẫn này sẽ tập trung vào Đăng nhập với Spring Security. 
- Chúng ta sẽ xây dựng dựa trên ví dụ Spring MVC 
2. The Maven Dependencies
- thêm dependencies 
```java 
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.3.3.RELEASE</version>
</dependency>
```
- lưu ý : theo dõi version mới nhất
3. Cấu hình 
-Hãy bắt đầu bằng cách tạo một lớp cấu hình Spring Security kế thừa WebSecurityConfigurerAdapter.

- thêm `@EnableWebSecurity` chúng ta sẽ nhận được hỗ trợ tích hợp Spring Security và MVC:
```java 
@Configuration
@EnableWebSecurity
public class SecSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(final AuthenticationManagerBuilder auth) throws Exception {
        // authentication manager (see below)
    }

    @Override
    protected void configure(final HttpSecurity http) throws Exception {
        // http builder configurations for authorize requests and form login (see below)
    }
}
```
3.1. quản lý xác thực
```java 
protected void configure(final AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
        .withUser("user1").password(passwordEncoder().encode("user1Pass")).roles("USER")
        .and()
        .withUser("user2").password(passwordEncoder().encode("user2Pass")).roles("USER")
        .and()
        .withUser("admin").password(passwordEncoder().encode("adminPass")).roles("ADMIN");
}
```
- Bắt đầu với Spring 5, chúng ta cũng phải xác định một bộ mã hóa mật khẩu. Trong ví dụ này chúng ta sẽ sử dụng BCryptPasswordEncoder:
```java 
@Bean 
public PasswordEncoder passwordEncoder() { 
    return new BCryptPasswordEncoder(); 
}
```
3.2. phân quyền

```java 
@Override
protected void configure(final HttpSecurity http) throws Exception {
    http
      .csrf().disable()
      .authorizeRequests()
      .antMatchers("/admin/**").hasRole("ADMIN")
      .antMatchers("/anonymous*").anonymous()
      .antMatchers("/login*").permitAll()
      .anyRequest().authenticated()
      .and()
      // ...
}
```
- Ở đây chúng ta cho phép tất cả truy cập được vào `/login` kể cả ẩn danh ( anonymous )
- Chúng ta sẽ hạn chế truy cập đối với các URL bắt đầu `/admin/` bằng role ADMIN
- Lưu ý rằng thứ tự của các phần tử `antMatchers()` là quan trọng, các quy tắc cụ thể hơn cần phải gọi trước, sau đó là các quy tắc chung hơn
  3.3. Cấu hình form đăng nhập

- Tiếp theo, chúng ta sẽ mở rộng cấu hình trên để đăng nhập và đăng xuất theo form :
```java 
@Override
protected void configure(final HttpSecurity http) throws Exception {
    http
      // ...
      .and()
      .formLogin()
      .loginPage("/login.html")
      .loginProcessingUrl("/perform_login")
      .defaultSuccessUrl("/homepage.html", true)
      .failureUrl("/login.html?error=true")
      .failureHandler(authenticationFailureHandler())
      .and()
      .logout()
      .logoutUrl("/perform_logout")
      .deleteCookies("JSESSIONID")
      .logoutSuccessHandler(logoutSuccessHandler());
}
```
  - loginPage() : trang đăng nhập tùy chỉnh
  - loginProcessingUrl () - URL để gửi tên người dùng và mật khẩu đến
  - defaultSuccessUrl () - trang chuyển hướng sau khi đăng nhập thành công
  - failUrl () - trang chuyển hướng sau khi đăng nhập không thành công
  - logoutUrl () - đăng xuất tùy chỉnh
4. Thêm Spring Security vào Web

- Để sử dụng cấu hình Spring Security được xác định ở trên, chúng ta cần gọi vào ứng dụng web.

- Chúng ta sẽ sử dụng WebApplicationInitializer, vì vậy chúng tôi không cần cung cấp bất kỳ web.xml nào:

```java 
public class AppInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext sc) {

        AnnotationConfigWebApplicationContext root = new AnnotationConfigWebApplicationContext();
        root.register(SecSecurityConfig.class);

        sc.addListener(new ContextLoaderListener(root));

        sc.addFilter("securityFilter", new DelegatingFilterProxy("springSecurityFilterChain"))
          .addMappingForUrlPatterns(null, false, "/*");
    }
}
```
5. Cấu hình XML 

- chúng ta cần nhập tệp cấu hình XML qua lớp Java @Configuration:
```java 
@Configuration
@ImportResource({ "classpath:webSecurityConfig.xml" })
public class SecSecurityConfig {
   public SecSecurityConfig() {
      super();
   }
}
```
- Và cấu hình Spring Security XML, webSecurityConfig.xml:
```java 
<http use-expressions="true">
    <intercept-url pattern="/login*" access="isAnonymous()" />
    <intercept-url pattern="/**" access="isAuthenticated()"/>

    <form-login login-page='/login.html' 
      default-target-url="/homepage.html" 
      authentication-failure-url="/login.html?error=true" />
    <logout logout-success-url="/login.html" />
</http>

<authentication-manager>
    <authentication-provider>
        <user-service>
            <user name="user1" password="user1Pass" authorities="ROLE_USER" />
        </user-service>
        <password-encoder ref="encoder" />
    </authentication-provider>
</authentication-manager>

<beans:bean id="encoder" 
  class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder">
</beans:bean>
```
6. web.xml
```java
<display-name>Spring Secured Application</display-name>

<!-- Spring MVC -->
<!-- ... -->

<!-- Spring Security -->
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
- Bộ lọc DelegateFilterProxy - chỉ cần ủy quyền cho một bean do Spring quản lý - FilterChainProxy - mà bản thân nó có thể hưởng lợi từ việc quản lý toàn bộ vòng đời của Spring bean .
7. Login Form
- Trang biểu mẫu đăng nhập sẽ được  với Spring MVC bằng cách sử dụng cơ chế đơn giản để ánh xạ views names to URLs. 
```java
registry.addViewController("/login.html");
```
- với `login.jsp` :
```java 
<html>
<head></head>
<body>
   <h1>Login</h1>
   <form name='f' action="login" method='POST'>
      <table>
         <tr>
            <td>User:</td>
            <td><input type='text' name='username' value=''></td>
         </tr>
         <tr>
            <td>Password:</td>
            <td><input type='password' name='password' /></td>
         </tr>
         <tr>
            <td><input name="submit" type="submit" value="submit" /></td>
         </tr>
      </table>
  </form>
</body>
</html>
```
- các thành phần trong Spring Login form :
  - login - URL nơi biểu mẫu được POST để kích hoạt quá trình xác thực
  - username – tên đăng nhập 
  - password – mật khẩu

8. Cấu hình thêm Spring Login 
```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.formLogin()
      .loginPage("/login.html")
      .loginProcessingUrl("/perform_login")
      .defaultSuccessUrl("/homepage.html",true)
      .failureUrl("/login.html?error=true")
}
```
- Hoặc cấu hình XML tương ứng:
```java 
<form-login 
  login-page='/login.html' 
  login-processing-url="/perform_login" 
  default-target-url="/homepage.html"
  authentication-failure-url="/login.html?error=true" 
  always-use-default-target="true"/>
```
   8.1. Trang đăng nhập
    - Tiếp theo, chúng ta sẽ định cấu hình trang đăng nhập tùy chỉnh bằng phương thức loginPage ():
```java
http.formLogin()
  .loginPage("/login.html")
  ```
  - Tương tự, chúng ta có thể sử dụng cấu hình XML:
  ```java 
  login-page='/login.html'
  ```
  - Nếu chúng ta không chỉ định điều này, Spring Security sẽ tạo một form đăng nhập có sẵn tại URL `/login`.
  8.2. POST URL 
  - Chúng ta có thể sử dụng phương thức loginProcessingUrl để ghi đè URL `/login` :
  
```java
http.formLogin()
  .loginProcessingUrl("/perform_login")
  ```
  - Chúng ta cũng có thể sử dụng cấu hình XML:
  ```java
  login-processing-url="/perform_login"
  ```
  - Bằng cách ghi đè URL mặc định này, giúp Chương trình được bảo mật hơn , tránh bị dò dỉ , đánh cắp thông tin.
  
  8.3. Chuyển hướng đăng nhập thành công
- Sau khi đăng nhập thành công, chúng ta sẽ được chuyển đến một trang mà mặc định là thư mục gốc của ứng dụng web.
- Chúng ta có thể ghi đè điều này thông qua phương thức defaultSuccessUrl ():
```java 
http.formLogin()
  .defaultSuccessUrl("/homepage.html")
  ```
  - Hoặc với cấu hình XML:
  ```java
  default-target-url="/homepage.html"
  ```
  - Nếu thuộc tính always-use-default-target được đặt thành true, thì người dùng luôn được chuyển hướng đến trang này. Nếu thuộc tính đó được đặt thành false, thì người dùng sẽ được chuyển hướng đến trang trước mà họ muốn truy cập trước khi được nhắc xác thực.
    8.4 Chuyển hướng đăng nhập không thành công
    - Tương tự như Trang Đăng nhập, Trang Lỗi Đăng nhập được Spring Security tự động tạo tại lỗi `/login?` Theo mặc định.

    - Để ghi đè điều này, chúng ta có thể sử dụng phương thức failUrl ():

```java
http.formLogin()
  .failureUrl("/login.html?error=true")
  ```
  - Hoặc với XML:
  ```java 
  authentication-failure-url="/login.html?error=true"
  ```
 9. Kết luận
- Trong Ví dụ Spring Login Example, chúng ta đã cấu hình một quy trình xác thực đơn giản. Chúng tôi cũng đã làm về form đăng nhập , Cấu hình bảo mật và một số tùy chỉnh 
[link bài dịch](https://www.baeldung.com/spring-security-login)

