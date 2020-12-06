---
title: "메이븐으로 프로젝트 생성하고 jar를 도커로 실행하기"
date: 2020-12-06 21:06:12 -0400
categories: maven-jar-docker
---
## 개요
메이븐의 archetype:generate로 간단 java 프로젝트 생성후 컴파일과 패키징하고 jar를 도커로 돌려본다.

[1. 메이븐 템플릿으로 프로젝트 생성](##-1.-메이븐-템플릿으로-프로젝트-생성)  
[2. 메이븐 컴파일](##-2.-메이븐-컴파일)  
[3. Pom.xml에 java 버전을 적어주어야 제대로 컴파일 가능](##-3.-Pom.xml에-java-버전을-적어주어야-제대로-컴파일-가능)  
[4. class 파일을 실행시켜보기](##-4.-class-파일을-실행시켜보기)  
[5. 메이븐 명령어로 jar 만들기](##-5.-메이븐-명령어로-jar-만들기)  
[6. 만들어진 jar를 실행해보기](##-6.-만들어진-jar를-실행해보기)  
[7. jar를 도커 이미지로 만들고 실행하기](##-7.-jar를-도커-이미지로-만들고-실행하기)  

## 1. 메이븐 템플릿으로 프로젝트 생성
```code
$ mvn archetype:generate -DgroupId=com.gchsj -DartifactId=javaprj -DarchetypeArtifactId=maven-archetype-quickstart
```
maven-archetype-quickstart이라는 템플릿을 사용하여 생성된 자바코드
```java
package com.gchsj;

/**
 * Hello world!
 *
 */
public class App 
{
    public static void main( String[] args )
    {
        System.out.println( "Hello World!" );
    }
}
```

## 2. 메이븐 컴파일
```code
$ mvn compile
```
자바 버전이 1.5 이상이어야 한다는 에러 발생! (실습에 사용한 Apache Maven 3.6.3 에서는 자바 버전이 1.5 이상이어야 한다.)

## 3. Pom.xml에 java 버전을 적어주어야 제대로 컴파일 가능
```xml
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

다시
```code
$ mvn compile
```
를 실행하면 target 디렉토리가 생성됨

## 4. class 파일을 실행시켜보기
main 메소드를 가진 클래스를 패키지 명을 모두 입력하여 실행시켜야 한다.
```code
$ java -cp target\classes com.gchsj.App
Hello World!
```
cp옵션은 클래스 패스를 설정하는 명령어. 해당 위치에서 com.gchsj.App 클래스파일을 실행시킨다.

## 5. 메이븐 명령어로 jar 만들기
```code
$ mvn package
```
target 디렉토리에 javaprj-1.0-SNAPSHOT.jar 파일이 생성됨

## 6. 만들어진 jar를 실행해보기
```code
$ java -cp target\javaprj-1.0-SNAPSHOT.jar com.gchsj.App
Hello World!
```
jar는 zip 파일처럼 해당 java 프로젝트를 묶은 것이므로 4번에서 App.class 파일을 실행시킨것처럼 java 명령어로 해당 클래스 파일을 실행시킨다.

$ java -jar 명령어로 실행시키려 한다면
.\target\javaprj-1.0-SNAPSHOT.jar에 기본 Manifest 속성이 없습니다.
과 같은 에러메시지가 나온다. manifest 속성을 넣어주면 해결 가능하다.

```xml
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <configuration>
          <archive>
            <manifest>
              <addClasspath>true</addClasspath>
              <mainClass>com.gchsj.App</mainClass>
            </manifest>
          </archive>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

```code
$ java -jar target\javaprj2-1.0-SNAPSHOT.jar
Hello World!
``` 

## 7. jar를 도커 이미지로 만들고 실행하기
jar를 실행할 베이스 openjdk 이미지 pull 받기
```code
$ docker search openjdk
$ docker pull adoptopenjdk/openjdk8
```
jar 파일이 위치한 곳에 Dockerfile 생성
```Dockerfile
FROM adoptopenjdk/openjdk8:latest
ADD javaprj2-1.0-SNAPSHOT.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

도커파일로 도커 이미지 생성
```code
$ docker build --tag javaprj2:0.1 ./
```
--tag 옵션으로 태그 설정하고 마지막 ./은 Dockerfile이 있는 위치를 지정해준다.(./은 현재위치)

도커이미지 컨테이터로 실행
```code
$ docker run --rm javaprj2:0.1
Hello World!
```
