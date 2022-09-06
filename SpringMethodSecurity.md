1. Tổng quan 
- trong bài viết này ta sẽ dùng Spring Sercurity method 
2. Enabling Method Security  
Đầu tiên, để sử dụng Spring Method Security, chúng ta cần thêm dependency spring-security-config :
```java 
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
</dependency>
```
theo dõi phiên bản mới nhất [tại đây](https://search.maven.org/search?q=spring-security-config)
- nếu dự án của bạn dùng Spring Boot thì hãy thêm dependency sau :
```java 
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
theo dõi phiên bản mới nhất [tại đây](https://search.maven.org/search?q=spring-boot-starter-security)

- tiếp theo ta sẽ kế thừa GlobalMethodSecurityConfiguration (cho phép bảo mật toàn cầu )
```java 
@Configuration
@EnableGlobalMethodSecurity(
  prePostEnabled = true, 
  securedEnabled = true, 
  jsr250Enabled = true)
public class MethodSecurityConfig 
  extends GlobalMethodSecurityConfiguration {
}
```
-  `prePostEnabled` annotation kích hoạt chú thích trước / sau của Spring Security
-  `securedEnabled` annotation xác định xem chú thích @Secured có nên được bật hay không.
-  `jsr250Enabled` annotation cho phép sử dụng chú thích @RoleAllowed.

3. áp dụng Method Security
  3.1. @Secured Annotation
 - annotation @Secured được sử dụng để chỉ định các role của một phương thức. Vì vậy, người dùng chỉ có thể truy cập phương thức đó nếu có ít nhất một trong các role được chỉ định.
  ```java
  @Secured("ROLE_VIEWER")
public String getUsername() {
    SecurityContext securityContext = SecurityContextHolder.getContext();
    return securityContext.getAuthentication().getName();
}
```
-> Ở đây, chú thích @Secured (“ROLE_VIEWER”) xác định rằng chỉ những người dùng có ROLE_VIEWER mới có thể thực thi method getUsername.

- Chúng ta có thể xác định nhiều role trong chú thích @Secured:
```java 
@Secured({ "ROLE_VIEWER", "ROLE_EDITOR" })
public boolean isValidUsername(String username) {
    return userRoleRepository.isValidUsername(username);
}
```
-> Trong trường hợp này, nếu người dùng có ROLE_VIEWER hoặc ROLE_EDITOR, thì người dùng đó có thể gọi phương thức isValidUsername.
  3.2. @RolesAllowed Annotation
- annotation @RolesAllowed là annotation tương đương với JSR-250 trong annotation @Secured.  

- Về cơ bản, chúng ta có thể sử dụng @RolesAllowed tương tự như @Secured.  

- Chúng ta có thể viết lại phương thức getUsername và isValidUsername như sau :  

```java 
@RolesAllowed("ROLE_VIEWER")
public String getUsername2() {
    //...
}
    
@RolesAllowed({ "ROLE_VIEWER", "ROLE_EDITOR" })
public boolean isValidUsername2(String username) {
    //...
}
```
 3.3. @PreAuthorize and @PostAuthorize Annotations
  - annotation `@PreAuthorize` và `@PostAuthorize` đều cung cấp khả năng kiểm soát truy cập dựa trên biểu thức. Vì vậy, có thể được viết bằng SpEL (Spring Expression Language).
- annotation `@PreAuthorize` kiểm tra biểu thức đã cho trước khi gọi method, trong khi `@PostAuthorize` xác minh nó sau khi thực hiện method và có thể thay đổi kết quả.  
```java 
@PreAuthorize("hasRole('ROLE_VIEWER')")
public String getUsernameInUpperCase() {
    return getUsername().toUpperCase();
}
```
- @PreAuthorize (“hasRole (‘ ROLE_VIEWER ’)”) có tác dụng tương đương @Secured (“ROLE_VIEWER”) mà chúng ta đã sử dụng ở phần trên.

- Do đó, annotation @Secured ({“ROLE_VIEWER”, ”ROLE_EDITOR”}) có thể được thay thế bằng @PreAuthorize (“hasRole (‘ ROLE_VIEWER ’) hoặc hasRole (‘ ROLE_EDITOR ’)”):
```java 
@PreAuthorize("hasRole('ROLE_VIEWER') or hasRole('ROLE_EDITOR')")
public boolean isValidUsername3(String username) {
    //...
}
```
- Hơn nữa, chúng ta thực sự có thể sử dụng đối số method như một phần của biểu thức:
```java 
@PreAuthorize("#username == authentication.principal.username")
public String getMyRoles(String username) {
    //...
}
```
- Ở đây người dùng chỉ có thể gọi phương thức getMyRoles nếu tên người dùng đối số giống với tên người dùng của hiện tại.

- lưu ý : các biểu thức `@PreAuthorize` có thể được thay thế bằng các biểu thức `@PostAuthorize`.

- có thể viết lại getMyRoles như sau :
```java 
@PostAuthorize("#username == authentication.principal.username")
public String getMyRoles2(String username) {
    //...
}
```
- Tuy nhiên, trong ví dụ trước, ủy quyền sẽ bị trì hoãn sau khi thực hiện phương thức.

- Ngoài ra, `@PostAuthorize` cung cấp khả năng truy cập kết quả phương thức:
```java 
  @PostAuthorize
  ("returnObject.username == authentication.principal.nickName")
public CustomUser loadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```
- Ở đây, phương thức loadUserDetail sẽ chỉ thực thi thành công nếu tên người dùng của CustomUser được trả về bằng với nickName hiện tại.

  3.4. @PreFilter and @PostFilter Annotations 
  - Spring Security cung cấp `@PreFilter` để lọc đối số trước khi thực thi phương thức:
 ```java 
  @PreFilter("filterObject != authentication.principal.username")
public String joinUsernames(List<String> usernames) {
    return usernames.stream().collect(Collectors.joining(";"));
}
  ```
  - Trong ví dụ này đang lọc tất cả các tên người dùng ngoại trừ tên đã được xác thực.

- Ở đây sử dụng filterObject để đại diện cho đối tượng hiện tại trong method.

- Tuy nhiên, nếu phương thức có nhiều đối số, chúng ta cần sử dụng thuộc tính filterTarget để chỉ định đối số nào chúng ta muốn lọc:
 ```java 
  @PreFilter
  (value = "filterObject != authentication.principal.username",
  filterTarget = "usernames")
public String joinUsernamesAndRoles(
  List<String> usernames, List<String> roles) {
 
    return usernames.stream().collect(Collectors.joining(";")) 
      + ":" + roles.stream().collect(Collectors.joining(";"));
}
  ```
  - Ngoài ra, cũng có thể lọc trả về của một phương thức bằng cách sử dụng ` @PostFilter`:
  ```java 
  @PostFilter("filterObject != authentication.principal.username")
public List<String> getAllUsernamesExceptCurrent() {
    return userRoleRepository.getAllUsernames();
}
```
- Trong trường hợp này, tên filterObject đề cập đến đối tượng hiện tại trong method được trả về.

- Với ví dụ trên Spring Security sẽ lặp qua danh sách trả về và xóa bất kỳ giá trị nào khớp với tên người dùng của chính.

  3.5. Method Security Meta-Annotation
  - dùng để xác định một siêu chú thích bảo mật:
 ```java
  @IsViewer
public String getUsername4() {
    //...
}
```
- Security Meta-Annotation chúng bổ sung nhiều ngữ nghĩa hơn và giúp tách logic nghiệp vụ của chúng ta khỏi khung bảo mật.

  3.6. Security Annotation at the Class Level
  - Nếu sử dụng cùng một chú thích bảo mật cho mọi phương thức trong một lớp, chúng ta có thể cân nhắc đặt chú thích đó ở cấp class :
 ```java 
  @Service
@PreAuthorize("hasRole('ROLE_ADMIN')")
public class SystemService {

    public String getSystemYear(){
        //...
    }
 
    public String getSystemDate(){
        //...
    }
}
  ```
  - Trong ví dụ trên, quy tắc bảo mật hasRole (‘ROLE_ADMIN ') sẽ được áp dụng cho cả hai phương thức getSystemYear và getSystemDate.
  
    3.7. Multiple Security Annotations on a Method
    - có thể sử dụng nhiều chú thích bảo mật trên một method:
   ```java 
   @PreAuthorize("#username == authentication.principal.username")
@PostAuthorize("returnObject.username == authentication.principal.nickName")
public CustomUser securedLoadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```
- Bằng cách này, Spring sẽ xác minh ủy quyền cả trước và sau khi thực thi phương thức secureLoadUserDetail.

4. Important Considerations
- Có hai điểm chúng ta cần nhắc lại liên quan đến method security:
  - Theo mặc định, Spring AOP proxy được sử dụng để áp dụng phương pháp bảo mật. Nếu một phương thức bảo mật A được gọi bởi một phương thức khác trong cùng một lớp, thì bảo mật trong A sẽ bị bỏ qua hoàn toàn. Điều này có nghĩa là phương thức A sẽ thực thi mà không có bất kỳ kiểm tra bảo mật nào. Điều này cũng áp dụng cho các private methods.
  - Spring SecurityContext là chuỗi liên kết. Theo mặc định, ngữ cảnh bảo mật không được truyền tới các chuỗi con.
  
 5. Testing Method Security
 5.1. Cấu hình
Để kiểm tra Spring Security với JUnit, chúng ta cần dependency Spring-security-test:
```java 
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
</dependency>
```

- theo dõi phiên bản mới nhất [tại đây](https://search.maven.org/search?q=spring-security-test)

- Tiếp theo, hãy cấu hình Spring Integration test đơn giản bằng cách chạy và cấu hình ApplicationContext:
```java 
@RunWith(SpringRunner.class)
@ContextConfiguration
public class MethodSecurityIntegrationTest {
    // ...
}
```
  5.2. Testing Username and Roles 
- Bây giờ cấu hình của chúng ta đã sẵn sàng, hãy thử kiểm tra phương thức getUsername đã bảo mật bằng chú thích @Secured (“ROLE_VIEWER”):
```java
@Secured("ROLE_VIEWER")
public String getUsername() {
    SecurityContext securityContext = SecurityContextHolder.getContext();
    return securityContext.getAuthentication().getName();
}
```
- Vì chúng ta sử dụng chú thích @Secured ở đây, nên yêu cầu người dùng phải được xác thực để gọi phương thức. Nếu không sẽ nhận được AuthenticationCredentialsNotFoundException.

- Vì vậy, cần cung cấp một người dùng để kiểm tra phương pháp bảo mật.

```java 
@Test
@WithMockUser(username = "john", roles = { "VIEWER" })
public void givenRoleViewer_whenCallGetUsername_thenReturnUsername() {
    String userName = userRoleService.getUsername();
    
    assertEquals("john", userName);
}
```

- Chúng ta đã cung cấp một người dùng được xác thực có tên người dùng là john và có vai trò là ROLE_VIEWER. Nếu chúng ta không chỉ định tên người dùng hoặc vai trò, tên người dùng mặc định là username và role mặc định là ROLE_USER.

- Lưu ý rằng không cần thiết phải thêm tiền tố ROLE_ vào đây vì Spring Security sẽ tự động thêm tiền tố đó.

- Ví dụ, hãy khai báo một phương thức getUsernameInLowerCase:
```java 
@PreAuthorize("hasAuthority('SYS_ADMIN')")
public String getUsernameLC(){
    return getUsername().toLowerCase();
}
```
Chúng ta có thể kiểm tra như sau :
```java
public void givenAuthoritySysAdmin_whenCallGetUsernameLC_thenReturnUsername() {
    String username = userRoleService.getUsernameInLowerCase();

    assertEquals("john", username);
}
```
-  nếu chúng ta muốn sử dụng cùng một người dùng cho nhiều trường hợp thử nghiệm, chúng ta có thể khai báo `@WithMockUser` :
```java
@RunWith(SpringRunner.class)
@ContextConfiguration
@WithMockUser(username = "john", roles = { "VIEWER" })
public class MockUserAtClassLevelIntegrationTest {
    //...
}
```
- Nếu muốn chạy thử nghiệm của mình với tư cách là người dùng ẩn danh, chúng ta có thể sử dụng `@WithAnonymousUser`:
```java
@Test(expected = AccessDeniedException.class)
@WithAnonymousUser
public void givenAnomynousUser_whenCallGetUsername_thenAccessDenied() {
    userRoleService.getUsername();
}
```
- Trong ví dụ trên, trả về AccessDeniedException vì người dùng ẩn danh không được cấp vai trò ROLE_VIEWER hoặc quyền SYS_ADMIN.
 
 5.3. Testing With a Custom UserDetailsService
- Đối với hầu hết các chương trình , thông thường sử dụng một class tùy chỉnh làm cơ sở xác thực. Trong trường hợp này, lớp tùy chỉnh cần triển khai giao diện org.springframework.security.core.userdetails.UserDetails.

-Chúng ta khai báo một lớp CustomUser extends UserDetails hiện có, là org.springframework.security.core.userdetails.
```java 
public class CustomUser extends User {
    private String nickName;
    // getter and setter
}
```
- Hãy xem lại ví dụ với `@PostAuthorize` trong Phần 3:
```java
@PostAuthorize("returnObject.username == authentication.principal.nickName")
public CustomUser loadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```
- Trong trường hợp này, method sẽ chỉ thực thi thành công nếu tên người dùng của CustomUser được trả về bằng với nickName hiện tại.

- Nếu muốn kiểm tra phương pháp đó, chúng ta có thể triển khai UserDetailsService dùng CustomUser dựa trên tên người dùng:

```java 
@Test
@WithUserDetails(
  value = "john", 
  userDetailsServiceBeanName = "userDetailService")
public void whenJohn_callLoadUserDetail_thenOK() {
 
    CustomUser user = userService.loadUserDetail("jane");

    assertEquals("jane", user.getNickName());
}
```
- Ở đây, `@WithUserDetails` nói rằng chúng ta sẽ sử dụng một UserDetailsService để khởi tạo người dùng đã xác thực, tham chiếu bởi thuộc tính userDetailsServiceBeanName.
- 
- chúng ta cũng có thể dùng `@WithUserDetails` ở cấp lớp, tương tự như những gì chúng ta đã làm với `@WithMockUser`.
  5.4. Testing With Meta Annotations
  - Chúng ta thường thấy mình sử dụng đi sử dụng lại cùng một user/role trong các ví dụ khác nhau nên ta sẽ tạo một meta-annotation.

  - Xem lại ví dụ trước @WithMockUser (username = ”john”, role = {“VIEWER”}), chúng ta có thể khai báo một meta-annotation:
 ```java
  @Retention(RetentionPolicy.RUNTIME)
@WithMockUser(value = "john", roles = "VIEWER")
public @interface WithMockJohnViewer { }
 ```
- Sau đó, chúng ta chỉ cần sử dụng @WithMockJohnViewer trong vdụdụ của mình:
```java
@Test
@WithMockJohnViewer
public void givenMockedJohnViewer_whenCallGetUsername_thenReturnUsername() {
    String userName = userRoleService.getUsername();

    assertEquals("john", userName);
}
```
- Tương tự như vậy, chúng ta có thể sử dụng meta-annotation để tạo người dùng cụ thể cho domain-specific bằng cách sử dụng @WithUserDetails.

 6. Kết luận
- Trong bài viết này, chúng tôi đã khám phá các cách dùng khác nhau để sử dụng Method Security trong Spring Security.
 [link bài dịch ](https://www.baeldung.com/spring-security-method-security) 
  
