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
자바의 ENUM 데이터를 데이터베이스와 매핑할때는 @Enumerated 어노테이션을 사용한다.  
@Column을 사용하여 컬럼 속성을 명시할 수 있다.  

**@Enumerated**  

| EnumType | 데이터 베이스 할당값 |
| ----------- | ---------------- |
| ORDINAL | INT 값으로 할당됨 |
| STRING | 문자열로 할당됨 |


```java
public enum MemberRole {
    USER,
    ADMIN
}
```

```java
@Column(name = "age", length = 100)
private Integer age;
@Column(name = "role")
@Enumerated(EnumType.STRING)
private MemberRole role;
```

#### 6.2.2.5 User Repository 클래스 작성
Repository는 Entity 조작에 필요한 쿼리를 메서드화 해서 사용할 수 있는 역할을 한다. 필드 검색을 하기 위해서 메서드 이름으로 쿼리를 생성할 수 있다.

```code
반환 타입 findBy필드명(매개변수)
```

```java
@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    Member findByName(@Param("name") String name);
}
```

### 6.2.3 연관 관계
연관(Association) 관계는 하나 이상의 객체가 연결되어 있는 상태를 나타낸다. 이때 연결은 하나의 객체가 다른 객체를 소유하거나 참조하는 형태가 된다.

#### 6.2.3.1 N:1(다대일)
JPA에서 다대일 관계를 객체로 표현하기 위해서 @ManyToOne 어노테이션과 @JoinColumn 어노테이션을 사용한다.

**@ManyToOne, @OneToMany**  

| fetch | 로딩 방법 |
| ----------- | ---------------- |
| EAGER | 즉시 로딩 |
| LAZY | 지연 로딩 |

**객체에서의 학생과 학교의 관계 표현**  

*학생*
```java
@Data
@Entity
@NoArgsConstructor
public class School {
    @Id
    @Column(name = "SCHOOL_ID")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "SCHOOL_NAME")
    private String name;
    private String address;
}
```

*학교*
```java
@Data
@Entity
@NoArgsConstructor
public class Student {
    @Id
    @Column(name = "STUDENT_ID")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "STUDENT_NAME")
    private String name;
    private String grade;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "SCHOOL_ID")
    private School school;
}
```

*연관 관계 설정*
```java
@Transactional
public void addSchoolToStudent() {
    School school = new School("신촌초등학교");
    schoolRepository.save(school);

    Student student1 = new Student("서준");
    Student student2 = new Student("서연");
    Student student3 = new Student("서언");

    student1.setSchool(school);
    student2.setSchool(school);
    student3.setSchool(school);

    studentRepository.save(student1);
    studentRepository.save(student2);
    studentRepository.save(student3);
}
```


#### 6.2.3.2 1:N 관계
JPA에서 일대다 관계를 객체로 표현하기 위해서 @OneToMany(mappedBy="mapping_param")을 사용한다. mappedBy는 연관 관계의 주인을 명시하기 위해 사용하는데 연관 관계의 주인은 다수 쪽이다. 

*연관 관계 설정*
```java
@OneToMany(mappedBy = "school")
private List<Student> students;
```

```java
@Transactional
public void addStudentsToSchool() {
    Student student1 = new Student("고길동");
    Student student2 = new Student("김연아");
    studentRepository.save(student1);
    studentRepository.save(student2);
    Student student3 = new Student("박찬호");
    Student student4 = new Student("손흥민");
    studentRepository.save(student3);
    studentRepository.save(student4);

    School school1 = new School("한국 고등학교");
    school1.setStudents(Arrays.asList(student1, student2));
    School school2 = new School("국제 고등학교");
    school2.setStudents(Arrays.asList(student3, student4));

    schoolRepository.save(school1);
    schoolRepository.save(school2);
}
```

[GitHub: firewood3's Spring-Data-Jpa](https://github.com/firewood3/spring/tree/master/spring-boot-data/spring-boot-data-jpa)

## 6.3 QueryDSL을 이용한 Type Safe한 쿼리 작성
QueryDSL은 일종의 표현식으로 데이터베이스의 질의를 별도의 SQL을 사용하지 않고 기존 언어의 메서드를 사용해서 데이터베이스에 질의를 할 수 있는 도구이다. SQL 쿼리는 문자 형식으로 작성하게 되는데 이는 타입 체크가 불가능하고 실행해 보기 전까지는 테스트가 안된다. QueryDSL은 SQL을 java로 type-safe하게 개발 할 수 있게 해준다.

[참고 QueryDSL 공식사이트](http://www.querydsl.com/static/querydsl/4.0.1/reference/ko-KR/html_single/#intro)

[참고 nklee 블로그 Type Safe](https://lng1982.tistory.com/285) 

[참고 SpringBoot + Maven + QueryDsl](https://noep.github.io/2017/05/03/springboot-querydsl/)

### 6.3.1 QueryDSL 설정

#### 의존성 추가
```maven
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
    <version>4.2.1</version>
</dependency>
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <version>4.2.1</version>
</dependency>
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-core</artifactId>
    <version>4.2.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

#### Q클래스 생성 설정
querysql에서 쿼리를 작성하려면 Q클래스라고 불리는 구현체를 생성해야한다. 컴파일시에 JPAAnnotationProcessor가 작동하도록 하여 Q 클래스를 생성하도록 한다.

```maven
<plugin>
    <groupId>com.mysema.maven</groupId>
    <artifactId>apt-maven-plugin</artifactId>
    <version>1.0.9</version>
    <executions>
        <execution>
            <goals>
                <goal>process</goal>
            </goals>
            <configuration>
                <outputDirectory>target/generated-sources/java</outputDirectory>
                <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                <options>
                    <querydsl.entityAccessors>true</querydsl.entityAccessors>
                </options>
            </configuration>
        </execution>
    </executions>
</plugin>
```

*Maven 빌드를 하면 Entity에 대한 Q클래스가 생성된다.*
```java
/**
 * QMemberEntity is a Querydsl query type for MemberEntity
 */
@Generated("com.querydsl.codegen.EntitySerializer")
public class QMemberEntity extends EntityPathBase<MemberEntity> {

    private static final long serialVersionUID = 1199583908L;

    public static final QMemberEntity memberEntity = new QMemberEntity("memberEntity");

    public final NumberPath<Integer> age = createNumber("age", Integer.class);

    public final NumberPath<Long> id = createNumber("id", Long.class);

    public final StringPath name = createString("name");

    public QMemberEntity(String variable) {
        super(MemberEntity.class, forVariable(variable));
    }

    public QMemberEntity(Path<? extends MemberEntity> path) {
        super(path.getType(), path.getMetadata());
    }

    public QMemberEntity(PathMetadata metadata) {
        super(MemberEntity.class, metadata);
    }

}
```

### 6.3.2 QueryDslRepositorySupport 활용
Spring Data JPA에서는 queryDSL을 함께 사용할 수 있는 기반 클래스를 제공하는데, 그 클래스가 QueryDslRepositorySupport이다. 그래서 Spring Data JPA를 사용할 때 기존 Repository에 기능을 추가하는 형태로 개발을 진행할 수 있다.   QueryDslRepositorySupport 추상 클래스의 구현체를 만드는 방법은 다음과 같다.

1. Q 클래스 생성
2. 커스텀 Repository 인터페이스 생성
3. QueryDslRepositorySupport 추상클래스와 커스텀 Repository의 구현체 생성

#### MemberRepositoryCustom 클래스 생성
```java
public interface MemberRepositoryCustom {
    List findAllLike(String keyword);
}
```

#### QueryDslRepositorysupport 추상클래스의 구현체 생성 및 커스텀 인터페이스에 맞는 쿼리 작성
```java
public class MemberRepositoryImpl extends QuerydslRepositorySupport implements MemberRepositoryCustom {
    public MemberRepositoryImpl() {
        super(MemberEntity.class);
    }

    @Override
    @Autowired
    public void setEntityManager(EntityManager entityManager) {
        super.setEntityManager(entityManager);
    }

    @Override
    public List findAllLike(String keyword) {
        QMemberEntity qMemberEntity = QMemberEntity.memberEntity;
        JPQLQuery<MemberEntity> query = from(qMemberEntity);
        List<MemberEntity> entityList = query.where(qMemberEntity.name.like(keyword)).fetch();
        return entityList;
    }
}
```

[GitHub: firewood3's Spring-Data-QueryDsl](https://github.com/firewood3/spring/tree/master/spring-boot-data/spring-data-querydsl)


***

참고도서
제목: 스프링 부트로 배우는 자바 웹 개발
지은이: 윤석진
펴낸곳: 제이펍

