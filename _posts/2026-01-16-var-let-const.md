---
title: "[JavaScript] var, let, const"
date: 2026-01-16 16:40 +0900
categories: [JavaScript]
tags: [javascript, var, let, const, scope, hoisting]
---

JavaScript에서 변수를 선언하는 방법은 `var`, `let`, `const`가 있습니다.  
요즘은 **대부분 `let`과 `const`를 사용**하고, `var`는 레거시 코드에서 주로 만납니다.

이 글은 아래 3가지만 확실하게 정리하는 게 목표입니다.

- **스코프(scope)**: 변수가 “어디까지” 보이는가?
- **재선언/재할당**: 같은 이름으로 다시 선언 가능한가? 값 변경 가능한가?
- **호이스팅(hoisting)**: 선언이 “어떻게” 끌어올려지는가?

---

<br />

## 1) 스코프: 함수 스코프(var) vs 블록 스코프(let/const)

### var는 함수 스코프

`var`는 `if`, `for` 같은 **블록 `{}`을 무시**하고, **함수 단위**로만 영역이 나뉩니다.

```js
function demo() {
  if (true) {
    var x = 1;
  }
  console.log(x); // 1 (블록 밖인데도 접근 가능)
}
demo();
```
<br />

### let/const는 블록 스코프

`let`, `const`는 **블록 `{}` 기준**으로 영역이 나뉩니다.

```js
function demo() {
  if (true) {
    let y = 2;
    const z = 3;
  }
  console.log(y); // ReferenceError
  console.log(z); // ReferenceError
}
demo();
```

<br /><br />

## 2) 재선언 / 재할당 차이

### var: 재선언 OK, 재할당 OK

```js
var a = 1;
var a = 2; // 재선언 가능
a = 3;     // 재할당 가능
```
<br />

### let: 재선언 NO, 재할당 OK

```js
let b = 1;
// let b = 2; // SyntaxError: Identifier 'b' has already been declared
b = 2;        // 재할당 가능
```
<br />

### const: 재선언 NO, 재할당 NO

```js
const c = 1;
// c = 2; // TypeError: Assignment to constant variable.
```

다만 `const`로 선언한 **객체/배열**은 내부 값 변경이 가능합니다.  
(변수 자체의 “참조”가 고정되는 것이지, 내부가 완전 불변은 아님)

```js
const obj = { count: 0 };
obj.count = 1; // OK (객체 내부 값 변경)
```

하지만 **참조 자체**를 바꾸는 건 불가능합니다.

```js
const obj = { count: 0 };
// obj = { count: 1 }; // TypeError
```
<br /><br />

## 3) 호이스팅: “된다/안된다”가 아니라 “어떻게 되냐”가 핵심

### var 호이스팅: 선언이 끌어올려지고, 초기값은 undefined

```js
console.log(v); // undefined
var v = 10;
console.log(v); // 10
```

위 코드는 내부적으로 아래처럼 동작합니다(개념적으로).

```js
var v;
console.log(v); // undefined
v = 10;
console.log(v); // 10
```

그래서 `var`는 의도치 않은 버그를 만들기 쉽습니다. <br /><br />

### let/const 호이스팅: 선언은 끌어올려지지만, TDZ 때문에 접근하면 에러

```js
console.log(l); // ReferenceError
let l = 10;

console.log(k); // ReferenceError
const k = 10;
```
---
<br /><br />

## 한눈에 비교표

| 구분 | 스코프 | 재선언 | 재할당 | 호이스팅 | 추천도 |
|---|---|---:|---:|---|---|
| `var` | 함수 스코프 | 가능 | 가능 | 가능(초기값 `undefined`) | 낮음 |
| `let` | 블록 스코프 | 불가 | 가능 | 가능(TDZ) | 높음 |
| `const` | 블록 스코프 | 불가 | 불가(참조 고정) | 가능(TDZ) | 매우 높음 |

> **TDZ(Temporal Dead Zone)**: 선언은 되었지만 초기화되기 전까지 접근하면 에러가 나는 구간

---

<br /><br />

## 마무리 정리

`var` : 함수 스코프 + undefined 호이스팅 때문에 예측이 어려워질 수 있음

`let` : 블록 스코프 + 재할당 가능

`const` : 블록 스코프 + 재할당 불가(가장 기본으로 추천)