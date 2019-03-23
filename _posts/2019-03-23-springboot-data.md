---
title: "스프링 부트와 데이터"
date: 2019-03-23 19:00:00 -0400
categories: java spring
---

## 6.1 데이터베이스 프로그래밍

다양한 데이터베이스를 공통적으로 대응할 수 있도록 자바에서는 JDBC(JAVA Database Connectivity)라는 공통된 스펙을 제공하고 벤더는 이를 기반으로 개발자에게 자바에서 사용할 수 있는 접속 라이브러리를 제공한다. 그리서 서로다른 데이터베이스라고 하더라도 벤더에서 제공하는 JDBC 라이브러리가 있으면 동일한 메서드를 사용해 서로다른 데이터베이스를 조작할 수 있다.

시스템의 규모가 커지고 로직이 복잡해짐에 따라서 길어진 쿼리문을 문자열과 소스가 결합된 형태로 개발하는 것에 대한 불편함이 생겼다. 그래서 이에 대한 해결책으로 SQLMapper(iBATIS, Mybatis)와 ORM(JPA, Hibernate)을 사용하여 개발하게된다.

## 6.2 ORM 도구의 활용
ORM(Object Relational Mapping)이란, 객체와 관계형 데이터베이스 간의 불일치 문제를 해결하기 위한 도구다. JDBC를 이용하여 개발을 하면 로직 처리를 SQL로 하게되고, 자바를 사용하지만 자바의 언어적인 기능을 사용하기보다는 쿼리와 결화를 매핑하는 용도로 사용되기 때문에 ORM이 등장하게 되었다.

### 6.2.1 Spring Data JPA
자바 진영에서 자주 쓰이는 ORM 기술은 JPA와 Hibernate가 있다. Spring에서는 Spring Data 프로젝트의 하위 프로젝트로 Spring Data Jpa를 지원한다.  
스프링 데이터는 NOSQL 또는 RDBMS 어느 한쪽만을 목표로 하지 않으므로 Srping Data의 추상회된 인터페이스를 통해서 MySQL, ElasticSearch, Redis 등 다양한 저장소를 활용할 수 있도록 해준다.  
[Spring Data 참고](https://res.infoq.com/articles/spring-data-intro/en/resources/spring_data_overview_small.jpg)

### 6.2.2 데이터베이스와 객체 매핑

#### 6.2.2.1 Entity 클래스 설정
클래스에 @Entity 어노테이션을 선언하여 데이터배이스의 스키마 내용을 자바 클래스로 표현할 수 있다.  
@Table 어노테이션을 사용하여 클래스명과 테이블명을 다르게 매칭할 수 있다.  
```java
@Data
@Entity
@Table(name = "tbl_member")
public class Member {
}
```

#### 6.2.2.2 데이터베이스와 키 매핑
데이터 베이스에서 제공하는 auto_increment, sequence와 같이 유일성을 보장하는 요소들과 키값 역할을 하는 클래스의 필드를 매핑할 때 @ID 어노테이션을 사용한다.  


**@ID 어노테이션의 생성 규칙**
| GenerationType | 키 값을 데이터베이스 테이블로 표현할 경우 |
| ----------- | ---------------- |
| Sequence | 데이터베이스의 시퀸스 객체와 매핑할 때 |
| Identify | Auto를 지원하지 않는 경우 Identify 속성으로 직접 지정해서 사용할 수 있다. |
| Auto | 데이터베이스에 자동으로 키 관련 객체를 생성한다. |

```java
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```

#### 6.2.2.3 콜백 메서드를 사용한 날짜 형식 매핑
날짜와 같이 매핑 시에 값 참조를 위해서 미리 인스턴스가 생성되어야 하는 필드는 JPA에서 제공하는 콜백 메서드를 사용해서 값을 세팅할 수 있다.  
날짜 타입에 createdAt 필드를 영속성 콘텍스트가 로드할 때 beforeCreate 메서드가 호출되어 createdAt 필드에 날짜 값이 할당된다.
```java
private Date createdAt;
@PrePersist
public void beforeCreate() {
    createdAt = new Date();
}
```
#### 6.2.2.4 값 매핑

```java
```
#### 6.2.2.5 User Repository 클래스 작성

```java
```

### 6.2.3 연관 관계

#### 6.2.3.1 N:1(다대일)

#### 6.2.3.2 1:N 관계



***

스터디 도서  
제목: 스프링 부트로 배우는 자바 웹 개발  
지은이: 윤석진  
펴낸곳: 제이펍 

