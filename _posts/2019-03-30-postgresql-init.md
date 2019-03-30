---
title: "PostgreSQL 간단 사용법"
date: 2019-03-30 15:52:00 -0400
categories: PostgreSQL
---

## CentOS6에 PostgreSQL10 설치하기

**리파지토리 추가**
```code
rpm -Uvh https://yum.postgresql.org/10/redhat/rhel-6-x86_64/pgdg-redhat10-10-2.noarch.rpm
```

**설치**
```code
yum install postgresql10-server postgresql10
```


**실행**
```code
service postgresql-10 start
```  

## PostgreSQL 간단 명령어

**PG 접속**
```code
sudo -u postgres psql
```

**PG 쉘에서 빠져나가기**
```code
\q
```

**계정 목록 조회**
```code
\du 
\du+(상세조회)
```

**계정 생성**
```code
create role 계정명;
```

**계정 패스워드 변경**
```code
ALTER ROLE 계정명 LOGIN password 패스워드;
```

**계정 권한 변경**
```code
ALTER USER TEST1 WITH CREATEUSER REPLICATION;
ALTER USER TEST1 WITH NOREPLICATION;
```

**계정 삭제**
```code
DROP USER name
```

**현재 연결 정보 보기**
```code
\conninfo
```

**데이터베이스 생성**
```code
sudo -u postgres createdb pg_notice
```

**데이터베이스 목록 조회**
```code
\list 또는 \l
\list+ 또는 \lt+
```

**사용자 권한 부여**
```code
GRANT CONNECT ON DATABASE my_db TO my_user;
GRANT USAGE ON SCHEMA public TO my_user;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO my_user;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO my_user;
```

**데이터베이스 사용하기**
```code
\c 데이터베이스명
```
