# 2.1 JSX란?

## JSX란?

JSX는 보통 리액트를 배우면서 처음 접하게 되기 때문에, React의 전용 문법이라고 착각하기 쉽다. 하지만 사실 JSX는 리액트에 종속되지 않으며 단순히 **구조화된 트리 데이터를 표현하기 위해 만들어진 문법**이다.

JSX는 브라우저가 직접 실행할 수 없기 때문에, Babel과 같은 *트랜스파일러*를 통해 JS 코드로 변환되어야 실행이 가능하다. (`@babel/plugin-transform-react-jsx` 플러그인을 사용한다.) JSX문법을 어떻게 JS 코드로 변환할지는 주석 혹은 Babel 설정 파일에서 pragma를 설정해 할 수 있다.

```jsx
// jsx
const element = <h1>Hello</h1>;

// babel이 변환했을 때(기본값: React.createElement)
const element = React.createElement("h1", null, "Hello");
```

```jsx
// Emotion에서 JSX 문법을 사용할 때, pragma 주석은 Babel에게 JSX를 jsx() 함수로 변환하라고 지시한다. 
// 이때 jsx는 Emotion이 제공하는 함수로, css prop이 동작하도록 돕는다.

/** @jsxImportSource @emotion/react */

function MyComponent() {
  return <div css={{ color: 'hotpink' }}>Hello</div>;
}

// 변환되었을 때
import { jsx as _jsx } from '@emotion/react'; // 자동 import 됨

function MyComponent() {
  return _jsx("div", {
    css: {
      color: "hotpink"
    },
    children: "Hello"
  });
}
```


> 👉 **트랜스파일러**
> 한 언어로 작성된 소스 코드를 다른 언어로 바꾸어 주는 것

> 👉 **Pragma**
> 컴파일러에게 파일 내용을 어떻게 다루어야할지 알려주는 문장
> ```jsx
> 'use strict' // 자바스크립트 코드를 더 엄격하게 검사해줘!
> /** @jsx h // babel아 h 함수를 써서 트랜스파일 해줘!
> /** @jsxImportSource @emotion/react */ // JSX를 emotion 스타일에 맞춰 바꿔줘!
> ```

# 2.2 가상 DOM과 리액트 파이버 / 2.4 렌더링은 어떻게 일어나는가?

## Virtual DOM의 탄생 배경

### Virtual DOM이 없을 때와 있을 때

SPA(Single Page Application)에서는 사용자의 클릭이나 입력 등 인터랙션이 많아 화면이 자주 바뀐다. 그런데 화면이 바뀔 때마다 스타일 계산, 레이아웃 계산, 페인팅, 컴포지팅 같은 렌더링 작업이 다시 수행되기 때문에 비용이 크다. 이런 상황에서 매번 어떤 DOM이 바뀌었는지 직접 추적해서 필요한 부분만 업데이트하도록 코드를 짜는 건 어렵고 실수하기도 쉽다.

이 문제를 해결하기 위해 **React는 Virtual DOM을 도입했다.** Virtual DOM은 실제 DOM을 흉내 낸 가벼운 JavaScript 객체 트리다. 상태나 props가 바뀌면 React는 이 Virtual DOM을 새로 만들고, 이전 Virtual DOM과 비교(diff)해서 어떤 부분이 바뀌었는지를 계산한다. 그리고 바뀐 부분만 실제 DOM에 반영해서 렌더링 성능을 높인다.

### Stack Reconciler의 한계

React 15까지는 Stack Reconciler를 사용해 렌더링을 수행했다. 이 방식은 재귀 호출 기반으로 컴포넌트 트리를 동기적으로 순회하며 렌더링을 수행했기 때문에, 렌더링 중간에 멈출 수 없었다. 이 구조는 트리가 무거워질수록 브라우저가 한 프레임(16.67ms) 내에 화면을 갱신하지 못하게 되어, 렌더링 중 프레임 드랍 현상이 발생하는 문제가 있었다.

## Fiber 개념 등장

React 16부터는 이러한 문제를 해결하기 위해 Fiber라는 새로운 렌더링 아키텍처가 도입되었다. [React Fiber의 설계 문서](https://github.com/acdlite/react-fiber-architecture)에 따르면, 이 아키텍처는 다음 네 가지 목표를 달성하고자 한다:

> - pause work and come back to it later.
> - assign priority to different types of work.
> - reuse previously completed work.
> - abort work if it's no longer needed.

Fiber는 **작업 단위의 조각**으로 볼 수 있다. 각 컴포넌트 또는 DOM 요소마다 하나의 Fiber Node가 생성되며, 이 Node들은 단일 연결 리스트 구조로 연결되어 있다. 이 구조 덕분에 렌더링 작업을 분할하고, 각 작업에 우선순위를 부여해 일시 중지하거나 나중에 다시 이어서 처리할 수 있게 되었다.

```jsx
function FiberNode(
  this: $FlowFixMe,
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // Instance
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;

  // Fiber
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  this.ref = null;
  this.refCleanup = null;

  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;

  // Effects
  this.flags = NoFlags;
  this.subtreeFlags = NoFlags;
  this.deletions = null;

  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  this.alternate = null;
```

## Fiber 트리 구조와 reconciliation

React는 렌더링을 수행할 때 두 개의 Fiber Tree를 사용한다. 

- `current`: 현재 화면에 렌더링된 트리
- `workInProgress`: 다음 렌더링을 위해 준비 중인 트리

이 구조를 더블 버퍼링(double buffering)이라고 부르며, 새로운 상태에 따라 `workInProgress` 트리를 생성하고, 작업이 끝난 후에 이를 `current`로 교체함으로써 렌더링 상태를 안전하게 갱신할 수 있다.

### Reconciliation

렌더링 조건을 만족하면 React는 새로운 Fiber Tree를 생성하면서 이전 Tree와 비교하는 Reconciliation(재조정) 과정을 거친다. 이 과정에서 변경된 부분에만 `flags`라는 변경 정보를 표시하고, 실제 DOM은 아직 조작하지 않는다. 이러한 작업은 렌더링 단계(Render Phase)와 커밋 단계(Commit Phase)의 두 단계로 나뉜다.

Render Phase는 비동기로 수행되며, 새로운 UI 상태에 따라 새로운 Fiber Tree를 만들고 비교(diff)를 수행한다. 이 단계는 실제 DOM에는 영향을 주지 않고, 어떤 DOM 변경이 필요한지만 `flags`에 저장한다. 이 단계에서는 Fiber를 순회하면서 `beginWork`와 `completeWork`라는 두 개의 주요 함수가 사용된다. Render Phase는 작업을 작은 단위로 쪼개고, `requestIdleCallback` 등을 이용해 브라우저가 여유 있을 때 처리하기 때문에 사용자 인터랙션을 블로킹하지 않는다.

이 과정을 통해 새로운 Fiber Tree가 완성되면, React는 두 번째 단계인 Commit Phase로 진입한다. 이 단계는 동기적으로 수행되며, `flags`에 따라 어떤 DOM 조작이 필요한지를 판단하여 실제 DOM에 적용한다. 이 시점에서 비로소 브라우저의 렌더 트리가 변경되며, 화면이 업데이트된다. 또한, 이 단계에서 useEffect와 같은 사이드 이펙트도 처리된다. Commit 단계는 중단이 불가능하기 때문에 모든 DOM 변경과 side-effect는 한 번에 처리되어 화면이 일관된 상태를 유지하게 된다.

## 렌더링이란?

리액트에서 렌더링이란 모든 컴포넌트가 자신의 props와 state의 값을 기반으로 UI를 구성하고 해당 결과를 브라우저에게 제공하는 과정이다. 

### 렌더링은 언제 일어날까?

크게 2가지로 나눌 수 있다. 사용자가 페이지에 최초로 접근할 때 웹사이트를 보여주기 위해 최초 렌더링을 수행한다. 이후에는 다양한 조건에 의해 렌더링이 다시 수행된다.

- 클래스 컴포넌트의 `setState` 호출
- 클래스 컴포넌트의 `forceUpdate` 호출
- 함수 컴포넌트의 `useState`의 setter 호출
- 함수 컴포넌트의 `useReducer`의 dispatch 호출
- 컴포넌트의 key prop 변경
- 컴포넌트의 props 변경
- 부모 컴포넌트의 리렌더링

# 2.3 클래스 컴포넌트와 함수 컴포넌트

## 클래스 컴포넌트 알아야 하는가?

React 16.8 이전에는 대부분 클래스 컴포넌트를 사용했다. 하지만 16.8 버전에서 Hooks가 도입되면서 함수 컴포넌트가 주류로 자리잡았고, 이후로 클래스 컴포넌트의 사용률은 크게 줄어들었다.

또한, 클래스 컴포넌트가 함수 컴포넌트보다 덜 선호되는 이유는 다음과 같다.

### 클래스 컴포넌트의 한계

- `this`를 사용해야 하기 때문에 의도와 다른 값이 렌더링될 수 있음
- `this`와 바인딩 등으로 인해 이해하기 어려움
- 메서드 실행 순서가 직관적이지 않아 데이터 흐름 추적이 어려움
- 로직 재사용이 어렵고, 기능이 늘어날수록 컴포넌트가 비대해짐
- 코드 크기 최적화 어려움(tree shaking이 안됨)
- 핫 리로딩 시 constructor 재호출로 상태가 초기화되는 문제가 있음

## …그렇다면 클래스 컴포넌트를 굳이 공부해야 할까?

그렇다. 왜냐하면 클래스 컴포넌트는 사라질 계획이 없어 보인다. 이미 16버전까지 많은 React 프로젝트가 클래스 컴포넌트 기반으로 개발되었고, 이를 모두 함수형 컴포넌트로 전환하는 것은 현실적으로 쉽지 않기 때문이다.

실무에서 직접 사용할 일이 많지는 않겠지만, 기존 코드베이스를 읽고 이해하려면 클래스 컴포넌트에 대한 이해가 필요하다. 또한, Error Boundary는 현재 클래스 컴포넌트에서만 구현 가능하기 때문에 이 부분에서도 여전히 쓰임새가 있긴 있다.

### 그래서 클래스 컴포넌트에 대해 간략히 정리해 보자면

- 생명주기를 가진다.
    - **마운트**: 컴포넌트가 마운팅되는 시점
    - **업데이트**: 이미 생성된 컴포넌트의 내용이 변경되는 시점
    - **언마운트**: 컴포넌트가 더 이상 존재하지 않는 시점
- 종류를 2개로 나눌 수 있다.
    - `React.Component`
    - `React.PureComponent`: state의 얕은 비교로 리렌더링 진행

# 2.5 컴포넌트와 함수의 무거운 연산을 기억해 두는 메모이제이션

## 메모이제이션은 언제 하는 게 좋을까?

성능 최적화를 위해 한다. 보통 무거운 계산이 자주 반복되거나 불필요한 리렌더링을 방지하고 싶을 때, 혹은 props가 변하지 않으면 동일한 값을 재사용하고 싶을 때 한다. 주로 `useMemo`이나 `useCallback`을 사용한다.

책에서 필자는 메모이제이션을 안 쓰는 것보다는 많이 쓰는 게 실보다 득이 크다고 주장한다. 하지만 처음 리액트를 공부한다면 조심히 적용해보라고 권유하고 있다.

# 참고

- [What is JSX pragma?](https://www.gatsbyjs.com/blog/2019-08-02-what-is-jsx-pragma/)
- [React Deep Dive - Fiber](https://blog.mathpresso.com/react-deep-dive-fiber-88860f6edbd0)
- [React 파이버 아키텍처 분석 | D2](https://d2.naver.com/helloworld/2690975)
- [Fiber 아키텍처의 개념과 Fiber Reconcilation 이해하기](https://m.blog.naver.com/dlaxodud2388/223195103660)
