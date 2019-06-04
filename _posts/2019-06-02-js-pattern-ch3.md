---
title: "자바스크립트 리터럴과 생성자"
date: 2019-06-02 19:20:01 -0400
categories: javascript design-pattern pattern
---

자바스크립트의 리터럴 표기법 패턴을 사용하면 좀더 정확하고 에러율은 낮은 방식으로 객체를 정의할 수 있다.

## 3.1 객체 리터럴
```js
// 리터럴 사용
var car = {goes: "far"};

// 내장 생성자 함수 사용
// 안티패턴
var car = new Object();
car.goes = "far";
```
리터럴 표기법의 장점
- 더 짧다.
- 객체란 단지 이름-값 쌍의 해시 테이블(프로퍼티의 모음)임을 확실히 보여준다.

내장 생성자 함수의 단점
- 지역 유효범위에 동일한 이름의 생성자가 있을 수 있기 때문에 Object()를 호출한 위치에서부터 전역 Object 생성자까지 인터프리터가 쭉 거슬러 올라가며 유효범위를 검색해야한다.

인터프리터의 유효범위 검색 발생(Object 함수의 위치를 찾는다.)
```js
function abcd() {
    var Object = function() {
    	this.a = 1;
    }
    var obj = new Object();
    console.log(obj);   // {a: 1}
    console.log(obj.a); // 1
}
```

>"new Object()를 사용하지 말고, 객체 리터럴을 사용하라!"

## 3.2 사용자 정의 생성자 함수
직접 생성자 함수를 만들어 객체를 생성할 수 있다.

다음 코드는 자바에서 Person이라는 클래스를 사용하여 객체를 생성하는 방식과 유사하다.
```js
var adam = new Person("Adam");
adam.say(); // "I am Adam"
```

다음 코드는 Person 생성자 함수를 정의한 코드이다.
```js
var Person = function (name) {
    this.name = name;
    this.say = function() {
        return "I am " + this.name;
    }
}
```

Person 생성자 함수를 자세히 들여다보면 다음 코드와 같은 방식으로 작동한다.
```js
var Person = function (name) {
    // 객체 리터럴로 새로운 객체를 생성한다.
    // var this = {};

    // 프로퍼티와 메서드를 추가한다.
    this.name = name;
    this.say = function () {
        return "I am " + this.name;
    };

    // this를 반환한다.
    // return this;
}
```

## 3.3 new를 강제하는 패턴
생성자를 호출할 때 new를 빼고 호출하면 생성자 내부의 this는 전역객체를 가리키게 된다.(브라우저에서라면 this는 window를 가리킨다.)

다음 코드는 new를 빼고 생성자를 호출한 코드이다.
```js
function Cup() {
    this.name = "mug";
}

// new를 사용하여 호출
var cup = new Cup();
cup.name // mug

// new를 생략하여 호출
var cup = Cup();
cup.name // cannot read property 'name'

// 전역객체에 name이 프로퍼티가 생성됨
name // mug
```

생성자를 호출 할때 new를 빼면 문법오류나 런타임 에러가 발생하지는 않지만, 논리적인 오류가 생겨 예기치 못한 결과가 나올 수 있다.

다음과 같은 코드를 통해서 new를 빼고 생성자를 호출하는 문제를 해결할 수 있다.
```js
function Cup() {
    // this가 해당 생성자의 인스턴스가 아니면 new를 사용하여 생성자를 재호출
    if (!(this instanceof Cup)) {
        return new Cup();
    }
    this.name = "mug";
}

var cup = Cup();
cup.name // mug
```

## 3.4 배열 리터럴
배열이란 0에서 인덱스를 시작하는 값이 목록이므로 배열 리터럴 문법이 더 간단하고 직관적이다.
```js
// Array() 생성자 사용: 안티패턴
var a = new Array("itsy", "bitsy", "spider");

// 리터럴 사용
var a = ["itsy", "bitsy", "spider"];
```

Array() 생성자에 숫자 하나를 전달할 경우, 이 값은 배열의 첫번째 원소값이 되는게 아니라 배열의 길이를 지정하게 되므로 오류가 있는 코드를 작성하기 쉽다.
```js
// 한 개의 원소를 가지는 배열
var a = [3];
a.length; // 1
a[0] // 3

// 세개의 원소를 가지는 배열
var a = new Array(3);
a.length; // 3
a[0] // "undefined" 
```

new Array()에 정수가 아닌 부동소숫점을 가지는 수를 전달할 경우 에러가 발생한다.
```js
var a = [3.14];
a[0] // 3.14

var a = new Array(3.14); // Invalid array length
```

해당 객체가 배열인지 확인하는 방법은 Array.isArray() 메서드를 사용하면 된다.
```js
Array.isArray([]); // true
```

## 3.5 JSON
JSON은 자바스크립트 객체 표기법(JavaScript Object Notation)의 준말로, 데이터 전송형식의 일종이다. JSON은 프로퍼티명을 따옴표로 감싸야 한다는 점이 객체 리터럴과 유일한 문법적 차이다.

JSON의 예
```json
{"name": "value", "some": [1, 2, 3]}
```

JSON 문자열을 객체로 변환할 때는 JSON.parse()를 사용하고, 객체를 JSON 문자열로 변환하기 위해서는 JSON.stringify() 메서드를 사용하면 된다.
```js
var jstr = `{"mykey": "my value"}`;
// JSON to Object
var data = JSON.parse(jstr);
data // {mykey: "my value"}

var dog = {
    name: "Fido",
    dob: new Date(),
    legs: [1, 2, 3, 4]
};
// Object to JSON
var jsonstr = JSON.stringify(dog);
jsonstr // {"name":"Fido","dob":"2019-06-02T12:26:03.589Z","legs":[1,2,3,4]}
```

## 3.6 정규 표현식 리터럴
자바스크립트에서 정규식은 객체이며 new RegExp() 생성자를 사용하거나 정규식 리터럴을 사용하여 생성할 수 있다.

다음 정규식은 역슬래시(\) 하나에 매치되는 정규식인데, 리터럴은 정규식의 역슬래시 문자 하나만을 이스케이프 해주면 되므로 역슬래시 문자가 두개 쓰인 반면, new RegEx() 생성자를 사용한 방식은 문자열의 역슬래시 자체를 이스케이프 해주어야 하기 때문에 역슬래시 문자가 네개 쓰였다.
```js
// 정규식 리터럴
var re = /\\/gm;
// 생성자
var re = new RegExp("\\\\","gm");
```

정규식의 리터럴 표기법은 매칭에 사용되는 정규식 패턴을 슬래시로 감싸고 두번째 슬러시 뒤에는 패턴변경자를 쓴다.

패턴 변경자
- g: 전역 매칭
- m: 여러줄 매칭
- i: 대소문자 구분 없이 매칭

패턴 변경자 g(전역 매칭)의 예제
```js
//한 개 이상의 문자열 뒤에 공백이 하나 있는 패턴
var re = /\w+\s/;
var str = "fee fi fo fum";
var myArray = str.match(re);
console.log(myArray); // ["fee "]

//한 개 이상의 문자열 뒤에 공백이 하나 있는 패턴 : 글로벌 검색
var re = /\w+\s/g;
var str = "fee fi fo fum";
var myArray = str.match(re);
console.log(myArray); // ["fee ", "fi ", "fo "]
```

정규식 리터럴을 사용하면 정규식 객체를 인자로 받는 String.prototype.repalce()와 같은 메서드를 호출할 때 좀더 정확한 코드를 작성할 수 있다.
```js
var no_letter = "abc123XYZ".replace(/[a-z]/gi);
no_letters // 123
```

그러나 매칭시킬 패턴을 미리 알 수 없고 런타임에 문자열로 만들어지는 경우네는 new RegExp()를 사용해야한다.

>일반적인 경우에는 정규식 리터럴을 사용하고 매칭시킬 패턴의 문자열이 런타임에 만들어 지는 경우에는 new RegExp()를 사용하라.

## 3.7 원시 데이터 타입 래퍼
숫자, 문자열, 불린의 원시 데이터 타입에는 래퍼 객체가 있다.

```js
var n = 100;
console.log(typeof n); // number

var nobj = new Number(100);
console.log(typeof nobj); // object
```

래퍼 객체에는 유용한 프로퍼티와 메서드가 있다.
- 숫자 객체: toFixed(), toExponential()
- 문자열: substring(), charAt(), toLowerCase(), length

원시 데이터 타입에서도 래퍼 객체의 메서드를 사용할 수 있는데, 메서드를 호출하는 순간 내부적으로는 원시 데이터 타입 값이 객체로 임시 변환되어 객체처럼 동작한다.
```js
var s = "hello";
console.log(s.toUpperCase()); // "HELLO"
```

원시 데이터 타입 값도 언제든 객체처럼 쓸 수 있기 때문에 장황한 래퍼 생성자를 쓸 필요는 사실 별로 없다.
```js
// 안티 패턴
var s = new String("my string");
var n = new Number(101);
var b = new Boolean(true);

// 권장안
var s = "my string";
var n = 101;
var b = true;
```

단, 값을 확장하기 위해 래퍼 객체를 쓰는 경우가 있다. 원시 데이터 타입은 객체가 아니기 떄문에 프로퍼티를 추가하여 확장할 수 없다.

```js
var str = "simple";
str.expand = "expand"; // 원시 타입을 확장할 경우 에러는 발생하지 않는다
str.expand // undefined

var str_wrapper = new String("simple");
str_wrapper.expand = "expand";
str_wrapper.expand // expand
```

## 3.8 에러 객체
자바스크립트에는 Error(), SyntaxError(), TypeError() 등 여러가지 에러 생성자가 내장되어 있다.

에러 생성자의 프로퍼티
- name : 에러 객체의 이름=> 일반적으로 생성자 함수의 이름이 된다.
- message: 객체를 생성할 때 생성자에 전달된 문자열

```js
function throwError() {
  throw new Error("occur error!");
}

try {
  throwError();
}
catch(e) {
  console.log(e.name); // Error
  console.log(e.message); // occur error!
}
```

throw문은 에러 생성자를 사용하지 않고 직접 에러 객체를 만들어 사용할 수 있다.

```js
function throwError() {
  throw {
	name: "custom error",
	message: "oops",
	extra: "extra property"
  }
}

try {
  throwError();
}
catch(e) {
  console.log(e.name); // custom error
  console.log(e.message); // oops
  console.log(e.extra); // extra property
}
```

## 정리
|내장 생성자(사용 자제)|리터럴과 원시 데이터 타입(권장안)|
|---|---|
|var o = new Object();   |var o = {};   |
|var a = new Array();   |var a = [];   |
|var re = new RegExp("[a-z]","g");   |var re = /[a-z]g/;   |
|var s = new String()   |var s = "";   |
|var n = new Number();   |var n = 0;   |
|var b = new Boolean();   |var b = false;   |
|throw new Error("un-oh");   |throw {name:"Error",message:"uh-oh"};<br> ... 또는 throw Error("uh-oh")   |


