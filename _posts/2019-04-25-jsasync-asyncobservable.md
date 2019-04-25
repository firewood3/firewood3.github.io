---
title: "JS의 비동기 처리 과정과 Observable을 비동기로 만들기"
date: 2019-04-25 14:15:12 -0400
categories: rxjs acyncObservable AsapScheduler observable webapi callstack
---

1. 자바스크립트는 어떻게 비동기를 처리하는가?
    - [참고1 비동기 queue 시각화](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
    - [참고2 비동기 WebApi 시각화](http://latentflip.com/loupe/)
    - [참고3 call stack과 webApi는 서로 다른 스레드로 동작하는가?](https://stackoverflow.com/questions/50283281/do-the-javascript-web-apis-run-on-a-different-thread-than-the-call-stack-thread)
2. Observable을 비동기로 처리하려면 어떻게 하는가? (RxJS 스케줄러 사용)
    - [참고1 rxjs-scheduler-demo](https://stackblitz.com/edit/rxjs-scheduler-demo)
    - [참고2 서로 다른 scheduler는 어떤 queue로 callback을 push 하는가?](https://blog.strongbrew.io/what-are-schedulers-in-rxjs/)

**JS의 비동기 처리 과정과 Observable을 비동기로 만들기**

***
## 자바스크립트가 비동기 코드를 처리하는 과정
1. 자바스크립트는 코드조각을 하나씩 call stack에 쌓아 그 코드 조각을 실행함
2. 자바스크립트의 call Stack은 하나의 Thread로 작동함
3. call stack의 코드조각이 비동기이면 코드 조각을 WebApi에게 전달하고 그 코드조각을 call stack에서 제거함
4. WebApi는 비동기 코드조각을 실행하고 실행완료하되면 callback을 queue에 push함
5. callstack에서 실행할 코드 조각이 없는 경우 queue에서 callback을 전달받아 실행함

### 비동기 호출이 완료된 콜백은 세가지 큐로 저장된다.
- Task Queue : SetTimeout의 콜백이 push됨.
- Microtask Queue : Promise나 MutationObserver의 콜백이 push됨.
- Animation frames Queue: requestAnimationFrame의 콜백이 push됨.

***Microtask Queue에 있는 callbcak은 Task Queue와 Animation fames Queue에 있는 callback보다 보다 먼저 호출된다.***

***Task queue와 Animation frames Queue사이에 우선순위는 없다. 어떤게 먼저 실행될지 모른다.***

javascript의 비동기 호출
```js
// call stack 에서 바로 실행됨
console.log("script start");

// setTimeout이 webApi에서 실행되고 cb은 taskqueue에 push됨
setTimeout(function(){
    console.log("setTimeout");
}, 0);

// Promise는 WebApi에서 실행되고 cb은 micro task queue에 push됨
Promise.resolve().then(function(){
    console.log("promise 1")
}).then(function(){
    console.log("promise 2")
});

// requestAnimationFrame은 WebApi에서 실행되고 cb은 Animation frames Queue에 push됨
requestAnimationFrame(function() {
    console.log("reqeustAnimationFrame");
});

// call stack에서 바로 실행됨
console.log("script end");

// 결과
// script start          // call stack 바로 실행
// script end            // call stack 바로 실행
// promise 1             // Microtask Queue에서 꺼내 실행
// promise 2             // Microtask Queue에서 꺼내 실행
// reqeustAnimationFrame // AnimationFrame Queue에서 꺼내 실행 (setTimeout과 실행 순서가 바뀔 수 있다.)
// setTimeout            // Task Queue에서 꺼내 실행 (reqeustAnimationFrame과 실행 순서가 바뀔 수 있다.)
```

***
## RxJS의 스케줄러를 사용하여 Observable을 비동기로 작동시키는 방법
[참고1 : 오퍼레이터를 사용한 비동기 Observable 코드]()

observeOn 오퍼레이터에 스케줄러의 타입을 명시하여 Observable을 비동기로 작동시킨다.
- asyncScheduler: cb(next())을 Task Queue에 push
- asapScheduler: cb(next())을 MicorTask Queue에 push
- animationFrameScheduler: cb(next())을 AnimationFrame Queue에 push

비동기로 작동하는 Observable
```js
console.log('start');

of(1,2,3,4,5)
.subscribe(x=>console.log(`${x} : Sync Observable`));

of(6,7,8,9,10)
.pipe(observeOn(asyncScheduler))
.subscribe({next: x=>console.log(`${x} : asyncScheduler`)});

of(11,12,13,14,15)
.pipe(observeOn(animationFrameScheduler))
.subscribe({next: x=>console.log(`${x} : animationFrameScheduler`)});

of(16,17,18,19,20)
.pipe(observeOn(asapScheduler))
.subscribe({next: x=>console.log(`${x} : asapScheduler`)});

console.log('end');

// start
// 1 : Sync Observable
// 2 : Sync Observable
// 3 : Sync Observable
// 4 : Sync Observable
// 5 : Sync Observable
// end
// 16 : asapScheduler                   // (asapScheduler의 cb은 microtask queue에 있다.)
// 17 : asapScheduler                   // (animationFrameScheduler의 cb은 AnimationFrame queue에 있다.)
// 18 : asapScheduler                   // (asyncScheduler cb은 task queue에 있다.) 
// 19 : asapScheduler                   // (AnimationFrame Queue와 Task Queue의 cb 호출 순서는 바뀔 수 있다.)
// 20 : asapScheduler        
// 11 : animationFrameScheduler        
// 12 : animationFrameScheduler        
// 13 : animationFrameScheduler
// 14 : animationFrameScheduler
// 15 : animationFrameScheduler
// 6 : asyncScheduler                   
// 7 : asyncScheduler
// 8 : asyncScheduler
// 9 : asyncScheduler
// 10 : asyncScheduler

```
   
***Observable을 비동기로 만들려면 Observable.pipe(observeOn(asyncScheduler))을 써라***
