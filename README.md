# springboot docker-jenkins 예제

## docker 간단 실행해보기
* ./gradlew bootJar or clean build
* Dockerfile 작성 (jre 11)
```
FROM openjdk:11-jre-slim
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```
* docker build -t demo/spring-docker .
```
docker build -t [<group>/<artifact>] [Dockerfile위치]
```
* docker run -p 8080:8080 demo/spring-docker
```
-p <호스트 포트>:<컨테이너 포트>
```
* container 삭제 후 image 삭제 가능
```
docker container ls -a
docker container rm [name]
docker images
docker images rm [REPOSITORY]
```

---
## 최적화 방법
소스 수정 후 gradlew build 후 
image build 하면 새로운 image로 인식이되고 jar 사이즈만큼 추가됨
프로젝트가 커질수록 시간이 소요된다.

1. layer 나누어 이미지 빌드
* ./gradlew clean
* ./gradlew bootJar
* mkdir -p build/dependency && (cd build/dependency; jar -xf ../libs/*.jar)
```
springboot 2.5 에서 gradle build 시 
-plain.jar 가 생성되는데 이것때문에 jar가 안풀림
./gradlew bootJar 로 하면 .jar 하나만 생성
```
* Docker 파일 재작성
```
FROM openjdk:11-jre-slim
ARG DEPENDENCY=build/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","hello.dockerjenkins.DockerJenkinsApplication"]
```
명령 순서대로 쌓인다.
```
ENTRYPOINT
COPY BOOT-INF
COPY META-INF
COPY
ARG DEPENDENCY=build/dependency
레이어가 쌓여서 캐싱하고, 변경된 layer만 교체됨
```
---
2. layered jar(multistage)
* build.gradle 추가 (2.4.0 이상버전에서 default로 제공)
```
bootJar {
   layered()
}
```
```
application
snapshot-dependencies
spring-boot-loader
dependencies
4개의 레이어로 나누어줌(아래로 갈수록 변경 적은순)
```
* DockerfIle 작성
```
FROM openjdk:11-jre-slim as builder
WORKDIR application
ARG JAR_FILE=build/libs/*.jar
ADD ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract

FROM openjdk:11-jre-slim
WORKDIR application
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./
ENTRYPOINT ["java","org.springframework.boot.loader.JarLauncher"]
```
