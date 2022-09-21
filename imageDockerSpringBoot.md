## Dockerize ứng dụng SpringBoot

1. Tạo file jar

```shell
mvn clean package -DskipTests  : bo qua test 

mvn package : dong goi all 
```

Sau bước này sẽ tạo ra 1 file có đuôi **.jar** trong thư mục **target**

Có thể kiểm tra chạy ứng dụng bằng câu lệnh sau

```shell
java -jar target/todolist-backend-0.0.1-SNAPSHOT.jar
```
-> chay thu 1 api trong controller de check xem file jar hd khong 

2. Viết Dockerfile

Trong thư mục gốc chứa project nganh hang voi thu muc target , tạo file `Dockerfile`

```dockerfile
FROM openjdk:17.0.2-oraclelinux8  : nho kiem tra ban update sao cho phu hop vs JDK cua du an 

COPY target/todolist-backend-0.0.1-SNAPSHOT.jar todolist-backend-0.0.1-SNAPSHOT.jar  : 

ENTRYPOINT ["java","-jar","/todolist-backend-0.0.1-SNAPSHOT.jar"]

CMD ["./mvnw","spring-boot:run"]
```

3. Build docker image
ls coi xem co Dockerfile khong neu co thi chay 
```shell
docker build --tag todolist-api .  (ten image o day dat la todolist-api) 
docker build -t 14thuhang/todolist-app:latest .  
```
kiem tra cac image : docker image ls = docker images
doi tag cho image : docker tag todolist-api:latest todolist-api:v1.0.0

4. Đẩy lên docker hub (nếu có tài khoản)

```shell
 docker tag todolist-api:v1.0.0 14thuhang/todolist-api:v1.0.0
 
  login vao docker  : 
 docker login
 
push len docker
❯ docker push 14thuhang/todolist-api:v1.0.0
kiem tra tren web coi co chua hoac chay lenh : docker ps -a 
xoa thu image bang id docker rmi -f f21e6af883bd (-f : xoa cac lien ket cua nos , f21e6af883bd : id cua image can xoa )

Pull and start/run a container :  docker run -dp 8085:8080 --name todolist-api -v "$(pwd):/app" 14thuhang/todolist-api:v1.0.0

docker ps : xem container nao dang chay 

push code & run : docker restart todolist-api-container
```

5. Run container chạy thử

```shell
docker run -d --name todolist -p 8888:8080 buihien0109/todolist-app:latest
```
