---
title: "[JavaScript] 네임스페이스(Namespace)란 무엇인가: 충돌을 막는 이름의 공간"
date: 2026-02-21 14:46 +0900
categories: [JavaScript]
tags: [javascript, namespace, scope, module]
---

네임스페이스(namespace)는 “이름이 들어갈 수 있는 공간”을 의미한다.  
같은 이름을 가진 변수나 함수가 여러 곳에서 생길 수 있기 때문에, 이름 충돌을 막고 코드를 구조화하기 위해 네임스페이스 개념이 사용된다.  
자바스크립트는 C#이나 Java처럼 언어 차원에서 `namespace` 문법을 제공하지는 않지만, 실제 개발에서는 다양한 방식으로 “네임스페이스 역할”을 구현해 왔다.

---
<br>

## 1. 네임스페이스가 필요한 이유

자바스크립트에서 이름 충돌은 주로 다음 상황에서 발생한다.

- 전역 공간(`window`)에 변수를 많이 올려두는 경우
- 여러 파일이 하나의 페이지에 함께 로드되는 경우
- 라이브러리/플러그인 코드가 섞이는 경우

예를 들어 아래처럼 전역에 함수를 막 선언하면, 다른 파일에서 같은 이름을 쓰는 순간 덮어써질 수 있다.

```js
function init() {
  console.log("A의 init");
}
```
즉, “이름을 어디에 소속시키고, 충돌을 어떻게 막을 것인가”가 네임스페이스의 핵심이다.

---
<br>

## 2. 자바스크립트의 기본 네임스페이스: 전역 객체

브라우저에서는 전역 객체가 **window**이며, Node.js에서는 `globalThis`를 통해 전역 객체에 접근할 수 있다.  
전역에 선언된 많은 식별자들이 사실상 이 전역 객체의 **이름 공간**에 들어가게 된다.

```js
var x = 1;
console.log(window.x); // 1 (브라우저 환경)
```
반면 let/const는 전역에 선언해도 window의 프로퍼티로 올라가지 않는 점이 다르다.

```js
let y = 2;
console.log(window.y); // undefined
```

다만 “전역 스코프에 이름이 생긴다”는 점에서, 여전히 충돌과 관리 문제를 일으킬 수 있다.

---
<br>

## 3. 객체를 네임스페이스로 사용하는 패턴

자바스크립트에서 가장 전통적인 네임스페이스 구현 방식은 **객체 하나를 만들고 그 안에 묶는 것**이다.

```js
const App = {
  init() {
    console.log("App.init");
  },
  utils: {
    clamp(n, min, max) {
      return Math.min(Math.max(n, min), max);
    },
  },
};

App.init(); // "App.init"
console.log(App.utils.clamp(10, 0, 5)); // 5
```

이 방식의 장점은 명확하다.

- 전역에 이름이 App 하나만 생긴다.
- 기능이 어디에 속하는지(App.utils)가 코드에서 드러난다.
- 같은 이름의 함수가 있어도 App.init, Admin.init처럼 구분 가능하다.

---
<br>

## 4. “즉시 실행 함수(IIFE) + 네임스페이스” 패턴

객체 네임스페이스는 유용하지만, 내부 구현 디테일까지 모두 공개되기 쉽다.  
이를 보완하기 위해 IIFE로 내부를 감추고, 외부로 필요한 것만 노출하는 방식이 자주 쓰였다.

```js
const App = (() => {
  // 내부 전용(외부에서 직접 접근 불가)
  let version = "1.0.0";

  function log(msg) {
    console.log(`[App] ${msg}`);
  }

  // 외부에 공개할 API만 반환
  return {
    init() {
      log(`init (v${version})`);
    },
    setVersion(v) {
      version = v;
      log(`version -> ${version}`);
    },
  };
})();

App.init(); // [App] init (v1.0.0)
App.setVersion("2.0.0"); // [App] version -> 2.0.0
```

이 패턴의 핵심은 다음과 같다.

- 네임스페이스(App)는 전역에 하나만 둔다.
- 내부 상태(version)는 숨겨서 보호한다.
- 공개 API만 외부에서 접근 가능하게 만든다.

---
<br>

## 5. 현대 방식: ES 모듈이 사실상 “네임스페이스” 역할을 한다

요즘은 네임스페이스 목적을 **ES 모듈**이 거의 대체한다.  
파일이 곧 독립된 스코프가 되며, 필요한 것만 `export`로 공개하고, 쓰는 쪽에서 `import`로 가져온다.

```js
// utils/math.js
export function clamp(n, min, max) {
  return Math.min(Math.max(n, min), max);
}
```

```js
// app.js
import { clamp } from "./utils/math.js";

console.log(clamp(10, 0, 5)); // 5
```

<br>

또는 `import * as` 문법을 사용하면 “네임스페이스처럼” 묶어서 쓸 수도 있다.

```js
import * as MathUtils from "./utils/math.js";

console.log(MathUtils.clamp(10, 0, 5)); // 5
```

이 방식은 다음 점에서 강력하다.

- 전역 오염이 거의 없다(모듈 스코프).
- 공개 범위가 명확하다(export/import).
- 파일 구조 자체가 곧 네임스페이스 역할을 한다.

---
<br>

## 6. 네임스페이스와 스코프의 차이

둘은 모두 “충돌과 관리” 문제를 다루지만, 해결하는 방식이 다르다.  

스코프(scope)는 **변수/함수가 접근 가능한 범위**를 의미한다.  
즉, 스코프는 “밖에서 아예 보이지 않게” 만들어 **접근 자체를 차단**한다.  

네임스페이스(namespace)는 **이름을 구분하기 위한 소속(이름의 공간)** 을 의미한다.  
즉, 네임스페이스는 “같은 이름도 공존할 수 있게” 만들어 **이름 충돌을 회피**한다.

예를 들어 객체를 네임스페이스처럼 사용하면, 같은 이름의 함수라도 소속이 달라 충돌하지 않는다.  
다만 이는 “스코프를 새로 만드는 것”이 아니라, **이름을 `A.init`, `B.init`처럼 묶어 구분**하는 방식이다.

```js
const A = { init() { console.log("A"); } };
const B = { init() { console.log("B"); } };

A.init(); // "A"
B.init(); // "B"
```

<br>

반면 `function`, `{}`, `module` 같은 스코프는 아예 경계를 만들어 내부 이름이 바깥으로 새지 않게 한다.  
즉, 네임스페이스가 **구분**이라면 스코프는 **격리**에 가깝다.

```js
{
  const init = () => console.log("block");
  init(); // "block"
}

console.log(typeof init); // "undefined"
```

---
<br>

## 마무리

네임스페이스는 이름 충돌을 막기 위해 “이름이 소속될 공간”을 만드는 개념이며,  
자바스크립트에서는 전통적으로 객체나 IIFE로 이를 구현했고,  
현대에는 ES 모듈이 파일 단위 스코프로 사실상 네임스페이스 역할을 수행한다.

