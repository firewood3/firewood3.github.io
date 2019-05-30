---
title: "옵저버 패턴"
date: 2019-04-13 20:31:00 -0400
categories: design-pattern observer_pattern
---

B 머신이 A 머신을 의존(B->A)하고 있는 경우, A 머신의 상태값이 바뀌면 B 머신은 오류를 일으킬 가능성이 커진다. 이럴때, 옵저버 패턴을 사용하면 이 문제를 해결할 수 있다.

## 옵저버와 서브젝트
- Observer는 특정 머신의 변화를 감지해야 하는 머신
- Subject는 옵저버들에게 특정 머신의 변화를 알려주는 머신

## 옵저버 패턴 사용법
- B 머신은 A 머신의 상태값을 전달 받는 인터페이스(Observer)를 구현한다.
- Subject는 B 머신을 구성 하고(복수 구성 가능) B 머신에게 상태변화를 알리는 인터페이스를 구현한다.
- A 머신은 자신의 상태변화를 Subject를 통해 B 머신에게 알린다.

## 옵저버 패턴 예시

### 일반적인 의존성 코드
setAstate 함수로 A 머신의 a_state 값을 변경해도 B 머신의 b_state는 변경되지 않음
```ts
class Bmechine {
    b_state = 0;
    construct(a_machine: Amachine) {
        this.b_state = a_machine + 1;
    }
}

class Amechine {
    a_state = 0;
    // setAstate 함수로 a_state 값을 변경해도 b_state는 변경되지 않음
    setAstate(a_state) {
        this.a_state = a_state;
    }
}
```

### 옵저버 패턴을 적용한 코드
setAstate 함수로 A 머신의 a_state 값을 변경하면 B 머신의 b_state도 변경됨
```ts
class Bmachine implements Observer {
    b_state = 0;
    update(a_state) {
        this.b_state = a_state + 1;            
    }
}

class Subject {
    oservers: Observer[];
    addMachine(machine: Observer) {
        this.observers.push(machine);
    }
    notify(state) {
        this.observers.forEach(machine => {
            machine.update(state);
        });
    }
}

class Amachine {
    a_state = 0;
    construct(subject: Subject) {
        this.subject = subject;
    }
    // setAstate 함수로 a_state 값을 변경하면 b_state도 변경됨
    setAstate(a_state) {
        this.a_state = a_state;
        this.subject.notify(this.a_state);
    }
}
```
