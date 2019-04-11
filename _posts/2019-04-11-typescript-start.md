---
title: "타입스크립트 시작하기"
date: 2019-04-11 11:15:00 -0400
categories: typescript javascript
---

## TypeScript 시작하기

### 자바스크립트 코드의 안좋은점?
```js
// 자바스크립트는 런타임시에 동적으로 타입을 결정함. 동적 타입 언어
// 타입스크립트는 컴파일 시에 타입 체크를 함. 정적 타입 언어
// 동일한 변수에 문자열, 숫자, 배열, 객체 할당 가능하므로 개발 로직 오류가 증가함
var num = 1;
num += 1;
console.log(typeof (num)); // number
console.log(num);
num = 'str';
console.log(typeof (num)); // string
num += 1;

var score = [1, 2, 3, 4, 5, 6];
console.log(score.slice(2, 4));
score = 10;
console.log(score.slice(2, 4)); // score가 배열인줄 알고 slice 함수를 호출함
```

```js
// ===의 사용은 == 보다 직관적이지 않다.
var result = '1';
console.log(result == 1); // true 문자열을 숫자로 변환하여 비교 
console.log(result === 1); // false ===은 변수의 타입을 강제 변환하지 않고 비교함

// 자바스크립트는 undifined나 null을 검사하는 방법을 제공해 주지 않는다.
// 그래서 변수에 접근할 때 마다 유효성 검사를 해야한다.
function print(data) {
  if (data !== null && data !== undefined) {
    alert(data);
  }
}

print(null);
print(undefined);
print('123');
```

### TypeScript 주요 기능
```ts
// 컴파일 타임에 타입 오류를 잡아준다.
let numVar = 1;
numVar = 'success'; // 에러. 빌드 실패함
```

```ts
// 하나의 변수에 여러 타입을 지정 할 수 있다.
var a: string | number;
a = 10; //가능
a = '10'; // 가능
a = false;  // 불가
```

```ts
// 캡슐화를 통해 필드와 메서드를 하나로 묶고 내용을 은닉화 할 수 있다.
class Person{
  public name: string;
  private phoneNumber: string;

  private log(message: string) {
    console.log(message);
  }

  public sendMessage(message: string) {
    this.log(message);
  }
}
```

```ts
// 상속을 사용하여 부모 클래스의 모든 public 멤버에 접근할 수 있다.
class Animal {
    // 생성자 파라미터에 접근제어자나 readonly와 함께 변수를 선언하면 그 변수는 객체의 프로퍼티로 초기화 된다.
  constructor(public type: string) { }
  log() { console.log(this.type); }
}

class Person extends Animal {
  constructor(public name: string) {
    super('human');
  }
}

class Dog extends Animal {
  constructor(public name: string) {
    super('dog');
  }
}

// 쉐이프(Shape) : 객체의 프로퍼티가 동일하다면 다른 식별자에도 객체를 할당할 수 있는 유연함을 제공
let animal = new Animal('animal');
animal.log();  // animal
animal = new Person('hong');
animal.log(); // human
animal = new Dog('happy');
animal.log(); // dog
```

```ts
// 인터페이스를 지원한다. 인터페이스의 구현체는 해당 인터페이스의 모든 프로퍼티를 구현해야한다.
interface Planet {
  name: string;
  weather: string;
  log(): void;
}

class Earth implements Planet {
  name: string;
  weather: string;
  constructor() { this.name = "earth"; }
  log(): void { console.log(this.name); }
} 

class Sun implements Planet {
  name: string;
  weather: string;
  constructor() { this.name = "sun"; }
  log(): void { console.log(this.name); }
}

// 쉐이프(Shape) : 객체의 프로퍼티가 동일하다면 다른 식별자에도 객체를 할당할 수 있는 유연함을 제공
let planet = new Earth();  
planet.log(); // earth
planet = new Sun();
planet.log(); // sun
```


### 타입 시스템(변수의 유효 범위)
[javascript var, let, const 차이점](https://gist.github.com/LeoHeo/7c2a2a6dbcf80becaaa1e61e90091e5d)

[JAVASCRIPT VAR 선언과 생략하는 경우의 차이점](http://www.freesens.com/x/?p=225)

함수 안에서 키워드 없이 변수를 선언했을 경우와 var 키워드로 변수를 선언했을 때 비교
```js
// summary 함수가 실행되고 나면 result변수는 전역변수로 등록됨
function summary() {
    result = 100;
}
console.log(typeof(result)); // undefined
console.log(result); // error
summary();
console.log(typeof(result)); // number
console.log(result); // 100

// 함수 안에서 var 키워드를 사용하여 변수를 선언하면 해당 변수는 함수 내에서 유효함
function summary() {
    var result = 100;
}
console.log(typeof(result)); // undefined
summary();
console.log(typeof(result)); // undefined
```

중첩함수에서의 지역변수
```js
// 중첩함수에서 내부함수는 외부 함수의 var const let변수에 접근 가능하다.
function outerFunction() {
  var value = 10; // const나 let도 동일하게 내부함수에서 접근 가능하다.
  function innerFunction() {
    console.log(value); // 내부함수에서 value 접근 가능
  }
  return innerFunction;
}
var func = outerFunction();
func();
```

var 키워드(function-scoped)와 끌어올림(hoisting)
```js
// var로 선언된 키워드는 끌어올림의 대상이 되기 때문에 변수가 선언되기 전에 사용하더라도 오류가 발생하지 않는다.
function scopingExample(hasValue) {
  if (hasValue) {
    value = 1;
  } else {
    value = 0;
  }
  console.log(value);
  var value; //let과 const 키워드는 끌어올림을 지원하지 않음.
}

scopingExample(true);
scopingExample(false);
```

var 키워드 변수의 중복선언 허용(let과 const는 중복선언 불가)
```js
var message = 'hello'
var message = 'hi'
console.log(message);
```

let은 선언하고 나중에 값 할당 가능
```js
let hello;
hello = 'hi';
console.log(hello);
```

const는 선언과 동시에 값을 할당해야함
```js
const hello = 'hi';
console.log(hello);
```

### TypeScript의 타입



### TypeScript의 클래스





***
참고도서: 예제로 배우는 타입스크립트 2.x  
저자: 사친 오호리  
옮김: 김창수  
출판사: 터닝포인트  
