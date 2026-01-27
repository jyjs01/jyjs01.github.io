---
title: "[Vanilla JS] DOM 요소 선택: querySelector부터 실전 패턴까지"
date: 2026-01-27 14:06 +0900
categories: [JavaScript]
tags: [javascript, vanilla-js, DOM, selector]
---

Vanilla JS에서 DOM을 다루는 첫 단계는 **요소 선택**이다.  
이 글에서는 DOM 요소를 선택하는 대표 방법과 각각의 차이, 그리고 실전에서 자주 쓰이는 선택 패턴을 정리한다.

---
<br />

## 1. DOM 요소 선택이란?

DOM(Document Object Model)은 HTML 문서를 자바스크립트에서 다룰 수 있도록 만든 객체 구조이다.  
DOM 조작은 대체로 다음 흐름을 따른다.

1) 요소를 선택한다.  
2) 속성/내용/스타일을 변경한다.  
3) 이벤트를 등록한다.

이 글은 그중 **1) 요소 선택**에 집중한다.

---
<br />

## 2. 가장 많이 쓰는 선택 API: querySelector, querySelectorAll

### document.querySelector(selector)

CSS 선택자 문법을 사용하여 **첫 번째로 일치하는 요소 하나**를 반환한다.  
일치하는 요소가 없다면 `null`을 반환한다.

```js
const title = document.querySelector("h1");
const loginBtn = document.querySelector("#login-btn");
const activeItem = document.querySelector(".menu .item.active");
```

> **반환값: Element \| null**

하나만 필요할 때 가장 자주 사용된다.

---
<br />

### document.querySelectorAll(selector)

CSS 선택자 문법을 사용하여 일치하는 요소 목록을 반환한다.  
반환값은 **NodeList**이며, 요소가 없어도 빈 목록이 반환된다.

```js
const items = document.querySelectorAll(".menu .item");

items.forEach((el) => {
  el.classList.add("ready");
});
```

> **반환값: NodeListOf&lt;Element&gt;**

여러 개를 한 번에 처리할 때 사용한다.

> NodeList는 배열과 유사하지만 완전히 같은 것은 아니므로, 필요한 경우 Array.from()으로 변환하여 사용한다.
{: .prompt-warning }

---
<br />

## 3. ID / Class 기반 선택: getElementById, getElementsByClassName

### document.getElementById(id)

id는 문서에서 유일해야 하므로, 특정 요소 하나를 빠르게 가져올 때 적합하다.

```js
const form = document.getElementById("login-form");
```

> **반환값: HTMLElement \| null**

id 전용이며 CSS 선택자(#id)를 쓰지 않는다.

<br />

### document.getElementsByClassName(className)

클래스 이름으로 요소들을 가져온다. 반환값은 **HTMLCollection**이다.

```js
const cards = document.getElementsByClassName("card");
```

> 반환값: **HTMLCollectionOf&lt;Element&gt;**

**라이브 컬렉션**이라는 점이 중요하다.

<br />

### 라이브 컬렉션?

DOM이 바뀌면 컬렉션 내용도 자동으로 따라 변하는 컬렉션을 의미한다.

```js
const live = document.getElementsByClassName("item");

console.log(live.length); // 예: 3

document.querySelector(".list").insertAdjacentHTML("beforeend", `<li class="item">new</li>`);

console.log(live.length); // 예: 4 (DOM 변경이 자동 반영)
```

실전에서는 이 동작이 의도치 않은 버그로 이어질 수 있으므로, 요소 목록은 querySelectorAll을 선호하는 경우가 많다.

---
<br /><br />

## 4. 태그 기반 선택: getElementsByTagName

특정 태그를 모두 선택할 때 사용한다.

```js
const inputs = document.getElementsByTagName("input");
```

> 반환값: **HTMLCollectionOf&lt;Element&gt;**

**라이브 컬렉션**이다.

---
<br />

## 5. NodeList와 HTMLCollection의 차이

둘 다 여러 요소를 담는 컬렉션이지만 성격이 다르다.

- querySelectorAll → NodeList (대부분의 경우 정적)
- getElementsByClassName, getElementsByTagName → HTMLCollection (라이브)

---
<br />

## 6. 선택 범위를 좁히는 방법: element.querySelector(...)

선택을 항상 document에서만 시작할 필요는 없다.
이미 선택한 부모 요소 내부에서만 다시 선택하면 성능과 가독성이 좋아진다.

```js
const modal = document.querySelector(".modal");
const closeBtn = modal?.querySelector(".close");
```

- 장점: 어느 영역의 요소인지 코드만 봐도 명확해진다.
- 실전 팁: 컴포넌트처럼 영역이 나뉘는 UI에서는 특히 유용하다.

---
<br />

## 7. 실전 선택 패턴

### 존재 여부를 먼저 확인하기

querySelector는 `null`을 반환할 수 있으므로, 바로 쓰기 전에 확인하는 습관이 필요하다.

```js
const el = document.querySelector(".toast");

if (!el) return;
el.textContent = "저장되었습니다.";
```
<br />

### 데이터 속성으로 선택하기(권장)

클래스는 스타일링 목적과 섞이기 쉬우므로, 기능 제어용 선택은 data-* 속성을 쓰면 유지보수가 편하다.

```html
<button data-role="save">저장</button>
```
```js
const saveBtn = document.querySelector('[data-role="save"]');
saveBtn?.addEventListener("click", () => {
  console.log("save");
});
```

- 장점: 스타일 변경(클래스 이름 변경)과 기능 로직이 덜 엮인다.

<br />

### 이벤트 위임을 고려한 선택

리스트 항목처럼 동적으로 추가/삭제되는 요소는, 개별 요소마다 이벤트를 달기보다 **부모에서 한 번만** 처리하는 패턴이 자주 쓰인다.  
이때도 선택이 핵심이 된다.

```js
const list = document.querySelector(".todo-list");

list?.addEventListener("click", (e) => {
  const target = e.target;

  if (!(target instanceof HTMLElement)) return;

  const item = target.closest(".todo-item");
  if (!item) return;

  console.log("클릭된 항목:", item.dataset.id);
});
```

---
<br />

## 어떤 선택 방법을 주로 쓰면 좋은가?

실무/개인 프로젝트 기준으로는 다음이 무난하다.

- 일반적인 선택: querySelector, querySelectorAll
- 고유한 단일 요소: getElementById도 유용
- 기능 선택자: data-* 속성을 활용한 querySelector

---
<br />

## 마무리

DOM 조작의 출발점은 요소 선택이며, 일반적으로는 querySelector/querySelectorAll을 중심으로 사용하되,  
범위를 좁히고(element.querySelector) 기능 선택은 data-*로 분리하는 것이 유지보수에 유리하다.