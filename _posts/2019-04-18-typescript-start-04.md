---
title: "Angular 간단 바인딩 및 프로미스 정리"
date: 2019-04-18 15:29:00 -0400
categories: typescript angular javascript
---

## Angular 간단 바인딩 정리
- 지시자(Directive): angular의 HTML Compiler에 의해 해석된 특정한 행위의 기능을 가진 DOM Element.
- @Input() : 상위 컴포넌트와 하위 컴포넌트의 프로퍼티를 바인딩 할 때 사용됨
- @Output(): EventEmitter와 함께 하위 컴포넌트의 데이터를 상위 컴포넌트로 전달할 때 사용함
- ElementRef: DOM Element의 참조를 지원하는 클래스
```ts
el: ElementRef;
const input = this.el.nativeElement
  .getElementsByClassName('board-title')[0]
  .getElementsByTagName('input')[0];
setTimeout(function () { input.focus(); }, 0);
```
- 스타일 바인딩: [style.속성]="속성값" 
```ts
editingTitle = false;
```
```html
<span [style.display]="editingTitle ? '' : 'none'">
    {{title}}
</span>
```
- 이벤트 바인딩
```ts
interface GlobalEventHandlers {
    ...
    onclick: ((this: GlobalEventHandlers, ev: MouseEvent) => any) | null;
    ...
}
```
```html
<!-- $event는 mouseEvent이다. -->
<div (click)="enableAddtask($event)" >
</div
```
- HTML 테그 속성 바인딩
```html
<div class="task" [attr.task-id]="task.id">
```
- Built-in pipe와 Custom Pipe
```ts
@Pipe({name: 'customSort'})
export class CustomSort implements PipeTransform {
  transform(value: Task[] | SubTask[], sort: boolean): Task[] | SubTask[]{
    if(sort) {
      return value.sort(this.compare);
    }else {
      return value;
    }
  }

  private compare(a,b) {
    if(a.title < b.title)
      return -1;
    if(a.title > b.title)
      return 1;
    return 0;
  }
}
```
- [Angular Template Syntax](https://angular.io/guide/template-syntax) 
- [Angular pipes](https://angular.io/guide/pipes) 
- [Angular lifecycle hook](https://angular.io/guide/lifecycle-hooks)
***

### 상위 컴포넌트의 프로퍼티와 하위 컴포넌트의 프로퍼티를 바인딩하기

**상위 컴포넌트의 Directive**
```html
<하위컴포넌트의Directive
    [하위 컴포넌트의 프로퍼티]="상위 컴포넌트의 프로퍼티"
    >
</하위컴포넌트의Directive>
```

**하위 컴포넌트의 프로퍼티에 @Input() 데코레이션 추가**
```ts
@Input()
하위프로퍼티: any;
```

### 하위 컴포넌트가 상위 컴포넌트에게 이벤트 전달하기

**하위 컴포넌트에서 EventEmitter 추가**
```ts
@Output()
dataEmitter: EventEmitter<any>;
```

**상위 컴포넌트의 HTML**
```html
<하위컴포넌트의Directive
    (하위 컴포넌트의 Emitter(dataEmitter))="ParentFunction(event)"
    >
</하위컴포넌트의Directive>
```

***

## 반복자
- 반복자는 값에 순차적으로 접근하기 위한 수단이다. 
- TypeScript에서 반복자는 배열, 맵, 세트와 문자열의 요소에 차레로 접근할 수 있게 해준다.
- 반복 가능한 모든 객체에는 Symbol.iterator 프로퍼티가 있어야 한다.

### JavaScript 반복자

```js
// 문자열에 Symbol.iterator 프로퍼티가 있음
let stringArray = "Learning TypeScript";
console.log(stringArray);
for(let c of stringArray){
    console.log(c);
}
console.log("-------");

// 명시적으로 반복자에 접근하기
let explicitly = "Learing TypeScript";
let iter = explicitly[Symbol.iterator]();
console.log(iter.next().value); // L
console.log(iter.next().value); // e
console.log(iter.next().value); // a

//반복자의 next()는 value와 done을 반환함
//value는 반복값을 반환함
//done은 반복이 끝났는지 여부를 반환함
```

### TypeScript 반복자

타입스크립트의 반복자 정의
```ts
interface IteratorResult<T> {
    done: boolean;
    value: T;
}

interface Iterator<T> {
    next(value?: any): IteratorResult<T>;
    return?(value?: any): IteratorResult<T>;
    throw?(e?: any): IteratorResult<T>;
}
```

타입스크립트 사용자 정의 반복자
```ts
class customCounter implements Iterator<number> {
    private calculatedVal: number = 0;
    next(value?: any): IteratorResult<number> {
        this.calculatedVal = this.calculatedVal > 99 ? 0 : ++this.calculatedVal;
        return {
            done: false,
            value: this.calculatedVal
        }
    }
}

// customCounter Iterator를 구현했으므로 반복할 수 있는 객체이다.
let c = new customCounter();
for (let i = 0; i < 101; i++) {
    console.log(c.next().value);
}
```

### TypeScript for...of와 for...in 루프

for...of 루프 : 요소의 값 반환
```ts
let sampleArray = ["typeScript", "Angular", "Node"];

for (let val of sampleArray) {
    console.log(val); // typeScript, Angular, Node
}
```

for...in 루프 : 요소의 인덱스 반환
```ts
let sampleArray = ["typeScript", "Angular", "Node"];

for (let val in sampleArray) {
    console.log(val); // 0, 1, 2
}
```

***

## 비동기 프로그래밍(콜백과 프로미스)
[동기와 비동기, 블록킹과 논블록킹](https://stackoverflow.com/questions/2625493/asynchronous-vs-non-blocking)

- A머신이 B머신을 호출 후 B머신의 응답이 도착하기 전까지 다른 작업을 할 수 있다.
<br>A머신과 B머신은 비동기이다. A머신은 논블록킹이다.
- A머신이 B머신을 호출 후 B머신의 응답이 도착하기 전까지 다른 작업을 할 수 없다.
<br>A머신과 B머신은 동기이다. A머신은 블록킹이다.


### 콜백 함수
- 콜백함수는 비동기 처리가 완료된 후에 호출되는 예약 함수.


인터페이스를 사용해서 콜백함수 만들기
```ts
interface ICallBack {
    (boards: Board[], status: string): void;
}

// 콜백 함수 인자를 인터페이스 타입으로 받음
function doWork(clientName: string, callback: ICallBack): void{
    let reesponse: Board[];

    callback(response, "Success");
}
```

### 프로미스(Promise)
[참고: developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- 프로미스는 비동기 호출을 감싸주는 객체이다.
- 프로미스는 비동기 코드를 가지며 콜백함수를 판별하여 성공이나 실패의 결과로 나태내 줄 수 있다.
- 비동기 코드의 시작 시점을 지정할 수 있다.
- 콜백 체이닝을 쉽게 구현할 수 있다.
<br>=> 단순히 콜백으로 짜면 콜백의 결과를 성공이나 실패 로직도 처리해주어야하고 비동기 시작 시점도 관리해주어야한다.
<br>=> 반면 프로미스는 콜백의 성공이나 실패 여부를 함수를 통해 처리할 수 있는 로직을 제공하고 비동기 시작 시점도 쉽 관리할 수 있다.
```html
<script>
    let myFirstPromise = new Promise((resolve, reject) => {
    
        setTimeout(function(){
            // 비동기 성공시 resolve() 함수 호출
            resolve("Success!");
            // 비동기 실패시 reject() 함수 호출
            reject("reject!");
        }, 1000);

        // const xhr = new XMLHttpRequest();
        // xhr.open("GET", url);
        // xhr.onload = () => resolve(xhr.responseText);
        // xhr.onerror = () => reject(xhr.statusText);
        // xhr.send();
    });

    myFirstPromise
        .then(value => {console.log(value)})
        .catch(error => {console.log(error)})
        ;
</script>
```

Promise chaining

```html
<script>
    let myFirstPromise = new Promise((resolve, reject) => {
        setTimeout(function(){
            resolve("Success!");
            reject("reject!");
        }, 1000);
    });
    let mySecondPromise = new Promise((resolve, reject) => {
        setTimeout(function(){
            resolve("second-Success!");
            reject("second-reject!");
        }, 1000);
    });
    myFirstPromise
        .then(value => {console.log(value); mySecondPromise.then(value=>console.log(value))})
        .catch(error => {console.log(error)})
        ;
</script>
```

### Async-await
[참고: Javascript.info](https://javascript.info/async-await)
- 함수에 async 키워드를 붙히면 프로미스를 반환.
- async로 정의된 함수 안에 await 키워드를 사용하여 다른 함수를 호출하면 이 함수는 비동기로 작동함
- async-await를 사용하면 개발자가 동기 방식의 코드를 작성하는 것처럼 비동기 코드를 작성할 수 있음
<br>=> 예제코드에서 비동기 체이닝을 동기 방식의 코드를 작성하는것처럼 코딩함

```html
<script>
    async function simple() {
    //async 키워드를 붙히면 함수는 Promise를 반환한다. 
      return 1;
    }
    simple().then(x=>console.log(x)); // 1
    
    async function f() {
        let result = "result : ";
        // 이 함수에서 프로미스는 비동기 요청을 만들기 위해서 사용됨
        let firstAsyncFunction = new Promise((resolve, reject) => {
          setTimeout(() => resolve("first done!"), 1000)
        });
        let secondAsyncFunction = new Promise((resolve, reject) => {
          setTimeout(() => resolve("second done!"), 1000)
        });

        let firstResult  = await firstAsyncFunction; // 비동기 함수가 끝날때까지 기다린다.
        result += firstResult;

        let secondResult  = await secondAsyncFunction; // 비동기 함수가 끝날때까지 기다린다.
        result += secondResult;
        
        // 리턴값은 resolve의 파라메터와 같다.
        return result;
    }
    f().then(x=>console.log(x));
</script>
```
