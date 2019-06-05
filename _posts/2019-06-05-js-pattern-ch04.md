---
title: "자바스크립트 함수"
date: 2019-06-05 09:00:00 -0400
categories: javascript design-pattern pattern
---

## 함수
### 함수의 특징
- 함수는 [일급(first-class) 객체](https://ko.wikipedia.org/wiki/%EC%9D%BC%EA%B8%89_%EA%B0%9D%EC%B2%B4)다.
- 함수는 유효범위(scope)를 제공한다.

### 함수 선언문(function declaration)
변수나 프로퍼티에 할당 할 수 없고, 함수 호출 시 인자로 함수를 넘길때도 사용할 수 없다. 전역 유효범위나 다른 함수의 본문 내부에서만 쓸 수 있다.
```js
function add (a, b) {
    return a + b;
}
add.name // add
```

자바스크립트는 함수 선언문을 호이스팅한다.
```js
console.log(add(1, 3)); // 4
function add (a, b) {
    return a + b;
}
```

### 함수 표현식(function expression)
변수나 프로퍼티에 할당할 수 있고, 함수 호출 시 인자로 함수를 넘길때 사용할 수 있다.

```js
// 기명함수 표현식(named function expression)
var add = function addFunction (a, b) {
    return a + b;
};
add.name // addFunction
```
 
```js
// 무명 함수 표현식(unnamed function expression, anonymous function)
var add = function (a, b) {
    return a + b;
};
add.name // add
```

자바스크립트는 함수 표현식의 변수 선언 부분을 호이스팅하고 함수 정의 부분은 호이스팅 하지 않는다.
```js
add(1,3); // TypeError: add is not a function
var add = function (a,b) {
    return a + b;
}
```





## 콜백 패턴
## 함수 반환하기
## 자기 자신을 정의하는 함수
## 즉시 실행 함수
## 즉시 객체 초기화
## 초기화 시점의 분리
## 함수 프로퍼티 - 메모제이션(Memoization) 패턴
## 설정 객체 패턴
## 커리(Curry)
## 요약

