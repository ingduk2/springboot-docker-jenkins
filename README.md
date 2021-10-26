# springboot docker-jenkins 예제

#### docker 간단 실행해보기
##### 1. ./gradlew bootJar or clean build
##### 2. Dockerfile 작성 (jre 11)
```
FROM openjdk:11-jre-slim
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```
##### 3. docker build -t demo/spring-docker .
```
docker build -t [<group>/<artifact>] [Dockerfile위치]
```
##### 4. docker run -p 8080:8080 demo/spring-docker
```
* -p <호스트 포트>:<컨테이너 포트>
```
   
* container 삭제 후 image 삭제 가능
```
docker container ls -a
docker container rm [name]
docker images
docker images rm [REPOSITORY]
```