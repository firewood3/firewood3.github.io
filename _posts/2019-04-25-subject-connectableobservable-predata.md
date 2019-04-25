---
title: "Subject와 ConnectableObservable에서 이전에 생성된 데이터를 Observable에게 전달하기"
date: 2019-04-25 11:10:12 -0400
categories: rxjs connectableObservable subject observable
---

Subject는 미리 생성해 놓은 데이터를 Observer에게 전달 할 수 없다.(Hot Observable의 특성)
<br>=> 이를 극복하기 파생 Subject를 제공하고 파생 Subject는 미리 생성된 데이터를 저장해 둘 수 있다.
<br>=> Multicasted Observable에서는 이 파생 Subject들을 Operator로 사용할 수 있다.

### Observer가 구독하기 전에 생성된 데이터를 전달 할 수 있는 Subject들
- behabiorSubject: 구독하기 전 데이터 1개를 전송
- replaySubject: 구독하기 전 데이터 n개 전송
- asyncSubject: 일반적인 subject와는 달리 데이터를 전달하지 않다가 Complete시에 마지막 데이터를 전달

### Connectable Observable(Multicast)에서 구독하기 전에 생성된 데이터 사용 오퍼레이터들
- publishBehavior(value)
- publishReplay(bufferSize, windowTime)
- publishLast()
- shareReplay(bufferSize, windowTime)

### CODE
Observable
```js
// Observable은 생성될 데이터를 정의해두고 이를 Observer에게 전달 할 수 있다.
// (Cold Observable)
const obs$ = Observable.create(function subscribe(observer){
    // === of(1,2)
    observer.next(1);
    observer.next(2);
});

obs$.subscribe({
    next: (v)=>console.log(`observer: ${v}`)
});

// observer: 1
// observer: 2
```

Subject
```js
// subject는 미리 생성된 데이터를 Observer에게 전달 할 수 없다.
// (Hot Observable)
const subject = new Subject();

subject.next(1);
subject.next(2);

subject.subscribe({
    next: (v)=>console.log(`observer: ${v}`)
});

// 출력없음
```

BehaviorSubject
```js
/*
behaviorSubject는 미리 생성된 데이터 1개를 Observer에게 전달할 수 있다.
*/

// 생성자를 통해 미리 생성될 데이터의 초기값을 지정할 수 있다.
const subject = new BehaviorSubject("start");

subject.subscribe({
    next: (v)=>console.log(`observerA: ${v}`)
});

subject.next(1);
subject.next(2);
subject.subscribe({
    next: (v)=>console.log(`observerB: ${v}`)
});

// observerA: start (초기값으로 저장된 데이터)
// observerA: 1
// observerA: 2
// observerB: 2 // (이전에 생성된 데이터)
```

ReplaySubject
```js
// ReplaySubject는 미리 생성된 데이터 n개를 Observer에게 전달할 수 있다.
const subject = new ReplaySubject(2);

subject.next(1);
subject.next(2);

subject.subscribe(
    (v) => console.log('observer :', v)
);

// observer: 1
// observer: 2
```

ReplaySubject(pre data clear)
```js
// 생성자의 두번째 매개변수로 ms 설정하면 ms 후는 미리 생성된 데이터를 전달하지 않게 된다.
const subject = new ReplaySubject(2, 1000);

subject.next(1);
subject.next(2);

subject.subscribe(
    (v) => console.log('Subscriber A:', v)
);

// subscriber B는 아무런 데이터를 전달받지 못한다.
setTimeout(() => {
  subject.subscribe(
      (v) => {console.log('Subscriber B:', v)}
  );
}, 2000);

// subscriber A: 1
// subscriber A: 2
```

AsyncSubject
```js
// AsyncSubject는 subject가 complete되기전까지는 데이터를 observer에게 전달하지 않다가
// subject가 complete하면 마지막으로 생성된 데이터만 Observer에게 전달해 준다.

// (WebApi와 task queue를 사용하는 비동기는 아니다.)
// (subject의 complete를 호출하면 Observer에게 마지막 데이터를 callback 해주는 계념인듯하다.)
const subject = new AsyncSubject();

subject.next(1);
subject.next(2);

subject.subscribe({
    next: (v) => console.log(`Subscriber: ${v}`),
    complete: () => console.log("Subscriber completed")
});

subject.next(3);
subject.next(4);
subject.next(5);

subject.complete();

// Subscriber: 5
// Subscriber completed
```

publishBehavior Operator
```js
// 미리 생성된 데이터 1개를 Observer에게 전달
const multiObs$ = fromEvent(document, "click")
.pipe(
    pluck("offsetX"),
    publishBehavior("init value")
);
multiObs$.connect();

multiObs$.subscribe({
    next: (v)=>console.log(`observerA: ${v}`)
});

setTimeout(()=>{
    multiObs$.subscribe({
        next: (v)=>console.log(`observerB: ${v}`)
    });
}, 2000);

// observerA : init value
// observerB : 25 (2초 전에 offsetX 25 부분을 클릭함)
```

publishReplay Operator
```js
// 미리 생성된 데이터 n개를 Observer에게 전달
const multiObs$ = fromEvent(document, "click")
.pipe(
    pluck("offsetX"),
    publishReplay(3, 3000) // 3초 후 미리 생성된 데이터 전달 해제
);
multiObs$.connect();

multiObs$.subscribe({
    next: (v)=>console.log(`observerA: ${v}`)
});

setTimeout(()=>{
    multiObs$.subscribe({
        next: (v)=>console.log(`observerB: ${v}`)
    });
}, 2000);

// ObserverC는 3초 전에 클릭한 데이터를 전달 받지 못함
setTimeout(()=>{
    multiObs$.subscribe({
        next: (v)=>console.log(`observerC: ${v}`)
    });
}, 5000);

// (offsetX 25,26,27 부분을 2초 전에 각각 클릭(총 3번 클릭)
// observerA: 25
// observerA: 26
// observerA: 27
// observerB: 25
// observerB: 26
// observerB: 27
```

publishLast
```js
// subject가 complete될때, subject는 마지막 값을 전달함
// complete 되기 전에는 데이터를 전달 받을 수 없음
const multiObs$=interval(1000)
.pipe(
    take(2), // 0, 1
    publishLast()
);
multiObs$.connect();

multiObs$.subscribe({
    next: (v)=>console.log(`subscriber: ${v}`),
    complete: ()=>console.log('subscriber completed')
});

// subscriber: 1
// subscriber completed
```

hot Observerable을 publishLast()사용하여 connectable Observable(multicast)로 만들고 
subject.complete를 호출하여 asyncSubject의 기능을 사용해보려 하였으나, 
connectable Observable에 내장되에 있는 subject의 complete를 호출하는 방법을 몰라서
Cold Observable을 connectable Observable로 만들어서 publishLast 테스트 코드를 만들었다.
connectable Observable에 내장되어있는 subject의 complete를 호출하는 방법을 참조하는 방법을 알고싶다..
connectable Observable의 unsubscribe로는 subject의 complete를 호출 안된다.


shareReplay
```js
// 미리 생성된 데이터 n개를 Observer에게 전달
const multiObs$ = fromEvent(document, "click")
.pipe(
    pluck("offsetX"),
    shareReplay(3, 3000) // 3초 후 미리 생성된 데이터 전달 해제
);

multiObs$.subscribe({
    next: (v)=>console.log(`observerA: ${v}`)
});

setTimeout(()=>{
    multiObs$.subscribe({
        next: (v)=>console.log(`observerB: ${v}`)
    });
}, 2000);

// ObserverC는 3초 전에 클릭한 데이터를 전달 받지 못함
setTimeout(()=>{
    multiObs$.subscribe({
        next: (v)=>console.log(`observerC: ${v}`)
    });
}, 5000);

// (offsetX 25,26,27 부분을 2초 전에 각각 클릭(총 3번 클릭)
// observerA: 25
// observerA: 26
// observerA: 27
// observerB: 25
// observerB: 26
// observerB: 27
```

share는 connectableObservable의 connect와 unsubscribe를 자동으로 해주므로
 초기값을 설정하는 BehaviorSubject와 
 subject가 complete될때 마지막값을 전달하는 asyncSubject를 사용하지 못하는것 같다.
