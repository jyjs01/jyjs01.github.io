---
title: "[Vanilla JS] DOM 요소 생성·추가·삭제: createElement부터 remove까지"
date: 2026-01-31 13:59 +0900
categories: [JavaScript]
tags: [javascript, vanilla-js, DOM, create, append, remove]
---

DOM 조작에서 **생성·추가·삭제**는 화면의 구조 자체를 바꾸는 작업이다.  
단순히 텍스트나 클래스만 바꾸는 수준을 넘어, 요소를 새로 만들고 원하는 위치에 끼워 넣거나 제거함으로써 UI를 동적으로 구성할 수 있다.  
본 글에서는 바닐라 JavaScript에서 DOM 요소를 **생성하고**, **추가하고**, **삭제하는** 대표 방법을 예시 HTML과 함께 정리한다.

---
<br />

## 1. 요소 생성: document.createElement

### 요소를 만들고 속성/텍스트를 준비하기

```html
<div class="wrap"></div>
```
```js
const wrap = document.querySelector(".wrap");
if (!wrap) return;

const p = document.createElement("p");
p.className = "msg";
p.textContent = "새로 생성된 문장입니다.";

wrap.append(p);

console.log(wrap.querySelector(".msg")?.textContent); // "새로 생성된 문장입니다."
```

---
<br />

## 2. 요소 추가: append, prepend, before, after

### append와 prepend

`append`는 마지막 자식으로 추가하고, `prepend`는 첫 번째 자식으로 추가한다.

```html
<ul class="list">
  <li class="item">A</li>
</ul>

<button class="btn-append">append</button>
<button class="btn-prepend">prepend</button>
```
```js
const list = document.querySelector(".list");
const btnAppend = document.querySelector(".btn-append");
const btnPrepend = document.querySelector(".btn-prepend");

let n = 1;

btnAppend?.addEventListener("click", () => {
  if (!list) return;

  const li = document.createElement("li");
  li.className = "item";
  li.textContent = `끝-${n}`;

  list.append(li);

  console.log(list.textContent?.trim()); // 예: "A끝-1" (공백/줄바꿈은 환경에 따라 다를 수 있음)
  n += 1;
});

btnPrepend?.addEventListener("click", () => {
  if (!list) return;

  const li = document.createElement("li");
  li.className = "item";
  li.textContent = `앞-${n}`;

  list.prepend(li);

  console.log(list.textContent?.trim()); // 예: "앞-1A"
  n += 1;
});
```

> `textContent` 출력은 HTML 줄바꿈/공백 구성에 따라 표시 형태가 달라질 수 있으나, “앞/뒤에 붙는다”는 점은 동일하다.

<br />

### before와 after

`before`와 `after`는 부모의 자식으로 넣는 것이 아니라, 선택한 요소 바깥의 **형제 위치**에 삽입한다.

```html
<p class="target">기준 요소</p>

<button class="btn-before">before</button>
<button class="btn-after">after</button>
```
```js
const target = document.querySelector(".target");
const btnBefore = document.querySelector(".btn-before");
const btnAfter = document.querySelector(".btn-after");

btnBefore?.addEventListener("click", () => {
  if (!target) return;

  const el = document.createElement("p");
  el.className = "mark";
  el.textContent = "앞에 추가";

  target.before(el);

  console.log(document.querySelectorAll(".mark").length); // 1, 2, 3 ... (클릭할 때마다 증가)
});

btnAfter?.addEventListener("click", () => {
  if (!target) return;

  const el = document.createElement("p");
  el.className = "mark";
  el.textContent = "뒤에 추가";

  target.after(el);

  console.log(document.querySelectorAll(".mark").length); // 1, 2, 3 ... (클릭할 때마다 증가)
});
```

---
<br />

## 3. 문자열로 추가: insertAdjacentHTML

요소를 하나씩 만들기 번거로울 때는 HTML 문자열로 한 번에 삽입할 수 있다.

### insertAdjacentHTML의 위치 옵션

- `beforebegin`: 대상 요소 바로 앞(형제)
- `afterbegin`: 대상 요소 안쪽 첫 번째 자식
- `beforeend`: 대상 요소 안쪽 마지막 자식
- `afterend`: 대상 요소 바로 뒤(형제)

```html
<div class="cards"></div>
<button class="btn">카드 추가</button>
```
```js
const cards = document.querySelector(".cards");
const btn = document.querySelector(".btn");

let id = 1;

btn?.addEventListener("click", () => {
  if (!cards) return;

  cards.insertAdjacentHTML(
    "beforeend",
    `<div class="card" data-id="${id}">카드 ${id}</div>`
  );

  console.log(cards.querySelectorAll(".card").length); // 1, 2, 3 ...
  id += 1;
});
```

> 외부 입력값(사용자 입력, 서버 응답 등)을 그대로 HTML 문자열로 합쳐 넣는 방식은 보안 문제가 될 수 있으므로 주의해야 한다.
{: .prompt-warning }

---
<br />

## 4. 요소 삭제: remove, removeChild, 자식 비우기

### remove로 요소 1개 삭제하기

remove()는 호출한 요소 자신을 DOM에서 제거한다.

```html
<ul class="list">
  <li class="item">A</li>
  <li class="item">B</li>
</ul>

<button class="btn">첫 항목 삭제</button>
```
```js
const btn = document.querySelector(".btn");

btn?.addEventListener("click", () => {
  const first = document.querySelector(".list .item");
  if (!first) {
    console.log("삭제할 항목이 없습니다.");
    return;
  }

  first.remove(); // 요소 1개 삭제
});
```

<br />

### removeChild로 삭제하기(레거시 방식)

removeChild는 부모 요소에서 특정 자식을 제거하는 방식이다.

```html
<ul class="list">
  <li class="item">A</li>
  <li class="item">B</li>
</ul>

<button class="btn">마지막 항목 삭제</button>
```
```js
const list = document.querySelector(".list");
const btn = document.querySelector(".btn");

btn?.addEventListener("click", () => {
  if (!list) return;

  const last = list.lastElementChild;
  if (!last) {
    console.log("삭제할 항목이 없습니다.");
    return;
  }

  list.removeChild(last);
  console.log(list.children.length); // 1 -> 0 -> 이후에는 "삭제할 항목이 없습니다."
});
```

<br />

### 자식 요소를 “전부” 비우기: replaceChildren, innerHTML = ""

**부모는 남기고, 자식만 모두 지우기**가 필요할 때가 많다.

```html
<ul class="list">
  <li class="item">A</li>
  <li class="item">B</li>
  <li class="item">C</li>
</ul>

<button class="btn-clear">자식 비우기</button>
<button class="btn-reset">다시 채우기</button>
```
```js
const list = document.querySelector(".list");
const btnClear = document.querySelector(".btn-clear");
const btnReset = document.querySelector(".btn-reset");

btnClear?.addEventListener("click", () => {
  if (!list) return;

  list.replaceChildren();
  console.log(list.children.length); // 0
});

btnReset?.addEventListener("click", () => {
  if (!list) return;

  list.innerHTML = `
    <li class="item">A</li>
    <li class="item">B</li>
    <li class="item">C</li>
  `;

  console.log(list.children.length); // 3
});
```

> `innerHTML`은 요소를 추가하는게 아닌, 기존 자식들을 통째로 갈아끼우는(교체하는) 동작이다.

`replaceChildren()`는 자식 요소를 모두 제거한다는 의도가 명확하다.  
innerHTML = ""도 동일하게 자식을 비우지만, HTML 문자열 기반이므로 DOM을 다시 채우는 코드까지 함께 쓸 경우 유지보수나 안전성 측면에서 주의가 필요하다.

---
<br />

## 5. 실전 패턴: 이벤트 위임으로 동적 목록 삭제하기

목록 항목이 동적으로 추가될 때는, 항목마다 이벤트를 붙이기보다 부모에서 한 번에 처리하는 방식이 흔하다.

### 동적 추가 + 개별 삭제

```html
<ul class="todo-list"></ul>

<button class="btn-add">항목 추가</button>
```
```js
const list = document.querySelector(".todo-list");
const btnAdd = document.querySelector(".btn-add");

let id = 1;

btnAdd?.addEventListener("click", () => {
  if (!list) return;

  list.insertAdjacentHTML(
    "beforeend",
    `
      <li class="todo-item" data-id="${id}">
        할 일 ${id}
        <button class="btn-remove" data-action="remove">삭제</button>
      </li>
    `
  );

  console.log(list.querySelectorAll(".todo-item").length); // 1, 2, 3 ...
  id += 1;
});

list?.addEventListener("click", (e) => {
  const t = e.target;
  if (!(t instanceof Element)) return;

  if (!t.matches('[data-action="remove"]')) return;

  const item = t.closest(".todo-item");
  if (!item) return;

  console.log(item.dataset.id); // 예: "1" (클릭한 항목 id)
  item.remove();

  console.log(list.querySelectorAll(".todo-item").length); // 삭제 후 남은 개수
});
```

---
<br />

## 마무리

DOM의 생성·추가·삭제는 UI 구조를 동적으로 바꾸는 핵심 작업이며,  
createElement로 안전하게 조립하고 append/prepend/before/after로 위치를 제어하며,  
삭제는 remove 또는 자식 비우기(replaceChildren)처럼 목적에 맞는 API를 선택하는 것이 중요하다.