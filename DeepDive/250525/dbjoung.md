# Next.js 톺아보기

## Next.js란?

Next.js는 Vercel이라는 미국 스타트업에서 만든 풀스택 웹 앱을 구축하기 위한 리액트 기반 프레임워크다.

PHP에 영감을 받아 만들어졌으며, 실제로도 PHP의 대용품으로 사용되기 위해 만들었다고 언급한 것으로 봐서 최초에 설계 당시부터 서버 사이드 렌더링을 염두에 뒀던 것으로 보인다.

## Next.js 시작하기

리액트 애플리케이션을 빠르게 만들 수 있는 create-react-app과 유사하게 Next.js는 create-next-app을 제공해 개발자가 빠르게 Next.js 기반 프로젝트를 생성할 수 있게 돕는다.

### package.json

- next : Next.js의 기반이 되는 패키지
- eslint-config-next : Next.js 기반 프로젝트에서 사용하도록 만들어진 ESLint 설정으로, 구글과 협업해 만든 핵심 웹 지표(core web vital)에 도움이 되는 규칙들이 내장돼 있다.
    - Next.js 기반 프로젝트라면 꼭 사용하는 것을 추천하며, `eslint-config-airbnb` 와 같은 기존에 사용하던 규칙이 있다면 이에 추가로 함께 사용하는 것을 추천한다.

### next.config.js

이 파일은 Next.js 프로젝트의 환경 설정을 담당한다. 

```jsx
/** @type {import('next').NextConfig} */
const nextConfig = {
	reactStrictMode : true,
	swcMinify : true
}

module.exports = nextConfig;
```

- 먼저 첫 줄에 있는 `@type`으로 시작하는 주석은 JS 파일에 타입스크립트의 타입 도움을 받기 위해 추가된 코드다. 해당 주석이 있다면 next의 NextConfig를 기준으로 타입의 도움을 받을 수 있는 반면, 없다면 일일이 타이핑해야 한다.
- reactStrictMode
    - 리액트의 엄격 모드와 관련된 옵션으로, 리액트 애플리케이션 내부에서 잠재적인 문제를 개발자에게 알리기 위한 도구다.
- swcMinify
    - Vercel에서는 SWC라 불리는 또 다른 오픈소스를 만들었는데, 이 도구는 번들링과 컴파일을 더욱 빠르게 수행하기 위해 만들어졌다.
    - 바벨의 대안이라고 볼 수 있다.
        - 바벨보다 빠를 수 있는 이유는 첫째, js 기반의 바벨과는 다르게 러스트(Rust)라는 언어로 작성했다는 점, 그리고 병렬로 작업을 처리한다는 점 등이 있다.
        - swcMinify는 이러한 SWC를 기반으로 코드 최소화 작업을 할 것인지 여부를 설정하는 속성이다.

### pages/_app.tsx

- pages 폴더가 경우에 따라서는 src 하단에 존재할 수도 있다. src에 있거나 혹은 프로젝트 루트에 있어도 동일하게 작동한다.
- _app.tsx, 그리고 내부에 있는 default export로 내보낸 함수는 애플리케이션의 전체 페이지의 시작점이다. 페이지의 시작점이라는 특징 때문에 웹 앱에서 공통으로 설정해야 하는 것들을 여기에서 실행할 수 있다. _app.tsx에서 할 수 있는 내용은 다음과 같다.
    - 에러 바운더리를 사용해 앱 전역에서 발생하는 에러 처리
    - reset.css 같은 전역 css 선언
    - 모든 페이지에 공통으로 사용 또는 제공해야 하는 데이터 제공 등
- 여기에서 SSR 프레임워크의 특징을 확인할 수 있는 사실이 있다.
    - _app.tsx 의 render() 내부에서 console.log() 를 추가해 아무 메시지나 기록해보자. 그리고 페이지를 새로고침하면 해당 로그가 브라우저 콘솔창이 아닌 Next.js를 실행한 터미널에 기록되는 것을 볼 수 있다.
    - 또, 여기에서 페이지를 전환하면 더 이상 서버에서 로깅되지 않고 브라우저에 로깅되는 것을 확인할 수 있다.
    
    ⇒ 이러한 사실로 미뤄 봤을 때 **최초에는 SSR을, 이후에는 클라이언트에서 _app.tsx의 렌더링이 실행된다는 것을 짐작**할 수 있다. 
    

### pages/_document.tsx

`create-next-app`으로 생성했을 때는 존재하지 않는 페이지이다. 필수 페이지가 아니기 때문이다. 

- _app.tsx가 앱 페이지 전체를 초기화하는 곳이라면, _document.tsx는 앱의 HTML을 초기화하는 곳이다. _app.tsx와 다음과 같은 몇 가지 차이가 있다.
    - <html>이나 <body>에 DOM 속성을 추가하고 싶다면, _document.tsx를 사용한다.
    - _app.tsx는 렌더링이나 라우팅에 따라 서버나 클라이언트에서 실행될 수 있지만, _document는 무조건 서버에서 실행된다. 따라서 이 파일에서 onClick과 같은 이벤트 핸들러를 추가할 수 없다. (이 작업은 클라이언트의 hydrate() 함수의 몫이다.)
    - getServerSideProps, getStaticProps 등 서버에서 사용 가능한 데이터 불러오기 함수는 여기에서 사용할 수 없다.
- CSS-in-JS의 스타일을 서버에서 모아 HTML로 제공하는 작업이 가능하다.
- 요약하자면, **_app.tsx는 Next.js를 초기화하는 파일로, Next.js 설정과 관련된 코드를 모아두는 곳이며, 경우에 따라 서버와 클라이언트 모두에서 렌더링**될 수 있다. _**document.tsx는 Next.js로 만드는 웹사이트의 뼈대가 되는 HTML 설정과 관련된 코드를 추가하는 곳이며, 반드시 서버에서만 렌더링**된다.

### pages/_error.tsx

- 마찬가지로 필수 페이지가 아니며, 클라이언틍서 발생하는 에러 또는 서버에서 발생하는 500 에러를 처리할 목적으로 만들어졌다.

### pages/404.tsx

- 404 페이지를 정의할 수 있는 파일이다. 만들지 않으면 Next.js에서 제공하는 기본 404 페이지를 볼 수 있고, 원하는 스타일의 404 페이지를 이곳에 만들 수 있다.

### pages/500.tsx

- 서버에서 발생하는 에러를 핸들링하는 페이지다. _error.tsx와 500.tsx가 모두 있다면 500.tsx가 우선적으로 실행된다. 마찬가지로 500이나 error 페이지가 없다면 기본적으로 Next.js에서 제공하는 페이지를 볼 수 있따.

### pages/index.tsx

앞서 소개한 _app.tsx, _error.tsx, _document.tsx, 500.tsx 등은 Next.js에서 예약어로 관리되는 페이지라면, index.tsx는 주로 웹사이트의 루트이지만 개발자가 자유롭게 명칭을 지정해 만들 수 있는 페이지다. 

- Next.js는 react-pages 처럼 라우팅 구조는 다음과 같이 `/pages` 디렉터리를 기초로 구성되며, 각 페이지에 있는 default export로 내보낸 함수가 해당 페이지의 루트 컴포넌트가 된다.
    - `/pages/index.tsx` : 웹사이트의 루트. [localhost:3000](http://localhost:3000) 과 같은 루트 주소를 의미한다.
    - `/pages/hello.tsx` : `/pages`가 생략되고, 파일명이 주소가 된다. 즉, 여기서는 `/hello` 이며, `localhost:3000/hello`로 접근할 수 있다.
    - `pages/hello/[greeting].tsx` : 여기서 `[]` 의 의미는 여기에 어떠한 문자도 올 수 있다는 뜻이다. 서버 사이드에서 greeting이라는 변수에 사용자가 접속한 주소명이 오게 된다.
    - `/pages/hi/[...props].tsx` : `/hi` 를 제외한 `/hi`의 모든 주소가 여기로 온다.


# 리액트와 상태 관리 라이브러리

## 상태 관리는 왜 필요한가?

먼저, **상태(State)**란 무엇인지 명확하게 정의할 필요가 있다. 흔히 **웹 앱을 개발할 때 이야기하는 상태는 어떠한 의미를 지닌 값이며 앱의 시나리오에 따라 지속적으로 변경될 수 있는 값을 의미**한다. 

웹 앱에서 상태로 분류될 수 있는 것들은 대표적으로 다음과 같은 것이 있다.

- UI : 기본적으로 웹 앱에서 상태라 함은 상호 작용이 가능한 모든 요소의 현재 값을 의미. 다크/라이트 모드, 라디오를 비롯한 각종 input, 알림창의 노출 여부 등 많은 종류의 상태가 존재한다.
- URL : 브라우저에서 관리되고 있는 상태값. `https://www.airbnb.co.kr/rooms/34113796?adults=2` 와 같은 주소가 있다고 가정해 보자. 이 주소에는 `rooomId=34113796` 과 `adults=2`라고 하는 상태가 존재하며 이 상태는 사용자의 라우팅에 따라 변경된다.
- 폼(form) : 폼에도 상태가 존재한다. 로딩 중인지(loading), 현재 제출됐는지(submit), 접근이 불가능한지(disabled), 값이 유효한지(validation) 등 모두가 상태로 관리된다.
- 서버에서 가져온 값 : 클라이언트에서 서버로 요청을 통해 가져온 값도 상태로 볼 수 있다. 대표적으로 API 요청이 있다.

이러한 상태를 어디에 둘 것인가? 전역 변수에 둘 것인가? 별도의 클로저를 만들 것인가? 그렇다면 그 상태가 유효한 범위는 어떻게 제한할 수 있을까? 상태의 변화에 따라 변경돼야 하는 자식 요소들은 어떻게 이 상태의 변화를 감지할 것인가? 이러한 상태 변화가 일어남에 따라 즉각적으로 모든 요소들이 변경되어 앱이 찢어지는 현상(tearing)을 어떻게 방지할 것인가? 등, 이처럼 현대 웹 앱에서 **상태 관리**란 어렵다고 해서 외면할 수 없는 주제가 됐다. 

## 리액트 상태 관리의 역사

프레임워크를 지향하는 Angular와 다르게 리액트는 단순히 사용자 인터페이스를 만들기 위한 라이브러리일 뿐이기 때문에 그 이상의 기능을 제공하지 않는다. 따라서 상태를 관리하는 방법이 개발자에 따라 다르고, 시간에 따라서도 많은 차이가 있다.

### Flux 패턴의 등장

우리가 순수 리액트에서 할 수 있는 전역 상태 관리 수단이라고 한다면 Context API를 떠올릴 것이지만, 리액트가 Context API를 선보인 것은 16.3 버전이었다. 

그러던 중 2014년 경 리액트의 등장과 비슷한 시기에 Flux 패턴을 기반으로 한 라이브러리인 Flux가 소개된다. 이 당시의 웹 개발 환경은 웹 앱이 비대해지고 상태(데이터)도 많아짐에 따라 어디서 어떤 일이 일어나서 해당 상태가 변했는지 등을 추적하고 이해하기가 매우 어려웠다. 

페이스북 팀은 이러한 문제의 원인을 **양방향 데이터 바인딩**으로 봤다. 양방향 데이터 바인딩에서는 뷰(HTML)가 모델(JS)을 변경할 수 있으며, 마찬가지로 모델도 뷰를 변경할 수 있다. 이 코드를 작성하는 입장에서는 간단할 수 있지만 코드의 양이 많아지고 변경 시나리오가 복잡해질수록 관리가 어려워진다.

페이스북 팀은 양방향이 아닌 단방향으로 데이터 흐름을 변경하는 것을 제안하는데, 이것이 바로 Flux 패턴의 시작이다. 

```jsx
// Flux의 기본적인 단방향 데이터 흐름
Action → Dispatcher → Model → View
```

- 액션(action) : 어떠한 작업을 처리할 액션과 그 액션 발생 시 함께 포함시킬 데이터를 의미. 액션 타입과 데이터를 각각 정의해 이를 디스패처로 보낸다.
- 디스패처(dispatcher) : 액션을 스토어에 보내는 역할을 한다. 콜백 함수 형태로 앞서 액션이 정의한 타입과 데이터를 모두 스토어에 보낸다.
- 스토어(store) : 여기에서 실제 상태에 따른 값과 상태를 변경할 수 있는 메서드를 가지고 있다. 액션의 타입에 따라 어떻게 이를 변경할지가 정의돼 있다.
- 뷰(view) : 리액트의 컴포넌트에 해당하는 부분으로, 스토어에서 만들어진 데이터를 가져와 화면을 렌더링하는 역할을 한다. 또한 뷰에서도 사용자의 입력이나 행위에 따라 상태를 업데이트하고자 할 수 있을 것이다.
    
    ```jsx
    // 위 경우에는 아래 처럼 뷰에서 액션을 호출하는 구조로 구성
    // 이처럼 Flux 패턴은 View에서 Action을 호출할 수 있었다.
    							 ↓-----(Action)--↰
    Action → Dispatcher → Store → View
    ```
    

이러한 단방향 데이터 흐름 방식은 사용자의 입력에 따라 데이터를 갱신하고 화면을 어떻게 업데이트해야 하는지도 코드로 작성해야 하므로 코드의 양이 많아지고 개발자도 수고스러워진다. 

하지만 데이터의 흐름은 모두 액션이라는 한 방향(단방향)으로 줄어들므로 데이터의 흐름을 추적하기 쉽고 코드를 이해하기가 한결 수월해진다.

### 시장의 지배자 리덕스의 등장

리액트의 단방향 데이터 흐름이 점점 두각을 드러내던 와중에 등장한 것이 리덕스다. 리덕스 또한 최초에는 이 Flux 구조를 구현하기 위해 만들어진 라이브러리 중 하나였다. 이에 한 가지 더 특별한 것은 여기에 Elm 아키텍처를 도입했다는 것이다.

Elm은 웹페이지를 선언적으로 작성하기 위한 언어다. 다음 예제는 Elm으로 HTML을 작성한 예제다.

```jsx
Module Main exposing (..)

import Browser
import Html exposing (Html, button, div, text)
import Html.Events exposing (onClick)

-- MAIN
main = 
	Browser.sandbox { init = init, update = update, view = view }
	
-- MODEL
type alias Model = Int

init : Model
init = 0

-- UPDATE
type Msg = Increment | Decrement

update : Msg -> Model -> Model
update : msg model = 
	case msg of 
		Increment ->
			model + 1
			
		Decrement ->
			model - 1
			
-- VIEW

view : Model -> Html Msg
view model = 
	div [] 
		[ button [ onClick Decrement ] [ text "-" ] 
		, div [] [ text (String.fromInt model) ]
		, button [ onClick Increment ] [ text "+" ]
		]
<div>
	<button>-</button>
	<div>2</div>
	<button>+</button>
</div>
```

- model, update, view가 Elm 아키텍처의 핵심이다.
    - 모델(Model) : 애플리케이션의 상태를 의미한다. 여기서는 Model을 의미하며, 초깃값으로는 0이 주어졌다.
    - 뷰(View) : 모델을 표현하는 HTML을 말한다. 여기서는 Model을 인수로 받아서 HTML을 표현한다.
    - 업데이트(Update) : 모델을 수정하는 방식을 말한다. Increment, Decrement를 선언해 각각의 방식이 어떻게 모델을 수정하는지 나타냈다.

즉, Elm은 Flux와 마찬가지로 데이터 흐름을 세 가지로 분류하고, 이를 단방향으로 강제해 웹 애플리케이션의 상태를 안정적으로 관리하고자 노력했다. 그리고 리덕스는 이 Elm 아키텍처의 영향을 받아 작성됐다. 

리덕스는 하나의 상태 객체를 스토어에 저장해 두고, 이 객체를 업데이트하는 작업을 디스패치해 업데이트를 수행한다. 이러한 작업은 reducer 함수로 발생시킬 수 있다. 이 함수의 실행은 웹 앱 상태에 대한 완전히 새로운 복사본을 반환한 다음, 앱에 이 새롭게 만들어진 상태를 전파하게 된다.

리덕스는 하나의 글로벌 상태 객체를 통해 이 상태 객체를 하위 컴포넌트에 전파할 수 있어 props 드릴링을 해결할 수 있었고, 스토어가 필요한 컴포넌트라면 connect를 써서 스토어에 바로 접근 가능했다.

Context API가 등장하기 전까지, 리덕스는 리액트 상태 관리에서 빼놓고 이야기할 수 없는 중요한 축으로 자리잡았다. 

### 리덕스의 단점

리덕스가 마냥 편하기만 한 것은 아니었다. 

리덕스는 단순히 하나의 상태를 바꾸고 싶어도 해야 할 일이 너무 많았다. 

어떤 액션인지 타입을 선언해야 하고, 이 액션을 수행할 creator 함수를 만들어야 한다. dispatcher와 selector도 필요하다. 즉, 하고자 하는 일에 비해 보일러플레이트가 너무 많다는 비판의 목소리다.

→ 하지만 이는 리덕스가 처음 등장했을 때의 비판으로, 지금은 이러한 작업이 많이 간소화됐다.

## Context API와 useContext

리액트의 **props 드릴링**을 해결하기 위한 방법은 계속 고민되어왔다. 리덕스가 있긴 했지만 단순히 상태를 참조하고 싶을 뿐인데 준비해야 하는 보일러플레이트가 부담스러웠다. 

리액트 팀은 리액트 16.3에서 전역 상태를 하위 컴포넌트에 주입할 수 있는 새로운 Context API를 출시했다. 

props로 상태를 넘겨주지 않아더라도 Context API를 사용하면 원하는 곳에서 Context Provider가 주입하는 상태를 사용할 수 있게 된 것이다.

### Context API의 문제점

하지만 다음과 같은 문제가 있었다.

1. 상위 컴포넌트가 렌더링되면 `getChildContext`도 호출됨과 동시에 `shouldComponentUpdate`가 항상 true를 반환해 불필요하게 렌더링이 일어남
2. `getChildContext`를 사용하기 위해서는 context를 인수로 받아야 하는데 이 때문에 컴포넌트와 결합도가 높아지는 등의 단점

위 두 가지를 해결하기 위해 16.3 버전에서는 새로운 context가 출시되었다. 

### 훅의 탄생, 그리고 React Query와 SWR

리액트는 16.8 버전에서 함수 컴포넌트에 사용할 수 있는 다양한 훅 API를 추가했다. 이 훅 API는 기존에 무상태 컴포넌트를 선언하기 위해서만 제한적으로 사용됐던 함수 컴포넌트가 클래스 컴포넌트 이상의 인기를 구가할 수 있도록 많은 기능을 제공했다.

가장 큰 변경점 중 하나는 **state를 매우 손쉽게 재사용 가능하도록 만들 수 있다는 것**이었다. 

이러한 훅과 state의 등장으로 **이전에는 볼 수 없던 방식의 상태 관리가 등장하는데, 바로 React Query와 SWR** 이다.

두 라이브러리는 모두 **외부에서 데이터를 불러오는 fetch를 관리하는 데 특화된 라이브러리**지만, **API 호출에 대한 상태를 관리하고 있기 때문에 HTTP 요청에 특화된 상태 관리 라이브러리**라 볼 수 있다.

```jsx
import React from 'react'
import useSWR from 'swr'

const fetcher = (url) => fetch(url).then((res)=>res.json());

export default function App() {
	const { data, error } = useSWR(
		'https://api.github.com/repos/vercel/swr',
		fetcher,
	);
	
	if (error) return 'An error has occurred.'
	if (!data) return 'Loading...'
	
	return (
		<p>{JSON.stringify(data)}</p>
	)
}
```

- `useSWR`의 첫 번째 인수로 조회할 API 주소를, 두 번째 인수로 조회에 사용되는 fetch를 넘겨준다. 첫 번째 인수인 API 주소는 키로도 사용되며, 이후에 다른 곳에서 동일한 키로 호출하면 재조회하는 것이 아니라 useSWR이 관리하고 있는 캐시의 값을 활용한다.
- 기존에 우리가 알고 있는 상태 관리 라이브러리보다는 제한적인 목적으로, 일반적인 형태와는 다르다는 점만 제외하면 분명히 SWR이나 React Query도 상태 관리 라이브러리의 일종이라 볼 수 있다.

### Recoil, Zustand, Jotai, Valtio에 이르기까지

훅이라는 새로운 패러다임의 등장에 따라 훅을 활용해 상태를 가져오거나 관리할 수 있는 다양한 라이브러리가 등장하게 된다. 

페이스북 팀에서 만든 Recoil을 필두로, Jotai, Zustand, Valtio 등 다양한 라이브러리가 선보이게 된다.

```jsx
// Recoil
const counter = atom({ key : 'count', default : 0 });
const todoList = useRecoilValue(counter);

// Jotai
const countAtom = atom(0);
const [ count, setCount ] = useAtom(countAtom);

// Zustand
const useCounterStore = create((set)=>{
	count : 0,
	increase : () => set((state)=>({ count : state.count + 1 })),
});
const count = useCounterStore((state) => state.count);

// Valtio
const state = proxy({ count : 0 });
const snap = useSnapshot(state);
state.count++;
```

새롭게 떠오르고 있는 많은 상태 관리 라이브러리는 기존의 리덕스 같은 라이브러리와는 차이점이 있는데, 바로 훅을 활용해 작은 크기의 상태를 효율적으로 관리한다는 것이다. 

Recoil Jotai, Zustand, Valtio의 저장소를 방문해보면 모두 peerDependencies로 리액트 16.8 버전 이상을 요구하고 있음을 확인할 수 있다.

이는 기존 상태 관리 라이브러리의 아쉬운 점으로 지적받던 전역 상태 관리 패러다임에서 벗어나 개발자가 원하는 만큼의 상태를 지역적으로 관리하는 것을 가능하게 만들었고, 훅을 지원함으로써 함수 컴포넌트에서 손쉽게 사용할 수 있다는 장점 또한 가지고 있다.

# 리액트 훅으로 시작하는 상태 관리

비교적 오랜 기간 리액트 생태계에서는 리액트 애플리케이션의 상태 관리를 위해 리덕스에 의존했다. 과거 리액트 코드를 보면 리액트와 리덕스가 함께 설치돼 있는 것을 흔히 볼 수 있었고, 일부 개발자들은 리액트와 리덕스를 마치 하나의 프레임워크 내지는 업계 표준으로 여기기도 했다.

그러나 현재는 새로운 Context API, useReducer, useState의 등장으로 컴포넌트에 걸쳐서 재사용하거나 혹은 컴포넌트 내부에 걸쳐서 상태를 관리할 수 있는 방법들이 점차 등장하기 시작했고, 덕분에 리덕스 외의 다른 상태 관리 라이브러리를 선택하는 경우도 많아지고 있다. 

그러한 새로운 방법을 채택한 라이브러리는 무엇이 있고, 어떻게 작동하는 지 알아보자.

## 가장 기본적인 방법 : useState와 useReducer

useState의 등장으로 리액트에서는 여러 컴포넌트에 걸쳐 손쉽게 동일한 인터페이스의 상태를 생성하고 관리할 수 있게 됐다. 리액트의 훅을 기반으로 만든 사용자 정의 훅은 함수 컴포넌트라면 **어디서든 손쉽게 재사용 가능하다는 장점**이 있다. 

useState와 비슷한 훅인 useReducer 또한 마찬가지로 지역 상태를 관리할 수 있는 훅이다. 하지만 이 두 가지 훅이 상태 관리의 모든 필요성과 문제를 해결해주지 않는다. 

- 훅을 사용할 때마다 컴포넌트 별로 초기화되므로 컴포넌트에 따라 서로 다른 상태를 가질 수밖에 없다. 아래처럼 useCount를 호출한 컴포넌트가 다수 존재할 경우, 각각의 컴포넌트는 모두 다른 count 상태를 가진다.
    - 이렇게 기본적인 useState를 기반으로 한 상태를 **지역 상태(local state)**라고 하며, 이 지역 상태는 해당 컴포넌트 내에서만 유효하다는 한계가 있다.
- 만약 모든 컴포넌트가 동일한 count 상태를 갖게 하려면 어떻게 해야 할까? 일단 count 상태를 컴포넌트 밖으로 한 단계 끌어올려 **전역 상태(global state)**로 만드는 방법이 있다.
- 하지만 이렇게 하면 상태를 props 형태로 자식에게 전달해야한다.

## 지역 상태의 한계를 벗어나보자 : useState의 상태를 바깥으로 분리하기

### 첫 번째 시도 - 상태를 외부로 뺀다.

**useState의 한계는 명확하다.** 위에서 언급했듯이 useState는 리액트가 만든 클로저 내부에서 관리되어 지역 상태로 생성되기 때문에 해당 컴포넌트에서만 사용할 수 있다는 단점이 있다. 

만약 useState가 이 리액트 클로저가 아닌 다른 JS 실행 문맥 어디에선가, 즉 완전히 다른 곳에서 초기화돼서 관리되면 어떨까?

그리고 그 상태를 참조하는 유효한 스코프 내부에서는 해당 객체의 값을 공유해서 사용할 수도 있지 않을까? 

- 결론부터 이야기하자면 이러한 방식은 리액트 환경에서 작동하지 않는다.
    - 예제 코드
        
        ```jsx
        // counter.js
        export type State = { counter : number }
        
        // 상태를 아예 컴포넌트 밖에 선언했다. 각 컴포넌트가 이 상태를 바라보게 할 것이다.
        let state : State = {
        	counter : 0
        }
        
        // getter
        export function get(): State {
        	return state
        }
        
        // useState와 동일하게 구현하기 위해 게으른 초기화 함수나 값을 받을 수 있게 한다.
        type Initializer<T> = T extends any ? T | ((prev: T)=> T) : never
        
        // setter 
        export function set<T>(nextState : Initializer<T>) {
        	state = typeof nextState === 'function' ? nextState(state) : nextState
        }
        
        // Counter
        function Counter() {
        	const state = get();
        	
        	function handleClick() {
        		set((prev: State)=>({ counter : prev.counter + 1 }));
        	}
        	
        	return (
        		<>
        			<h3>{state.counter}</h3>
        			<button onClick={handleClick}>+</button>
        		</>
        	)
        }
        ```
        
    - 작동하지 않는 이유는, 컴포넌트가 리렌더링되지 않기 때문이다. setter, getter, state 업데이트 등 모든 로직이 원하는대로 작동하지만, 리액트의 랜더링 방식으로 인해 값이 변경되어도 리렌더링이 발생하지 않는다.

### 두 번째 시도 - useState, useReducer 활용

리렌더링을 발생시키기 위해, useState 인수로 컴포넌트 밖에서 선언한 state를 넘겨주는 방식으로 구현해본다.

```jsx
function Counter1() {
	const [ count, setCount ] = useState(state);
	
	function handleClick() {
		// 외부에서 선언한 set 함수 내부에서 다음 상태값을 연산한 다음,
		// 그 값을 로컬 상태값에도 넣었다.
		set((prev: State)=>{
			const newState = { counter : prev.counter + 1 };
			// setCount가 호출되면서 컴포넌트 리렌더링을 야기한다.
			setCount(newState);
			return newState;
		});
	}
	
	return (
		<>
			<h3>{ count.counter }</h3>
			<button onClick={handleClick}>+</button>
		</>
	);
}

function Counter2() {
	const [count, setCount] = useState(state);
	
	// 위 컴포넌트와 동일한 작동을 추가했다.
	function handleClick() {
		set((prev: State)=>{
			const newState = { counter : prev.counter + 1 }
			setCount(newState);
			return newState;
		});
	}
	
	return (
		<>
			<h3>{count.counter}</h3>
			<button onClick={handleClick}>+</button>
		</>
	);
}
```

- 예제 코드에서는 억지로 전역에 있는 상태를 참조하게 만들었다. useState의 초깃값으로 컴포넌트 외부에 있는 값을 사용하는 위와 같은 방식은 일반적인 리액트 코드 작성 방식과 동일하다.
- 하지만 해당 방법은 아래의 두 가지 문제점이 있다.
    - 외부에 이미 상태가 있음에도 컴포넌트 내에서 동일한 state를 관리하고 있다는 비효율이 존재한다.
    - 작동 문제가 있다. Counter1의 버튼을 클릭해도 Counter2는 업데이트가 되지 않는다.
        
        → Counter1의 이벤트를 발생시켜도 Counter2의 상태는 바뀌지 않기 때문이다.
        

### 정리

위 첫번째 시도, 두 번째 시도를 통해, **함수 외부에서 상태를 참조하고 이를 통해 렌더링까지 자연스럽게 일어나려면**  다음과 같은 조건을 만족해야한다는 결론에 도달할 수 있다.

- 꼭 window나 global에 있어야 할 필요는 없지만 **컴포넌트 외부 어딘가에 상태를 두고** 여러 컴포넌트가 같이 쓸 수 있어야 한다.
- 외부에 있는 상태를 사용하는 컴포넌트는 **상태의 변화를 알아챌 수 있어야 하고 ,상태가 변화될 때마다 리렌더링이 일어나서 컴포넌트를 최신 상태값 기준으로 렌더링**해야 한다.
    - 이 상태 감지는 상태를 변경시키는 컴포넌트뿐만 아니라 이 상태를 참조하는 모든 컴포넌트에서 동일하게 작동해야 한다.
- 상태가 객체(not 원시값)인 경우에 그 객체에 내가 **감지하지 않는 값이 변할 경우에는 리렌더링이 발생해서는 안된다**.
    - 예를 들어, { a : 1, b : 2 } 라는 상태가 있으며 어느 컴포넌트에서 a를 2로 업데이트했다고 가정해 보자.
    - 이러한 객체 값의 변화가 단순히 b의 값을 참조하는 컴포넌트에서는 리렌더링을 일으켜서는 안 된다는 뜻이다.

### 세 번째 시도

위에서 정리한 조건을 만족하는 **컴포넌트 레벨의 지역 상태를 벗어난 새로운 상태 관리 코드**를 만들어보자.

- 먼저 해당 상태는 객체일 수도, 원시값일 수도 있으므로 범용적 이름인 `store`로 정의한다.
- 그리고 2번 조건을 만족하기 위해 store의 값이 변경될 때마다 변경됐음을 알리는 callback 함수를 실행해야 하고, 이 callback을 등록할 수 있는 subscribe 함수가 필요하다.

```jsx
type Initializer<T> = T extends any ? T | ((prev: T)=> T) : never;

type Store<State> = {
	get : () => State
	set : (action : Initializer<State>) => State
	subscribe : (callback : () => void) => () => void
}
```

- get은 항상 최신값을 가져와야 하므로 함수로 구현.
- set은 기존에 리액트 개발자가 널리 사용하고 있는 useState와 동일하게 값 또는 함수를 받을 수 있도록 구현.
- subscribe는 이 store의 변경을 감지하고 싶은 컴포넌트들이 자신의 callback 함수를 등록해 두는 곳

```jsx
export const createStore = <State extends unknown>(
	initialState : Initializer<State>,
) : Store<State> => {
	// useState와 마찬가지로 초깃값을 게으른 초기화를 위한 함수 또한 
	// 그냥 값을 받을 수 있도록 함
	// state의 값은 스토어 내부에서 보관해야 하므로 변수로 선언
	let state = typeof initialState !== 'function' ? initialState : initialState();
	
	// callbacks는 자료형에 관계없이 유일한 값을 저장할 수 있는 Set을 사용한다.
	const callbacks = new Set<()=>void>();
	// 언제든 get이 호출되면 최신값을 가져올 수 있도록 함수로 만든다.
	const get = () => state;
	const set = (nextState : State | ((prev : State) => State)) => {
		// 인수가 함수라면 함수를 실행해 새로운 값을 받고, 
		// 아니라면 새로운 값을 그대로 사용
		state = typeof nextState === 'function' ? 
			(nextState as (prev : State) => State)(state)
			: nextState;
			
		// 값의 설정이 발생하면 콜백 목록을 순회하면서 모든 콜백을 실행
		// (state를 구독한 모든 컴포넌트 렌더링 유도)
		callbacks.forEach((callback)=>callback());
		
		return state;
	}
	
	// subscribe는 콜백을 인수로 받는다.
	const subscribe = (callback : () => void) => {
		// 받은 함수를 콜백 목록에 추가한다.
		callbacks.add(callback);
		
		// 클린업 실행 시 이를 삭제해서 반복적으로 추가되는 것을 막는다.
		return () => {
			callbacks.delete(callback);
		}
	}
	
	return { get, set, subscribe };
}
```

- 요약하자면 createStore는 자신이 관리해야 하는 상태를 내부 변수로 가진 다음, get 함수로 해당 변수의 최신값을 제공하며, set 함수로 내부 변수를 최신화하고, 이 과정에서 등록된 콜백을 모조리 실행하는 구조를 띠고 있다.
- 위 코드에 더해 마지막으로 createStore로 만들어진 store 값을 참조하고, 이 값의 변화에 따라 컴포넌트 렌더링을 유도할 사용자 정의 훅이 필요하다.
    - useStore라는 훅을 만들어 이 store의 변화를 감지할 수 있게 코드를 작성해보자.
    
    ```jsx
    export const useStore = <State extends unknown>(store : Store<State>) => {
    	const [state, setState] = useState<State>(() => store.get());
    	
    	useEffect(()=>{
    		const unsubscribe = store.subscribe(() => {
    			setState(store.get());
    		});
    		
    		return unsubscribe;
    	}, [store]);
    	
    	return [state, store.set] as const;
    }
    ```
    

→ 위 코드는 컴포넌트 1에서 상태가 바뀌었을 때 같은 상태를 가진 컴포넌트2까지 함께 리렌더링 되어야한다는 조건을 잘 만족한다. 하지만 여전히 아래의 문제가 있다. 

- 스토어의 구조가 원시값이라면 상관 없지만 객체인 경우에는, 그 객체의 속성값이 변할 때마다 다른 속성값을 구독한 컴포넌트까지 리렌더링 된다.

이번에는 이를 해결해보자. 

### 마지막 시도 - `정리`의 조건 만족하도록 `useStoreSelector` 작성

```jsx
export const useStoreSelector = <State extends unknown, Value extends unknown>(
	store : Store<State>,
	selector : (state : State) => Value,
) => {
	const [state, setState] = useState(()=>selector(store.get());
	
	useEffect(()=>{
		const unsubscribe = store.subscribe(()=>{
			const value = selector(store.get());
			setState(value);
		});
		
		return unsubscribe;
	});
	
	return state;
}
```

- 위 `useStoreSelector`가 이전에 만들었던 `useStore`와 다른 점은 두 번째 인수로 `selector` 라는 함수를 받는 것이다.
- 이 함수는 store의 상태에서 어떤 값을 가져올 지 정의하는 함수다. 이 함수를 사용해 `store.get()`을 수행한다. useState는 값이 변경되지 않으면 리렌더링을 수행하지 않으므로 store의 값이 변경됐다 하더라도 `selector(store.get())` 이 변경되지 않는다면 리렌더링이 일어나지 않는다.
- 사용 예제는 다음과 같다.

```jsx
const store = createStore({ count : 0, text : 'hi' });

function Counter() {
	const counter = useStoreSelector(
		store,
		useCallback((state) => state.count, [])
		// useCallback으로 고정시키지 않으면 리렌더링될 때마다 함수가 새로 생성됨.
	)
	
	function handleClick() {
		store.set((prev)=>({ ...prev, count : prev.count + 1 }));
	}
	
	useEffect(()=>{
		console.log('Counter Renderer');
	});
	
	return (
		<>
			<h3>{counter}</h3>
			<button onClick={handleClick}>+</button>
		</>
	)
}

const textSelector = (state : ReturnType<typeof store.get>) => state.text;

function TextEditor() {
	const text = useStoreSelector(store, textSelector);
	
	useEffect(()=>{
		console.log('TextEditor Renderer');
	});
	
	function handleChange() {
		store.set((prev)=>({ ...prev, text : e.target.value }));
	}
	
	return (
		<>
			<h3>{text}</h3>
			<input value={text} onChange={handleChange} />
		</>
	);
}
```

- GPT의 정리
    1. `store.get()` → `{ count: 5, text: "hi" }`
    2. `selector`로 `count`만 추출 → `5`
    3. `useState(5)`로 이 값을 React 내부 상태로 보존
    
    → 이후 `store.set({ count: 6, text: "hi" })`이 발생하면:
    
    - `store.subscribe()`에 등록된 콜백 실행
    - `selector(store.get())` → `6` 반환
    - `useState(5)` → `setState(6)` 실행 → React 리렌더링
    
    즉, **store는 단일 소스로 외부에서 관리 되**고,
    
    **React는 그걸 selector를 활용해 컴포넌트 단위로 따로 복사해 추적하면서 렌더링을 조율**하는 구조야.
    

### 페이스북의 `useSubscription`

위에서 알아본 상태 관리 방법으로 구현된 훅이 이미 존재한다. 페이스북 팀에서 만든 useSubscription 훅이다. 

```jsx
function NewCounter() {
	const subscription = useMemo(
		()=> ({
			// 스토어의 모든 값으로 설정해 뒀지만 selector 예제와 마찬가지로 
			// 특정한 값에서만 가져오는 것도 가능하다.
			getCurrentValue : () => store.get(),
			subscribe : (callback: () => void) => {
				const unsubscribe = store.subscribe(callback);
				return () => unsubscribe();
			}
		}),
		[]
	)
	
	const value = useSubscription(subscription)
	
	return <>{JSON.stringify(value)}</>
}
```

- 위 예제처럼 `useSubscripttion`을 사용하면 앞선 예제에서 `useStoreSelector` 를 사용했던 것 처럼 외부에 있는 store를 가져와서 사용하고 리렌더링까지 정상적으로 수행할 수 있다.
    - useSubscription은 외부 store를 리액트와 연결해서, state 변경 시 변화를 자동으로 감지하여 구독 중인 모든 컴포넌트를 리렌더링 할 수 있도록 연결해주는 훅이다.

## useState와 useContext 동시에 사용해보기

앞서 살펴본 useStore과 useStoreSelector 훅을 활용한 방식은 **반드시 하나의 스토어만 가지게 된다는** 단점이 있다. 

하나의 스토어를 가지면 이 스토어는 마치 전역 변수처럼 작동하게 되어 동일한 형태의 여러 개의 스토어를 가질 수 없게 된다. 

만약 **훅을 사용하는 서로 다른 스코프에서 스토어의 구조는 동일하되, 여러 개의 서로 다른 데이터를 공유해 사용하고 싶다면** 어떻게 해야 할까?

### 첫 번째 방법 : createStore를 이용해 동일한 타입으로 스토어 여러개 생성

단순히 createStore를 여러번 호출하는 방법이다. 하지만 상당히 번거로운 방법인데, 

- 해당 스토어가 필요할 때마다 반복적으로 스토어를 생성해야 하고, 훅은 스토어와 의존적인 1:1 관계를 맺고 있으므로 스토어를 만들 때마다 해당 스토어에 의존적인 useStore와 같은 훅을 동일한 개수로 생성해야 한다. (useStore 안에 외부 상태의 복사본이 들어가 있으므로…)
- 또한, 1:1로 훅을 하나씩 만들었다고 하더라도 이 훅이 어느 스토어에서 사용 가능한지를 가늠하려면 오직 훅의 이름이나 스토어의 이름에 의지해야 한다는 어려움이 있다.

### 두 번째 방법 : Context API 활용

Context를 활용해 해당 스토어를 위한 하위 컴포넌트에 주입한다면 컴포넌트에서는 자신이 주입된 스토어에 대해서만 접근할 수 있게 될 것이다. 스토어와 Context를 함께 사용하는 예제는 아래와 같다.

```jsx
// Context를 생성하면 자동으로 스토어도 함께 생성
export const CounterStoreContext = createContext<Store<CounterStore>>(
	createStore<CounterStore>({ count : 0, text : 'hello' })
);

export const CounterStoreProvider = ({
	initialState, 
	children
} : PropsWithChildren<
	initialState : CounterStore
>) => {
	const storeRef = useRef<Store<CounterStore>>();
	
	// 스토어를 생성한 적이 없다면 최초에 한 번 생성
	if (!storeRef.current) {
		storeRef.current = createStore(initialState);
	}
	
	return (
		<CounterStoreContext.Provider value={storeRef.current}>
			{children}
		</CounterStoreContext.Provider>
	)
};
```

- storeRef를 사용해서 스토어를 제공하는 이유는, Provider로 넘기는 props가 불필요하게 변경돼서 리렌더링되는 것을 막기 위해서다.
    - useRef를 사용했기 때문에 CounterStoreProvider는 오직 최초 렌더링에서만 스토어를 만들어서 값을 내려주게 될 것이다.

이제 이 Context에서 내려주는 값을 사용하기 위해 useStoreSelector와는 다른 접근이 필요하다. 해당 훅은 스토어에 직접 접근하고 있었지만, 이번 예제에서는 **Context를 통해 store에 접근**해야하기 때문이다.

```jsx
export const useCounterContextSelector = <State extends unknown>(
	selector: (state: CounterStore) => State,
) => {
	const store = useContext(CounterStoreContext);
	//useStoreSelector를 사용해도 동일하다.
	const subscription = useSubscription(
		useMemo(
			() => ({
				getCurrentValue : () => selector(store.get()),
				subscribe : store.subscribe
			}), 
			[store, selector]
		)
	);
	
	return [ subscription, store.set ] as const
}
```

이렇게 Context와 Provider 기반으로 각 store를 격리해서 관리하면 장점이 많다.

- 스토어를 사용하는 컴포넌트는 해당 상태가 어느 스토어에서 온 상태인지 신경 쓰지 않아도 된다. 단지 해당 스토어를 기반으로 어떤 값을 보여줄지만 고민하면 되므로 좀 더 편리하게 코드를 작성할 수 있다.

### 정리

현재 리액트 생태계에는 많은 상태 관리 라이브러리가 있지만 이것들이 **작동하는 방식은 결국 다음과 같이 요약**할 수 있다.

- useState, useReducer가 가지고 있는 한계, 컴포넌트 내부에서만 사용할 수 있는 지역 상태라는 점을 극복하기 위해 외부 어딘가에 상태를 둔다.
    - 이는 컴포넌트 최상단 내지는 상태가 필요한 부모가 될 수도 있고, 혹은 격리된 JS 스코프 어딘가일 수도 있다.
- 이 외부 상태 변경을 각자의 방식으로 감지해 컴포넌트의 렌더링을 일으킨다.

## 상태 관리 라이브러리 Recoil, Jotai, Zustand 살펴보기

- Recoil과 Jotai는 Context와 Provider, 그리고 훅을 기반으로 가능한 작은 상태를 효율적으로 관리하는 데 초점을 맞추고 있다.
- Zustand는 리덕스와 비슷하게 하나의 큰 스토어를 기반으로 상태를 관리하는 라이브러리다.
    - Recoil이나 Jotai와는 다르게 **이 하나의 큰 스토어**는 **Context가 아니라 스토어가 가지는 클로저를 기반으로 생성**되며, **이 스토어의 상태가 변경되면 이 상태를 구독하고 있는 컴포넌트에 전파해 리렌더링을 알리는 방식**이다.

### 페이스북 상태 관리 라이브러리 Recoil

- Recoil은 리액트를 만든 페이스북에서 만든 리액트를 위한 상태 관리 라이브러리다.
- 리액트에서 훅의 개념으로 상태 관리를 시작한 최초의 라이브러리 중 하나이며, 최소 상태 개념인 `Atom` 을 처음 리액트 생태계에서 선보이기도 했다.
- Recoil 팀에서는 리액트 18에서 제공되는 동시성 렌더링, 서버 컴포넌트, Streaming SSR 등이 지원되지 전까지는 1.0.0을 릴리스하지 않을 것이라 밝힌 적 있다.
    - **현재 시점 기준 개발 종료….**
- RecoilRoot, atom, useReciolState, useRecoilValue 등의 도구가 있다.

```jsx
export default function App() {
	return <RecoilRoot></RecoilRoot>
}

function RecoilRoot(props: Props) : React.Node {
	const { override, ...propsExceptOverride } = props
	
	const ancestorStoreRef = useStoreRef();
	if (override === false && ancestorStoreRef.current) !== defaultStore) {
		// If ancestorStoreRef.current !== defaultStore, it means that this 
		// RecoilRoot is not nested within another.
		return props.children;
	}
	
	return <RecoilRoot_INTERNAL {...propsExceptOverride} />
}
```

- 여기서 주목할 것은 `useStoreRef`다. useStoreRef로 ancestorStoreRef의 존재를 확인하는데, 이는 Recoil에서 생성되는 atom과 같은 상태값을 저장하는 스토어를 의미한다.
- 스토어 기본값인 defaultStore는 스토어의 아이디 값을 가져올 수 있는 `getNextStoreID()` 와 스토어 값을 가져오는 함수인 `getState`, 값을 수정하는 함수인 `replaceState` 등으로 이뤄져 있다.
    - 해당 스토어 아이디를 제외하고는 모두 에러로 처리돼 있는데, 이를 통해 **`<RecoilRoot>` 로 감싸지 않은 컴포넌트에서는 스토어에 접근할 수 없다는 것**을 알 수 있다.
- 또한, 상태가 변할 때 이 변경된 상태를 하위 컴포넌트로 전파해 컴포넌트에 리렌더링을 일으키는 `notifyComponents`가 있는 것도 확인할 수 있다.
    - notifyComponent는 store, 그리고 상태를 전파할 storeState를 인수로 받아 이 스토어를 사용하고 있는 하위 의존성을 모두 검색한 다음, 여기에 있는 컴포넌트들을 모두 확인해 콜백을 실행한다.
    - 값이 변경됐을 때 콜백을 실행해 상태 변화를 알리는 부분은 앞서 구현해 본 바닐라 스토어와 크게 다르지 않다.
- RecoilRoot의 구조를 대략 파악하여 정리하면 다음과 같다.
    - Recoil의 상태값은 RecoilRoot로 생성된 Context의 스토어에 저장된다.
    - 스토어의 상태값이 접근할 수 있는 함수들이 있으며, 이 함수를 활용해 상태값에 접근하거나 상태값을 변경할 수 있다.
    - 값의 변경이 발생하면 이를 참조하고 있는 하위 컴포넌트에 모두 알린다.

#### atom

Recoil의 핵심 개념인 atom은 상태를 나타내는 Recoil의 최소 단위다. 

```jsx
type Statement = {
	name : string
	amount : number
}

const InitialStatements: Array<Statement> = [
	{ name : '과자', amount : -500 },
	{ name : '용돈', amount : 10000 },
	{ name : '네이버페이충전', amount : -5000 },
]

// Atom 선언
const statementsAtom = atom<Array<Statement>>({
	key : 'statements',
	default : InitialStatements,
});
```

- atom은 key 값을 필수로 가지며, 이 키는 다른 atom과 구별하는 식별자가 되는 필수 값이다. (유일한 값이어야 함)
- default는 atom의 초깃값을 의미
- atom의 값을 컴포넌트에서 읽어오고 이 값의 변화에 따라 컴포넌트를 리렌더링하려면 다음 두 가지 훅을 사용하면 된다.
    - useRecoilValue : atom의 값을 읽어오는 훅이다. `외부의 값을 구독해 렌더링을 강제로 일으킨다`라는 원리를 따르고 있다.
    - useRecoilState : useRecoilValue가 단순히 atom의 값을 가져오기 위한 훅이었다면, useRecoilState는 좀 더 useState와 유사하게 값을 가져오고, 또 이 값을 변경할 수도 있는 훅이다.
- selector라는 함수 또한 사용되는데, 한 개 이상의 atom 값을 바탕으로 새로운 값을 조립할 수 있는 API다.
    
    ```jsx
    // atom을 기반으로 또 다른 상태를 만듬
    const isBiggerThan10 = selector({
    	key : 'above10State',
    	get : ({ get }) => {
    		return get(counterState) >= 10
    	},
    });
    ```
    
- 이 외에도 atom에 비동기 작업도 추가할 수 있으며, `useRecoilStateLoadable` , `waitForAll`, `waitForAny`, `waitForAllSettled`와 같이 강력한 비동기 작업을 지원하기 위한 API도 있다.

### Recoil에서 영감을 받은, 그러나 조금 더 유연한 Jotai

- Jotai는 공식 홈페이지에도 나와있는 것처럼, Recoil의 atom 모델에 영감을 받아 만들어진 상태 관리 라이브러리다.
- 상향식(bottom-up) 접근법을 취하고 있으며 이는 리덕스와 같이 하나의 큰 상태를 애플리케이션에 내려주는 방식이 아니라, 작은 단위의 상태를 위로 전파할 수 있는 구조를 취하고 있음을 의미한다.
- 또한 앞서 언급했던 **리액트 Context**의 문제점인 **불필요한 리렌더링이 일어나는 문제**를 해결하고자 설계되어 **개발자들이 추가적으로 메모이제이션이나 최적화를 거치지 않아도 리렌더링이 발생되지 않도록 설계**되어 있다.

#### atom

- Jotai에도 atom 개념이 존재하며, 마찬가지로 **최소 단위의 상태**를 의미한다.
    - 다만 Recoil과는 다르게 atom 하나만으로도 상태를 만들 수도, 이에 파생된 상태를 만들 수도 있다.
    - atom이 최소한의 상태 단위라는 것까지는 동일하지만 **atom 하나로 파생된 상태까지 만들 수 있다는 점에서 차이가 있다.**
- 아래는 atom을 만드는 방법과 초기값이다.

```jsx
const counterAtom = atom(0);
console.log(counterAtom);

// ...
// {
//		init: 0,
//		read : (get) => get(config),
//		write : 
//			(get, set, update) => set(config, typeof update === 'function' ? 
//																update(get(config)) :
//																update)  
// }
```

- Jotai 내부 atom 구조를 살펴보면 이름과 개념적인 원리는 Recoil에서 따왔지만 구현 자체에는 약간의 차이가 있음을 알 수 있다.
    - 각 atom을 생성할 때마다 고유한 key가 필요했던 Recoil과는 다르게, Jotai는 atom을 생성할 대 별도의 key를 넘겨주지 않아도 된다.
    - atom 내부에는 key라는 변수가 존재하긴 하지만 외부에서 받는 값은 아니며 단순히 toString()을 위한 용도로 한정돼 있다.
    - config 객체를 반환하는데, 이 config에는 초깃값을 의미하는 init, 값을 가져오는 read, 값을 설정하는 write만 존재한다.
        - 즉, **Jotai에서 atom에 따로 상태를 저장하고 있지 않다.**

#### useAtomValue

- useAtomValue의 구조를 보면 `useReducer`가 있는데, 여기서 반환하는 상태값은 3가지로 `[ version, valueFromReducer, atomFromReducer ]` 이다.
    - 첫번째는 스토어의 버전, 두 번째는 atom에서 get을 수행했을 때 반환되는 값, 세 번째는 atom 그 자체를 의미한다.
- Recoil과는 다르게 컴포넌트 루트 레벨에서 Context가 존재하지 않아도 되는데, Context가 없다면 Provider가 없는 형태로 기본 스토어를 루트에 생성하고 이를 활용해 값을 저장하기 때문이다.
- 그리고, **atom의 값은 store에 존재**한다. store에 atom 객체 그 자체를 키로 활용해 값을 저장한다.
    - 이러한 방식을 위해 `WeakMap`이라고 하는, JS에서 객체만을 키로 가질 수 있는 독특한 방식의 Map을 활용해 **recoil과는 다르게 별도의 key를 받지 않아도 스토어에 값을 저장**할 수 있다.
- rerenderIfChanged는 리렌더링을 일으키기 위해 사용한다. 아래 두 가지 경우에 일어난다.
    - 넘겨받은 atom이 Reducer를 통해 스토어에 있는 atom과 달라지는 경우.
    - subscribe를 수행하고 있다가 어디선가 이 값이 달라지는 경우
    
    → 이러한 로직 덕에 atom의 값이 어디서 변경되더라도 `useAtomValue`로 값을 사용하는 쪽에서는 언제든 최신 값의 atom을 사용해 렌더링할 수 있게 된다.
    

#### useAtom

- useState와 동일한 형태의 배열을 반환한다. 첫 번째로는 atom의 현재 값을 나타내는 useAtomValue 훅의 결과를, 두 번째로는 useSetAtom 훅을 반환한다.

#### 정리

- Jotai에서 상태를 선언하기 위해서는 atom 이라는 API를 사용하는데, 이는 useState와 다르게 컴포넌트 외부에서도 선언할 수 있다. 또한 atom은 함수를 인수로 받을 수도 있어, 이를 통해 **파생된 atom**을 만들 수 있다.
    
    ```jsx
    // atom 생성
    const counterState = atom(0);
    // 파생된 atom 생성
    const isBiggerThan10 = atom((get) => get(counterState) > 10);
    ```
    
- 이 외에도 localStorage와 연동해 영구적으로 데이터를 저장하거나, Next.js, 리액트 네이티브와 연동하는 등 상태와 관련된 다양한 작업을 Jotai에서 지원한다.
- 또한, Recoil에서 영감을 받은 만큼 그와 유사한 면과 Recoil이 가지고 있는 몇 가지 한계점을 극복하기 위한 노력이 엿보인다.
    - Recoil의 atom 개념을 도입하면서도 API가 간결하다.
        - Recoil의 atom에서는 각 상태값이 모두 별도의 키를 필요로 하기 때문에 이 키를 별도로 관리해야 하는데, Jotai는 이러한 부분을 추상화해 사용자가 키를 관리할 필요가 없다.
        - Jotai가 별도의 문자열 키가 없어도 각 값들을 관리할 수 있는 것은 객체의 참조를 통해 값을 관리하기 때문이다. (객체를 WeakMap에 보관)
- Recoil에서는 atom에서 파생된 값을 만들기 위해 selector API가 필요했지만, Jotai에서는 selector가 없어도 atom 만으로 atom 값에서 또 다른 파생된 상태를 만들 수 있다.

### 작고 빠르며 확장에도 유연한 zustand

- Jotai가 Recoil에 영감을 받아 만들어졌다면, Zustand는 리덕스에 영감을 받아 만들어졌다.
    - atom이라는 개념으로 최소 단위를 관리하는 것이 아니라 **하나의 스토어를 중앙 집중형으로 활용해 이 스토어 내부에서 상태를 관리**하고 있다.
- 스토어 만드는 코드를 살펴보면, 앞서 만들어본 바닐라 스토어와 유사하게 **state의 값을 외부에서 관리하는 것**을 볼 수 있다.
- **`createStore`는 `getState, setState, subscribe, destroy` 를 반환한다.**
    - `setState` 는 state 값을 변경하는 용도의 함수지만, partitial과 replace로 나뉘어 있다. 각각 state의 일부분만 변경하고 싶을 때, state를 완전히 새로운 값으로 변경하고 싶을 때 사용한다.
        - state의 값이 객체일 때 필요에 따라 나눠서 사용할 수 있음
    - `getState`는 클로저의 최신 값을 가져오기 위해 함수로 만들어져 있다.
    - `subscribe`는 listener를 등록하는데, `listener`는 마찬가지로 Set 형태로 선언되어 추가와 삭제, 그리고 중복 관리가 용이하게끔 설계돼 있다.
        - 즉, 상태값이 변경될 때 리렌더링이 필요한 컴포넌트에 전파될 목적으로 만들어졌음을 알 수 있다.
    - `destroy`는 listeners를 초기화하는 역할을 한다.
    - createStore로 스토어를 만들 때 set이라는 인수를 활용할 수 있다. set을 통해 현재 스토어의 값을 재정의할 수도 있고, 두번째 인수로 get을 추가해 현재 스토어의 값을 받아올 수도 있다.
        
        ```jsx
        const store = createStore<CounterStore>((set)=>{
        	count : 0,
        	increase : (num : Number) => set((state)=>({ count : state.count + num }));
        });
        ```
        
        - 이렇게 생성된 스토어는 getState나 setState를 통해 현재 스토아의 값을 받아오거나 재정의할 수 있다. 또한 subscribe를 통해 스토어의 값이 변경될 때마다 특정 함수를 실행할 수도 있다.
- 특이한 점은 createStore가 구현되어있는 vanilla.ts 파일은 어떤 것도 import 하고 있지 않다는 점이다. 따라서 이 파일의 이름 처럼, createStore는 리액트를 비롯한 모든 프레임워크에 독립적으로 구성돼 있다. 즉, 순수한 바닐라 JS에서도 해당 store를 사용할 수 있다.

#### Zustand의 리액트 코드

- Zustand를 리액트에서 사용하기 위해서는 어디선가 store를 읽고 리렌더링을 해야 한다. 이에 도움을 주는 함수들은 `./src/react.ts`에 있다. 타입을 제외하고, 이 파일에서 export 하는 함수는 바로 `useStore`와 `create`다.
    - useStore : useSyncExternalStoreWithSelector를 사용해서 앞선 useStore의 subscribe, getState를 넘겨주고, 스토어에서 선택을 원하는 state를 고르는 함수인 selector를 넘겨주고 끝난다.
        - useSyncExternalStoreWithSelector는 useSyncExternalStore와 완전히 동일하지만 원하는 값을 가져올 수 있는 selector와 동등 비교를 할 수 있는 equalityFn 함수를 받는다는 차이가 있다.
        - useSyncExternalStore는 리액트 18에서 새롭게 만들어진 훅으로, 리액트 외부에서 관리되는 상태값을 리액트에서 사용할 수 있도록 도와준다.
- 또한, `./src/react.ts`에서는 `create` 메소드도 export하는데, 이는 리액트에서 사용할 수 있는 스토어를 만들어주는 변수다.
    - 바닐라의 createStore를 기반으로 만들어졌기 때문에 거의 유사하다고 볼 수 있다.
    - 차이점은 useStore를 사용해 해당 스토어가 즉시 리액트 컴포넌트에서 사용할 수 있도록 만들어졌다는 것이다.
- 아래는 리액트 환경에서 바닐라 createStore를 사용한 예제다.
    
    ```jsx
    interface Store {
    	count : number
    	text : string
    	increase : (count : number) => void
    	setText : (text : string) => void
    }
    
    const store = createStore<Store>((set) => ({
      count: 0,
      text: '',
      increase: (num) => set((state) => ({ count: state.count + num })),
      setText: (text) => set({ text })
    }));
    
    const counterSelector = ({ count, increase }) => ({
    	count,
    	increase
    });
    
    function Counter() {
    	const { count, increase } = useStore(store, counterSelector);
    	
    	function handleClick() {
    		increase(1);
    	}
    	
    	return (
    		<>
    			<h3>{count}</h3>
    			<button onClick={handleClick}>+</button>
    		</>
    	)
    }
    
    const inputSelector = ({ text, setText } : Store) => ({
    	text,
    	setText
    });
    
    function Input() {
    	const { text, setText } = useStore(store, inputSelector);
    	
    	useEffect(()=>{
    		console.log('Input Changed');
    	});
    	
    	function handleChange(e : ChangeEvent<HTMLInputElement>) {
    		setText(e.target.value);
    	}
    	
    	return (
    		<div>
    			<input value={text} onChange={handleChange} />
    		</div>
    	)
    }
    ```
    
- create 메소드로 store를 만든 예제는 다음과 같다.
    
    ```jsx
    import { create } from 'zustand'
    
    const useCounterStore = create(()=>{
    	count : 1,
    	inc : () => set((state) => ({ count : state.count + 1 })),
    	dec : () => set((state) => ({ count : state.count - 1 }))
    });
    
    function Counter() {
    	const { count, inc, dec } = useCounterState();
    	return {
    		<div>
    			<span>{ count }</span>
    			<button onClick={inc}>up</button>
    			<button onClick={dec}>down</button>
    		</div>
    	}
    }
    ```
    
    - create를 사용해 스토어를 만들고, 반환 값으로 컴포넌트 내부에서 사용할 수 있는 훅을 받았다.

#### 정리

- Zustand는 특별히 많은 코드를 작성하지 않아도 빠르게 스토어를 만들고 사용할 수 있다는 큰 장점이 있다. 스토어를 만들고 이 스토어에 파생된 값을 만드는 데 단 몇 줄의 코드면 충분하다.
- Bundlephobia 기준으로 79.1kB인 Recoil, 13.1kb인 Jotai와 다르게 Zustand는 고작 2.9kb밖에 되지 않는다.
- 리덕스와 마찬가지로 미들웨어를 지원한다.