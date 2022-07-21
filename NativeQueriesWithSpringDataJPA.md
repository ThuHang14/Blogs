## Native Queries with Spring Data JPA
- Spring Data JPA hỗ trợ nhiều cách khác nhau để lấy dữ liệu từ cơ sở dữ liệu của bạn. Một số cách rất dễ sử dụng như các truy vấn dẫn xuất ( derived queries), truy vấn tùy chỉnh (custom queries). Trong đó truy vấn gốc (Native queries) là tùy chọn linh hoạt nhất và mạnh mẽ nhất giúp bạn đọc dữ liệu.
- Các Native queries trong JPA, chúng cung cấp cho bạn quyền truy cập vào tất cả các tính năng được cơ sở dữ liệu của bạn hỗ trợ. Điều đó làm cho chúng phù hợp cho tất cả các trường hợp sử dụng yêu cầu truy vấn phức tạp để trích xuất thông tin cần thiết. Tất nhiên, điều này cũng đúng nếu bạn đang sử dụng các Native queries với Spring Data JPA vì bạn đang sử dụng cùng một cơ chế nội bộ. Spring Data JPA chỉ làm cho việc định nghĩa và thực thi một Native queries dễ dàng hơn một chút.
-------------------------------------------------------
1 Defining a Native Query   
1.1 Write Operations as Native Queries  
2 Limitations of Native Queries With Spring Data JPA  
2.1 Add a Count Query to Enable Pagination  
2.2 No Dynamic Sorting  
3 Conclusion  
--------------------------------------------------------
--------------------------------------------------------
## 1. Xác định một Native queries
- Khi sử dụng JPA thuần hoặc Hibernate, việc xác định và thực hiện một truy vấn gốc yêu cầu nhiều bước. Annotation  `@Query` của Spring Data sẽ xóa tất cả mã soạn sẵn. với @Query bạn có thể khai báo câu query cho các method trong repository

- Khi xác định một truy vấn gốc, bạn chú thích phương thức lưu trữ của mình bằng annotation `@Query`, đặt thuộc tính `nativeQuery=true` và cung cấp một câu lệnh SQL làm giá trị. Như được hiển thị trong đoạn mã sau, bạn có thể sử dụng các tham số liên kết theo cách giống như trong một truy vấn JPQL tùy chỉnh.
```java
@Repository
public interface AuthorRepository extends CrudRepository<Author, Long>, PagingAndSortingRepository<Author, Long> {
 
    @Query(value="select * from author a where a.first_name= :firstName", nativeQuery=true)
    List<Author> getAuthorsByFirstName(String firstName);
 
 
}
```

Sau đó, bạn chỉ cần gọi phương thức getAuthorsByFirstName trong repository với firstName mà bạn muốn tìm kiếm.
```java
List<Author> authors = authorRepository.getAuthorsByFirstName("Janssen");

```
- khi chạy câu lệnh Spring Data sẽ cung cấp mã bắt buộc để khởi tạo truy vấn cho câu lệnh được cung cấp trong  `@Query` của bạn. Sau đó, nó đặt giá trị được cung cấp làm tham số ràng buộc trên truy vấn đó và thực thi nó.
## 1.1. Write Operations as Native Queries 

- để tối ưu hiệu suất Hibernate bạn nên dùng bulk operations để thêm số lượng lớn các dữ liệu bằng cách sử dụng JPQL, Criteria hoặc navtive queries


- Bạn có thể sử dụng annotation `@Query` để xác định một câu lệnh JPQL hoặc native SQL. Vì các thao tác ghi cần được thực hiện khác với các thao tác đọc, bạn cũng cần chú thích phương thức kho lưu trữ bằng annotation `@Modizing`. Đó là sự khác biệt duy nhất đối với các câu lệnh navtive SQL SELECT .
```java
@Repository
public interface AuthorRepository extends CrudRepository<Author, Long>, PagingAndSortingRepository<Author, Long> {
 
    @Modifying
    @Query(value="delete from author a where a.last_name= :lastName", nativeQuery = true)
    void deleteAuthorByLastName(@Param("lastName") String lastName);
     
    @Modifying
    @Query(value="update author set last_name= :lastName where first_name = :firstName", nativeQuery=true)
    void updateAuthorByFirstName(String firstName, String lastName);

}
```

## 2. Limitations of Native Queries With Spring Data JPA
- khi dùng native queries, có những hạn chế sau :  
1. Spring Data JPA và persistence provider của bạn không điều chỉnh truy vấn theo phương ngữ SQL cụ thể của cơ sở dữ liệu của bạn. Do đó, bạn cần đảm bảo rằng tất cả RDBMS được hỗ trợ bởi ứng dụng của bạn có thể xử lý câu lệnh được cung cấp.
2. Khi phân trang kết quả Native Queries yêu cầu thêm một bước.
3. Spring Data JPA không hỗ trợ sắp xếp tự động cho các câu lệnh SQL gốc.

- Chúng ta cùng xem xét kỹ hơn hạn chế thứ 2 và thứ 3.

### 2.1 Thêm truy vấn đếm để phân trang
- Khi làm việc với truy vấn JPQL tùy chỉnh, bạn có thể thêm tham số kiểu ,gắn vào phương thức lưu trữ của mình. Điều này cho phép phân trang cho kết quả truy vấn của bạn. Spring Data JPA sau đó sẽ thêm tất cả mã soạn sẵn cần thiết để truy xuất kết quả truy vấn từng trang một.

- để phân trang bạn Làm tương tự với một native query nhưng yêu cầu thêm một bước truy vấn đếm trả về tổng số bản ghi có trong kết quả không phân trang. Một cách để làm điều đó là cung cấp Chuỗi truy vấn làm giá trị của thuộc tính countQuery của annotation `@Query`.

```java
@Repository
public interface AuthorRepository extends CrudRepository<Author, Long>, PagingAndSortingRepository<Author, Long> {
     
    @Query(value="select * from author a where a.last_name= ?1", 
            countQuery = "select count(id) from author a where a.last_name= ?1", 
            nativeQuery = true)
    Page<Author> getAuthorsByLastName(String lastname, Pageable page);
     
    ...
}
```

- Nếu phương thức kho lưu trữ của bạn tham chiếu đến native query được đặt tên, bạn cần cung cấp truy vấn đếm dưới dạng truy vấn được gọi lại tên lần thứ hai và thêm hậu tố `.count` vào tên của nó.
```java 
@NamedNativeQuery(name = "Author.getAuthorsByLastName", 
                    query = "select * from author a where a.last_name= ?1", 
                    resultClass = Author.class)
@NamedNativeQuery(name = "Author.getAuthorsByLastName.count", 
                    query = "select count(id) from author a where a.last_name= ?1")
@Entity
public class Author { ... }
```

### 2.2 Không có phân trang tự động
- Khi làm việc với truy vấn JPQL, bạn có thể thêm tham số kiểu Sắp xếp vào phương thức kho lưu trữ của mình. Điều này cho phép bạn xác định tiêu chí sắp xếp trong thời gian chạy. Spring Data JPA sau đó tạo mệnh đề ORDER BY bắt buộc dựa trên giá trị tham số được cung cấp.

- Rất tiếc, Spring Data JPA không hỗ trợ tính năng này cho các truy vấn gốc. Làm điều đó sẽ yêu cầu Spring Data phân tích câu lệnh được cung cấp và tạo mệnh đề ORDER BY trong phương ngữ dành riêng cho cơ sở dữ liệu. Đây sẽ là một hoạt động rất phức tạp và hiện không được Spring Data JPA hỗ trợ.

- Tất nhiên, bạn có thể thêm mệnh đề ORDER BY của riêng bạn vào truy vấn của bạn. Nhưng điều đó giới hạn bạn ở một thứ tự cụ thể cho mỗi truy vấn. Nếu bạn cần hỗ trợ nhiều hơn, sử dụng composite repository là cách tốt hơn. Sau đó, bạn có thể triển khai phương pháp truy vấn của riêng mình bằng cách sử dụng API của JPA và chỉ định mệnh đề ORDER BY dựa trên các tham số đầu vào được cung cấp.

##  kết luận
- Truy vấn gốc là cách mạnh mẽ và linh hoạt nhất để triển khai các hoạt động đọc của bạn. Chúng cho phép bạn sử dụng tất cả các tính năng được cơ sở dữ liệu của bạn hỗ trợ và Spring Data JPA xử lý hầu hết tất cả các mã soạn sẵn cần thiết.

- Nhưng việc sử dụng chúng tốn nhiều công sức hơn so với derived query và chúng cung cấp một vài hạn chế so với custom JPQL query. Những điều đáng chú ý nhất là:

1. Để sử dụng phân trang cho kết quả truy vấn của bạn, bạn cần cung cấp một truy vấn đếm. Bạn có thể làm điều đó bằng cách đặt thuộc tính countQuery của annotation `@Query`.
2. Spring Data JPA không hỗ trợ sắp xếp tự động cho các truy vấn gốc. Nếu bạn muốn truy xuất kết quả truy vấn của mình theo một thứ tự cụ thể, bạn cần bao gồm mệnh đề `ORDER BY` trong truy vấn của mình.
3. Spring Data JPA và persistence provider của bạn không điều chỉnh câu lệnh truy vấn gốc của bạn theo phương ngữ SQL trong cơ sở dữ liệu của bạn. Do đó, bạn cần đảm bảo rằng tất cả các DBMS được hỗ trợ đều hỗ trợ câu lệnh SQL của bạn.

[Code Demo](https://github.com/ThuHang14/Blogs/blob/main/blog1.md)
