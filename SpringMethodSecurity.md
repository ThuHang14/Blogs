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

3. Applying Method Security
  3.1. @Secured Annotation
 - annotation @Secured được sử dụng để chỉ định danh sách các vai trò trên một phương thức. Vì vậy, người dùng chỉ có thể truy cập phương thức đó nếu cô ấy có ít nhất một trong các vai trò được chỉ định.
  ```java
  @Secured("ROLE_VIEWER")
public String getUsername() {
    SecurityContext securityContext = SecurityContextHolder.getContext();
    return securityContext.getAuthentication().getName();
}
```
-> Ở đây, chú thích @Secured (“ROLE_VIEWER”) xác định rằng chỉ những người dùng có vai trò ROLE_VIEWER mới có thể thực thi phương thức getUsername.

- Bên cạnh đó, chúng ta có thể xác định danh sách các vai trò trong chú thích @Secured:
```java 
@Secured({ "ROLE_VIEWER", "ROLE_EDITOR" })
public boolean isValidUsername(String username) {
    return userRoleRepository.isValidUsername(username);
}
```
-> Trong trường hợp này, cấu hình nói rằng nếu người dùng có ROLE_VIEWER hoặc ROLE_EDITOR, thì người dùng đó có thể gọi phương thức isValidUsername.
  3.2. @RolesAllowed Annotation
-  Chú thích @RolesAllowed là chú thích tương đương của JSR-250 của chú thích @Secured.  

- Về cơ bản, chúng ta có thể sử dụng chú thích @RolesAllowed theo cách tương tự như @Secured.  

- Bằng cách này, chúng ta có thể xác định lại phương thức getUsername và isValidUsername:  

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
- Tương tự, chỉ người dùng có vai trò ROLE_VIEWER mới có thể thực thi getUsername2.

- Một lần nữa, người dùng chỉ có thể gọi isValidUsername2 nếu họ có ít nhất một trong các vai trò ROLE_VIEWER hoặc ROLER_EDITOR.
  3.3. @PreAuthorize and @PostAuthorize Annotations
  - Cả chú thích @PreAuthorize và @PostAuthorize đều cung cấp khả năng kiểm soát truy cập dựa trên biểu thức. Vì vậy, các vị từ có thể được viết bằng SpEL (Spring Expression Language).
- Chú thích @PreAuthorize kiểm tra biểu thức đã cho trước khi nhập phương thức, trong khi chú thích @PostAuthorize xác minh nó sau khi thực hiện phương thức và có thể thay đổi kết quả.  
```java 
@PreAuthorize("hasRole('ROLE_VIEWER')")
public String getUsernameInUpperCase() {
    return getUsername().toUpperCase();
}
```
- @PreAuthorize (“hasRole (‘ ROLE_VIEWER ’)”) có cùng ý nghĩa với @Secured (“ROLE_VIEWER”) mà chúng ta đã sử dụng trong phần trước. Vui lòng khám phá thêm chi tiết các biểu thức bảo mật trong các bài viết trước.

- Do đó, chú thích @Secured ({“ROLE_VIEWER”, ”ROLE_EDITOR”}) có thể được thay thế bằng @PreAuthorize (“hasRole (‘ ROLE_VIEWER ’) hoặc hasRole (‘ ROLE_EDITOR ’)”):
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
- Ở đây người dùng chỉ có thể gọi phương thức getMyRoles nếu giá trị của tên người dùng đối số giống với tên người dùng của chính hiện tại.

- Cần lưu ý rằng các biểu thức @PreAuthorize có thể được thay thế bằng các biểu thức @PostAuthorize.

- Hãy viết lại getMyRoles:
```java 
@PostAuthorize("#username == authentication.principal.username")
public String getMyRoles2(String username) {
    //...
}
```
- Tuy nhiên, trong ví dụ trước, ủy quyền sẽ bị trì hoãn sau khi thực hiện phương thức đích.

- Ngoài ra, chú thích @PostAuthorize cung cấp khả năng truy cập kết quả phương thức:
```java 
  @PostAuthorize
  ("returnObject.username == authentication.principal.nickName")
public CustomUser loadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```
- Ở đây, phương thức loadUserDetail sẽ chỉ thực thi thành công nếu tên người dùng của CustomUser được trả về bằng với biệt hiệu của hiệu trưởng xác thực hiện tại.

- Trong phần này, chúng tôi chủ yếu sử dụng các biểu thức Spring đơn giản. Đối với các tình huống phức tạp hơn, chúng tôi có thể tạo các biểu thức bảo mật tùy chỉnh
  3.4. @PreFilter and @PostFilter Annotations 
  - Spring Security cung cấp chú thích @PreFilter để lọc đối số tập hợp trước khi thực thi phương thức:
 ```java 
  @PreFilter("filterObject != authentication.principal.username")
public String joinUsernames(List<String> usernames) {
    return usernames.stream().collect(Collectors.joining(";"));
}
  ```
  - Trong ví dụ này, chúng tôi đang kết hợp tất cả các tên người dùng ngoại trừ tên đã được xác thực.

- Ở đây, trong biểu thức của chúng tôi, chúng tôi sử dụng tên filterObject để đại diện cho đối tượng hiện tại trong bộ sưu tập.

- Tuy nhiên, nếu phương thức có nhiều đối số là kiểu tập hợp, chúng ta cần sử dụng thuộc tính filterTarget để chỉ định đối số nào chúng ta muốn lọc:
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
  - Ngoài ra, chúng tôi cũng có thể lọc bộ sưu tập trả về của một phương thức bằng cách sử dụng chú thích @PostFilter:
  ```java 
  @PostFilter("filterObject != authentication.principal.username")
public List<String> getAllUsernamesExceptCurrent() {
    return userRoleRepository.getAllUsernames();
}
```
- Trong trường hợp này, tên filterObject đề cập đến đối tượng hiện tại trong bộ sưu tập được trả về.

- Với cấu hình đó, Spring Security sẽ lặp qua danh sách trả về và xóa bất kỳ giá trị nào khớp với tên người dùng của chính.

  3.5. Method Security Meta-Annotation
  - Chúng tôi thường thấy mình trong một tình huống mà chúng tôi bảo vệ các phương pháp khác nhau bằng cách sử dụng cùng một cấu hình bảo mật.

  - Trong trường hợp này, chúng tôi có thể xác định một siêu chú thích bảo mật:
 ```java
  @IsViewer
public String getUsername4() {
    //...
}
```
- Chú thích meta bảo mật là một ý tưởng tuyệt vời vì chúng bổ sung nhiều ngữ nghĩa hơn và tách logic nghiệp vụ của chúng ta khỏi khung bảo mật.
  3.6. Security Annotation at the Class Level
  - Nếu chúng tôi thấy mình đang sử dụng cùng một chú thích bảo mật cho mọi phương thức trong một lớp, chúng tôi có thể cân nhắc đặt chú thích đó ở cấp lớp:
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
    - Chúng tôi cũng có thể sử dụng nhiều chú thích bảo mật trên một phương pháp:
   ```java 
   @PreAuthorize("#username == authentication.principal.username")
@PostAuthorize("returnObject.username == authentication.principal.nickName")
public CustomUser securedLoadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```
- Bằng cách này, Spring sẽ xác minh ủy quyền cả trước và sau khi thực thi phương thức secureLoadUserDetail.
4. Important Considerations
- Có hai điểm chúng tôi muốn nhắc lại liên quan đến method security:
  - Theo mặc định, Spring AOP proxy được sử dụng để áp dụng phương pháp bảo mật. Nếu một phương thức bảo mật A được gọi bởi một phương thức khác trong cùng một lớp, thì bảo mật trong A sẽ bị bỏ qua hoàn toàn. Điều này có nghĩa là phương thức A sẽ thực thi mà không có bất kỳ kiểm tra bảo mật nào. Điều tương tự cũng áp dụng cho các phương pháp riêng tư.
  - Spring SecurityContext là chuỗi liên kết. Theo mặc định, ngữ cảnh bảo mật không được truyền tới các chuỗi con.
 5. Testing Method Security
 5.1. Cấu hình
Để kiểm tra Spring Security với JUnit, chúng ta cần sự phụ thuộc của Spring-security-test:
```java 
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
</dependency>
```

- Chúng tôi không cần chỉ định phiên bản phụ thuộc vì chúng tôi đang sử dụng plugin Spring Boot. Chúng tôi có thể tìm thấy phiên bản mới nhất của sự phụ thuộc này trên Maven Central.

- Tiếp theo, hãy định cấu hình một bài kiểm tra Tích hợp mùa xuân đơn giản bằng cách chỉ định trình chạy và cấu hình ApplicationContext:
```java 
@RunWith(SpringRunner.class)
@ContextConfiguration
public class MethodSecurityIntegrationTest {
    // ...
}
```
  5.2. Testing Username and Roles 
- Bây giờ cấu hình của chúng tôi đã sẵn sàng, hãy thử kiểm tra phương thức getUsername mà chúng tôi đã bảo mật bằng chú thích @Secured (“ROLE_VIEWER”):
```java
@Secured("ROLE_VIEWER")
public String getUsername() {
    SecurityContext securityContext = SecurityContextHolder.getContext();
    return securityContext.getAuthentication().getName();
}
```
- Vì chúng tôi sử dụng chú thích @Secured ở đây, nó yêu cầu người dùng phải được xác thực để gọi phương thức. Nếu không, chúng tôi sẽ nhận được AuthenticationCredentialsNotFoundException.

- Vì vậy, chúng tôi cần cung cấp một người dùng để kiểm tra phương pháp bảo mật của chúng tôi.

- Để đạt được điều này, chúng tôi trang trí phương pháp thử nghiệm với @WithMockUser và cung cấp người dùng và các vai trò:
```java 
@Test
@WithMockUser(username = "john", roles = { "VIEWER" })
public void givenRoleViewer_whenCallGetUsername_thenReturnUsername() {
    String userName = userRoleService.getUsername();
    
    assertEquals("john", userName);
}
```

- Chúng tôi đã cung cấp một người dùng được xác thực có tên người dùng là john và có vai trò là ROLE_VIEWER. Nếu chúng tôi không chỉ định tên người dùng hoặc vai trò, tên người dùng mặc định là người dùng và vai trò mặc định là ROLE_USER.

- Lưu ý rằng không cần thiết phải thêm tiền tố ROLE_ vào đây vì Spring Security sẽ tự động thêm tiền tố đó.

- Nếu chúng ta không muốn có tiền tố đó, chúng ta có thể xem xét sử dụng quyền hạn thay vì vai trò.

- Ví dụ, hãy khai báo một phương thức getUsernameInLowerCase:
```java 
@PreAuthorize("hasAuthority('SYS_ADMIN')")
public String getUsernameLC(){
    return getUsername().toLowerCase();
}
```
Chúng tôi có thể kiểm tra điều đó bằng cách sử dụng các cơ quan chức năng:
```java
public void givenAuthoritySysAdmin_whenCallGetUsernameLC_thenReturnUsername() {
    String username = userRoleService.getUsernameInLowerCase();

    assertEquals("john", username);
}
```
Thuận tiện, nếu chúng ta muốn sử dụng cùng một người dùng cho nhiều trường hợp thử nghiệm, chúng ta có thể khai báo chú thích @WithMockUser tại lớp thử nghiệm:
```java
@RunWith(SpringRunner.class)
@ContextConfiguration
@WithMockUser(username = "john", roles = { "VIEWER" })
public class MockUserAtClassLevelIntegrationTest {
    //...
}
```
- Nếu chúng tôi muốn chạy thử nghiệm của mình với tư cách là người dùng ẩn danh, chúng tôi có thể sử dụng chú thích @WithAnonymousUser:
```java
@Test(expected = AccessDeniedException.class)
@WithAnonymousUser
public void givenAnomynousUser_whenCallGetUsername_thenAccessDenied() {
    userRoleService.getUsername();
}
```
- Trong ví dụ trên, chúng tôi mong đợi một AccessDeniedException vì người dùng ẩn danh không được cấp vai trò ROLE_VIEWER hoặc quyền SYS_ADMIN.
  5.3. Testing With a Custom UserDetailsService
- Đối với hầu hết các ứng dụng, thông thường sử dụng một lớp tùy chỉnh làm cơ sở xác thực. Trong trường hợp này, lớp tùy chỉnh cần triển khai giao diện org.springframework.security.core.userdetails.UserDetails.

- Trong bài viết này, chúng tôi khai báo một lớp CustomUser mở rộng việc triển khai UserDetails hiện có, là org.springframework.security.core.userdetails.
```java 
public class CustomUser extends User {
    private String nickName;
    // getter and setter
}
```
- Hãy xem lại ví dụ với chú thích @PostAuthorize trong Phần 3:
```java
@PostAuthorize("returnObject.username == authentication.principal.nickName")
public CustomUser loadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```
- Trong trường hợp này, phương thức sẽ chỉ thực thi thành công nếu tên người dùng của CustomUser được trả về bằng với biệt hiệu của hiệu trưởng xác thực hiện tại.

- Nếu chúng tôi muốn kiểm tra phương pháp đó, chúng tôi có thể cung cấp triển khai UserDetailsService có thể tải CustomUser của chúng tôi dựa trên tên người dùng:

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
- Ở đây, chú thích @WithUserDetails nói rằng chúng tôi sẽ sử dụng một UserDetailsService để khởi tạo người dùng đã xác thực của chúng tôi. Dịch vụ được tham chiếu bởi thuộc tính userDetailsServiceBeanName. UserDetailsService này có thể là một triển khai thực hoặc một giả cho mục đích thử nghiệm.
- Ngoài ra, dịch vụ sẽ sử dụng giá trị của giá trị thuộc tính làm tên người dùng để tải UserDetails.
- Thuận tiện, chúng ta cũng có thể trang trí bằng chú thích @WithUserDetails ở cấp lớp, tương tự như những gì chúng ta đã làm với chú thích @WithMockUser.
  5.4. Testing With Meta Annotations
  - Chúng tôi thường thấy mình sử dụng đi sử dụng lại cùng một người dùng / vai trò trong các thử nghiệm khác nhau.

  - Đối với những tình huống này, thật tiện lợi để tạo một chú thích meta.

  - Xem lại ví dụ trước @WithMockUser (username = ”john”, role = {“VIEWER”}), chúng ta có thể khai báo một meta-annotation:
 ```java
  @Retention(RetentionPolicy.RUNTIME)
@WithMockUser(value = "john", roles = "VIEWER")
public @interface WithMockJohnViewer { }
 ```
- Sau đó, chúng tôi chỉ cần sử dụng @WithMockJohnViewer trong thử nghiệm của mình:
```java
@Test
@WithMockJohnViewer
public void givenMockedJohnViewer_whenCallGetUsername_thenReturnUsername() {
    String userName = userRoleService.getUsername();

    assertEquals("john", userName);
}
```
- Tương tự như vậy, chúng tôi có thể sử dụng chú thích meta để tạo người dùng cụ thể cho miền bằng cách sử dụng @WithUserDetails.

 6. Kết luận
- Trong bài viết này, chúng tôi đã khám phá các tùy chọn khác nhau để sử dụng Method Security trong Spring Security.

- Chúng tôi cũng đã trải qua một số kỹ thuật để dễ dàng kiểm tra tính bảo mật của phương pháp và học cách sử dụng lại những người dùng bị bắt chước trong các thử nghiệm khác nhau.
 [link bài dịch ](https://www.baeldung.com/spring-security-method-security) 
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
