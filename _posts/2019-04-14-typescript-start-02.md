---
title: "타입스크립트 두번째"
date: 2019-04-14 17:31:00 -0400
categories: design-pattern observer_pattern
---

***

## first-end: Sports News Combinator 프로젝트 구현
- export 키워드를 사용하면 클래스를 애플리케이션의 다른 컴포넌트에서 사용할 수 있다.
- 클래스 내부의 프로퍼티의 기본 접근자는 pubilc 이다.
- 컴포넌트의 메타데이터는 @Component 데코레이터를 사용하여 정의한다.
```ts
@Component({
  selector: 'app-news',
  templateUrl: './news.component.html',
  styleUrls: ['./news.component.css']
})
```
- import 키워드는 외부의 클래스에 접근할 수 있도록 해준다.
- 데이터 단방향 바인딩
- 지시자 *ngFor를 사용하여 Iterable 객체를 바인딩 할 수 있다.

***
## 함수

### JavaScript의 this
JavaScript의 this는 함수의 호출 방식에 따라 값이 달라진다.
1. 함수에서 호출
2. 메서드에서 호출
3. 생성자에서 호출
4. apply에서 호출
5. bind에서 호출

- 함수호출 시 this는 상위 스코프가 따로 없기 때문에 전역 객체를 가리키는데, 웹 브라우저에서의 전역 객체는 window이다.
- 다만 ECMAScript 5.1에서 코드 안정성과 오류 검증을 용이하게 하기 위해 만든 엄격모드(use strict)를 사용하면 함수에서의 this는 undefined이다.
- obj.myFunc(); 형태로 호출하면 메서드가 실행되지만 var someFunc = obj.myFunc; someFunc(); 형태로 호출하거나 setTimeout(obj.myFunc,1000); 형태로 호출하면 메서드가 객체로부터 분리되어 함수형태로 호출된다.

- 생성자에서 호출
```js
function ConstructorFunc(msg) {
    this.msg = msg;
}
ConstructorFunc.protorype.hello = function() {
    console.log(this.msg); // 이 this는 ConstructorFunc의 this이다.
}
var obj = new ConstructorFunc("Hello World!");
obj.hello(); // Hello World!
```

- apply 호출: apply호출은 apply 함수를 사용해 호출하여 함수 내부에서 사용할 this를 지정하는 것이다. apply 함수의 첫번째 인자로 null을 넘기면 전역 객체인 window가 this가 된다.


### 선택적(Optional) / 기본(default) 파라미터
```ts
function Book(title: string, length?: number) {}
```

```ts
function Book(titile: string, length:number=300) {}
```

### 나머지 Rest 파라미터
```ts
function School(name: string. ...id:number[]){}
let harvad = new School("Harvard", 1,2,3,4,5,6,7);
```

### 함수 로버로딩
Ts에서는 함수 오버로딩의 구현이 어색하다.
```ts
function getCustomer(name:string):string;
function getCustomer(id:number):string;
function getCustomer(property:any)string{
    if(typeof property == 'string') {}
    else if(typeof property == 'number'){}
    return "customer";
}
```

## 클래스
객체지향 프로그래밍
- 상속
- 다형성
- 캡슐화
- 추상화

### 생성자
하나의 생성자가 여러 형식을 가진 것처럼 작동할 수 있다.
```ts
constructor(author: string, title?: string)
```

public 키워드를 생성자 파라미터 이름 앞에 배치하면 파라마티와 같은 프로퍼티가 생성된다.
```ts
constructor(public author: string, public title?: string)
```

### 접근자(accessor)
프로퍼티의 getter setter 함수 용도로만 사용되는 키워드
```ts
class Book {
    public _title: string;
    get title(): string{
        console.log("dd");
        return this._title;
    }
    set title(value: string) {
        if (value != "")
            this._title = value;
    }
}

let typeScript = new Book();
typeScript.title = "TypeScript By Example";
console.log(typeScript.title);
```

### static 프로퍼티
static 으로 선언한 프로퍼티는 인스턴스 공통이다. 인스턴스에서 변경할 수 없다.

### 상속과 super
상속하면 부모의 프로퍼티에 접근할 수 있다.
자식 클래스에서 생성자를 정의 했다면 반드시 super키워드를 사용하여 명시적으로 부모 클래스를 호출해야만한다.
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

### 추상클래스
추상클래스에서 정의한 모든 메서드를 구현할 필요가 없는 특별한 타입의 클래스이다.
<br>추상클래스는 인스턴스화 되지 못한다.
```ts
abstract class book {
    constructor(public author: string) { }

    abstract getFullTitle(): string;
}

class TypeScript extends book {
    getFullTitle(): string {
        return "dd";
    }
}

let a = new TypeScript("aa");
```

### 덕타이핑
명시적으로 Bclass가 Atype 인터페이스로 선언하지 않았지만 news함수에서 Bclass를 전달할 수 있다.
```ts
interface Atype{
    name: string;
    age: number;
}

class Bclass{
    name: string;
    age: number;
}

function news(atype: Atype) {

}

let bclass = new Bclass();
news(bclass);
```

### 인터페이스 확장
```ts
interface IArticle{
    author: string;
    title: string;
}

interface IEspn extends IArticle {
    description: string;
}

let news: IEspn = {
    author: "ESPN",
    title: "",
    description: ""
}
```
### 인터페이스 구현
클래스가 인터페이스를 구현하면 모든 프로퍼티와 메서드가 클래스에 구현되어야 한다.


***

## second-end: Sports News Combinator 프로젝트 구현

### app component
- app.module.ts 에서 bootstrap부분으로 시작 컴포넌트를 지정해준다.
- app.module.ts 컴포넌트들을 정의하고 외부 모듈을 불러온다.

### 지시자(Directive)???? Selector???
```html
<div>
  <app-header></app-header>
  <app-news></app-news>
  <app-footer></app-footer>
</div>
```

### Angular 내장 지시자(Directive)
- *ngIf
- *ngFor

### 바인딩
- 단방향 바인딩
```html
<h4>{{source}}</h4>
<h4>{{'News Outlet: ' + source}}</h4>
<h4 innerText={{source}}></h4>
<h4>{{getSource()}}</h4>
```
- HTML 엘리먼트의 프로퍼티와 바인딩
```html
<img [src] = 'article.url'></img>
<img src = {{article.url}}></img>
```
- 양방향 바인딩
<input [(ngModel)] = "name">

***

## 데코레이터

### 데코레이터 디자인 패턴
데코레이터 디자인 패턴은 클래스를 수정하지 않고 클래스의 기능을 확장할 수 있는 구조적 패턴의 하나이다.

### tsconfig.json의 데코레이터 사인
자바스크립트에서 데코레이터의 사용이 아직 확정되지 않은 기능이므로 데코레이터를 사용하기 위해서는 tsconfig.json에서 데코레이터 사용을 켜주어야한다.
```json
{
    "compileroptions": {
        "experimentalDecorators" : true
    }
}
```

***

## Thrid-end : Sports News Combinator 프로젝트 구현


### 서비스 구현
API를 받아오는 서비스를 Root Provider에 등록
```ts
@Injectable({
  providedIn: 'root'
})
```

### 라우터 구현
```code
ng generate module app-routing --flat --module=app
```
```ts
@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
```

### HTTPClient 사용
```ts
public fetchNewsFeed(source: string): Observable<News> {
  return this
    ._http
    .get('${this.baseUrl}/?source=${source}&sortBy=top&apiKey=${NewsApiService.apiKey}')
    .pipe(
      tap(x => console.log("11")),
      tap(x => console.log(x)),
      catchError(this.handleError<any>('fetchNewsFeed', []))
    );
}
```

***
참고도서: 예제로 배우는 타입스크립트 2.x  
저자: 사친 오호리  
옮김: 김창수  
출판사: 터닝포인트  
