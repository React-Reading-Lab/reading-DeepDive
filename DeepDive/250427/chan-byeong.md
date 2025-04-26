# 3. 리액트 훅 깊게 살펴보기

## 3.1 리액트의 모든 훅 파헤치기

### 3.1.1 useState

함수형 컴포넌트는 렌더링 시 매번 함수를 실행시킨다.

즉, 일반적으로 선언한 변수의 경우 매 렌더링마다 초기화된다.

렌더링 간에도 값을 계속해서 유지하기 위해서 리액트에서는 `useState`라는 훅을 도입하였고, `useState` 훅은 내부에서 `클로저`를 활용하여
값을 렌더링 간에도 유지한다.

- `useState`의 게으른 초기화

일반적으로 useState의 초기값으로 원시값을 넣어서 초기화한다.

하지만 어떤 값을 반환하는 함수를 초기값으로 넣어 초기화할 수도 있다.

해당 함수는 초기값이 없을 경우 즉, 초기 렌더링에서만 실행되고 이후 렌더링에서는 해당 함수는 실행되지 않는다.

이러한 게으른 초기화는 무겁고 복잡한 연산을 초기 렌더링에서만 수행할 수 있게 해줌으로써 성능을 높이는데 도움을 준다.

### 3.1.2 useEffect

`useEffect`는 의존성 배열에 존재하는 값들을 이전 값들과 비교하여 변경사항이 있을 경우 콜백함수를 실행시킨다.

이때 값들의 비교는 `얕은 비교`을 통해서 이루어진다.

그렇다면 어떻게 이전 값들과 새로운 값을 비교할 수 있을까? Fiber 아키텍처에서 이전 값을 저장하기 때문에 값의 비교를 할 수 있다.

- Clean up function

clean up function은 `이전 state`를 참조하여 실행된다.

실행 순서:

```text
phase1     ->    phase2
useEffect-1      useEffect-1's clean up function
                 useEffect-2
```

- 의존성 배열

1. 의존성 배열이 없는 경우

매 렌더링 시 실행된다.

그렇다면 `useEffect`를 쓰지 않는 것과 어떤 차이가 있을까?

- `useEffect`는 클라이언트 사이드에서 실행되는 것을 보장한다. 따라서 `window` 객체에 접근도 가능하다.
- `useEffect`는 렌더링이 완료된 후 실행된다. 만약 서버사이드 렌더링이라면 무거운 연산을 `useEffect` 외부에서 수행한다면 서버에서 렌더링하는 시간이 더욱 길어질 것이다.

- 그외의 팁들

  1. 복잡한 함수의 경우 분리하거나 익명함수가 아닌 **기명함수**를 쓰는 것이 좋다.
  2. 거대한 `useEffect`는 자제하도록
  3. `useEffect`의 내부에서 사용하는 함수는 내부에 정의하고 사용하는 것이 깔끔하다.

- `useEffect`의 콜백으로 비동기 함수는 왜 안될까?

이전 실행과 이후 실행 간 `race condition`이 발생할 수 있기 때문이다.

이전 실행이 비동기로 약 5초, 이후 실행이 약 1초만에 완료된다면, 이전 실행과 이후 실행이 접근하는 값이 잘못된 경우가 존재할 수 있다.

하지만 콜백 함수 내부에서 비동기 함수를 정의하고 실행하는 것은 가능하다.

이때는 클린업 함수에서 이전 비동기 함수에 대한 처리를 해주는 것이 좋다.

### 3.1.3 useMemo

값을 기억하는 훅

- 첫번째 인자: 어떤 값을 반환하는 함수
- 두번째 인자: 해당 함수가 의존하는 값의 배열

### 3.1.4 useCallback

함수를 기억하는 훅

`useMemo`와 동일하지만 첫번째 인자로 함수를 받는다.

- `useCallback`의 쓰임

만약 `props`로 함수를 받고, 해당 함수는 `React.memo`를 통해 메모이제이션이 된 경우.

일반적으로 `props`의 변경이 없다면 해당 함수는 리렌더링되지 않는다.

하지만 상위 컴포넌트에서 `props`로 넘기는 함수를 메모이제이션하지 않는다면 해당 함수는 새롭게 생성되어

`props` 비교 시 변경된 것으로 인지하여 하위 `React.memo`를 통해 메모이제이션한 컴포넌트도 리렌더링되게 된다.

=> 이러한 현상은 `useCallback`을 통해 함수 자체를 메모이제이션하여 막을 수 있다.

### 3.1.5 useRef

- `useRef`의 반환 객체의 `current`로 값에 접근 또는 변경이 가능하다.
- `useRef`의 값이 변경되어도 렌더링되지 않는다.

### 3.1.6 useContext

리액트에서 Context란 무엇인지부터 알아보자

- Context

Context를 사용함으로써 명시적 `props` 전달 없이도 하위 컴포넌트에서 값을 사용할 수 있게 해준다.

Context를 함수 컴포넌트에서 사용 가능하게 해주는 것이 `useContext`이다.

```javascript
const Context = createContext<{val: string} | undefined>(undefined)

<Context.Provider value={val: "123"}>
  <App />
</Context.Provider>
```

`useContext`는 가장 가까운 `Provider`의 값을 참조한다.

_주의할 점_

- Context 사용 시 하위 컴포넌트는 `Provider`에 대한 의존성을 가지는 것이다. 이는 컴포넌트의 재사용성을 떨어뜨린다.

- Context는 단순히 상태를 주입하는 개념일 뿐 전역 상태 관리 라이브러리가 아니다.

### 3.1.7 useReducer

복잡한 상태를 관리하는데 적합한 훅

- 반환값:

  1. state: 현재 `reducer`가 가지고 있는 값
  2. dispatcher: state를 업데이트하는 함수.

- 인자:
  1. reducer: action을 정의하는 함수.
  2. initialState: `useReducer`의 초기값을 의미
  3. init: 초기값을 지연 생성하고 싶을 때 사용하는 함수 인자. initialState를 인수로 init 함수가 실행된다.

```javascript
import React, { useReducer } from "react";

// 액션 타입 정의
type Action = { type: "INCREMENT" } | { type: "DECREMENT" } | { type: "RESET" };

// 리듀서 함수
function counterReducer(state: number, action: Action): number {
  switch (action.type) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    case "RESET":
      return 0;
    default:
      return state;
  }
}

function Counter() {
  // useReducer 사용
  const [count, dispatch] = useReducer(counterReducer, 0);

  return (
    <div>
      <h1>카운터: {count}</h1>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>증가</button>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>감소</button>
      <button onClick={() => dispatch({ type: "RESET" })}>리셋</button>
    </div>
  );
}
```

### 3.1.8 useImperativeHandle

- fowardRef

  컴포넌트 간 ref 전달에 일관성을 주기 위해서 등장하였다.

  원래는 ref는 예약어라서 props로 넘기지 못했다.

```javascript
  const ChildComponent = forwardRef((props, ref) => {
    useEffect(() => {
      console.log(ref)
    })
    return <div>{ref}</div>
  })

  function ParentComponent = () => {
    const inputRef = useRef()

    return (
      <>
        <input ref={inputRef} />
        <ChildComponent ref = {inputRef} />
      </>
    )
  }
```

- useImperativeHandle

부모에게서 넘겨받은 ref를 원하는 대로 수정할 수 있는 훅이다.

ref의 값에 원하는 값을 넣거나, 액션을 정의할 수 있다.

```javascript
import React, { forwardRef, useImperativeHandle, useRef } from 'react';

// 부모 컴포넌트에서 사용할 ref의 타입 정의
interface InputRef {
focus: () => void;
clear: () => void;
getValue: () => string;
}

// 자식 컴포넌트
const CustomInput = forwardRef<InputRef, { placeholder?: string }>((props, ref) => {
const inputRef = useRef<HTMLInputElement>(null);

// 부모 컴포넌트에서 사용할 수 있는 메서드 정의
useImperativeHandle(ref, () => ({
  focus: () => {
    inputRef.current?.focus();
  },
  clear: () => {
    if (inputRef.current) {
      inputRef.current.value = '';
    }
  },
  getValue: () => {
    return inputRef.current?.value || '';
  }
}));

return <input ref={inputRef} placeholder={props.placeholder} />;
});

// 부모 컴포넌트
function ParentComponent() {
const inputRef = useRef<InputRef>(null);

const handleFocus = () => {
  inputRef.current?.focus();
};

const handleClear = () => {
  inputRef.current?.clear();
};

const handleGetValue = () => {
  const value = inputRef.current?.getValue();
  console.log('현재 입력값:', value);
};

return (
  <div>
    <CustomInput ref={inputRef} placeholder="입력하세요" />
    <button onClick={handleFocus}>포커스</button>
    <button onClick={handleClear}>지우기</button>
    <button onClick={handleGetValue}>값 확인</button>
  </div>
);
}

export default ParentComponent;
```

### 3.1.9 useLayoutEffect

`useEffect`와 시그니처가 동일하고, 실행시점에 차이가 존재한다.

모든 DOM의 변경 후에 동기적으로 발생한다.

**실행 순서**

1. 리액트 DOM 업데이트
2. `useLayoutEffect` 실행
3. 브라우저 변경 사항 반영
4. `useEffect` 실행

성능 상의 이슈는 존재하나 화면에 반영되기 전에 하고 싶은 작업을 할 때 유용하다.

### 3.1.10 useDehubValue

개발 과정에서 사용되는 훅.

오직 다른 훅 내부에서만 실행이 가능하다.

### 3.1.11 훅의 규칙

- 최상위에서만 훅을 호출해야한다.
- 반복문, 조건문, 중첩된 함수 내에서 훅을 호출할 수 없다. (`use` 훅 예외)
- 훅은 함수 컴포넌트 혹은 사용자 정의 훅 내부에서만 호출 가능하다.

리액트 내부에서 훅의 순서대로 링크드리스트로 관리하기 때문에 조건문이나 반복문 등의 호출 순서에 영향이 가는 로직이 있는 경우 올바른 훅의 호출이 이어질 수 없다.
