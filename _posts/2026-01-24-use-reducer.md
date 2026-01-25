---
title: "[React] useReducer 정리"
date: 2026-01-25 16:50 +0900
categories: [React]
tags: [React, Hooks, UseReducer, State, Reducer, Dispatch]
---

`useReducer`는 React에서 상태를 관리하기 위한 훅(Hook) 중 하나이다.  
`useState`가 “값을 직접 바꾸는 방식”에 가깝다면, `useReducer`는 **상태 변화 규칙을 함수로 분리**하여 상태를 갱신하는 방식에 가깝다.

특히 상태가 복잡해지거나, 여러 이벤트에 따라 동일한 상태를 다양한 방식으로 갱신해야 할 때 `useReducer`는 코드의 구조를 안정적으로 만들어준다.

---
<br />

## useReducer의 기본 형태

`useReducer`의 기본 사용 형태는 다음과 같다.

- **Reducer**: `(state, action) => nextState` 형태의 함수
- **State**: 현재 상태
- **Dispatch**: 액션을 전달하여 상태 변경을 요청하는 함수

즉, “상태를 어떻게 바꿀지”는 `reducer`가 책임지고, 컴포넌트는 “무슨 일이 일어났는지(action)”만 전달한다.

```tsx
import { useReducer } from "react";

type State = {
  // 예: 폼/리스트 등 원하는 상태로 확장 가능
  value: number;
};

type Action =
  | { type: "SetValue"; payload: number }
  | { type: "Reset" };

const initialState: State = { value: 0 };

// reducer는 보통 "컴포넌트 밖"에 둔다(재사용/테스트 용이)
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "SetValue":
      return { ...state, value: action.payload };
    case "Reset":
      return initialState;
    default:
      return state;
  }
}

export default function Example() {
// useReducer는 보통 "컴포넌트 최상단(훅 영역)"에 둔다
  const [state, dispatch] = useReducer(reducer, initialState);

  // 기본 형태 예시: 상태 변경은 dispatch로 "요청"만 한다
  const setTo10 = () => dispatch({ type: "SetValue", payload: 10 });
  const reset = () => dispatch({ type: "Reset" });

  return (
    <div>
      <p>Value: {state.value}</p>
      <button onClick={setTo10}>Set 10</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

```

---
<br />

## 핵심 개념: Reducer와 Action

### Reducer란 무엇인가

Reducer는 상태 변경 규칙을 정의하는 함수이다.  
현재 상태(`state`)와 이벤트 정보(`action`)를 입력으로 받아, 다음 상태(`nextState`)를 반환한다.

Reducer는 다음 원칙을 따르는 것이 바람직하다.

- 같은 입력이면 같은 결과를 반환한다.
- 외부 값을 변경하는 부작용을 만들지 않는다.
- 기존 상태를 직접 수정하지 않고, 새로운 상태를 만들어 반환한다.

### Action이란 무엇인가

Action은 “어떤 일이 발생했는지”를 나타내는 객체(혹은 값)이다.  
일반적으로 다음과 같은 형태로 사용한다.

- `type`: 동작을 구분하는 식별자
- `payload`: 상태 변경에 필요한 추가 데이터

액션은 단순히 reducer에게 전달되는 “요청서”에 가깝다.

---
<br />

## useReducer가 유리해지는 상황

`useReducer`는 모든 상태 관리에 필수인 도구는 아니다.  
그러나 아래 조건에서는 확실한 장점이 있다.

### 1) 상태 구조가 복잡한 경우

객체나 배열 형태의 상태가 커지고, 여러 필드가 함께 움직이는 경우가 많다면 `useState`로는 관리가 산만해지기 쉽다.  
`useReducer`는 “변경 규칙”을 한 곳에 모아둘 수 있으므로, 전체 흐름을 파악하기 쉬워진다.

### 2) 상태 변경 케이스가 많은 경우

예를 들어 “추가 / 수정 / 삭제 / 초기화 / 토글”처럼 이벤트가 다양해지면, 컴포넌트 내부에서 상태 변경 코드가 흩어진다.  
이때 reducer에 로직을 모으면, 이벤트별 동작이 명확해진다.

### 3) 테스트와 유지보수를 고려하는 경우

Reducer는 입력과 출력이 명확한 함수이므로, 단위 테스트가 상대적으로 쉽다.  
또한 UI 코드와 상태 변경 규칙이 분리되어 유지보수 비용이 내려가는 편이다.

---
<br />

## useState와 useReducer의 관점 차이

두 훅은 경쟁 관계라기보다 “적합한 상황이 다른 도구”에 가깝다.

- `useState`: 단순한 값, 단일 입력으로 갱신되는 상태에 적합
- `useReducer`: 복잡한 상태, 다양한 액션으로 갱신되는 상태에 적합

실제로는 하나의 컴포넌트 안에서도 두 방식을 혼합하여 쓰는 경우가 많다.

---
<br />

## 초기 상태와 Lazy Initialization

`useReducer`는 초기 상태를 단순 값으로 줄 수도 있고, 초기값 계산 함수를 두어 지연 초기화(Lazy Initialization)를 적용할 수도 있다.

지연 초기화는 다음과 같은 상황에서 유용하다.

- 초기 상태를 만들기 위한 연산 비용이 큰 경우
- 컴포넌트 렌더링마다 초기값 계산이 반복되는 것을 피하고 싶은 경우

즉, 초기 상태 계산을 “첫 렌더링 1회”로 제한하고 싶을 때 의미가 있다.

{% raw %}
```tsx
import { useReducer } from "react";

type ExpensiveState = { items: number[] };
type ExpensiveAction = { type: "Reverse" };

// 초기 상태를 만드는 비용이 크다고 가정한 함수
function initExpensiveState(size: number): ExpensiveState {
  return { items: Array.from({ length: size }, (_, i) => i) };
}

function expensiveReducer(state: ExpensiveState, action: ExpensiveAction): ExpensiveState {
  switch (action.type) {
    case "Reverse":
      return { items: [...state.items].reverse() };
    default:
      return state;
  }
}

export function ExpensiveList() {
  // useReducer(reducer, initialArg, init)
  // initialArg(여기서는 10000)을 init 함수로 넘겨 초기 상태를 "1회" 생성한다
  const [state, dispatch] = useReducer(expensiveReducer, 10000, initExpensiveState);

  return (
    <div style={{ display: "grid", gap: 12 }}>
      <button onClick={() => dispatch({ type: "Reverse" })}>Reverse</button>
      <div>Items: {state.items.length}</div>
    </div>
  );
}
```
{% endraw %}

---
<br />

## Reducer 작성 시 자주 발생하는 실수

### 1) 기존 상태를 직접 수정하는 경우

객체/배열 상태를 `push`, `splice` 등으로 직접 수정하면 React는 변경을 예측하기 어려워지고, 렌더링 동작이 꼬일 수 있다.  
상태는 **불변성을 지키며 새 객체로 반환**하는 방식이 기본이다.

```ts
type State = { todos: string[] };
type Action = { type: "Add"; payload: string };

function badReducer(state: State, action: Action): State {
  if (action.type === "Add") {
    // 나쁜 예: 기존 배열을 직접 수정
    state.todos.push(action.payload);
    return state;
  }
  return state;
}

function goodReducer(state: State, action: Action): State {
  if (action.type === "Add") {
    // 좋은 예: 새 배열을 만들어 반환
    return { todos: [...state.todos, action.payload] };
  }
  return state;
}
```

### 2) Reducer에서 부작용을 처리하는 경우

Reducer는 “상태를 계산하는 함수”로 유지하는 편이 좋다.  
네트워크 요청, 로컬스토리지 접근, 타이머 등록 같은 부작용은 reducer가 아니라 Effect 계층에서 처리하는 편이 안정적이다.

### 3) Action 설계가 불명확한 경우

액션 타입이 너무 세분화되거나, 반대로 너무 뭉뚱그려져 있으면 reducer가 읽기 어려워진다.  
“이 액션이 왜 존재하는지”가 드러나도록 이름과 payload 구조를 잡는 것이 중요하다.

---
<br />

## 예시: 카운터에서 상태 규칙 분리하기

아주 단순한 예시로도 `useReducer`의 구조적 장점이 드러난다.

{% raw %}
```tsx
import { useReducer } from "react";

type CounterState = { count: number };

type CounterAction =
  | { type: "Increment" }
  | { type: "Decrement" }
  | { type: "Reset" }
  | { type: "AddBy"; payload: number };

const initialState: CounterState = { count: 0 };

function counterReducer(state: CounterState, action: CounterAction): CounterState {
  switch (action.type) {
    case "Increment":
      return { count: state.count + 1 };
    case "Decrement":
      return { count: state.count - 1 };
    case "Reset":
      return initialState;
    case "AddBy":
      return { count: state.count + action.payload };
    default:
      return state;
  }
}

export default function Counter() {
  const [state, dispatch] = useReducer(counterReducer, initialState);

  const onAdd5 = () => dispatch({ type: "AddBy", payload: 5 });

  return (
    <div style={{ display: "grid", gap: 12, maxWidth: 280 }}>
      <h3>Count: {state.count}</h3>

      <div style={{ display: "flex", gap: 8, flexWrap: "wrap" }}>
        <button onClick={() => dispatch({ type: "Decrement" })}>-1</button>
        <button onClick={() => dispatch({ type: "Increment" })}>+1</button>
        <button onClick={onAdd5}>+5</button>
        <button onClick={() => dispatch({ type: "Reset" })}>Reset</button>
      </div>
    </div>
  );
}
```
{% endraw %}

- 컴포넌트는 버튼 클릭 시 `dispatch`만 호출한다.
- 상태 변화는 reducer에만 존재한다.

이 방식은 기능이 늘어날수록 차이가 커진다.  
처음에는 `useState`가 더 간단해 보이지만, 액션이 늘어날수록 reducer 기반 구조가 유지보수에 유리해지는 경향이 있다.

---
<br />

## useReducer와 Redux의 관계

`useReducer`는 Redux와 직접적으로 연결된 기능은 아니나, **상태 관리 방식의 철학과 구조가 매우 유사**하다고 한다.  
Redux 또한 `Reducer(state, action) => nextState` 형태로 상태 변경 규칙을 정의하고, 컴포넌트는 `dispatch`를 통해 `action`을 전달하는 흐름을 따른다고 한다.

따라서 `useReducer`를 이해하면 Redux에서 가장 중요한 개념인 **Reducer / Action / Dispatch**를 이미 익힌 셈이 된다.  
이후 Redux를 학습할 때는 “새로운 개념”보다는, 동일한 패턴을 **전역(Store)으로 확장**하고 **연결 도구(Provider, Selector 등)**를 사용하는 방식에 집중하면 된다.

다만 모든 상태를 전역으로 올리는 것은 오히려 복잡도를 높일 수 있으므로,  
규모가 작은 상태는 `useState`나 `useReducer`로 관리하고, 전역 공유가 필요한 경우에만 Redux와 같은 전역 상태 관리 도구를 고려하는 편이 합리적이다.

---
<br />

## 마무리

`useReducer`는 상태 관리 자체를 “복잡하게 만들기 위한 도구”가 아니라,  
복잡해질 수밖에 없는 상태를 **정돈된 규칙으로 관리하기 위한 도구**이다.

정리하면 다음과 같다.

- 상태가 단순하면 `useState`가 효율적이다.
- 상태 구조가 복잡하거나 변경 케이스가 많으면 `useReducer`가 유리하다.
- reducer는 순수 함수로 유지하고, 부작용은 별도로 분리하는 것이 안정적이다.

React에서 상태는 결국 UI의 근거가 된다.  
`useReducer`는 그 근거를 “규칙 중심”으로 관리하게 해주며, 규모가 커질수록 효과가 두드러지는 방식이라 할 수 있다.
