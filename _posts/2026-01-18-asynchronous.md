---
title: "[JavaScript] 비동기 처리 방식의 차이: 콜백, Promise, async/await"
date: 2026-01-18 16:40 +0900
categories: [JavaScript]
tags: [javascript, async, promise, callback]
---

자바스크립트는 **싱글 스레드 언어**로,  
기본적으로 한 번에 하나의 작업을 처리하는 방식으로 동작한다.  
따라서 오래 걸리는 작업(서버 요청, 파일 읽기, 타이머 등)을 그대로 기다리면 화면이 멈춘 것처럼 보일 수 있다.  
이 문제를 피하기 위해 자바스크립트는 오래 걸리는 작업을 **비동기 처리**로 넘기고, 작업이 끝났을 때 결과를 받아 이어서 처리하는 방식을 사용한다.

이 글에서는 비동기 처리를 대표하는 세 가지 방식인 **콜백(Callback)**, **Promise**, **async/await**의 차이를 정리한다.

---
<br />

## 1. 콜백(Callback)

### 1) 개념
콜백은 “나중에 실행할 함수”를 인자로 넘기는 방식이다.  
즉, 어떤 작업이 끝났을 때 실행할 로직을 함수로 전달해 두는 구조이다.

### 2) 예시
```js
function fetchUser(callback) {
  setTimeout(() => {
    callback(null, { id: 1, name: "Jaeyoung" });
  }, 500);
}

fetchUser((err, user) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(user);
});
```

### 3) 장단점

장점: 단순한 작업에서는 구현이 빠르고 직관적이다.

단점: 작업이 여러 단계로 이어지면 콜백이 중첩되어 코드가 복잡해진다.

---
<br />

## 2. 콜백 지옥(Callback Hell) 문제

콜백 방식은 비동기 작업이 늘어날수록 들여쓰기와 분기 처리가 계속 깊어진다.
특히 에러 처리와 단계별 로직이 섞이면 흐름을 한눈에 파악하기 어려워진다.

```js
login((err, token) => {
  if (err) return handleError(err);

  fetchProfile(token, (err, profile) => {
    if (err) return handleError(err);

    fetchPosts(profile.id, (err, posts) => {
      if (err) return handleError(err);

      render(posts);
    });
  });
});
```
이처럼 “콜백 안에 콜백이 계속 들어가는 구조”를 흔히 **콜백 지옥**이라고 부른다.

---
<br />

## 3. Promise

### 1) 개념

Promise는 비동기 작업을 “객체”로 감싸서, 작업이 끝난 뒤 이어서 할 일을 then/catch로 연결하는 방식이다.  
즉, “나중에 완료될 결과”를 약속(Promise) 형태로 다루는 것이다.  

- 성공하면 `then()`으로
- 실패하면 `catch()`로 흐름을 분리할 수 있다.

### 2) 예시
```js
function fetchUser() {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve({ id: 1, name: "Test" }), 500);
  });
}

fetchUser()
  .then((user) => {
    console.log(user);
  })
  .catch((err) => {
    console.error(err);
  });
```

### 3) 체이닝(Chaining)

Promise는 반환값을 이어받아 다음 작업을 자연스럽게 연결할 수 있다.

```js
fetchUser()
  .then((user) => fetchPosts(user.id))
  .then((posts) => render(posts))
  .catch((err) => handleError(err));
```

### 4) Promise의 장점

콜백 중첩이 줄어들어 흐름이 비교적 읽기 쉬워진다.  

에러 처리를 `catch()`로 모을 수 있어 구조가 정돈된다.

---
<br />

## 4. async/await

### 1) 개념

async/await는 Promise 기반 코드를 **동기 코드처럼 순서대로 보이게** 작성하도록 도와주는 문법이다.  
겉보기에는 “위에서 아래로” 흐르는 코드처럼 작성할 수 있어 가독성이 좋아진다.  

async가 붙은 함수는 항상 Promise를 반환한다.  

await는 Promise가 끝날 때까지 기다린 뒤 결과를 반환받는다.

### 2) 예시

```js
async function run() {
  try {
    const user = await fetchUser();
    const posts = await fetchPosts(user.id);
    render(posts);
  } catch (err) {
    handleError(err);
  }
}

run();
```

### 3) 장점

- 단계별 흐름을 눈으로 따라가기 쉬워진다.   
- try/catch로 에러 처리를 한 곳에 모을 수 있다.

---
<br />

## 5. 세 방식의 비교 정리

### 1) 코드 흐름(읽기 쉬운 정도)

콜백 : 단순할 때는 괜찮지만, 단계가 늘면 급격히 복잡해진다.  

Promise : then/catch로 흐름이 정리되지만, 체인이 길어지면 가독성이 떨어질 수 있다.  

async/await : 대체로 가장 읽기 쉽고, 로직 설명이 자연스럽다.  

### 2) 에러 처리

콜백 : 단계마다 if (err) ... 처리가 반복되기 쉽다.  

Promise : catch()로 모을 수 있다.  

async/await : try/catch로 모을 수 있다.  

### 3) 실제 사용 추천

간단한 1회성 비동기 : 콜백도 가능하나, 요즘은 Promise/async를 더 많이 사용한다.  

여러 단계가 이어지는 비동기 : Promise 또는 async/await가 적합하다.  

유지보수/가독성이 중요한 코드 : async/await가 특히 유리하다.  

---
<br />

## 마무리

콜백은 가장 기본적인 방식이지만, 단계가 늘면 코드가 쉽게 복잡해진다.

Promise는 then/catch 체이닝으로 흐름과 에러 처리를 정리해준다.

async/await는 Promise 코드를 동기 코드처럼 읽히게 만들어 가독성과 유지보수에 유리하다.