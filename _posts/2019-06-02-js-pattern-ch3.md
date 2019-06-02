# 3장 리터럴과 생성자
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

