---
title: "[JavaScript] 단축 평가와 옵셔널 체이닝: 안전한 접근을 위한 자바스크립트 패턴"
date: 2026-02-15 12:15 +0900
categories: [JavaScript]
tags: [javascript, short-circuit, optional-chaining, nullish-coalescing]
---

단축 평가(short-circuit evaluation)는 논리 연산자(`&&`, `||`)가 왼쪽부터 평가될 때,  
결과가 이미 확정되면 **오른쪽을 평가하지 않고** 멈추는 동작을 의미한다.  
옵셔널 체이닝(optional chaining, `?.`)은 객체의 중첩 속성에 접근할 때,  
중간 값이 `null` 또는 `undefined`이면 에러를 내지 않고 **즉시 `undefined`를 반환**하도록 돕는 문법이다.    
두 기능은 모두 “값이 없을 수 있는 상황”에서 코드를 안전하고 간결하게 만드는 데 목적이 있다.

---
<br>

## 1. 단축 평가란 무엇인가

### `&&`의 단축 평가

`&&`는 왼쪽 값이 **Falsy**이면 전체 결과가 이미 Falsy로 확정되므로 오른쪽을 평가하지 않는다.  
반대로 왼쪽이 Truthy이면 오른쪽까지 평가한 뒤, 최종적으로 **마지막으로 평가된 값**을 반환한다.

```js
const user = null;
const name = user && user.name;

console.log(name); // null
```
user가 null이므로 user.name 접근은 실행되지 않는다. 따라서 에러를 피할 수 있다.

<br>

### `||`의 단축 평가

||는 왼쪽 값이 Truthy이면 결과가 확정되므로 오른쪽을 평가하지 않는다.  
왼쪽이 Falsy일 때만 오른쪽을 평가하며, 최종적으로 마지막으로 평가된 값을 반환한다.

```js
const input = ""; // 사용자가 입력하지 않았다고 가정
const label = input || "기본 제목";

console.log(label); // "기본 제목"
```

<br>

### Falsy 값 목록

단축 평가에서 자주 혼동되는 부분은 “Falsy”의 범위이다. 다음 값들은 Falsy로 평가된다.

- false
- 0, -0
- "" (빈 문자열)
- null
- undefined
- NaN

---
<br>

## 2. 단축 평가의 실전 패턴

### 조건부 실행

```html
<button class="btn">저장</button>
```
```js
const btn = document.querySelector(".btn");

btn && btn.addEventListener("click", () => {
  console.log("클릭됨"); // 버튼 클릭 시 "클릭됨"
});
```

<br>

### 기본값 처리의 함정: `||`는 0과 ""도 덮어쓴다

`||`는 “Falsy면 오른쪽”이므로, 의도치 않게 0이나 "" 같은 값도 기본값으로 바꿔버릴 수 있다.

```js
const page = 0; 
const safePage = page || 1; // 0은 Falsy로 평가

console.log(safePage); // 1  (원래 0을 유지하고 싶었는데 덮어씀)
```

이런 경우에는 `??`(널 병합 연산자)가 더 적합하다.

---
<br>

## 3. 널 병합 연산자 `??`와의 관계

`??`는 왼쪽 값이 null 또는 undefined일 때만 오른쪽을 사용한다.  
즉, 0이나 "" 같은 값은 그대로 유지한다.

```js
const page = 0;
const safePage = page ?? 1;

console.log(safePage); // 0
```

---
<br>

## 4. 옵셔널 체이닝(?.)이란 무엇인가

옵셔널 체이닝은 중간 값이 null 또는 undefined일 때 에러를 내지 않고 undefined를 반환한다.  
대표적으로 “깊은 객체 접근”에서 효과가 크다.

### 중첩 속성 접근

```js
// profile이 없다고 가정
const user = { id: 1, profile: null };
const city = user.profile?.address?.city;

console.log(city); // undefined
```
- user.profile이 null이므로 그 뒤 접근은 중단된다.
- 에러 대신 undefined가 반환된다.

<br>

### 메서드 호출에도 사용 가능

```js
const modal = null;

modal?.open?.();
console.log("실행 완료"); // "실행 완료"
```
modal이 없으면 `open()`을 호출하지 않고 넘어간다.

---
<br>

## 5. 단축 평가 vs 옵셔널 체이닝

둘 다 “없을 수 있는 값”을 다루지만, 목적과 표현이 다르다.

### 단축 평가가 자연스러운 경우

- 조건부로 실행하고 싶을 때
- 간단한 fallback(기본값)을 주고 싶을 때

```js
const name = "";
const label = name || "이름 없음";

console.log(label); // "이름 없음"
```

<br>

### 옵셔널 체이닝이 자연스러운 경우

- 깊은 객체 접근이 많고, 중간 값이 없을 가능성이 클 때
- “존재하면 접근하고, 없으면 undefined”가 기대 동작일 때

```js
const user = { profile: null };
const email = user.profile?.contact?.email;

console.log(email); // undefined
```

---
<br>

## 6. 자주 하는 실수와 주의점

### 옵셔널 체이닝은 “없는 값”을 자동으로 만들어주지 않는다.  

옵셔널 체이닝은 에러를 막아줄 뿐, 값이 없으면 undefined를 준다.  
따라서 기본값이 필요하면 ??와 조합하는 것이 일반적이다.

```js
const user = { profile: null };
const email = user.profile?.contact?.email ?? "이메일 없음";

console.log(email); // "이메일 없음"
```

<br>

### ||와 ??를 섞을 때 괄호가 필요할 수 있다

연산자 우선순위 때문에 의도대로 동작하지 않거나 문법 오류가 날 수 있으므로,  
섞어 쓸 때는 괄호를 명시하는 편이 안전하다.

```js
const v = null;
const result = (v ?? "기본") || "대체";

console.log(result); // "기본"
```

---
<br>

## 마무리

단축 평가는 `&&`, `||`가 결과가 확정되면 평가를 멈추는 특성을 이용해 조건 실행과 기본값 처리를 간단히 만드는 방법이며,  
옵셔널 체이닝(?.)은 중간 값이 null/undefined일 때 에러 없이 undefined를 반환해 깊은 객체 접근을 안전하게 해준다.