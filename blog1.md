### Truy vấn dữ liệu Spring JPA
- Spring Data là một module của Spring Framework. Mục đích của Spring Data JPA là giảm thiểu việc thực hiện quá nhiều bước để có thể implement được JPA.
  Spring Data JPA là một phần của Spirng Data và có hỗ trợ Hibernate
- Ở bài viết này bạn sẽ Custom Query , define thông qua chú thích @Query
-  với @Query bạn có thể khai báo câu query cho các method trong repository , đặt thuộc tính `nativeQuery = true` để dùng query thuần giống các câu lệnh select trong
   database
- Để bắt đầu , bạn hãy tạo dự án SpringBoot và thêm các dependencies sau vào file `pom.xml` :
 ```java
      
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
```
- ngoài ra bạn hãy thêm dependencies faker (tạo dữ liệu ảo trong database)
```java
        <dependency>
            <groupId>com.github.javafaker</groupId>
            <artifactId>javafaker</artifactId>
            <version>1.0.2</version>
        </dependency>
```
2. Cấu hình file `application.properties` : 
```java
        # DATABASE
        # ===============================
        spring.datasource.url=jdbc:mysql://localhost:3306/db-test-author?createDatabaseIfNotExist=true&useSSL=false //phần này đổi localhost theo máy của bạn
        spring.datasource.username=root  //phần này đổi uername theo máy bạn
        spring.datasource.password=123 //phần này đổi pass theo máy bạn
        spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
        spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect

        # ===============================
        # JPA / HIBERNATE
        # ===============================
        spring.jpa.show-sql=true
        spring.jpa.generate-ddl=true
        spring.jpa.hibernate.ddl-auto=update
        spring.jpa.properties.hibernate.format_sql=true
        spring.jpa.hibernate.use-new-id-generator-mappings=false
```
3. tạo thư mục entity với 2 file con `Author.java` `Book.java`
- trong `Author.java` :
```java
package com.example.demospringdatajpa.entity;

import lombok.*;

import javax.persistence.*;

@Builder
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
@Entity
@Table(name = "author")
public class Author {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id", nullable = false)
    private Long id;

    @Column(name = "name")
    private String name;

    @Column(name = "address")
    private String address;

}
```
- trong `Book.java` :
```java
package com.example.demospringdatajpa.entity;

import lombok.*;

import javax.persistence.*;
import java.time.LocalDateTime;

@Builder
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
@Entity
@Table(name = "book")
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id", nullable = false)
    private Long id;

    @Column(name = "name")
    private String name;

    @Column(name = "published_at")
    private LocalDateTime publishedAt;

//phần này là khóa 1:N 
    @ManyToOne
    @JoinColumn(name = "author_id")
    private Author author_id;


    @PrePersist
    public void prePersist() {
        publishedAt = LocalDateTime.now().minusMonths(1);
    }
} 
```
4. chạy dự án và mở database kiểm tra kết quả sẽ được như sau :
   ![image](https://user-images.githubusercontent.com/96046778/180000514-7710aa77-9088-47e8-8788-6dd70ac91c1c.png)
5. giờ ta sẽ thêm dữ liệu vào bằng faker . Hãy tạo 1 file trong thư mục test của dự án và chèn dữ liệu như sau :
```java
package com.example.demospringdatajpa;

import com.example.demospringdatajpa.entity.Author;
import com.example.demospringdatajpa.entity.Book;
import com.example.demospringdatajpa.repository.AuthorRepository;
import com.example.demospringdatajpa.repository.BookRepository;
import com.github.javafaker.Faker;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import javax.servlet.http.HttpServletRequest;
import java.util.List;
import java.util.Random;

@SpringBootTest
public class InitData {

    @Autowired
    private BookRepository bookRepository;
    @Autowired
    private AuthorRepository authorRepository;
    @Autowired
    private HttpServletRequest httpServletRequest;
    @Autowired
    private Faker faker;
    @Autowired
    private Random random;


    //tạo 30 dữ liệu cho phần Author
    @Test
    void initDataAuthor() {
        for (int i = 0; i < 30; i++) {
            Author author = Author.builder()
                    .name(faker.name().fullName())
                    .address(faker.address().streetAddress())
                    .build();

            authorRepository.save(author);
        }
    }

//    tạo 50 dữ liệu cho phần Book
    @Test
    void initDataBook() {
        List<Author> authorList = authorRepository.findAll();

        for (int i = 0; i < 50; i++) {
            Author authorRd = authorRepository.getAuthorById(random.nextLong(authorList.size()));
            Book book = Book.builder()
                    .name(faker.lorem().sentence(20))
                    .author_id(authorRd)
                    .build();

            bookRepository.save(book);
        }
    }
}
 
```
6. giờ mở database xem kết quả ta được như sau : 
- dữ liệu Author
  ![image](https://user-images.githubusercontent.com/96046778/180002066-b15edc3f-451b-4606-97d9-6f90a403571b.png)
- dữ liệu Book
  ![image](https://user-images.githubusercontent.com/96046778/180002365-72cccc33-a820-49ff-8fe5-8be81a5d11b6.png)
7.Giờ bạn tạo thêm thư mục repository với 2 file con là `AuthorRepository.java` `BookRepository.java`
- trong AuthorRepository.java 
```java
package com.example.demospringdatajpa.repository;

import com.example.demospringdatajpa.entity.Author;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.CrudRepository;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface AuthorRepository extends JpaRepository<Author, Long>, CrudRepository<Author, Long>, PagingAndSortingRepository<Author, Long> {
   //câu truy vấn tìm Author bằng tên
    @Query(value = "select * from author a where a.name= ?1", nativeQuery = true)
    Author getAuthorsByName(String Name);

    //câu truy vấn cập nhập địa chỉ Author thông qua id
    @Modifying
    @Query(value = "update author set address= ?2 where id = ?1", nativeQuery = true)
    void updateAuthorAddressById(Long Id, String address);


    Author getAuthorById(Long id);

}
```
- trong BookRepository.java : 
```java
package com.example.demospringdatajpa.repository;

import com.example.demospringdatajpa.entity.Book;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.CrudRepository;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface BookRepository extends JpaRepository<Book, Long>, CrudRepository<Book, Long>, PagingAndSortingRepository<Book, Long> {
    
    //câu truy vấn xóa 1 Book bằng id
    @Modifying
    @Query(value = "delete from book b where b.id= ?1", nativeQuery = true)
    void deleteAuthorById(Long id);

    //truy vấn lấy ra những quyển sách được viết bởi tác giả bằng id tác giả
    @Query(value = "select * from book b where b.id= ?1",
            countQuery = "select b.id b.name from book b where b.author_id= ?1",
            nativeQuery = true)
    Page<Book> getBooksByIdAuthor(Long id, Pageable page); // nếu bạn muốn dùng @Param cho id thì trong @Qery hãy đổi ?1 thành :id
//Pageable dùng để phân trang , tương tự như limit 
}
```
- trong file trên `@Modifying` dùng để chú thích phuơng thức kho lưu trữ , nếu không có chương trình sẽ báo lỗi `could not extract resultset`
- cấu trúc thư mục sẽ như thế này :
  ![image](https://user-images.githubusercontent.com/96046778/180003185-48395b2b-adf2-4f69-ad58-b98dfc61f418.png)

7. Giờ thì chạy test cho các câu query đã viết ở trên :
```java
package com.example.demospringdatajpa;

import com.example.demospringdatajpa.entity.Author;
import com.example.demospringdatajpa.repository.AuthorRepository;
import com.example.demospringdatajpa.repository.BookRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.transaction.annotation.Transactional;

@SpringBootTest
public class TestQueries {


    @Autowired
    private AuthorRepository authorRepository;
    @Autowired
    private BookRepository bookRepository;

    @Test
    void getAuthorByName() {
        Author author = authorRepository.getAuthorsByName("Abel Sipes");
        System.out.println(author);
    }

    @Transactional
    @Test
    void updateAuthorAddressById() {
        authorRepository.updateAuthorAddressById(2L, "123 Hai Duong");
    }

    @Transactional
    @Test
    void deleteBookById() {
        bookRepository.deleteAuthorById(2L);
    }

    @Transactional
    @Test
    void getBookPageById() {
        Pageable pageable = PageRequest.of(0, 2);
        bookRepository.getBooksByIdAuthor(8L, pageable);
    }
}
 
```
- khi chạy trương trình test hãy thêm `@Transactional` để chương trình thực hiện truy vấn 
