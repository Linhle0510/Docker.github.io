## Các bước build ứng dụng với Dockerfile
### Cách 1: Dùng multi-stage
Chia làm 2 stage:

1. Stage 1

- Sử dụng base image ***maven:ibmjava-alpine***  
```
FROM maven:ibmjava-alpine as builder
```
- Copy source code vào image
```
COPY . /app
```

- Từ source code build ra file ***websocket-demo-0.0.1-SNAPSHOT.jar*** bằng lệnh: `mvn clean package`.
```
RUN mvn clean package
```

- File ***websocket-demo-0.0.1-SNAPSHOT.jar*** sẽ nằm trong thư mục **target** của source code

2. Stage 2

- Sử dụng base image ***openjdk:8-alpine***
```
FROM openjdk:8-alpine
```

- Copy file ***websocket-demo-0.0.1-SNAPSHOT.jar*** từ stage 1 sang stage 2
```
WORKDIR /spring-app

COPY --from=builder /app/target/websocket-demo-0.0.1-SNAPSHOT.jar /spring-app
```
- Dùng lệnh sau để start container: `java -Djava.security.egd=file:/dev/./urandom -jar websocket-demo-0.0.1-SNAPSHOT.jar`
```
CMD ["java", "-Djava.security.egd=file:/dev/./urandom ", "-jar", "websocket-demo-0.0.1-SNAPSHOT.jar"]
```
3. Build và run image
- Build image
```
docker build -f Dockerfile.cach1 -t spring-app:v1 .
```
- Run 
```
docker run -d --name spring-app-1 -p 9001:8080 spring-app:v1
```
### Cách 2: build source code ra thư mục target chứa các file .jar, sau đó copy thư mục target vào image
1. Đóng gói
- build source code ra thư mục target dùng lệnh:
``` 
./mvnw package
```
- Sử dụng base image ***openjdk:8-alpine***
```
FROM openjdk:8-alpine
```
- Copy file websocket-demo-0.0.1-SNAPSHOT.jar vào docker image
```
 ARG JAR_FILE=target/*.jar

COPY ${JAR_FILE} app.jar
```
- Start container
```
CMD ["java", "-Djava.security.egd=file:/dev/./urandom ", "-jar", "/app.jar"]
``` 
2. Build và run image
- Build image
 ```
 docker build -f Dockerfile.cach2 -t spring-app:v2 .
```
- Run 
```
docker run -d --name spring-app-2 -p 9002:8080 spring-app:v2
```
### Kết quả
![image](https://user-images.githubusercontent.com/80322605/131101982-5358b254-dd37-4697-a04a-9d662e380792.png)
![containers](https://user-images.githubusercontent.com/80322605/131102006-5bdf9bca-1065-4764-94d1-daabc0a6735a.png)
![web 1](https://user-images.githubusercontent.com/80322605/131102036-daa3cb54-fd1a-4f8a-9d29-4e49a67b6317.png)
![web 2](https://user-images.githubusercontent.com/80322605/131102051-01ea2213-fcfe-4544-9aec-59b35e4dba50.png)
