---
title: "merge, mergeAll, mergeMap"
date: 2019-04-19 11:00:01 -0400
categories: rxjs javascript
---

- merge: 여러개의 Observable들을 하나로 합쳐 Observable을 만드는 오퍼레이터
- mergeAll: Observable안에 있는 여러개의 Observable을 꺼내서 하나의 Observable로 만드는 오퍼레이터
- mergeMap: mergeMap은 map과 mergeAll을 합친 오퍼레이터
<br>=>map을 사용하여 변환한 객체가 Observable 일때 map의 리턴값은 Observable of Observables가 되는데 이것을 하나의 Observable로 만들고 싶을 때 사용한다.

예시
```js
const { of } = rxjs;
const { tap, map, mergeMap, mergeAll, merge } = rxjs.operators;

console.log("---------merge-----------")
// merge는 여러개의 Observable들을 하나로 합쳐 Observable을 만드는 오퍼레이터
of('A','B','C')
.pipe(
    // Observable{A, B, C, 1, 2, 3}
    merge(of(1,2,3))
)
.subscribe(
    value=>{console.log(value)}
);

console.log("---------map + mergeAll-----------")
//mergeAll은 Observable안에 있는 여러개의 Observable을 꺼내서 하나의 Observable로 만드는 오퍼레이터
of('A','B','C')
.pipe(
    //Observable{ Observable{A1,A2,A3}, Observable{B1,B2,B3}, Observable{C1,C2,C3}}
    map(value=>of(value+1,value+2, value+3)),
    //Observable{ A1, A2, A3, B1, B2, B3, C1, C2, C3}
    mergeAll()
)
.subscribe(
    value=>{console.log(value)}
);
console.log("---------mergeMap-----------")
// mergeMap은 map과 mergeAll을 합친 오퍼레이터
of('A','B','C')
.pipe(
    //Observable{ A1, A2, A3, B1, B2, B3, C1, C2, C3}
    mergeMap(value=>of(value+1,value+2, value+3))
)
.subscribe(
    value=>{console.log(value)}
);
```
