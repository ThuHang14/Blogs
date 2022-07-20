### Truy vấn dữ liệu Spring JPA
- Spring Data là một module của Spring Framework. Mục đích của Spring Data JPA là giảm thiểu việc thực hiện quá nhiều bước để có thể implement được JPA. 
Spring Data JPA là một phần của Spirng Data và có hỗ trợ Hibernate 
- Ở bài viết này bạn sẽ Custom Query , define thông qua chú thích @Query
-  với @Query bạn có thể khai báo câu query cho các method trong repository , đặt thuộc tính `nativeQuery = true` để dùng query thuần giống các câu lệnh select trong
database
- Để bắt đầu , bạn hãy tạo dự án SpringBoot và thêm các dependencies sau : 
 ![image](https://user-images.githubusercontent.com/96046778/179996984-ed439ed7-df6f-4732-823b-db5985bca1c4.png)
- ngoài ra bạn hãy thêm dependencies faker (tạo dữ liệu ảo trong database)
```java
        <dependency>
            <groupId>com.github.javafaker</groupId>
            <artifactId>javafaker</artifactId>
            <version>1.0.2</version>
        </dependency>
```
