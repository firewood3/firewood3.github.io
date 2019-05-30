---
title: "Anguler 컴포넌트 테스트하기"
date: 2019-04-22 13:51:00 -0400
categories: typescript angular
---

***
## 테스트 작성 시기의 두 가지 견해
1. 테스트를 먼저 작성 후 테스트를 통과하는 코드를 작성하자는 견해(TDD)
    - 장점: 요구사항에 맞는 테스트를 모두 만들어 놓고 이를 통과하는 코드를 만들므로 안정성이 높아진다.
    - 단점: 요구사항이 변경될때마다 테스트 코드를 수정해야하거나 이전 테스트코드를 버리고 새로운 테스트 코드를 만들어야 한다.
2. 코드를 만들면서 테스트를 함께 작성하자는 견해
    - 장점: 요구사항이 변경되어도 기존 테스트 코드를 위한 추가 작업을 하지 않아도 된다.
    - 단점: TDD 보다 적은 경우를 생각하게 된다.

***
## Angular의 두 가지 테스팅 기법
1. 단위 테스트
    - 목적: 특정 코드의 정확성을 확인하는 것
    - 의의: 의존성과 상관없이 특정 코드(단위 코드)의 정확성을 확인한다.
    - 테스트 컴포넌트 생성 방법: 컴포넌트의 인스턴스를 직접 생성한다.(new component-name)
    <br>=> 단위 테스트에서 의존성은 모의(Mocking, 더미 객체)을 사용하여 해결한다.
2. 종단간(end-to-end) 테스트
    - 목적: 전체 시스템을 테스트하는 것
    - 의의: 여러 시스템(DB, HTTP..)간의 의존성을 확인하고 전체 시스템이 예상대로 작동하는지 확인한다.
    - 테스트 컴포넌트 생성 방법: 모듈을 설정하고 모듈을 사용하여 컴포넌트를 생성한다.(module.createComponent(component-name))
    <br>=> 컴포넌트와 템플릿 연결: 모듈에서 생성된 컴포넌트를 사용하므로 바인딩된 템플릿에 접근할 수 있다.
    <br>=> 컴포넌트와 서비스의 연결: 모듈에 서비스를 provider로 추가하여 컴포넌트에서 주입받을 수 있도록 한다.

***
## 테스트 절차
1. 준비(Arrange) : 단위 테스트의 초기 상태를 작성하는 단계
2. 작동(Act) : 실제로 단위 테스트를 치르는 단계
3. 확인(Assert): 테스트를 검증하는 단계

***
## Angular의 테스트 도구
1. 자스민(Jasmine) : 테스트 케이스를 작성하고 관리하는 라이브러리

```ts
describe() { // describe : 자스민 테스트 함수들의 컨테이너
    let str:string;
    beforeEach() { // beforeEach : Arrange 부분에 해당되는 함수
        str = "name";
    }
    it() { // it : Act 부분에 해당하는 함수
        expect(str).toBe("name"); // true // Assert 부분에 해당하는 함수
        expect(str).toContain("g"); // false // Assert 부분에 해당하는 함수
    }
}
```
2. 카르마(Karma) : 테스트 케이스를 식별하고 브라우저를 통해 테스트 케이스를 실행하는 도구

실행명령어
```code
ng test
```

***
## 테스트 코드 작성해보기
### 파이프의 테스트 
파이프는 외부 의존성을 가지지 않으므로 테스트 코드를 비교적 쉽게 만들 수 있다.
```ts
describe('Custom Built-in Pipe: Sort',()=>{
  let pipe: CustomSort;
  let tasks: Task[];
  
  beforeEach(()=>{
    pipe = new CustomSort();
    tasks = Array.of(
      new Task(3, "t2", Array.of<SubTask>(),"3"),
      new Task(1, "t1", Array.of<SubTask>(),"1"),
      new Task(2, "t2", Array.of<SubTask>(),"2"),
      new Task(4, "t4", Array.of<SubTask>(),"4"),
    );
  });

  it('task 배열의 오름차순 정렬 테스트 케이스', ()=>{
    let expectedTask: Task[] = Array.of(
      new Task(1, "t1", Array.of<SubTask>(),"1"),
      new Task(3, "t2", Array.of<SubTask>(),"3"),
      new Task(2, "t2", Array.of<SubTask>(),"2"),
      new Task(4, "t4", Array.of<SubTask>(),"4")
    );
    expect(pipe.transform(tasks, true)).toEqual(expectedTask);
  });
});
```

### HTTP 서비스의 테스트
API 서비스는 HttpClient라는 의존 객체의 get()메소드에 의존성을 가지므로 HttpClient의 get()메소드를 더미객체로 만들어 주고 API를 생성될때 주입해준다.
```ts
describe('TrelloApiService', () => {
  beforeEach(() => TestBed.configureTestingModule({}));

  let trelloService: TrelloApiService;
  let mockHTTP;
  let fakeBoards: Board[];

  beforeEach(()=>{
    mockHTTP = jasmine.createSpyObj('mockHTTP', ['get','post']);
    trelloService = new TrelloApiService(mockHTTP);
  });

  it('Board 배열을 받아오는 HTTP 서비스의 get 테스트 케이스', ()=>{
    fakeBoards = Array.of(new Board(1, "board1", []), new Board(2, "board2", []));
    mockHTTP.get.and.returnValue(of(fakeBoards));
    trelloService.getBoards().subscribe(
      boards => {
        expect(boards[0].title).toEqual("board1");
        // expect(boards[0].title).toEqual("board2");
        expect(boards).toEqual(Array.of(new Board(1, "board1", []), new Board(2, "board2", [])));
      }
    );
  });
});
```
### 독립된 컴포넌트의 테스트
컴포넌트를 템플릿 부분을 고려하지 않고 테스트 할 수 있다.
  - 장점: 컴포넌트의 논리를 비교적 간단하게 테스트 할 수 있다.
  - 단점: 템플릿 부분에 대한 포괄적인 검증을 할 수 없다.

```ts
describe('BoardComponent', () => {
  let boardComponent: BoardComponent;
  let mockElementRef, mockRoute, mockTrelloService;

  beforeEach(()=>{
    boardComponent = new BoardComponent(mockElementRef, mockRoute, mockTrelloService);
  });

  it('기존 작업에 추가 테스트', ()=>{
    boardComponent.addtaskText = "더미";
    boardComponent.board = new Board(1, "보드 1", Array.of());
    boardComponent.board.task.push(new Task(1, "작업 1", Array.of(), "1"));
    boardComponent.addtask();

    expect(boardComponent.board.task.length).toBe(2);
    expect(boardComponent.board.task[1].title).toBe("더미");
  });

  it('첫 번째 작업 추가 테스트', ()=>{
    boardComponent.addtaskText = "더미";
    boardComponent.board = new Board(1, "보드 1", Array.of());
    boardComponent.addtask();
    expect(boardComponent.board.task.length).toBe(1);
    expect(boardComponent.board.task[0].id).toBe(1);
    expect(boardComponent.board.task[0].title).toBe("더미");
  });
});
```

### 통합 컴포넌트의 테스트
템플릿 부분을 고려하고 컴포넌트를 테스트 한다.
  - 장점: 템플릿 부분에 대한 포괄적인 검증을 할 수 있다.
  - 단점: 컴포넌트의 인스턴스만 테스트할 때와 비교하여 많은 설정을 해야 한다.(모듈 설정)

통합 테스트를 위한 객체들(모듈을 사용하여 컴포넌틀를 생성하기 위한 객체들)
  - TestBed: 통합 컴포넌트를 위한 모듈을 설정하고 생성한다.
  - ComponentFixture: TestBed의 반환값으로 통합 컴포넌트의 인스턴스를 생성하고 관리한다.(컴포넌트의 변화감지 및 템플릿에 접근)
  - async 함수: 테스트의 Arrange 단계인 TestBed의 모듈 설정은 비동기 방식으로 이루어 지는데, Act 단계에서는 모듈 설정이 완료된 TestBad를 원하므로 Arrange단계와 Act 단계의 동기를 맞춰주기 위해 사용한다.
  <br> => 비동기 함수를 async 함수로 감싸면 비동기 함수가 끝날때까지 기다린다.
  <br> => async-await의 await 키워드와 동일하다. (두 머신간의 동기를 맞춰주는 키워드)

```ts
describe('HomepageComponent', () => {
  let component: HomepageComponent;
  let fixture: ComponentFixture<HomepageComponent>;

  beforeEach(async(() => {
    const mockTrelloService={
      getBoards: ()=>of([])
    };

    TestBed.configureTestingModule({
      declarations: [ HomepageComponent ],
      imports: [RouterModule.forRoot([])],
      providers: [
        {provide: TrelloApiService, useValue:mockTrelloService}]
    })
    .compileComponents();
  }));

  beforeEach(() => {
    fixture = TestBed.createComponent(HomepageComponent);
    component = fixture.componentInstance;
    fixture.detectChanges(); // 컴포넌트의 변화를 템플릿에 반영한다.
  });

  it('Component가 Module로부터 잘 만들어졌는지 확인', async(() => {
    expect(component).toBeTruthy();
  }));

  it('2개 보드가 있는지 확인', () =>{
    component.boards = Array.of(
      new Board(1, "보드 1", []),
      new Board(2, "보드 2", [])
    );
    fixture.detectChanges();
    const elements = fixture.debugElement.nativeElement;
    let title = elements.querySelectorAll('.title');
    console.log(title);
    expect(title[0].textContent).toContain('보드 1');
    expect(title[1].textContent).toContain('보드 2');
  });

  it('새 보드 추가 확인', ()=>{
    component.addBoard();
    fixture.detectChanges();
    expect(component.boards.length).toBe(1);
    const elements = fixture.debugElement.nativeElement;
    let title = elements.querySelectorAll('.title');
    expect(title[0].textContent).toContain('New Board');
  });
});
```

***
참고도서: 예제로 배우는 타입스크립트 2.x  
저자: 사친 오호리  
옮김: 김창수  
출판사: 터닝포인트  
