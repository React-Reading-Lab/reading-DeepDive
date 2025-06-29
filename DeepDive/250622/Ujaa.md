# 17버전 살펴보기

16버전과 다르게 새롭게 추가된 기능이 없다. 그냥 다음 버전 업을 위한 준비 버전이라고 봐도 무방하다.

## 점진적 업그레이드

리액트 17부터는 이후에 18로 업그레이드를 할 때 일부 기능에 대해서는 17에 머물러 있는 것이 가능하다. 버전을 전체적으로 올리기 부담스러울 때 사용할 수 있다.

## 이벤트 위임 방식 변경

리액트는 이벤트 핸들러를 해당 이벤트 핸들러를 추가한 각각의 DOM 요소에 부탁하는 것이 아니라, 이벤트 타입당 하나의 핸들러를 루트에 부착한다. 리액트 16에서는 이 이벤트 위임을 document에서 수행했지만 17부터는 리액트 컴포넌트 최상단 트리 즉 루트 요소로 바뀌었다.

## import React from ‘react’가 더 이상 필요 없다

기존에는 jsx를 바벨이나 타입스크립트를 활용해 일반적인 JS로 변환이 필요했다. 이때 변환될 코드에 `React.createElement`가 필요했기 때문에 항상 `import React from ‘react’`를 작성해야 했지만 17버전부터는 사용하지 않아도 된다. 무려 불필요한 import 구문을 삭제해 번들링 크기도 약간 줄여준다.

## 기타 변경 사항

### 이벤트 풀링 제거

리액트는 원래 이벤트를 처리하기 위한 SyntheticEvent라는 이벤트가 있다. 이는 브라우저의 기본 이벤트를 한 번 감싼 이벤트 객체다. 때문에 이벤트를 처리할 때마다 브라우저 이벤트를 합성 이벤트로 감싸는 작업이 필요했다. 이런 작업을 줄이기 위해 SyntheticEvent 풀을 만들어서 이벤트가 발생할 때마다 가져와서 사용했다. 사용한 syntheticEvent은 모든 이벤트 필드를 null로 바꾸고 풀에 돌려놓는데 이 점 때문에 비동기적으로 이벤트에 접근할 경우 null로 접근이 되는 문제가 발생했다. 이를 해결하기 위해 e.persist() 함수를 사용할 수 있지만 이게 불편하고 크게 성능향상에 도움이 안되어 이벤트 풀링이 제거되었다.

### useEffect 클린업 함수의 비동기 실행

클린업 함수가 화면 업데이트가 완전히 끝난 이후에 실행되도록 변경되었다.  commit까지 걸리는 시간을 조금 줄여주었다.

### 컴포넌트 undefined 반환에 대한 일관적인 처리

16에서는 forwardRef나 memo에서 undefined를 반환하는 것에 대해 에러가 나지 않았는데 17에서는 컴포넌트 내부에서 undefined를 반환하면 일관적으로 에러가 나도록 처리했다.

# 18버전 살펴보기

17버전과 다르게 새로운 기능이 추가되었다. 가장 큰 변경점은 동시성 지원이다.

## 새로 추가된 훅들

### useId

컴포넌트별로 유니크한 값을 생성하는 새로운 훅. 클라이언트와 서버의 불일치를 피하면서도 컴포넌트 내부의 고유한 값을 생성할 수 있다. :로 감싸져 CSS 선택자나 querySelector 에서 작ㅈ동하지 않도록 하기 위한 의도적 결과다.

### useTransition

UI 변경을 가로막지 않고 상태를 업데이트할 수 있는 리액트 훅이다. 상태 업데이트를 긴급하지 않은 것으로 간주해 무거운 렌더링 작업을 조금 미룰 수 있다. (실수로 무거운 연산을 요청하는 상호작용을 했다가 다른 상호작용을 하면 다른 상호작용으로 넘어감)

```jsx
const [isPending, startTransition] = useTransition()
```

### useDeferredValue

리렌더링이 급하지 않은 부분을 지연할 수 있게 도와주는 훅이다. 디바운스왕 비슷한 느낌이지만 고정된 지연 시간 없이 첫 번째 레더링이 완료된 이후에 이 useDeferredValue로 지연된 렌더링을 수행한다.

### useSyncExternalStore

17버전의 useSubscription 훅이 변경된 것이다. 우선 18버전에서는 동시성을 지원하면서 같은 값을 참조해도 해당 값이 중간에 변경됨에 따라 다른 렌더링 결과를 나타내는 테어링 현상이 리액트가 관리하지 못하는 document.body, window.innerWidth 등을 대상으로 생길 수 있게 되었다. useSyncExternalStore은 이러한 문제를 해결하기 위한 훅이다.

### useInsertionEffect

CSS-in-JS 라이브러리를 위한 훅이다. DOM이 실제로 변경되기 전에 동기적으로 실행된다. 브라우저가 레이아웃을 계산하기 전에 실행된다. useInsertionEffect → useLayoutEffect → useEffect 순으로 실행된다.

## react-dom/client

리액트 트리를 만들 때 사용하는 API가 변경되었다.

### createRoot

```jsx
// before
ReactDOM.render(<App />, container)

// after
ReactDOM.createRoot(container).render(<App />)
```

### hydrateRoot

```jsx
// before
ReactDOM.hydrate(<App />, container)

// after
ReactDOM.hydrateRoot(container, <App />)
```

## react-dom/server

### renderToPipeableStream

react 컴포넌트를 HTML로 렌더링하는 메서드로 스트리밍을 지원한다.

### renderToReadableStream

renderToPipeableStream는 Node.js 환경에서 렌더링을 위해 사용되고, renderToReadableStream는 웹 스트리밍을 기반으로 작동한다.

## 자동 배치

자동 배치는 여러 상태 업데이트를 하나의 리렌더링에 묶어서 성능을 향상시키는 것으로 react 18부터는 동기 비동기 관계 없이 자동 배치가 일어나며 루트 컴포넌트를 createRoot로 사용해서 만들면 된다.

만약 자동 배치를 하고 싶지 않으면 flushSync를 사용하면 된다.

## 더욱 엄격해진 엄격 모드

<StrictMode>를 통해 리액트는 잠재적인 버그를 찾는다. 개발자 모드에서만 작동하며 이 기능이 더 엄격해졌다.

- 더 이상 안전하지 않은 특정 생명주기를 사용하는 컴포넌트에 대한 경고(componentWillMount, componentWillReceiveProps, componentWillUpdate)
- 문자열 ref 사용 금지(충돌여지 있음, 어떤 ref에서 참조된건지 알기 어려움, ref추적이 안 되어서 성능 이슈)
- findDOMNode에 대한 경고 출력(클래스 컴포넌트 인스턴스에서 실제 DOM 요소에 대한 참조를 가져올 수 있는 함수)
- 구 context API 사용 시 발생하는 경고(childContextTypes와 getChildContext를 사용하는 구 리액트 Context API를 사용하면 에러 출력함)
- 클래스 컴포넌트의 constructor, render, shoudlCompleteUpdate, getDerivedStateFromProps, 클래스 컴포넌트의 setState의 첫 인수, 함수 컴포넌트 body, useState, useMemo, useReducer는 모두 이중으로 호출된다. 항상 순수한 결과물을 내놓고 있는지 확인하기 위함이다.
- 컴포넌트 최초에 마운트될 때 자동으로 모든 컴포넌트를 해제하고 두 번째 마운트에서 이전 상태를 복원한다.

## Suspense 기능 강화

- 마운트되기 전에 effect가 실행되는 버그 수정.
- 컴포넌트 스스로 suspense에 의해 실제로 보여지고 있는지 숨겨져 있는지 알 수 있음
- 이제 서버에서도 suspense 실행 가능.
- suspense 내에 throttling 추가.

## 인터넷 익스플로러 지원 중단에 따른 추가 폴리필 필요

- promise, symbol, object.assign 세 기능을 기본적으로 사용할 수 있다고 가정하에 배포하기 때문에 만약 이를 지원하지 않는 브라우저에 배포할 예정이면 polyfill을 스스로 준비해야 한다.
