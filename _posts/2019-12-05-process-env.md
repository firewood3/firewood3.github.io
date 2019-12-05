---
title: "nodejs, process.env를 사용하여 로컬 환경변수 및 도커 환경변수 읽어오기"
date: 2019-12-05 09:11:12 -0400
categories: javascript nodejs process.env environment
---

환경 변수 설정은 데이터 베이스 접속 정보나 api키 값을 설정할 때 유용하게 사용됨<br>
### 참고자료
[How To Pass Environment Info During Docker Builds](https://blog.bitsrc.io/how-to-pass-environment-info-during-docker-builds-1f7c5566dd0e)<br>
[Node.js Everywhere with Environment Variables!](https://medium.com/the-node-js-collection/making-your-node-js-work-everywhere-with-environment-variables-2da8cdf6e786)
### 목차
[1. 커맨드 라인으로 입력된 환경변수 읽기](#1.-커맨드-라인으로-입력된-환경변수-읽기)<br>
[2. .env파일의 환경변수 읽기](#2.-.env파일의-환경변수-읽기)<br>
[3. 도커 환경변수 읽기](#3.-도커-환경변수-읽기)<br>
[4. 도커 빌드 시 매개변수를 입력하고 이를 환경변수로 할당하기](#4.-도커-빌드-시-매개변수를-입력하고-이를-환경변수로-할당하기)

## 1. 커맨드 라인으로 입력된 환경변수 읽기
### 1.1 다음과 같은 server.js 파일 생성
```js
console.log(`server port is ${process.env.PORT}`);
console.log(`db name is ${process.env.DB_NAME}`);
console.log(`db user is ${process.env.DB_USER}`);
console.log(`db pw is ${process.env.DB_PW}`);
console.log(`db host is ${process.env.DB_HOST}`);
console.log(`db port is ${process.env.DB_PORT}`);
```

### 1.2 커맨드 라인에 환경변수를 입력하고 server.js 실행
```code
PORT=3000 DB_NAME='pg' DB_USER='firewood' DB_PW=1111 DB_HOST='127.0.0.1' DB_PORT=5432 node server.js
```

### 1.3 결과
```code
server port is 3000
db name is pg
db user is firewood
db pw is 1111
db host is 127.0.0.1
db port is 5432
```

## 2. .env파일의 환경변수 읽기

### 2.1 package.json 파일 생성
```code
npm init -y
```

### 2.2 dotenv 설치
```code
npm install dotenv
```

### 2.3 .env 파일 생성
.env 파일은 루트 경로에 생성해야함
```config
PORT=3000
DB_NAME='pg'
DB_USER='firewood'
DB_PW='1111'
DB_HOST='127.0.0.1'
DB_PORT=5432
```

### 2.4 server.js에서 dotenv를 사용하여 .env파일의 환경변수 읽기
server.js에 dotenv 추가
```js
const dotenv = require('dotenv');
dotenv.config();

console.log(`server port is ${process.env.PORT}`); // 3000
console.log(`db name is ${process.env.DB_NAME}`);  // pg
console.log(`db user is ${process.env.DB_USER}`);  // firewood
console.log(`db pw is ${process.env.DB_PW}`);      // 1111
console.log(`db host is ${process.env.DB_HOST}`);  // 127.0.0.1
console.log(`db port is ${process.env.DB_PORT}`);  // 5432
```
### 2.5 express 설치
```code
npm i express
```

### 2.6 server.js 수정

```js
const express = require('express');
const dotenv = require('dotenv');
dotenv.config();
const app = express(),
      port = process.env.PORT || 3080;

let db_config = {
    user: process.env.AG_USER,
    password: process.env.AG_PASSWORD,
    database: process.env.AG_DATABASE,
    host: process.env.AG_HOST,
    port: process.env.AG_PORT
}

module.exports = {
    db_config: db_config
};

app.get('/', (req,res) => {
    res.send(`server port is ${process.env.PORT}
             db name is ${process.env.DB_NAME}
             db user is ${process.env.DB_USER}
             db pw is ${process.env.DB_PW}
             db host is ${process.env.DB_HOST}
             db port is ${process.env.DB_PORT}`);
});

app.listen(port, () => {
    console.log(`Server listening on the port::${port}`);
});
```

### 2.7 결과
- 서버 실행 : node server.js
- input : http://localhost:3000
- output : 
```code
server port is 3000
db name is pg 
db user is firewood 
db pw is 1111 
db host is 127.0.0.1 
db port is 5432
```

## 3. 도커 환경변수 읽기
### 3.1 도커 파일의 작성
```docker
# base image
FROM node:slim

# setting the work direcotry
WORKDIR /usr/src/app

# env
ENV DB_PORT=3000
ENV DB_NAME='docker'
ENV DB_USER='docker_user'
ENV DB_PW='docker_pw'
ENV DB_HOST='docker_host'
ENV DB_PORT='docker_port'

# copy package.json
COPY ./package.json .

# install dependencies
RUN npm install

# COPY server.js
COPY ./server.js .

EXPOSE 3000

CMD ["node","server.js"]
```

### 3.2 실행
- docker build -t server:v1 .
- docker run -p 3000:3000 -d server:v1
- 환경변수 변경 가능: docker run -e DB_NAME='changed_name' -e DB_HOST='changed_host' -p 3000:3000 -d server:v1음

### 3.3 결과
- input: http://localhost:3000
- output :
```code
server port is 3000
db name is docker (changed_name)
db user is docker_user
pw is docker_pw
db host is docker_host (chaged_host)
db port is docker_port
```

## 4. 도커 빌드 시 매개변수를 입력하고 이를 환경변수로 할당하기

### 4.1 도커 파일에 매개변수 추가

```docker
# base image
FROM node:slim

# args
ARG ARG_DB_PORT=3000
ARG ARG_DB_NAME='arg_docker'
ARG ARG_DB_USER='arg_docker_user'
ARG ARG_DB_PW='arg_docker_pw'
ARG ARG_DB_HOST='arg_docker_host'
ARG ARG_DB_PORT='arg_docker_port'

# setting the work direcotry
WORKDIR /usr/src/app

# env
ENV DB_PORT=$ARG_DB_PORT
ENV DB_NAME=$ARG_DB_NAME
ENV DB_USER=$ARG_DB_USER
ENV DB_PW=$ARG_DB_PW
ENV DB_HOST=$ARG_DB_HOST
ENV DB_PORT=$ARG_DB_PORT

# copy package.json
COPY ./package.json .

# install dependencies
RUN npm install

# COPY server.js
COPY ./server.js .

EXPOSE 3000

CMD ["node","server.js"]
```

### 4.2 도커 빌드시 매개변수를 입력하여 환경변수로 할당
- docker build --build-arg ARG_DB_NAME='changed_arg_name' -t server:v2 .
- docker run -p 3000:3000 -d server:v2

### 4.3 결과
- input: http://localhost:3000
- output :
```code
server port is 3000
db name is changed_arg_name
db user is arg_docker_user
pw is arg_docker_pw
db host is arg_docker_host
db port is arg_docker_port
```


