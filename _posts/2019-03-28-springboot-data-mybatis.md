---
title: "스프링 부트에서 Mybatis 사용하기"
date: 2019-03-28 10:50:00 -0400
categories: java spring mybatis
---

MyBatis는 iBatis의 후속버전으로 JDBC에서 문자열로 쿼리를 작성했던 부분을 XML로 작성할 수 있게하는 쿼리 매퍼이다. 이 글에서는 SrpingBoot에서 MyBatis를 사용하여 findAll, findOne, insert 기능을 구현할 것이다. 그리고 DBMS는 로컬에서 간단하게 사용할 수 있는 HSQLDB를 사용할 것이다.

프로젝트에 사용된 의존성은 다음과 같다.
- mybatis-spring-boot-starter
- hsqldb
- lombok

먼저 프로젝트에 사용될 데이터를 정의하자. DDL과 DML을 .sql로 작성한다. 

```sql
drop table TBL_NOTICE if exists;

CREATE TABLE TBL_NOTICE (
  id integer NOT NULL,
  title varchar(100) NOT NULL,
  content varchar(500) NOT NULL
);
```

```sql
insert into TBL_NOTICE values(1, 'title1', 'content1');
insert into TBL_NOTICE values(2, 'title2', 'content2');
insert into TBL_NOTICE values(3, 'title3', 'content3');
```

TBL_NOTICE 테이블과 id, title, content라는 세 개의 컬럼이 있다.

그리고 매핑될 자바 객체를 만들어 주자.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Notice {
    private Long id;
    private String title;
    private String content;
}
```

그 다음으로는 mybatis 매퍼를 xml로 작성하자. <_resultMap> 태그를 사용하여 자바객체와 테이블 컬럼을 연결해주고 있고, <_select> 테그를 사용하여 select 절을, <_insert> 테그를 사용하여 insert 절에 사용될 메서드를 정의하고 있다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="mapper.noticeMapper">

    <resultMap id="userResultMap" type="com.firewood.springbootdatamybatis.notice.Notice">
        <result property="id" column="id"/>
        <result property="title" column="title"/>
        <result property="content" column="content"/>
    </resultMap>


    <select id="findAllNotice" resultType="hashMap">
    <![CDATA[
		SELECT id, title, content
		FROM TBL_NOTICE
	]]>
    </select>

    <insert id="addNotice" >
    <![CDATA[
        insert into TBL_NOTICE(id ,title, content)
        values(#{id},#{title}, #{content})
     ]]>
    </insert>

    <select id="findNoticeById" parameterType="map" resultMap="userResultMap">
    <![CDATA[
      SELECT
            id,
            title,
            content
      FROM TBL_NOTICE
      WHERE id =#{id}
      ]]>
    </select>
</mapper>
```

그 다음으로는 위에서 작성한 myBatis 매핑 xml 파일을 스프링과 연결하기 위한 작업을 진행하자. 세 객체를 빈으로 등록하여 데이터베이스와 연결하고 Mybatis 매퍼를 스프링에 등록한다.
- datasource : 물리적인 데이터 소스를 관리하는 객체 
- sqlSession(sqlSessionTemplate) : 쿼리 매핑 구문을 실행하기 위해 사용됨
- sqlSessionFactory : sqlSession을 생성하기 위해 사용됨

```java
@Configuration
public class MybatisConfig {

    @Bean
    public DataSource dataSource(){
        return new EmbeddedDatabaseBuilder()
                .setName("firewooddb")
                .setType(EmbeddedDatabaseType.HSQL)
                .addScript("schema-hsqldb.sql")
                .addScript("data-hsqldb.sql")
                .build();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory() throws Exception{
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource());
        sqlSessionFactoryBean.setConfigLocation((new PathMatchingResourcePatternResolver().getResource("classpath:mybatis-config.xml")));
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/*.xml"));
        return sqlSessionFactoryBean.getObject();
    }
}
```

마지막으로 Repository를 생성하고 그 안에서 이전에 빈으로 등록한 sqlSesstionTemplate를 주입받아. Mybatis 매핑 XML에 등록한 메서드들을 사용하면 된다.

```java
@Repository
public class NoticeRepository {
    private static final String MAPPER_NAME_SPACE = "mapper.noticeMapper.";

    private final SqlSessionTemplate sqlSessionTemplate;

    @Autowired
    public NoticeRepository(SqlSessionTemplate sqlSessionTemplate) {
        this.sqlSessionTemplate = sqlSessionTemplate;
    }

    public List findAllNotice() {
        return sqlSessionTemplate.selectList(MAPPER_NAME_SPACE + "findAllNotice");
    }

    public Notice findById(Long id) {
        Map<String,Object> params = new HashMap();
        params.put("id", id);

        return sqlSessionTemplate.selectOne(MAPPER_NAME_SPACE + "findNoticeById", params);
    }

    public void addNotice(Notice notice) {
        sqlSessionTemplate.insert(MAPPER_NAME_SPACE + "addNotice", notice);
    }
}
```

[SampleCode : firewood3' GitHub springboot-data-mybatis](https://github.com/firewood3/spring/tree/master/spring-boot-data/springboot-data-mybatis)

***

참고도서
제목: 스프링 부트로 배우는 자바 웹 개발
지은이: 윤석진
펴낸곳: 제이펍
