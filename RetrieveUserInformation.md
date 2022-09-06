1. Mở đầu
- bài viết này sẽ chỉ ra cách lấy thông tin người dùng trong Spring Security.
2. lấy thông tin người dùng trong Bean
- Cách đơn giản nhất để lấy thông tin người dùng đã được xác thực là thông qua SecurityContextHolder:
```java 
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
String currentPrincipalName = authentication.getName();
```
- hoặc bạn có thể kiểm tra xem người dùng đã xác thực hay không trước khi lấy ra thông tin người dùng như sau :
```java 
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
if (!(authentication instanceof AnonymousAuthenticationToken)) {
    String currentUserName = authentication.getName();
    return currentUserName;
}
```
- Tuy nhiên có vài nhược điểm khi dùng  này và khả năng kiểm tra không cao, chúng ta sẽ tìm hiểu thêm các giải pháp thay thế cho cách này ở phần dưới .
3. lấy thông tin người dùng trong Controller
- Chúng ta có thể xác định trực tiếp hàm chính dưới dạng đối số phương thức :
```java 
@Controller
public class SecurityController {

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserName(Principal principal) {
        return principal.getName();
    }
}
```
- Ngoài ra, có thể sử dụng mã thông báo xác thực:
```java 
@Controller
public class SecurityController {

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserName(Authentication authentication) {
        return authentication.getName();
    }
}
```
- API của lớp Authentication rất nhiều framework linh hoạt. Vì lý do này, mã chính của Spring Security chỉ có thể được truy xuất dưới dạng một Đối tượng và cần được ép kiểu đến UserDetails chính xác:
```java 
UserDetails userDetails = (UserDetails) authentication.getPrincipal();
System.out.println("User has authorities: " + userDetails.getAuthorities());
```
- Cuối cùng là gọi đến HTTP request:
```java 
@Controller
public class GetUserWithHTTPServletRequestController {

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserNameSimple(HttpServletRequest request) {
        Principal principal = request.getUserPrincipal();
        return principal.getName();
    }
}
```

4. lấy thông ra thông tin người dùng thông qua giao diện tùy chỉnh

- để có thể truy xuất xác thực ở mọi nơi, không chỉ trong `@Controller bean`, chúng ta cần Override Authentication lại:
```java 
public interface IAuthenticationFacade {
    Authentication getAuthentication();
}
@Component
public class AuthenticationFacade implements IAuthenticationFacade {

    @Override
    public Authentication getAuthentication() {
        return SecurityContextHolder.getContext().getAuthentication();
    }
}
```

- giờ chỉ cần Autowired lại là ta có thể gọi ra ở bất cứ đâu :
```java 
@Controller
public class GetUserWithCustomInterfaceController {
    @Autowired
    private IAuthenticationFacade authenticationFacade;

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserNameSimple() {
        Authentication authentication = authenticationFacade.getAuthentication();
        return authentication.getName();
    }
}
```

5. lấy ra thông tin người dùng trong JSP

- để lấy ra thông tin người dùng trong JSP ta sẽ dùng Spring Security Taglib
- Đầu tiên, chúng ta cần xác định thẻ trong Page :
```java 
<%@ taglib prefix="security" uri="http://www.springframework.org/security/tags" %>
```
- Tiếp theo, chúng ta có thể gọi đến principal :
```java 
<security:authorize access="isAuthenticated()">
    authenticated as <security:authentication property="principal.username" /> 
</security:authorize>
```
6. lấy ra thông tin người dùng  Thymeleaf
- Đầu tiên, chúng ta cần dependency thymeleaf-spring5 và thymeleaf-extras-springecurity5 để tích hợp Thymeleaf với Spring Security:

```java
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
</dependency>
```
- Bây giờ chúng ta có thể tham chiếu đến phần tử chính trong trang HTML bằng cách sử dụng thuộc tính sec:authorize :
```java 
<html xmlns:th="https://www.thymeleaf.org" 
  xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
<body>
    <div sec:authorize="isAuthenticated()">
      Authenticated as <span sec:authentication="name"></span></div>
</body>
</html>
```

7. Kết luận
- Bài viết này đã chỉ ra cách lấy thông tin người dùng trong ứng dụng Spring, bắt đầu với cơ chế truy cập tĩnh phổ biến trong bean, sau đó là một số cách tốt hơn để lấy thông tin người dùng.
 [link bài viết](https://www.baeldung.com/get-user-in-spring-security)
































