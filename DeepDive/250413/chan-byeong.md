# 2장 리액트 핵심요소 깊게 살펴보기

## 2.1 JSX 란?

- JSX는 ECMAScript라고 불리는 자바스크립트 표준의 일부가 아니다. 따라서 V8 엔진 또는 브라우저에서 동작되지 않는다.
  - ECMAScript :
- JSX의 설계 목적은 츠리 구조를 토큰화하여 ECMAScript로 변환하기 위함이다.
- JSX -> JS 변환 : 트랜스파일러

### 2.1.1 JSX 정의

- 구성요소 : JSX Element, JSX Attribute, JSX children, JSX string

- JSX Element:

  - Element는 반드시 대문자로 시작해야 한다. 이유는 다른 HTML 태그와 `Custom Component`를 비교하기 위함이다.
  - 네이밍 규칙
    1. `$`, `_` 이외의 다른 특수문자로 시작할 수 없다.
    2. identifier`:`identifier ( `:`로 이어주기 가능, 하나까지만 가능)
    3. indentifer`.`identifier`.`identifier (`.`로 이어주기 가능, 여러개 가능)

- JSX Attribute:

  - JSXSpreadAttributes: 자바스크립트의 연개 연산자와 동일한 역할을 한다고 볼 수 있다.
  - JSXAttribute: 커스텀 태그의 속성을 `키`-`값`으로 짝을 이루어 표현한다.

- JSX Children:

  - JSXElement의 자식 값을 나타낸다.

- JSX String:
  - JSX <-> HTML 간 복붙을 쉽게 할 수 있도록 설계되었다.
  - `\`를 이스케이프 문자로 처리하지 않는다. (HTML에서도 이스케이프 문자로 처리하지 않음)

### 2.1.3 JSX는 어떻게 자바스크립트에서 변환될까?

`@babel/plugin-transform-react-jsx` 플러그린을 알아야한다. 해당 플러그인은 JSX 구문을 자바스크립트가 이해할 수 있는 형태로 변환한다.

해당 플러그인의 특징은 다음과 같다.

1. JSX Element를 첫번째 인수로 선언해 요소를 정의한다.
2. 옵셔널인 Attibutem children, string은 이후 인수로 넘겨 처리한다.

```javascript
// JSX
<h1>
  <h2>Text</h2>
</h1>;

// Transform
React.createElement("h1", null, React.createElement("h2", null, "Text"));

// 플러그인의 특징을 이용하여 간결하게 표현한 방식
// JSX
isHeading ? (
  <h1 className='text'>{children}</h1>
) : (
  <span className='text'>{children}</span>
);
// Transform
React.createElement(
  isHeading ? "h1" : "span",
  {
    className: "text",
  },
  children
);
```

### 2.1.4 정리

- JSX는 자바스크립트 코드 내부에서 HTML과 같은 트리 구조를 가진 컴포넌트를 표현할 수 있다는 점에서 각광받고 있다.
- 일부에서는 JSX가 HTML 문법과 JS 문법이 뒤섞여서 가독성을 해친다는 의견도 있다.

## 2.2 가상 DOM과 리액트 파이버

### 2.2.1 DOM과 브라우저 렌더링 과정

_(https://www.youtube.com/watch?v=R23JmhbPnVo)_

1. 브라우저가 사용자가 요청한 주소를 방문해 HTML 파일을 다운로드한다.
2. 브라우저의 렌더링 엔전은 HTML을 파싱하며 태그와 속성을 발견하고 이를 DOM 트리로 만든다. (DOM 트리는 메모리에 저장된다.)
3. 2번 과정에서 CSS 파일(CSS 링크, `style`태그)을 만나면 해당 CSS 파일도 다운로드하고 파싱한다.
4. CSS를 파싱하여 CSSOM 트리를 생성한다.
5. DOM 트리과 CSSOM 트리를 결함하여 렌더트리를 생성한다.
6. 렌더 트리는 브라우저 상에서 요소의 위치와 크기를 결정하는 `reflow` 과정을 거친다.
7. 다음으로 요소의 색상 등 시각적 요소를 결정하는 `repaint` 과정을 거친다.

(\*HTML 파싱 도중 `script` 태그를 만나면 브라우저는 HTML 파싱을 중단하고 `script` 태그를 다운로드 및 실행한다.)

### 2.2.2 가상 DOM의 탄생 배경

브라우저가 웹페이지를 렌더링하는 과정은 매우 복잡하고 많은 비용이 든다.

SPA에서는 DOM 트리의 DOM 노드를 변경하여 사용자에게 화면의 변화를 보여준다. 유저의 인터렉션에 의해 DOM 요소의 변화가 많이 생긴다.

이때 개발자나 브라우저는 모든 DOM의 변경을 아는 것보다 최종적으로 만들어지는 DOM 결과물 하나만 아는 것이 훨씬 유용하다.

이러한 문제점을 해결하기 위해 탄생한 것이 `가상 DOM`이다.

가상 DOM은 메모리 상에 존재하고 메모리 위에서 DOM의 구조를 변경 후 브라우저로 반영한다.

하지만 무조건 빠른 것은 아니다. 다만, 충분히 빠르다.

```markdown
- 가상 DOM : Javascript 객체로 메모리에 직접 접근 가능 (Javascript의 힙 메모리에 존재)
- 실제 DOM : 브라우저의 렌더링 엔진을 통해 간접적 접근 가능 (렌더링 엔지 내 DOM 트리 메모리)
  - DOM API를 통한 간접적 접근
  - 변경 마다 렌더링 엔진과의 상호작용 필요
  - 변경할 때마다 `reflow`와 `repaint` 발생 가능
```

### 2.2.3 가상 DOM을 위한 아키텍처, 리액트 파이버

가상 DOM 렌더링 최적화 비결은 `React Fiber`이다.

`React fiber`란 리액트에서 관리하는 평범한 자바스크립트 객체(가급적이면 파이버 객체는 재사용된다.)이다. 파이버는 파이버 재조정자가 관리한다.

- _Fiber 객체는 하나의 Element와 1:1 관계를 가진다. 이를 구분하는 키가 `tag` 속성이다._

재조정자는 가상DOM과 실제DOM을 비교해 변경 사항을 수집하며 만약 이둘 사이에 차이가 있으면 변경에 관련된 정보를 가지고 있는 파이버를 기준으로 화면에 렌더링을 요청하는 역할을 한다.

- Fiber가 수행하는 일
  1. 작업을 잔은 단위로 분할하고 쪼갠 다음, 우선순위를 매긴다.
  2. 이러한 작업을 일시 중단하고 다시 시작할 수 있다.
  3. 이전에 했던 작업을 다시 재사용하거나 필요하지 않은 경우 폐기할 수 있다.

이러한 모든 과정은 `비동기`로 일어난다는 것이다.

- _과거에는 스택 알고리즘으로 이루어져 동기적으로 작업이 이루어짐_

파이버는 어떻게 구현되어 있을까?

파이버는 하나의 작업 단위(Unit of Work)로 구성되어 있다. 리액트는 이러한 작업 단위를 하나씩 처리하고 `finishedWork()`라는 작업으로 마무리한다.

- _작업 단위란?_

  - React가 처리하는 최소 단위의 작업
  - 각 Fiber 노드가 하나의 작업 단위

  ```javascript
  function App() {
    const [count, setCount] = useState(0);

    return (
      <div>
        <h1>Count: {count}</h1>
        <button onClick={() => setCount((c) => c + 1)}>Increment</button>
      </div>
    );
  }

  // 작업 단위 처리 과정
  // 1. App 컴포넌트 렌더링 (작업 단위 1)
  // 2. div 엘리먼트 생성 (작업 단위 2)
  // 3. h1 엘리먼트 생성 (작업 단위 3)
  // 4. button 엘리먼트 생성 (작업 단위 4)
  // 5. 상태 업데이트 이펙트 처리 (작업 단위 5)
  ```

1. 렌더 단계

- 가상 DOM을 생성하거나 업데이트하는 단계
- 실제 DOM에 변경사항을 반영하지 않음
- 중단, 재개, 취소가 가능한 비동기 단계

2. 커밋 단계

- 실제 DOM에 변경사항을 반영하는 단계
- 동기적으로 실행되며 중단할 수 없음
- 변경사항을 한번에 적용

파이버도 트리 구조를 가진다. (child, sibling, return 속성 활용)

- child: 첫번째 요소만 자식으로 가진다.
- sibling: 첫번째 요소를 제외한 그 형제들
- return: 부모 fiber를 가르킨다.

#### 리액트 파이버 트리

파이버 트리는 리액트 내부에서 2개가 존재한다.

- 현재 모습을 담은 `current` 파이버 트리
- 작업 중인 상태를 담은 `workInProgress` 트리

리액트 파이버의 작업이 끝나면 리액트는 단순히 `포인터`만 변경해 `wIP`트리를 `current`트리로 바꾼다.

이러한 기술을 `더블 버퍼링`이라고 한다. (`더블 버퍼링`은 `커밋 단계`에서 수행된다.)

#### 파이버의 작업 순서

1. `beginWork()` 함수 실행. 더 이상 **자식**이 없는 파이버를 만날 떄까지 트리 형식으로 시작된다.
2. 1번에서 작업이 끝나면 `completeWork()` 함수를 실행해 파이버 작업을 완료한다.
3. **형제**가 있다면 **형제**로 넘어간다.
4. 2,3번이 모두 끝났다면 `return`으로 돌아가 자신의 작업이 완료됐음을 알린다.

```javascript
function App() {
  return (
    <div>
      <Header />
      <Content>
        <Sidebar />
        <Main />
      </Content>
    </div>
  );
}
```

```text
// 파이버 트리 구조
App Fiber
  ├── child: div Fiber
        ├── child: Header Fiber
        └── sibling: Content Fiber
              ├── child: Sidebar Fiber
              └── sibling: Main Fiber
```

```javascript
// 1. beginWork() 실행 순서
function beginWork(currentFiber) {
  // App Fiber
  //   ↓
  // div Fiber
  //   ↓
  // Header Fiber (자식 없음)
  //   ↓
  // Content Fiber
  //   ↓
  // Sidebar Fiber (자식 없음)
  //   ↓
  // Main Fiber (자식 없음)
}

// 2. completeWork() 실행 순서
function completeWork(fiber) {
  // Header Fiber (자식 없음)
  //   ↓
  // Sidebar Fiber (자식 없음)
  //   ↓
  // Main Fiber (자식 없음)
  //   ↓
  // Content Fiber
  //   ↓
  // div Fiber
  //   ↓
  // App Fiber
}
```

### 2.2.4 파이버와 가상 DOM

리액트 컴포넌트에 대한 정보를 1:1로 가지고 있는 것이 파이버이며, 이 파이버는 리액트 아키텍처 내부에서 비동기로 이뤄진다.

실제 브라우저 구조인 DOM에 반경하는 것은 동기적으로 이루어져야 하며, 처리하는 작업이 많아 화면에 불완전하게 표시될 수있는 가능성이 높으므로 이러한 작업을 가상에서, 즉 메모리 상에서 먼저 수행해서 최종결과만을 실제 브라우저 DOM에 반영한다.

### 2.2.5 정리

가상 DOM과 리액트의 핵심은 브라우저의 DOM을 더욱 빠르게 그리고 반영하는 것이 아니라 바로 값으로 UI를 표현하는 것이다.

화면에 표시되는 UI를 자바스크립트의 문자열, 배열 등과 마찬가지로 값으로 관리하고 이러한 흐름을 효율적으로 관리하기 위한 메커니즘이 바로 리액트의 핵심이다.

## 2.3 클래스 컴포넌트와 함수 컴포넌트

초기에는 단순히 정적 렌더링을 위해 함수형 컴포넌트를 사용, 훅이 등장하면서 상태나 생명주기 메서드를 흉내낼 수 있으면서 함수형 컴포넌트가 각광 받았다.

### 2.3.1 클래스 컴포넌트

클래스 컴포넌트의 종류

- React.Component
- React.PureComponent

이 둘의 차이는 `shouldComponentUpdate` 메서드에서 차이가 난다.

- _`shouldComponentUpdate`는 props나 state의 변화가 있는 경우만 컴포넌트를 리랜더링한다._

```javascript
  shouldComponentUpdate(nextProps: Props, nextState: State) { ... }
```

- React.Component:

  - shouldComponentUpdate가 기본적으로 true를 반환
  - 모든 업데이트에서 리렌더링 발생

- React.PureComponent:
  - props와 state의 얕은 비교를 수행
  - 실제 값이 변경된 경우에만 리렌더링
  - 불변성(immutability)을 기반으로 동작

#### 화살표 함수와 일반 함수

메서드를 일반함수로 작성할 경우 `this` 바인딩을 해줘야한다.

일반함수는 **실행 시점**에서 this가 상위 스코프로 정해지기 때문에 예측하기 어렵고 클래스 인스턴스를 가르키지 않을 수 있다.

```javascript
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };

    // 일반 함수의 경우 this 바인딩 필요
    this.increment = this.increment.bind(this);
  }

  // 일반 함수
  increment() {
    // this가 undefined가 될 수 있음
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return <button onClick={this.increment}>증가</button>;
  }
}
```

화살표 함수의 경우 **작성 시점**에 this가 상위 스코프로 결정되기 때문에 따로 `this` 바인딩을 해주지 않아도 된다.

#### 클래스 컴포넌트의 생명주기 메서드

생명주기 메서드가 실행되는 시점은 크게 3개로 나눌 수 있다.

- 마운트: 컴포넌트가 생성되는 시점
- 업데이트: 이미 생성된 컴포넌트의 내용이 변경되는 시점
- 언마운트: 컴포넌트가 더 이상 존재하지 않는 시점

에러 상황에서 발생하는 메서드

`getDerivedStateFromError()`

자식 컴포넌트가 에러가 발생했을 때 호출되는 메서드이다.

해당 메서드는 `static`메서드로 `error`를 인수로 받는다.

## 2.4 렌더링은 어떻게 일어나는가?

### 2.4.1 리액트에서 렌더링이란?

### 2.4.2 리액트의 렌더링이 일어나는 이유

1. 최초 렌더링

2. 리렌더링

- setState가 호출되는 경우 (클래스형 컴포넌트, 함수형 컴포넌트)
- forceUpdate가 실행되는 경우 (클래스형 컴포넌트)
- useReducer의 dispatch가 호출되는 경우 (함수형 컴포넌트)
- 컴포넌트의 `key props`가 변경되는 경우

  리액트에서의 key는 리렌더링이 발생하는 동안 형제 요소들 사이에서 동일한 요소를 식별하는 값이다.

  파이버 트리의 current 트리와 wip 트리를 비교할 때 같은 컴포넌트인지 비교할 때 사용하는 것이 `key`값이다.

  current 트리와 wip 트리를 비교 시 `key`값은 컴포넌트의 식별과 재사용을 위한 고유 식별자 역할을 한다.

  `key`는 리스트에서 각 항목의 고유 식별자로, React가 컴포넌트의 재사용 여부를 결정하고 DOM 업데이트를 최적화하는 데 사용된다.

  이떄 `key`값의 유의 사항으로는

  1. 고유해야 한다.
  2. 배열의 인덱스를 key로 사용하지 말아야 한다.
  3. key는 변경되지 않아야 한다.
  4. key는 형제 요소 간에만 고유하면 된다.

### 2.4.3 리액트의 렌더링 프로세스

current 트리와 wip 트리를 비교하여 **모든** 변경사항을 하나의 동기 시퀀스로 DOM에 적용한다.

이떄 리액트의 렌더링은 **렌더 단계**와 **커밋 단계**로 구분된다.

### 2.4.4 렌더와 커밋

**렌더 단계**

컴포넌트를 렌더링하고 변경 사항을 계산하는 모든 작업을 말한다.

컴포넌트를 실행한 결과와 이전 가상 DOM을 비교하여 변경이 필요한 컴포넌트를 체크한다.

이때 비교하는 것은 `type`, `props`, `key`이다.

**커밋 단계**

렌더 단계의 변경 사항을 실제 DOM에 적용하여 사용자에게 보여주는 단계이다.

해당 단계가 끝나야 비로소 브라우저의 렌더링이 발생한다.

```text
React 렌더링
  ↓
커밋 단계
  ↓
DOM 업데이트
  ↓
브라우저 렌더링 시작
  ├── DOM 트리 업데이트
  ├── CSSOM 트리 업데이트
  ├── 렌더 트리 생성
  ├── 레이아웃 계산
  └── 페인팅
```

커밋 단계에서 먼저 더블 버퍼링을 통해 workInProgress 트리를 current 트리로 전환한 후, 실제 DOM 업데이트를 수행한다.

커밋 단계에서 변경 사항들을 모두 업데이트한 뒤 리액트 내부에서는 참조를 업데이트한다.

그다음 생명주기 개념이 있는 클래스 컴포넌트에서는 `componentDidMount`, `componentDidUpdate` 메서드를 호출,

함수형 컴포넌트에서는 `useLayoutEffect` 훅을 호출 한다.

```javascript
// 1. 렌더 단계
// - 가상 DOM 생성/업데이트
// - 변경사항 계산

// 2. 커밋 단계
// 2.1. 더블 버퍼링 (참조 업데이트)
// - workInProgress 트리를 current 트리로 전환
// - 포인터만 변경하여 빠른 전환

// 2.2. DOM 업데이트
// - 실제 DOM에 변경사항 적용
// - 동기적으로 실행

// 2.3. 생명주기 메서드/훅 실행
// - componentDidMount/Update
// - useLayoutEffect
```

여기서 중요한 사실은 **리액트의 렌더링이 일어난다고 해서 무조건 DOM 업데이트가 일어나는 것은 아니다.**

```javascript
// 예시
function Counter() {
  const [count, setCount] = useState(0);

  // 이 함수는 렌더링을 발생시키지만 DOM 업데이트는 일어나지 않음
  const handleClick = () => {
    setCount(0); // count가 이미 0인 상태에서 다시 0으로 설정
  };

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={handleClick}>리셋</button>
    </div>
  );
```

## 2.5 컴포넌트와 함수의 무거운 연산을 기억해 두는 메모이제이션

잘못된 memo로 인해 지불해야 하는 비용

- props에 대한 얕은 비교
- 메모이제이션을 위해 이전 렌더링 결과물을 저장 (CPU, Memory 사용)
-

memo를 하지 않았을 경우 발생할 수 있는 문제

- 렌더링 비용
- 컴포넌트 내부의 로직 재실행
- 하위 컴포넌트의 렌더링

### 2.5.3 결론 및 정리

정답은 없으나 의심 가는 곳에 메모이제이션을 활용한 최적화는 도움이 될 가능성이 크다.
