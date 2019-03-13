---
title: "스프링 부트로 RestAPI 만들기"
date: 2019-03-13 15:25:00 -0400
categories: java spring
---

[firewood3's GitHub Code](https://github.com/firewood3/assignment)


## 공지사항을 불러오는 RestAPI


**모든 공지사항 불러오기**
- method: get  
- url: /notice/get/all  
- Http 응답: 성공-200  
- responseBody
```json
[
    {
        "id": 1,
        "title": "[EBS 직업 유튜브 구독 이벤트 당첨 안내]",
        "content": "<EBS직업 유튜브 구독 이벤트> 구독 및 댓글 이벤트에 참여해주셔서 감사합니다.\n\n\n\n이벤트 상품에 당첨되신 분들은 아래와 같습니다."
    },
    {
        "id": 2,
        "title": "EBS 무료 이용 안내",
        "content": "1. 서비스 대상 및 내용\n1) 서비스 대상 : 기초생활수급자, 국가 유공자, 장애인 각 본인만 해당\n2) 서비스 내용\n가. EBS 방송 VOD/AOD 무료 이용 가능합니다.\n나. 중학 프리미엄 무료 이용 가능합니다.\n다. EBS 명품 공인중개사, 공무원 온라인 강좌 50% 할인됩니다."
    },
    ...
]
```

**id 별로 공지사항 불러오기**
- method: get
- url: /notice/get/one/{id}
- Http 응답: 성공-200, 실패-404
- responseBody
```json
{
    "id": 3,
    "title": "EBS 사이트 시스템 점검 안내 (02월 18일)",
    "content": "안녕하세요 EBS입니다.\n\nEBS를 이용해 주시는 회원 여러분께 진심으로 감사 드립니다.\n\nEBS 사이트 안정화를 위해 서비스 점검을 아래와 같이 계획하고 있습니다.\n\n서비스 이용에 불편을 드린 점 양해를 부탁드립니다. (일부 일정은 변동될 수 있습니다.)"
}
```

**공지사항 추가하기**
- method: post
- url: /notice/create
- Http 응답: 성공0-200, 실패-400
- requestBody :
```json
{
    "title": "",
    "content" :"",
}
```
- responseBody :
```json
{
    "id": 7,
    "title": "새로운 공지사항 생성 제목",
    "content": "새로운 공지사항 생성 내용"
}
```


**공지사항 수정하기**
- method: put
- url: /notice/update/{id}
- Http 응답: 성공-200, 실패-400
- requestBody :
```json
{
    "title": "",
    "content" :"",
}
```
-responseBody :
```json
{
    "id": 3,
    "title": "업데이트 공지사항",
    "content": "업데이트 공지사항"
}
```

**공지사항 삭제하기**
- method: delete
- url: /notice/delete/{id}
- http 응답: 성공-200, 실패-404


## 다국어를 불러오는 API

**다국어 불러오기**
- method: get
- url: /word/{language_code}/{key}
- http 응답: 성공-200, 실패-400
- 지원하는 language_code : en, de, ko
- 지원하는 key : home, program
- responseBody :
```json
{
    "key": "home",
    "word": "ko",
    "language": "나는 집에 산다."
}
```

