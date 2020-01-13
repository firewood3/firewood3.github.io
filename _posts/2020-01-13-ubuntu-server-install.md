---
title: "우분투 서버 설치 기록"
date: 2020-01-13 10:01:12 -0400
categories: ubuntu-server-install
---

레노버 SR530 서버에 우분투를 설치한 기록을 남긴다.  

[OS설치](##-1.-OS-설치)  
[openssh 설치](##-2.-ssh-설치)  
[타임존 변경](##-3.-OS-타임존-변경-UTC-=>-KST)  
[Maria DB 설치](##-4.-Maria-DB-설치)  
[아파치 설치](##-5.-아파치-설치)  
[php 설치](##-6.-php-설치)  
[openjdk 설치](##-7.-openjdk-11-설치)  

## 1. OS 설치
1. 우분투 다운로드 : Ubuntu on Lenovo ThinkSystem SR530 : https://certification.ubuntu.com/hardware/201706-25573
2. 우분투 부트 usb 만들기: https://hiseon.me/linux/ubuntu/ubuntu-install-usb/
    - 필자는 윈도우에서 Rufus 프로그램을 사용하여 부트 usb를 만듦
    - USB는 3.0이상의 16GB이상을 사용하는 것을 추천
3. 두 가지 OS 설치 방법:  
SR530을 시작하면 바이오스 화면이 나타나는데 여기서 리눅스를 설치하기 위한 방법은 2가지가 있다. 첫째는 OS의 installer를 사용하여 설치하는 방법이고, 다른 하나는 직접 리눅스 서버를 USB로 부트하여 설치하는 방법이다.  OS의 Installer를 사용하게 되면 Enterprise 버전의 Redhat 기반의 리눅스를 설치하게 된다. 필자는 우분투 서버를 USB로 설치하였다.
4. 우분투 설치:
우분투 이미지가 들어있는 USB를 서버의 USB포트에 연결시키고 서버의 파워를 켠다. 바이오스 화면에서 F12 를 계속 누른다. 그러면 어떤 장치로 시작할지 정할 수 있다. USB flush drive를 선택하면 우분투의 설치가 진행된다. 설치가 완료된 후 서버를 재부팅 하면 다시 바이오스 화면이 뜨는데 이때 F12를 계속해서 누르면 어떤 장치로 시작할지 정할 수 있는데 이때 맨 위에 [Ubuntu]라고 나와있는 부분을 선택하고 Enter를 입력하면 Ubuntu가 부팅될 것이다.

## 2. ssh 설치
22번 포트를 사용하여 ssh 연결을 하기 위해서는 openssh-server를 설치해야한다.  
1. 설치확인 : dpkg -l | grep openssh
2. 설치: sudo apt-get install openssh-server
3. ssh 시작: sudo service ssh start
4. 서버 시작 확인: service --status-all | grep +
5. 점유 포트 확인: sudo netstat -antp

## 3. OS 타임존 변경 UTC => KST
우분투를 설치하고 나면 타임존이 UTC로 설정되어있다. 터미널에서 date 명령어를 치면 현재 시간을 알 수 있다.(타임존과 함께 표시된다.) 타임존을 KST로 변경하여 현지 시간에 맞출 필요가 있다.
```code
date // 현재 시간 확인 (UTC)
sudo update
tzselect // tzselect 명령어를 실행후 KST로 변경
sudo ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime // 링크걸기. 링크를 걸지 않으면 반영안됨.
date // 현재 시간 확인 (KST)
```
## 4. Maria DB 설치
### 4.1 DB 설치
```code
sudo apt install mariadb-server
```
### 4.2 루트 비밀번호 등록
1. $sudo mysql -u root use mysql;
2. update user set password=password('비밀번호') where user='root';
3. flush privileges;
### 4.3 외부에서 Maria DB를 사용할 수 있도록 설정
데이터베이스를 설치한 직후 원격으로 데이터베이스 접근하려 하면 계속 실패하게 된다. 데이터베이스에 접근할 수 있는 IP가 로컬 호스트만 허용되고 다른 IP는 허용되지 않아서 접근이 실패하는 것이다.  

다음과 같은 위치의 파일을 열어서
```code
/etc/mysql/mariadb.conf.d/50-server.cnf
```
다음과 같이 변경한다.
```code
#bind-address = 127.0.0.1
bind-address = 0.0.0.0
```

재실행을 위해서 다음과 같은 명령어를 입력한다.
```code
sudo systemctl restart mariadb.service 
sudo systemctl restart mysql.service 
sudo service mariadb restart
sudo service mysql restart
```

## 5. 아파치 설치
다음과 같이 설치하고 브라우저에서 서버의 ip로 접근하면 /var/www/html 경로의 index.html 페이지가 보일 것이다.
```code
sudo apt install apache2
apache2 -v
```

## 6. php 설치
아래와 같은 명령어를 입력해 설치한다.
```code
sudo apt install php
php -v
```

php에서 MariaDB를 사용하기 위해 php-mysql을 설치한다.
```code
apt install php-mysql
```

적용하기 위해 아파치를 재시작한다.
```code
service apache2 restart
```

/var/www/html 경로에 phpinfo.php파일을 다음과 같이 만든다.
```code
<?php phpinfo();?>
```
브라우저에서 [서버 ip]]/phpinfo.php로 접근하면 [Apache2 Ubuntu Default Page] 페이지가 보일 것이다.

## 7. openjdk-11 설치
기존 JDK의 저작권 문제가 대두되어 오픈소스 진영에서 JDK를 대체하기 위해 openjdk를 만들었다. JDK는 MYSQL처럼 라이센스가 불투명 하므로 openjdk를 설치하여 사용할 것을 권장한다.
```code
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt install openjdk-11-jdk
java -version
javac -version
```
