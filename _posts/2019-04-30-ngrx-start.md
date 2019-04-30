---
title: "Redux와 Ngrx"
date: 2019-04-30 13:40:12 -0400
categories: rxjs redux ngrx ngrx/store ngrx/effects anguler typescript javascript
---

1. Ngrx로 상태관리하기
2. Ngrx로 간단 상태 바꾸기
3. CanActivate 인터페이스로 라우트 요청 검수하기
4. [[외부 자료]](https://mherman.org/blog/authentication-in-angular-with-ngrx/)로그인 상태를 관리하기 위해 Ngrx 사용하기

***
## Ngrx의 Redux 패턴
[Redux pattern 참고](https://jobs.zalando.com/tech/blog/design-patterns-redux/?gh_src=4n3gxh1)

### Redux 패턴 = singleton pattern(immutable) + observer pattern(reactive)
```ts     
        side effect(action chaining)           change state(Pure function)
      [Action<enum>] -----> [Store<State>] <--> [Reducers(action): state]
         ^                       |
         | dispatch(action)      V      select():observable
          ----------------- [Components]    
```
- Store는 옵저버 패턴의 Subject역할을 한다. state를 update하는 역할을 한다.
- component는 옵저버 패턴의 Ovserver역할을 한다. 
- Store가 관리하는 state는 singleton이며 immutable하다.
- Action은 state의 상태 변경을 정의한다.
- Reducer는 state의 상태변경을 실행한다.(순수함수로..)
- component는 store의 dispatch(=update as observer pattern) 메서드로 특정 action을 호출하여 state를 변경할 수 있다.
- component는 store의 select(=subscribe as observer pattern) 메서드로 Store의 특정 상태를 구독할 수 있다.
- Action이 일어났을 때, side effect를 함수를 만들어 실행하게 만들 수 있다. 
- side effect 함수에서 연속적으로 Action을 호출할 수 있다.

***
## Ngrx로 간단 상태 바꾸기

```code
npm install @ngrx/store --save
npm install @ngrx/effects --save
```

### State
```ts
export interface RGBState {
  color: string;
  name: string | null;
  errorMessage: string | null;
}

export const initialRGBState: RGBState = {
  color: 'init red',
  name: null,
  errorMessage: null
};
```

### Action
```ts
export enum RGBActionType {
  RED = '[RGB] RED',
  GREEN = '[RGB] GREEN',
  BLUE = '[RGB] BLUE',
  BLUE_SUB_1 = '[RGB] BLUE SUB 1',
  BLUE_SUB_2 = '[RGB] BLUE SUB 2'
}

export class Red implements Action {
  readonly type = RGBActionType.RED;
  constructor() {}
}

export class Green implements Action {
  readonly type = RGBActionType.GREEN;
  constructor() {}
}

export class Blue implements Action {
  readonly type = RGBActionType.BLUE;
  constructor() {}
}

export class BlueSub1 implements Action {
  readonly type = RGBActionType.BLUE_SUB_1;
  constructor() {}
}

export class BlueSub2 implements Action {
  readonly type = RGBActionType.BLUE_SUB_2;
  constructor() {}
}

export type All =
  | Red
  | Green
  | Blue
  | BlueSub1
  | BlueSub2;
```

### Reducer
```ts
export function rgbReducer(state = initialRGBState, action: All): RGBState {
  switch (action.type) {
    case RGBActionType.RED: {
      return {
        ...state,
        color: 'Red Color',
        name: 'Red is fire',
        errorMessage: null
      };
    }
    case RGBActionType.GREEN: {
      return {
        ...state,
        color: 'Green Color',
        name: 'Green is tree',
        errorMessage: null
      };
    }
    case RGBActionType.BLUE: {
      return {
        ...state,
        color: 'Blue Color',
        name: 'Blue is water',
        errorMessage: null
      };
    }
    case RGBActionType.BLUE_SUB_1: {
      return {
        ...state,
        color: 'Blue Color',
        name: 'Blue is water',
        errorMessage: null
      };
    }
    default: {
      return state;
    }
  }
}

export const reducers = {
  rgb: rgbReducer
};

export const selectRgbState = createFeatureSelector<RGBState>('rgb');
```

### Effects
```ts
@Injectable()
export class RgbStateEffects {

  constructor(private actions: Actions) {}

  @Effect({ dispatch: false })
  red: Observable<any> = this.actions.pipe(
    ofType(RGBActionType.RED),
    tap(value=>console.log('red effects', value))
  );

  @Effect({ dispatch: false })
  greed: Observable<any> = this.actions.pipe(
    ofType(RGBActionType.GREEN),
    tap(value=>console.log('green effects', value))
  );

  @Effect()
  blue: Observable<any> = this.actions.pipe(
    ofType(RGBActionType.BLUE),
    tap(value=>console.log('blue effects', value)),
    map(()=>{
      return new BlueSub1();
    })
  );

  @Effect({ dispatch: false })
  blue_sub_1: Observable<any> = this.actions.pipe(
    ofType(RGBActionType.BLUE_SUB_1),
    tap(value=>console.log('blue_sub_1', value))
  );
}
```

### NgModule 등록
```ts
@NgModule({
  imports:[
    StoreModule.forRoot(reducers, {}),
    EffectsModule.forRoot([RgbStateEffects]),
  ]
}) 
```

### Ngrx state 사용
```ts
@Component({
  selector: 'app-rgb',
  templateUrl: './rgb.component.html',
  styleUrls: ['./rgb.component.css']
})
export class RgbComponent implements OnInit {

  getRgbState: Observable<any>;
  color = null;

  constructor(private store: Store<RGBState>) {
    this.getRgbState = this.store.select(selectRgbState);
  }

  ngOnInit() {
    this.getRgbState.subscribe((state) => {
      this.color=state.color;
    });
  }

  redClick() {
    console.log("red click");
    this.store.dispatch(new Red());
  }

  greenClick() {
    console.log("green click");
    this.store.dispatch(new Green());
  }

  blueClick() {
    console.log("blue click");
    this.store.dispatch(new Blue());
  }

}
```

***
## CanActivate로 라우트 요청 검수하기
```ts
@Injectable()
export class OtherPageGuardService implements CanActivate{
  constructor(public router: Router){}
  canActivate(): boolean {
    this.router.navigateByUrl("/"); // 다른곳으로 라우팅
    return false; // false 는 라우트 되지 않음(빈페이지 나타냄) // true로 해야 라우트 됨
  }
}
@NgModule({
  imports:[
    RouterModule.forRoot([
      {path: 'other', component: OtherPageComponent, canActivate: [OtherPageGuardService]},
      {path: '', component: RgbComponent},
      {path: '**', redirectTo: '/'},
    ]),
  ]
}) 
```

***
## 로그인 상태를 관리하기 위해 Ngrx, HTTP_INTERCEPTOR, CanActivate를 사용할 수 있다.
[Origin](https://mherman.org/blog/authentication-in-angular-with-ngrx/)

- login State를 관리하기 위해 ngrx/store를 사용.
- loginSuccess action시, ngrx/effects로 jwt 토큰 호출 후 localstorage에 저장.
- HTTP_INTERCEPTORS 모듈을 사용해 전송 request에 jwt header 포함하여 전송.
- response에 401 Unauthorized 에러가 나면 localstorage의 jwt 토큰을 제거한다.
- CanActivate 인터페이스를 사용하면 라우터 요청이 왔을 때 검사 후 컴포넌트를 생성할지 말지를 결정할 수있다.(다른 곳로 라우팅 할 수도 있다.) 



