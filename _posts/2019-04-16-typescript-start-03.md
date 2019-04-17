---
title: "타입스크립트 세번째"
date: 2019-04-16 15:31:00 -0400
categories: typescript angular
---

## 유니버설 네임스페이스
JavaScript는 기본적으로 모든 클래스와 함수를 window 네임스페이스에 추가한다.

*JS에서는 이전함수를 덮음, TS에서는 컴파일 오류*
```js
function getData() {
    return "true";
}
function getData() {
    return "false"
}
window.getData(); // false
```

***

### TypeScript 네임스페이스
TypeScript는 namespace라는 키워드를 제공하여 관련 함수, 클래스와 인터페이스를 하나의 우산아래 캡슐화 할수 있다.

```ts
namespace WebServiceResponse{

}
```

네임스페이스에 접근 방법
```code
<<네임스페이스 이름>>.<<함수/클래스 이름>>
```

export 키워드는 네임스페이스의 특정 맴버를 외부에 공개 하도록 한다.
```ts
namespace WebServiceResponse{
    export class WebResponse{}
}
```

중첩네임스페이스 : 네임스페이스 안에 또다시 네임스페이스를 추가할 수 있다.

*네임스페이스 예제*
```ts
namespace WebServiceResponse{
    export let url: string;
    export function getName() {
        return "name";
    }
    export namespace ServiceResponse {
        export class WebResponse {
            getResponse() {
                return "success";
            }
            sendResponse() {
                return "200 Ok";
            }
        }
    }

    let localResponse = new ServiceResponse.WebResponse();
    localResponse.getResponse();
    localResponse.sendResponse();
}
let localResponse = new WebServiceResponse.ServiceResponse.WebResponse();
localResponse.getResponse();
localResponse.sendResponse();

WebServiceResponse.url;
WebServiceResponse.getName;
```

JavaScript에서의 false
1. false
2. null
3. NaN
4. 0
5. 빈 문자열("" or '')
6. undefined

네임스페이스의 호출은 자바스크랩트 내부에서 chort-circuit 이라는 (a || b) a가 true이면 b는 실행 안됨으로 만들어진다.

***

## TypeScript 모듈
TypeScript 모듈은 네임스페이스와 동일한 역할을 수행한다. 네임스페이스 대신 모듈을 사용하는 가장 큰 장점은 모듈을 사용하면 모듈 로더를 사용할 수 있으므로 비동기 동작을 제공하고 결과적으로 애플리케이션의 실행 속도를 높일 수 있다.

TypeScript의 역할은 네임스페이스와 동일하지만, 모듈 로더를 사용하므로써 비동기 동작을 제공하고 앱의 실행 속도를 높일 수 있다.

TypeScript 모듈은 파일 시스템에 의해 정의된다. 즉, 모든 파일이 별도의 모듈이고 파일 이름이 모듈 이름이 된다. 


- 모듈은 파일이다.
- 모듈의 역할은 네임스페이스와 같다.
- 장점: 모듈 로더를 사용해서 비동기 동작하고 앱 의 실행속도를 높일 수 있다.
- 캡술화 되므로 멤버를 공개할때는 export, 맴버를 사용할때는 import 키워드를 사용한다.
- import할때 별칭도 추가할 수 있다. 
```ts
import {BoardService as Service} from './Modules/Service';
```


## TypeScript 제네릭
- 제네릭은 함수 또는 클래스를 정의할때 타입을 명시적으로 정의하지 않는 것.
- 함수 또는 클래스를 호출할때 타입을 정의하는 것.
- 제네릭은 코드 재사용성을 높인다.
- extends 키워드를 사용하여 제네릭 타입에 제약을 걸 수 있다.

제약 조건
```ts
class GenericClass<T extends Title>
// Title 타입으로 파생된 타입 T만 받겠다.
```

예1
```ts
let data = [];
function pushGenericToArray<T>(item: T) {
    this.data.push(item);
}

this.pushGenericToArray<string>("10");
this.pushGenericToArray<number>(10);
console.log(data);
```

예2
```ts
class GenericClass<T> {
    items: T[] = [];
    pushData(val: T) {
        this.items.push(val);
    }
    getData(index: number): T {
        return this.items[index];
    }
}

let numClass = new GenericClass<number>();
numClass.pushData(10);
numClass.pushData(20);
console.log(numClass.getData(0));
```

예3
```ts
interface Title {
    title: string;
}

class GenericClass<T extends Title> {
    items: T[] = [];
    pushData(val: T) {
        this.items.push(val);
    }
    getData(index: number): T {
        return this.items[index];
    }
}

class MyBook implements Title{
    title: string = 'myBook';
}

const genericClass = new GenericClass<MyBook>();
const myBook = new MyBook();
genericClass.pushData(myBook);

console.log(genericClass.getData(0).title); // myBook
```

***

## 트렐로 예제 애플리케이션
1. TypeScript 모듈 사용
2. 컴포넌트 통신
   - 서비스 사용(계층구조가 없는 컴포넌트 사이의 통신)(홈페이지와 보드는 라우터로 통신함(pathVariable 넘겨주기))
   - 직접 데이터 통신(게층구조가 존재하는 컴포넌트의 통신)(@Input, @Output)
3. observable을 사용하며 제네릭 살펴보기
4. 컴포넌트 생명주기

```code
          [앱]
    _______|________
   |       |       |
[홈페이지] [보드] [트렐로서비스]
           |
         [작업] 
           |
        [하위작업] 
```

- [작업]과 [하위작업]은 [보드]의 자식 컴포넌트이다.
- [홈페이지]와 [보드]의 통신은 서비스를 사용
- [보드]와 [작업/하위작업] 사이의 통신은 @Input과 Output 속성을 사용

컴팩트 생명주기
- OnInit : 컴포넌트를 초기화하고 데이터 바인딩된 프로퍼티를 표시한다.
- OnDestroy: 컴포넌트 파괴 시 호출, 타이머나 작업을 중지하고 메모리를 정리하기에 좋은 장소이다.
- OnChange: 컴포넌트에 바인딩된 프로페티에서 변경사항이 생겼을 때 발생하는 이벤트

보드 컴포넌트
- 라우팅을 사용하여 해당 보드 페이지로 이동한다.
- 라우팅 파라미터를 사용해서 어떤 보드를 선택했는지 확인한다.
- 트렐로서비스를 사용하여 작업과 하위 작업을 가져온다.
- [작업/하위작업] 컴포넌트들을 화면에 표시한다.
- [작업/하위작업] 컴포넌트와의 통신은 @Input과 @Output 프로퍼티를 사용한다.

라우팅에서 PathParameter 가져오기
```ts
ngOnInit() {
let boardId = this._route.snapshot.params['id'];
}
```

단일책임원칙(single responsibility principle): 하나의 컴포넌트는 하나의 기능만 담당하므로 새로운 기능으로 수정하거나 기능을 추가해야 하는 경우 다른 컴포넌트에 대한 영향을 걱정할 필요가 없다.

**@Input : 부모 컴포넌트에서 자식 컴포넌트의 데이터를 참조하기**
<br>자식 TS
```ts
@Input()
task: Task;
```
부모 HTML(Selector)
```html
<app-task [task]="task"> </app-task>
```

**@Output : 자식 컴포넌트의 데이터를 부모 컴포넌트로 PUSH하기**
<br>자식 TS
```ts
@Output()
public onAddsubTask: EventEmitter<SubTask>;
```
부모 HTML(Selector)
```html
<app-task (onAddsubTask)="addsubTask($event)"></app-task>
```
자식 TS
```ts
this.onAddsubTask.emit(newsubTask);
```
부모 TS
```ts
addsubTask(event) {
  console.log('Event Fired');
  console.log(event);
}
```



