# 4.3 Next.js 톺아보기

## 4.1 Next.js 란?

Next.js는 vercel이라는 미국 스타트업에서 만든 리액트 기반 풀스택 웹 애플리케이션 프레임워크다. SSR를 지원하는 프레임워크 중 가장 사용자도 많고 vercel의 전폭적인 지원을 받고 있기 때문에 SSR를 고려하고 있다면 Next.js를 선택하는 것이 가장 합리적이다.

<aside>
👉

Next.js는 사실 PHP와 react-page의 영향을 받았다. react-page는 현재는 중단된 페이스북에서 만들던 리액트 기반 SSR 프로젝트로 디렉토리 기반 라우팅이 바로 여기에서 영향을 받은 것이다.

</aside>

## 4.2 Next.js 시작하기

### 명령어로 Next.js 프로젝트 설치하기

```powershell
npx create-next-app@latest
```

### 프로젝트 구조 살피기

```
project-root/
├── src/                     # (선택) source folder
│   ├── app/                 # app router
└── public/                  # static assets
```

**Routing Files**

Next.js는 특정 파일 이름을 가지면(`layout`, `page`, `loading`, `not-found`, `error`, `global-error`, `route`, `template`, `default`) 그 파일들이 자동으로 특수한 역할을 수행하는 컴포넌트로 변환되며 다음과 같은 중첩 순서를 가진다. 

또한, `app/` 내부의 위치에 따라 중첩 렌더링된다. (ex. `/app/layout.tsx` → `/app/dashboard/layout.tsx` 가 계층적으로 감싸는 구조)

```html
<Layout>
	<Template>
		<ErrorBoundary fallback={<Error />}>
			<Suspense fallback={<Loading/>}>
				<ErrorBoundary fallback={<NotFound/>}>
					<Page/>
				</ErrorBounary>
			</Suspense>
		</ErrorBoundary>
	</Template>
</Layout>
```

**Routes**

| 종류 | 예시 & 설명 |
| --- | --- |
| folder | 경로 |
| folder/folder | 중첩 경로 |
| [folder] | 동적 경로(/dashboard/1) |
| […folder] | 다중 필수(/doc/a/b/c → slug: [a, b, c]) |
| [[…folder]] | 다중 선택(/doc 또는 /doc/a/b/c → undefined 또는 [a, b, c]) |
| (folder) | 그룹핑 |
| _folder | 완전 무시 |
| @folder | 병렬 라우팅할 때 슬롯 이름 지을 때 사용 |
| (.)folder, (..)folder, (…)folder | 왼쪽부터 같은 레벨, 상위, 루트를 intercept |

### Server Routing vs Client Routing

(정리중)

# 5.1 상태 관리는 왜 필요한가?

## 상태란?

상태? 어떠한 의미를 지닌 값, 변경될 수 있는 값

ex. UI, URL, form, 서버에서 가져온 값

<aside>
❓

tearing: 하나의 상태에 따라 서로 다른 결과물을 사용자에게 보여주는 것

</aside>

## 상태 관리 방법들

리액트는 처음에는 단순히 사용자 인터페이스를 만들기 위한 라이브러리였기 때문에 상태를 관리하는 방법이 역사적으로 계속 변화해왔다.

### Flux 패턴 등장

2014년경 당시 웹개발은 MVC 패턴의 형태를 띄었는데 뷰가 모델을 변경할 수 있고 모델이 뷰를 변경할 수 있는 이른바 양방향 데이터 바인딩은 코드를 간단히 작성할 수 있다는 이점이 있지만 변경 시나리오가 복잡해지면 관리하기 어려워진다. 그래서 이때 페이스북 팀은 단방향 데이터 흐름을 제안하며 Flux 패턴을 선보이기 시작했다.

```html
   Action : 액션 타입과 데이터를 정의해 dispatcher로 보내는 역할을 한다.
-> Dispatcher : Actions를 Store에 보내는 역할을 한다.
-> Store : 상태 데이터를 가지고 있으며, 액션 타입에 따라 상태를 변경하는 역할을 한다.
-> View : 화면을 렌더링하는 역할을 한다. 또한, 뷰에서 사용자의 입력에 따라 Action을 호출하게 된다.
```

```jsx
type StoreState = {
  count: number
}

type Action = {
 type: 'add',
 payload: number
}

function reducer(prev: StoreState, action: Action) {
  const { type } = action;
  if (type === "add") {
    return { count: prev.count + action.payload }
  }
  throw new Error(`Unexpected Action [${ActionType}]`);
}

export default function App() {
  const [state, dispatcher] = useReducer(reducer, {count: 0})
  
  function handleClick() {
    dispatcher({ type: 'add', payload: 1 })
  }
  
  return (
    <div>
      <h1>{state.count}</h1>
      <button onClick={handleClick}>+</button>
    </div>
  )
}
```

### Redux 등장

단방향 흐름은 데이터 흐름을 추적하기 쉽다는 명확한 이점을 보여준 상태에서 동시에 우후죽순처럼 다양한 Flux 패턴을 따르는 라이브러리가 생기기 시작했다. Redux 또한 이 Flux 구조를 구현하기 위해 만들어진 라이브러리이며, Redux는 거기에 더해 특이하게 Elm이라는 아키텍처에 영향을 받았다. Redux는 하나의 상태 객체를 스토어에 저장하고 업데이트가 실행되면 새로운 상태 객체 복사본을 만들어 애플리케이션에 전파한다.

<aside>
❓

**Elm이란?**

Elm은 웹페이지를 선언적으로 작성하기 위한 언어로, Flux와 마찬가지로 단방향 데이터 흐름을 가진다. Elm은 model, update, view라는 3가지 구조를 가진다.

</aside>

Redux는 props drilling 문제를 해결하고 상태에 더 쉽게 접근할 수 있도록 도왔다. ContextAPI가 등장하기 전까지 많이 사용했다. 그러나 초반에 단점은 하나의 상태를 바꾸기 위해 작성해야할 코드가 너무 많다는 것이었다.

### Context API와 useContext

redux가 나온 이후에도 리액트는 상태를 어떻게 적절히 주입해야 하는가 계속해서 고민했다. redux를 사용하면 props drilling 문제를 해결할 수 있지만 단순히 상태를 참조하는 것인데도 많은 코드를 작성해야 한다는 점이 부담으로 다가왔다. 그래서 리액트 팀은 리액트 16.3 버전에서 전역 상태를 하위 컴포넌트로 주입할 수 있는 새로운 Context API를 출시했다. props로 상태를 넘겨주지 않아도 Context API를 사용하면 원하는 곳에서 Context Provider가 주입하는 상태를 사용할 수 있다. 현재는 함수형 컴포넌트에 맞게 사용하지만 첫 등장 당시에는 클래스형 컴포넌트에 맞는 형태로 사용했다.

### 훅 탄생

리액트는 16.8버전부터 함수 컴포넌트에 사용할 수 있는 다양한 훅 API(ex. useState)를 추가했다. 이런 훅의 등장으로 새로운 방식의 상태 관리 라이브러리가 등장했으며, React Query와 SWR이 있다. 두 라이브러리 모두 외부에서 데이터를 불러오는 fetch, 특히 http 요청으로 받아오는 상태를 관리하는데 특화된 라이브러리다.

### Recoil, Zustand, Jotai, Valtio

훅이라는 새로운 패러다임 등장 이후 훅을 활용한 범용적으로 쓸 수 있는 상태 관리 라이브러리들이 많이 등장하기 시작했다. 이 라이브러리들의 공통점은 훅을 활용해 작은 크기의 상태를 효율적으로 관리한다는 것이다. 

# 5.2 리액트 훅으로 시작하는 상태 관리

## 리액트 훅으로 시작하는 상태 관리

### useState, useReducer

useState를 이용해 지역 상태를 관리하는 커스텀 훅을 만들어 여러 컴포넌트에서 사용할 수 있다. 또한 useState는 내부적으로 useReducer로 구현되어 있기 때문에 useReducer 훅을 이용할 수도 있다.

```jsx
function useCounter(initCount: number = 0) {
  const [counter, setCounter] = useState(initCount);
  
  function inc() {
    setCounter(prev => prev + 1);
  }
  
  return {counter, inc}
}
```

그러나 아래 코드처럼 훅을 사용할 경우 훅을 사용할 때마다 컴포넌트별로 초기화되기 때문에 각 컴포넌트에서만 사용 가능한 지역 상태로밖에 사용할 수 없다.

```jsx
function Counter1() {
  const {counter, inc} = useCounter();
  
  return (
	  <>
      <h3>Counter1: {counter}</h3>
      <button onClick={inc}>+</button>
    </>
  )
}

function Counter2() {
  const {counter, inc} = useCounter();
  
  return (
	  <>
      <h3>Counter2: {counter}</h3>
      <button onClick={inc}>+</button>
    </>
  )
}
```

두 컴포넌트가 같은 상태를 사용하고 싶다면 부모에서 선언해 props로 상태와 업데이트 함수를 전달하는 방법도 있지만 사용하기 불편하다.

```jsx
function Parent() {
  const {counter, inc} = useCounter();
  
  return (
    <>
      <Counter1 counter={counter} inc={inc} />
      <Counter2 counter={counter} inc={inc} />
    </>
  )
}
```

### useState 바깥으로 분리하기

```tsx
export type State = { counter: number }

let state: State = {
  counter: 0
}

export function get(): State {
  return state
}

type Initializer<T> = T extends any ? T | ((prev: T) => T) : never // 뭔말이야

export function set<T>(nextState: Initializer<T>) {
  state = typeof nextState == 'function' ? nextState(state) : nextState
}
```

## 상태 관리 라이브러리 Recoil, Jotai, Zustand 살펴보기

Recoil과 Jotai는 Context, Provider그리고 훅을 기반으로 가능한 작은 상태를 효율적으로 관리하는데 초점을 맞추고 있다. Zustand는 리덕스와 비슷하게 하나의 큰 스토어를 기반으로 상태를 관리하며 클로저 기반이다.

- 라이브러리 내부에서 어떻게 상태를 관리하니?
- 상태를 각 컴포넌트로 어떻게 전파해 렌더링을 일으키니?

### Recoil

페이스북에서 만든 리액트 상태 관리 라이브러리. 훅 기반이다.
