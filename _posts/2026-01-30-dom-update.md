---
title: "[Vanilla JS] DOM 요소 수정: 텍스트·속성·클래스·스타일·구조 변경"
date: 2026-01-30 15:01 +0900
categories: [JavaScript]
tags: [javascript, vanilla-js, DOM, update]
---

**DOM 요소 수정**은 화면에 보이는 값이나 상태를 실제로 바꾸는 단계이다.    
사용자 입력에 따라 텍스트를 갱신하거나, 버튼 상태를 변경하고, 클래스·스타일을 조정하며,  
필요하다면 DOM 구조 자체를 추가·삭제하기도 한다.  
본 글에서는 바닐라 JavaScript에서 DOM을 수정하는 대표 방법을 예시 HTML과 함께 정리한다.

---
<br />

## 1. 텍스트 수정

### textContent로 텍스트 바꾸기

```html
<h2 class="title">안녕하세요</h2>
<button class="btn">문구 변경</button>
```
```js
const title = document.querySelector(".title");
const btn = document.querySelector(".btn");

btn?.addEventListener("click", () => {
  if (!title) return;

  title.textContent = "반갑습니다";
  console.log(title.textContent); // "반갑습니다"
});
```

<br />

### innerHTML로 마크업까지 바꾸기(주의)

```html
<div class="message">기본 문구</div>
<button class="btn">강조 문구</button>
```
```js
const msg = document.querySelector(".message");
const btn = document.querySelector(".btn");

btn?.addEventListener("click", () => {
  if (!msg) return;

  msg.innerHTML = "<strong>중요</strong> 문구로 변경";
  console.log(msg.innerHTML); // "<strong>중요</strong> 문구로 변경"
});
```

> `innerHTML`은 편리하지만, 사용자 입력값을 그대로 넣는 방식은 보안 문제(XSS)로 이어질 수 있으므로 주의해야 한다.
{: .prompt-warning }

---
<br />

## 2. 속성 수정

### setAttribute로 속성 변경

```html
<img class="avatar" src="/img/cat.png" alt="고양이" />
<button class="btn">이미지 변경</button>
```
```js
const img = document.querySelector(".avatar");
const btn = document.querySelector(".btn");

btn?.addEventListener("click", () => {
  if (!img) return;

  img.setAttribute("src", "/img/dog.png");
  img.setAttribute("alt", "강아지");
  console.log(img.getAttribute("src")); // "/img/dog.png"
});
```

<br />

### 프로퍼티로 상태 변경(value/checked 등)

```html
<label>
  <input id="email" type="email" value="test@example.com" />
  <input id="agree" type="checkbox" />
  약관 동의
</label>
<button class="emailBtn">이메일 변경</button>
<button class="agreeBtn">강제 체크</button>
```
```js
const input = document.querySelector("#email");
const agree = document.querySelector("#agree");
const emailBtn = document.querySelector(".emailBtn");
const agreeBtn = document.querySelector(".agreeBtn");

emailBtn?.addEventListener("click", () => {
  if (!(input instanceof HTMLInputElement)) return;

  input.value = "great@example.com";
  console.log(input.value); // "great@example.com"
})

agreeBtn?.addEventListener("click", () => {
  if (!(agree instanceof HTMLInputElement)) return;

  agree.checked = true;
  console.log(agree.checked); // true
});
```

<br />

### data-* 수정: dataset

```html
<li class="todo-item" data-status="todo">할 일</li>
<button class="btn">상태 변경</button>
```
```js
const item = document.querySelector(".todo-item");
const btn = document.querySelector(".btn");

btn?.addEventListener("click", () => {
  if (!item) return;

  item.dataset.status = "done";
  console.log(item.dataset.status); // "done"
});
```

---
<br />

## 3. 클래스 수정

### classList.add / remove / toggle

```html
<button class="btn-add">추가(add)</button>
<button class="btn-remove">제거(remove)</button>
<button class="btn-toggle">토글(toggle)</button>

<p class="desc">설명 문구</p>
```
```js
const desc = document.querySelector(".desc");

const btnAdd = document.querySelector(".btn-add");
const btnRemove = document.querySelector(".btn-remove");
const btnToggle = document.querySelector(".btn-toggle");

btnAdd?.addEventListener("click", () => {
  if (!desc) return;

  desc.classList.add("active");
  console.log(desc.className); // "desc active"
});

btnRemove?.addEventListener("click", () => {
  if (!desc) return;

  desc.classList.remove("active");
  console.log(desc.className); // "desc"
});

btnToggle?.addEventListener("click", () => {
  if (!desc) return;

  const next = desc.classList.toggle("active"); // 토글 결과(추가되면 true, 제거되면 false)
  console.log(next); // true 또는 false
  console.log(desc.className); // "desc active" 또는 "desc"
});
```

---
<br />

## 4. 스타일 수정

### 인라인 스타일 변경(style)

```html
<div class="box">BOX</div>
<button class="btn">색 변경</button>
```
```js
const box = document.querySelector(".box");
const btn = document.querySelector(".btn");

btn?.addEventListener("click", () => {
  if (!box) return;

  box.style.backgroundColor = "black";
  box.style.color = "white";

  console.log(box.style.backgroundColor); // "black"
});
```

<br />

### CSS 변수(--var) 수정

```html
<style>
  :root { --main-size: 16px; }
  .text { font-size: var(--main-size); }
</style>

<p class="text">텍스트</p>
<button class="btn">크기 키우기</button>
```
```js
const btn = document.querySelector(".btn");

btn?.addEventListener("click", () => {
  document.documentElement.style.setProperty("--main-size", "22px");

  const v = getComputedStyle(document.documentElement).getPropertyValue("--main-size").trim();
  console.log(v); // "22px"
});
```

---
<br />

## 5. DOM 구조 수정

### 요소 추가: createElement + append

```html
<ul class="list"></ul>
<button class="btn">항목 추가</button>
```
```js
const list = document.querySelector(".list");
const btn = document.querySelector(".btn");

let n = 1;

btn?.addEventListener("click", () => {
  if (!list) return;

  const li = document.createElement("li");
  li.textContent = `항목 ${n}`;

  list.append(li);

  console.log(list.children.length); // 1, 2, 3 ... (클릭할 때마다 증가)
  n += 1;
});
```

<br />

### 요소 삭제: remove

```html
<ul class="list">
  <li class="item">A</li>
  <li class="item">B</li>
</ul>
<button class="btn">첫 항목 삭제</button>
```
```js
const list = document.querySelector(".list");
const btn = document.querySelector(".btn");

btn?.addEventListener("click", () => {
  const first = list?.querySelector(".item");
  if (!first) {
    console.log("삭제할 항목이 없습니다.");
    return;
  }

  first.remove();
  console.log(list.children.length); // 1 ... (클릭할 때마다 감소)
});
```

<br />

### 템플릿 문자열로 삽입: insertAdjacentHTML

```html
<div class="cards"></div>
<button class="btn">카드 추가</button>
```
```js
const cards = document.querySelector(".cards");
const btn = document.querySelector(".btn");

btn?.addEventListener("click", () => {
  if (!cards) return;

  cards.insertAdjacentHTML(
    "beforeend",
    `<div class="card"><strong>새 카드</strong>가 추가되었습니다.</div>`
  );

  console.log(cards.querySelectorAll(".card").length); // 1, 2, 3 ... (추가된 카드 수)
});
```

> `insertAdjacentHTML`은 빠르고 간단하지만, 외부 입력값을 그대로 넣는 방식은 보안 문제로 이어질 수 있다.
{: .prompt-warning }


안전이 중요하다면 `createElement` 기반 조립이 더 적합하다.

---
<br />

## 6. 폼 입력값 초기화 및 상태 리셋

### value 초기화와 focus 이동

```html
<input class="name" type="text" value="test" />
<button class="btn">초기화</button>
```
```js
const input = document.querySelector(".name");
const btn = document.querySelector(".btn");

btn?.addEventListener("click", () => {
  if (!(input instanceof HTMLInputElement)) return;

  input.value = "";
  input.focus();

  console.log(input.value); // ""
});
```

---
<br />

## 마무리

DOM 수정은 
- 텍스트(textContent)
- 속성(setAttribute/프로퍼티/dataset)
- 클래스(classList)
- 스타일(style/CSS 변수)
- 구조(추가·삭제·삽입)

예시 HTML 구조를 함께 두고 “어떤 상태를 바꾸는지”를 명확히 하여 코드를 작성하는 것이 중요하다.