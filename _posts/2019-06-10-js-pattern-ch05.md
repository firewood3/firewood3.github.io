---
title: "자바스크립트 객체 생성 패턴"
date: 2019-06-10 17:00:00 -0400
categories: javascript design-pattern pattern
---

## 네임스페이스 패턴
네임스페이스는 전역 객체를 하나 만들고 모든 기능을 이 객체에 추가한다. 주의해야 할 점은 추가하려는 프로퍼티가 이미 존재할 수도 있기 때문에, 추가하려는 프로퍼티가 존재하는지 여부를 검사하는 해야한다.

네임스페이스를 적용하지 않은 코드
```js
function Parent() {};
function Child() {};

var some_var = 1;

var module1 = {};
module1.data = {a: 1, b: 2};
var module2 = {};
```

네임스페이스를 적용한 코드
```js
var MYAPP = MYAPP || {};

MYAPP.some_var = 1;

MYAPP.Parent = function () {};
MYAPP.Child = function () {};

MYAPP.modules = {};

MYAPP.modules.module1 = {};
MYAPP.modules.module1.data = {a: 1, b: 2};
MYAPP.modules.moudle2 = {};
```

## 의존 관계 선언
의존 관계 선언 패턴은 함수나 모듈 내 최상단에 의존관계가 있는 모듈을 선언하는 것이다.

장점
- 함수의 첫 머리에 의존 관계가 선언되기 때문에 의존 관게를 찾아내고 이해하기 쉽다.
- 의존 관계가 명시적으로 선언되어 있기 때문에 코드를 사용하는 사람이 페이지 내에 반드시 포함 시켜야하는 스크립트 파일이 무엇인지 알 수 있다.
- dom과 같이 지역변수를 사용하는 것이 YAHOO.util.dom처럼 전역 변수의 중첩프로퍼티를 사용하는 것 보다 빠르다.

```js
var myFunction = function() {
    // 의존관계가 있는 모듈을 지역변수로 선언한다.
    var event = YAHOO.util.Event, 
        dom = YAHOO.util.dom;

    // event와 dom을 사용한다.
}
```

## 비공개 프로퍼티와 메서드
자바스크립트에서는 비공개 멤버에 대한 별도의 문법이 없지만 클로저를 사용해서 구현할 수 있다. 생성자 함수 안에서 클로저를 만들면, 클로저 유효범위 안의 변수는 생성자 함수 외부에 노출되지 않지만 객체의 공개 메서드 안에서는 쓸 수 있다.

```js
function Gadget() {
    // 비공개 멤버
    var name = 'iPod';
    //공개된 함수
    this.getName = function () {
        return name;
    };
}

var toy = new Gadget();
console.log(toy.name); // undefined
console.log(toy.getName()); // iPod
```

클로저에서 객체를 반환하는 경우에는 새로운 객체를 만들어 반환해야한다.
```js
function Gadget() {
    // 비공개 멤버
    var specs = {
        screen_width: 320,
        screen_height: 480,
        color: "white"
    };

    // 공개함수
    this.getSpecs = function () {
        var clonedSpecs = {};
        clonedSpecs.screen_width = specs.screen_width;
        clonedSpecs.screen_height = specs.screen_height;
        clonedSpecs.color = specs.color;
        return clonedSpecs;
    };
}
```

위 방식은 비공개 프로퍼티를 가지는 객체를 생성자 함수로 만드는 방식이었다. 다음은 리터럴 객체로 비공개 멤버를 구현한 코드이다.

```js
var myobj;
(function () {
    // 비공개 멤버
    var name = "my, oh my";

    myobj = {
        getName: function () {
            return name;
        }
    };
}());

myobj.getName(); // my, oh my
```

```js
var myobj = (function () {
    // 비공개 멤버
    var name = "my, oh my";

    return {
        getName: function () {
            return name;
        }
    };
}());

myobj.getName(); // my, oh my
```

생성자를 사용하여 비공개 멤버를 만들 경우, 생성자를 호출할 때마다 비공개 멤버가 재 생성된다. 생성자의 프로토타입에 프로퍼티와 메서드를 추가하면 메모리를 절약하고 비공개 멤버가 재 생성되지 않도록 할 수있다.

```js
function Gadget() {
    // 비공개 메서드
    var name = 'iPod';

    // 공개함수
    this.getName = function () {
        return name;
    }

    this.setName = function (arg) {
        name = arg;
    }
}

// 생성자의 프로토타입에 비공개 멤버와 공개 함수를 추가한 경우
Gadget.prototype = (function () {
    // 비공개 멤버
    var browser = "Mobile Webkit";

    return {
        // 공개 함수
        getBrowser: function () {
            return browser;
        },
        setBrowser: function (arg) {
            browser = arg;
        }
    };
}());

var gadget1 = new Gadget();
var gadget2 = new Gadget();

gadget1.setName('gadget1'); 
gadget2.setName('gadget2');
console.log(gadget1.getName()); // gadget1
console.log(gadget2.getName()); // gadget2

gadget1.setBrowser('Edge');
gadget2.setBrowser('Chrome');
console.log(gadget1.getBrowser()); // Chrome
console.log(gadget2.getBrowser()); // Chrome
```

