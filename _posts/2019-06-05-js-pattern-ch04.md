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
### 콜백 함수
[콜백함수](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function)는 다른 함수의 매개변수로 입력되는 함수이며, 외부 함수 내부에서 호출되어 일종의 루틴이나 액션을 완료한다.
```js
function greeting(name) {
  alert('Hello ' + name);
}

function processUserInput(callback) {
  var name = prompt('Please enter your name.');
  callback(name);
}

processUserInput(greeting);
```

### 콜백과 유효범위
콜백이 객체의 메서드일 때, 콜백안에서 콜백이 속해있는 객체를 참조하기 위해 this를 사용하기 위해서는 [call()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)이나 [apply()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 함수를 사용해 콜백이 속해있는 객체를 바인드 해야 한다. (콜백이 일회성의 익명 함수나 전역 함수 인 경우는 문제가 되지 않는다.)

콜백 안의 this가 콜백이 속해있는 객체를 참조한 것이 아니라 전역의 window를 참조한 경우
```js
var myapp = {};
myapp.color = "green";
myapp.paint = function (item) {
    item.color = this.color;
}

var canvas = function (callback) {
    var item = {};
    callback(item);
    console.log(item.color);
}

canvas(myapp.paint);

// 콘솔출력
// undefined
```

콜백 안의 this가 콜백이 속해있는 객체를 참조하기위해 call() 메소드를 사용하여 바인드 한 경우
```js
var myapp = {};
myapp.color = "green";
myapp.paint = function (item) {
    item.color = this.color;
}

var canvas = function (callback, callback_obj) {
    var item = {};
    callback.call(callback_obj, item);
    console.log(item.color);
}

canvas(myapp.paint, myapp);

// 콘솔 출력
// green
```

### 비동기 이벤트 리스너
페이지의 엘리먼트에 이벤트 리스너를 붙이는 것은 실제로 이벤트가 발생했을 때 호출될 콜백 함수의 포인터를 전달하는 것이다.

```js
document.addEventListener("click", console.log, false);
```

## 함수 반환하기
함수는 객체이기 때문에 반환 값으로 사용될 수 있다.

함수 반환 예제: private 데이터 저장을 위해 [클로저](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)를 사용
```js
var setup = function () {
    var count = 0; // private 변수
    return function () {
        return (count += 1);
    };
};

// next() 함수에서는 setup() 함수의 count에 접근 불가.
var next = setup();
next(); // 1
next(); // 2
next(); // 3
```

## 자기 자신을 정의하는 함수
함수의 본문 내에서 이전의 함수 포인터가 새로운 함수를 가리키도록 하므로써 자기자신을 정의하는 함수를 만들 수 있다.

- 장점: 어떤  초기화 준비 작업을 단 한번만 수행할 경우에 유용하다.
- 단점: 자기 자신을 재정의한 이후에는 이전에 원본 함수에 추가했던 프로퍼티들을 모두 찾을 수 없다.

```js
var exFunction = function () {
    console.log("foo");
    exFunction = function () {
        console.log("boo");
    }
}

exFunction.property = "property";

exFunction.property // property
exFunction(); // foo
exFunction.property; // undefined
exFunction(); // boo
exFunction(); // boo
```

## 즉시 실행 함수
## 즉시 객체 초기화
## 초기화 시점의 분리
## 함수 프로퍼티 - 메모제이션(Memoization) 패턴
## 설정 객체 패턴
## 커리(Curry)
## 요약

