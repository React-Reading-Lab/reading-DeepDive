# 10장 리액트 17, 18의 변경사항 살펴보기

## 10.1 리액트 17 버전 살펴보기

17 버전은 새롭게 추가된 기능은 없으면 기존 코드의 수정을 최소화했다는 점이 가장 큰 특징이다.

### 10.1.1 리액트의 점진적인 업그레이드

리액트 17 버전부터는 점진적인 업그레이드가 가능하다.

이러한 점진적 업그레이드를 지원하기 위한 리액트의 일부 컴포넌트 변경이 리액트 17 업데이트의 주요 변경 사항 중 하나이다.

'한 어플리케이션 내에 여러 버전의 리액트가 존재하는 시나리오'는 어떤 것일까?

### 10.1.2 이벤트 위임 방식의 변경

리액트에서 이벤트가 어떻게 추가되는가?

```javascript
export default function Button() {
  const buttonRef = useRef(null);

  useEffect(() => {
    if (buttonRef.current) {
      buttonRef.current.onclick = function click() {
        alert("hello");
      };
    }
  }, []);

  function hello() {
    alert("hello");
  }

  return (
    <>
      <button onClick={hello}>React button</button>
      <button ref={buttonRef}>Normal button</button>
    </>
  );
}
```

`Normal button`의 이벤트 리스너를 확인해보면 `click` 이벤트가 추가된 것을 확인할 수 있다.
`React button`의 이벤트 리스너를 확인해보면 `noop`라는 이벤트가 추가되었다. 이때 `noop`는 no operation으로 아무일도 하지 않는 이벤트이다.

리액트는 이벤트를 처리할 때 해당 버튼에 이벤트를 위임하는 것이 아니라 이벤트 타입(click, change ...) 당 하나의 핸들러를 `루트`에 부착한다.
이를 이벤트 위임이라고 한다.

이로인해 실제 이벤트 핸들러의 역할은 해당 버튼이 아니라 루트에 존재하는 이벤트 핸들러가 역할을 수행하게 된다.

_이벤트 전파 단계_

_캡처링 -> 타켓 -> 버블링_

리액트 16 버전까지는 이벤트 위임을 모두 `document`에 했지만 리액트 17버전 부터는 최상단 노드인 `root`노드에 이벤트 위임을 한다.

**이렇게 변경하게된 이유는 무엇일까?**

그것은 앞서 이야기한 점진적인 업그레이드 지원으 하기 위함이다.

만약 하나의 어플리케이션에 여러 개의 리액트 버전이 혼재해있을 경우 아래와 같은 html 트리가 구성되었다고 보자

```html
<html>
  <body>
    <div id="react-16-14">
      <div id="react-16-8"></div>
    </div>
  </body>
</html>
```

16 버전에서는 이벤트 위임을 `document`에 한다.

만약 16-8 컴포넌트에서 `e.propagation`을 통해 이벤트 전파를 막는다 하더라도 이미 모든 이벤트가 `document`에 위임된 상태이기 때문에 상위 컴포넌트인 16-14 컴포넌트에도 이벤트는 날라간다.

이처럼 서로 다른 리액트 버전에서 발생할 수 있는 문제를 해결하기 위해 `document`가 아닌 최상위 컴포넌트로 이벤트 위임 대상을 변경했다.

### 10.1.3 import React from 'react`가 더 이상 필요없다: 새로운 JSX transform

jsx는 브라우저가 처리할 수 없는 언어이므로 js로 변환과정이 반드시 필요하다.

16버전까지는 이러한 jsx 변환을 위해 코드 내에서 Reeact를 사용하는 구문이 없더라도 import React from 'react'가 필요했고 이것이 없다면 에러가 발생했다.

그러나 17 버전부터는 import 구문없이도 JSX를 변환할 수 있게 됐다.

### 10.1.4 그 밖의 주요 변경 사항

#### 리액트 풀링 제거

리액트 16 버전에서는 `이벤트 풀링` 기능이 존재했다. 리액트에서는 이벤트를 처리하기 위한 `SyntheticEvent`라는 이벤트가 있는데 이 이벤트는 브라우저의 기본 이벤트를 한 번 더 감싼 이벤트 객체이다.

리액트는 이렇게 브라우저 기본 이벤트가 아닌 한번 리팽한 이벤트를 사용하기 때문에 이벤트가 발생할 때마다 해당 이벤트를 새로 생성했어야 했다.

그 과정에서느 항상 새로 이벤트를 만들 때마다 메모리 할당 작업이 일어날 수 밖에 없다.또한 메모리 누수를 방지하기 위해서 이렇게 만든 이벤트를 주기적으로 해제해야 하는 번거로움도 있다. 여기서 이벤트 풀링이란 SyntheticEvent 풀을 만들어서 이벤트가 발생할 때마다 가져오는 것을 의미한다.

즉, 이벤트 풀링 시스템에서는 다음과 같이 이벤트가 발생한다.

1. 이벤트 핸들러가 이벤트를 발생시킨다.
2. 합성 이벤트 풀에서 합성 이벤트 객체에 대한 참조를 가져온다.
3. 이 이벤트 정보를 합성 이벤트 객체에 넣어준다.
4. 유저가 지정한 이벤트 리스너가 실행된다.
5. 이벤트 객체가 초기화되고 다시 이벤트 풀로 돌아긴다.

언뜻 보기에는 이벤트 풀에 존재하는 합성 이벤트(Synthetic Event)를 반복적으로 사용할 수 있어 효과적으로 보이지만 풀에서 이벤트를 받아오고 이벤트가 종료되자마자 다시 초기화 하는 방식으로 사용하는 입장에서 직관적이지않다.

```javascript
export default function App() {
  const [value, setValue] = useState('')
  function handleChange(e) {
    setValue(() => return e.target.value)
  }
}
```

위의 코드는 에러가 발생한다. 왜냐하면 이벤트가 호출된 이후 다른 이벤트를 위해 이벤트 필드를 모두 null로 초기화한다.
비동기 코드 내부에서 일관된 합성 이벤트 e에 접근하기 위해서는 e.persist()같은 작업이 필요하다.

이와 같은 방식이 성능 개선에 크게 도움이 되지 않고 사용자에게 직관적이지 않아서 삭제되었다.

최신 브라우저들의 자바스크립트 엔진 성능이 매우 향상되어 객체를 생성하고 해제하는 비용이 크게 줄었다. 오히려 이벤트 풀링을 위해 내용을 지우고 관리하는 비용이 더 크게 든다.

#### useEffect 클린업 함수의 비동기 실행

16 버전까지는 클린업 함수가 동기적으로 실행되어 다른 작업에 방해가 될 수 있었다.

17 버전부터는 화면이 완전히 업데이트된 이후에 클린업 함수가 비동기적으로 실행된다. 엄밀히 이야기해보면 클린업 함수는 컴포넌트의 커밋 단계가 완료될 떄까지 지연된다.

#### 컴포넌트의 undefined 반환에 대한 일관적인 처리

16, 17 버전은 컴포넌트 내부에서 undefined를 반환하면 오류가 발생한다.

리액트 16에서는 fowardRef나 memo에서 undefined를 반환하는 경우 별다른 에러가 발생하지 않는 문제가 있었는데 이를 수정했다.

## 10.2 리액트 18 버전 살펴보기

리액트 18 버전에서는 다양한 기능들이 추가되었다. 가장 큰 변경점은 `동시성` 지원이다.

### 10.2.1 새로 추가된 훅 살펴보기

#### useId

usedId는 컴포넌트 별로 유니크한 값을 생성하는 새로운 훅이다.

useId는 클라이언트와 서버에서 불일치를 피하면서 컴포넌트 내부의 고유한 값을 생성할 수 있게 됐다.

```javascript
const id = useId();
```

#### useTransition

useTransition 훅은 UI 변경을 가로막지 않고 상태를 업데이트할 수 있는 리액트 훅이다. 이를 활용하면 상태 업데이트를 긴급하지 않은 것으로 간주해 무거운 렌더링 작업을 조금 미룰 수 있으며 사용자에게 조금 더 나은 사용자 경험을 제공할 수 있다.

useTransition은 아무것도 인수로 받지 않으며, isPending과 startTransition이 담긴 배열을 반환한다. startTransition은 긴급하지 않은 상태 업데이트로 간주할 set 함수를 넣어둘 수 있는 함수를 인수로 받는다.

과거 리액트의 모든 렌더링은 동기적으로 작동해 느린 렌더링 작업이 있을 경우 어플리케이션 전체적으로 영향을 끼쳤으나 useTransition과 같은 동시성을 지원하는 기능을 사용하면 느린 렌더링 과정에서 로딩 화면을 보여주거나 혹은 지금 진행 중인 렌더링을 버리고 새로운 상태값으로 다시 렌더링하는 등의 작업을 할 수 있다.

만약 훅을 사용할 수 없는 환경일 경우 `startTransition`을 직접 import하여 사용할 수도 있다.

```
import { startTransition } from 'react'
```

사용 시 주의할 점을 살펴보자

- startTransition 내부에는 반드시 setState와 같은 상태를 업데이트하는 함수와 관련된 작업만 넘길 수 있다. 만약 props나 사용자 정의 훅에서 반환하는 값 등을 사용하고 싶다면 `useDefferedValue`를 사용하면 된다.
- startTransition으로 넘겨주는 상태 업데이트는 다른 모든 동기 상태 업데이트로 인해 실행이 지연될 수 있다. 예를 들어 타이핑을 통해 setState가 일어나는 경우 타이핑이 끝날 때까지 useTransition으로 지연시킨 상태 업데이트는 일어나지 않는다.
- startTransition으로 넘겨주는 함수는 반드시 동기함수여야 한다. 만약 비동기 함수를 넣을 시 제대로 작동하지 않는다.

#### useDeferredValue

useDeferredValue는 리액트 컴포넌트 트리에서 리렌더링이 급하지 않은 부분을 지연할 수 있게 도와주는 훅이다.

특정 시간 동안 발생하는 이벤트를 하나로 인식해 한번만 실행시켜주는 디바운스와 비슷하지만 디바운스 대비 useDeferredValue만이 가진 장점이 몇가지 있다.

먼저 디바운스는 고정된 지연 시간을 필요로 하지만 useDeferredValue는 고정된 지연 시간 없이 첫 번째 렌더링이 완료된 이후에 useDeferredValue로 지연된 렌더링을 실행한다.

```javascript
import React, { useState, useDeferredValue, useMemo } from "react";

const SlowList = ({ text }) => {
  // useMemo를 사용해 text 값이 바뀔 때만 오래 걸리는 계산을 다시 하도록 합니다.
  const items = useMemo(() => {
    const list = [];
    // 200개의 아이템을 생성하여 렌더링에 부하를 줍니다.
    for (let i = 0; i < 200; i++) {
      list.push(
        <div key={i}>
          입력값: '{text}'에 대한 검색 결과 {i + 1}
        </div>
      );
    }
    return list;
  }, [text]);

  return <>{items}</>;
};

export default function App() {
  const [query, setQuery] = useState("");

  // useDeferredValue를 사용합니다.
  // deferredQuery는 query의 업데이트를 즉시 따라가지 않고,
  // 브라우저가 다른 긴급한 작업을 처리한 후에 업데이트됩니다. (지연된 값을 가집니다)
  const deferredQuery = useDeferredValue(query);

  return (
    <div>
      <h1>
        <code>useDeferredValue</code> 예제
      </h1>
      <p>
        아래 입력창에 빠르게 타이핑해보세요. <br />
        입력 필드는 즉시 반응하지만, 아래 목록은 약간의 지연 후 갱신되어 부드러운
        사용자 경험을 제공합니다.
      </p>

      {/* 이 입력 필드의 값은 즉시 업데이트됩니다. */}
      <input
        type='text'
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder='빠르게 입력해보세요...'
        style={{ fontSize: "1.2rem", padding: "8px" }}
      />
      <hr />

      {/* 
        이 리스트 컴포넌트는 '지연된' 값인 deferredQuery를 사용합니다.
        따라서 이 컴포넌트의 느린 렌더링이 위 입력 필드의 반응성을 해치지 않습니다.
      */}
      <SlowList text={deferredQuery} />
    </div>
  );
}
```

_useDeferredValue와 useTransition의 차이점_

- useTransition은 값을 업데이트하는 함수를 감싸서 사용하지만 useDeferredValue는 값 자체를 감싸서 사용한다.

사용하는 방식에는 위와 같은 차이가 존재하지만 하는 역할을 동일하다.

따라서 상황에 맞는 방법을 선택하여 활용하면 된다.

#### useSyncExternalStore

_tearing 현상이란?_

https://github.com/reactwg/react-18/discussions/69

리액트에서는 하나의 상태가 있음에도 서로 다른 값을 기준으로 렌더링되는 현상을 말한다.

17 버전에서는 이와 같은 일이 벌어질 일이 없지만 18 버전에서는 동시성 관련 훅을 사용하면 렌더링을 일시중지하거나 뒤로 미루는 등의 최적화가 가능해져 동시성 이슈가 발생할 수 있다.

예를 들어 useTransition으로 렌더링을 일시 중지했다고 했을 때 일시 중지 과정에서 값이 업데이트되면 동일한 하나의 변수에 대해서 서로 다른 컴포넌트 형태가 나타날 수 있다.

테어링이 발생하는 과정

1. 첫 번째 컴포넌트에서 외부 데이터 스토어의 값이 파란색이었으므로 파란색을 렌더링한다.
2. 그리고 나머지 컴포넌트들도 파란색으로 렌더링을 준비하고 있다.
3. 그러다 갑자기 외부 데이터 스토어의 값이 빨간색으로 변경되었다.
4. 나머지 컴포넌트들은 렌더링 도중에 바뀐 색을 확인해 빨간색으로 렌더링한다.
5. 결과적으로 같은 데이터 소스를 바라보고 있음에도 렌더링 결과는 다른 테어링 현상이 발생했다.

리액트의 외부 데이터 소스를 사용하면 테어링 현상이 발생할 수 있다.이러한 문제를 해결하기 위해 useSyncExternalStore라는 훅을 만들었다.

```javascript
import { useSyncExternalStore } from 'react'

useSyncExternalStore(
  subscribe: (callback) => Unsubscribe
  getSnapshot: () => State
) => State
```

- 첫번째 인수는 subscribe로, 콜백 함수를 받아 스토어에 등록하는 용도로 사용된다. 스토어에 있는 값이 변경되면 이 콜백이 호출돼야 한다. 그리고 useSyncExternalStore는 이 훅을 사용하는 컴포넌트를 리렌더링한다.

- 두번째 인수는 컴포넌트에 필요한 현재 스토어의 데이터를 반환하는 함수다. 이 함수는 스토어가 변경되지 않았다면 매번 함수를 호출할 떄마다 동일한 값을 반환한다.

- 마지막 인수는 옵셔널 값으로 서버 사이드 렌더링 시에 내부 리액트를 하이드레이션하는 도중에만 사용된다. 서버 사이드에서 렌더링되는 훅이라면 반드시 이 값을 넘겨주어야 한다.

useSyncExternalStore에는 컴포넌트 렌더링을 발생시키기 위해 어딘가에 렌더링 트리거가 존재한다.

[예시 코드와 설명]

#### useInsertionEffect

useInsertionEffect는 CSS-in-js 라이브러리를 위한 훅이다.

CSS의 추가 및 수정은 브라우저에서 렌더링하는 작업 대부분을 다시 계산해 작업해야 하는데 이는 리액트 관점으로 본다면 모든 리액트 컴포넌트에 영향을 미칠 수도 있는 매우 무거운 작업이다.

따라서 17버전과 styled-components에서는 클라이언트 렌더링 시에 이러한 작업이 발생하지 않도록 서버 사이드에서 스타일 코드를 삽입했다.

그러나 이 작업을 훅으로 처리하는 것은 지금까지 쉽지 않았는데 훅에서 이러한 작업을 할 수 있도록 해주는 것이 바로 useInsertionEffect이다.

useInsertionEffect는 기본적으로 useEffect와 동일한 구조이다. 하지만 실행시점에서 차이가 존재한다.

useInsertionEffect는 DOM이 실제로 변경되기 전에 동기적으로 실행된다. 이 훅 내부에서 스타일을 삽입하는 코드를 집어넣음으로써 브라우저가 레이아웃을 계산하기 전에 실행될 수 있게끔 해서 좀 더 자연스러운 스타일 삽입이 가능하다.

정리하자면 useInsertionEffect -> useLayoutEffect -> useEffect 순으로 실행된다.

useLayoutEffect는 useInsertionEffect와 마찬가지로 브라우저에 DOM이 렌더링되기 전에 실행되지만 useLayoutEffect는 DOM 변경 작업이 다 끝난 후에 실행된다.

useInsertionEffect는 useSyncExternalStore와 마찬가지로 라이브러리를 작성하는 경우가 아니라면 참고만 하고 실제 어플리케이션에서는 가급적 사용하지 않는 것이 좋다

### 10.2.2 react-dom/client

클라이언트에서 리액트 트리를 만들 때 사용되는 API가 변경됐다.

#### createRoot

기존의 `render`를 대체하는 메서드이다.

#### hydrateRoot

서버 사이드 렌더링 어플리케이션에서 하이드레이션을 하기 위한 메서드이다.

### 10.2.3 react-dom/server

#### renderToPipeableStream

리액트 컴포넌트를 HTML로 렌더링하는 메서드이다. 스트림을 지원하는 메서드로 HTML을 점진적으로 렌더링하고 클라이언트에서는 중간에 script를 삽입하는 등의 작업을 할 수 있다.

서버에서는 Suspense를 사용해 빠르게 렌더링이 필요한 부분을 먼저 렌더링할 수 있고, 값비싼 연산으로 구성된 부분은 이후에 렌더링될 수 있게끔한다.

_기존의 renderToNodeStream_

문제는 무조건 렌더링을 순서대로 해야 하고 그리고 그 순서에 의존적이기 때문에 이전 렌더링이 완료되지 않으면 이후 렌더링도 완료되지 않는 문제점이 있었다. 따라서 렌더링 중간에 지연이 발생하면 뒤의 렌더링 작업도 마찬가지로 길어진다.

이러한 문제를 해소하기 위해 renderToPipeableStream는 렌더링 순서나 오래 걸리는 렌더링에 영향 받을 필요없이 빠르게 렌더링을 가능하게 해준다.

[뒷 내용??]

#### renderToReadableStream

renderToPipeableStream이 nodejs 환경에서의 렌더링을 위해 사용된다면, renderToReadableStream은 웹 스트림을 기반으로 작동한다는 차이가 있다.

### 10.2.4 자동 배치(Automatic Batching)

Automatic Batching은 리액트가 여러 상태 업데이트를 하나로 묶어서 한번의 렌더링으로 상태를 업데이트하는 방식이다.
[?]
18 버전부터는 createRoot를 사용해서 만들면 모든 업데이트가 배치 작업으로 최적화할 수 있게 됐다.

만약 의도적으로 배치작업을 사용하지 않으려면 `flushSync` 함수를 사용하여 래핑해주면 된다.

```javascript
function handleClick() {
  flushSync(() => {
    setCounter((c) => c - 1);
  });

  flushSync(() => {
    setFlag((f) => !f);
  });
}
```

### 10.2.5 더욱 엄격해진 엄격 모드

#### 리액트의 엄격 모드

어플리케이션에 발생할 수도 있는 잠재적인 버그를 찾는데 도움이 되는 컴포넌트이다. `<StrictMode>`

- 더 이상 안전하지 않은 특정 생명주기를 사용하는 컴포넌트에 대한 경고

`componentWillMount`, `componentWillReceiveProps`, `componentWillUpdate`를 사용하게 되면 엄격모드에서 아래와 같은 경고가 뜬다.

[사진]

- 문자열 ref 사용 금지

과거 리액트에서는 레거시 문자열 ref라 해서 createRef가 없어도 컴포넌트 내부에서 문자열로 ref를 생성하고, 이를 사용해 DOM 노드를 참조하는 것이 가능했다.

```javascript
class UnsafeClassComponent extends Component {
  componentDidMount() {
    //'refs' is deprecated
    // <input type='text' />
    console.log(this.ref.myInput);
  }

  render() {
    return (
      <div>
        <input type='text' ref='myInput' />
      </div>
    );
  }
}
```

render()에 있는 ref는 단순 문자열을 ref에 할당했다. 문자열을 바탕으로 DOM에 접근할 수 있지만 몇가지 문제가 있어 사용이 금지되었다.

- findDOMNode에 대한 경고 출력

findDOMNode는 클래스 컴포넌트 인스턴스에서 실제 DOM 요소에 대한 참조를 가져올 수 있지만 현재는 사용하는 것이 권장되지 않는 메서드이다.

_findDOMNode가 권장되지 않는 이유_

- findDOMNode를 사용하면 부모가 특정 자식만 별도로 렌더링하는 것이 가능해진다. 이는 리액트가 추구하는 트리 추상화 구조를 무너뜨린다.

- 구 Context API 사용 시 발생하는 경고

childContextTypes와 getChildContext를 사용하는 구 리액트 Context API를 사용하면 엄격 모드에서는 에러가 발생한다.

- 예상치 못한 부작용(side-effects) 검사

엄격 모드 내부에서는 다음 내용을 의도적으로 이중으로 호출된다.

1. 클래스 컴포넌트의 constructor, render, shouldComponentUpdate, getDerivedStateFromProps

2. 클래스 컴포넌트의 setState의 첫번째 인수

3. 함수 컴포넌트의 body

4. useState, useMemo, useReducer에 전달되는 함수

_왜 2번씩 실행될까?_

함수형 프로그래밍의 원칙에 따라 리액트의 모든 컴포넌트는 항상 순수하다고 가정하기 때문이고, 엄격 모드에서는 앞에서 언급한 내용이 실제로 지켜지는지, 즉 항상 순수한 결과물을 내고 있는지 개발자에게 확인시켜 주기 위해 두 번 실행되는 것이다.

리액트에서는 state, props, context가 변경되지 않으면 항상 JSX를 반환해야한다.

#### 리액트 18에서 추가된 엄격 모드

향후 리액트에서는 컴포넌트가 마운트 해제된 상태에서도 컴포넌트 내부의 상태값을 유지할 수 있는 기능을 제공할 예정이라고 밝혔다.

이러한 기능을 향후에 지원하기 위해 엄격 모드의 개발 모드에 새로운 기능을 도입했다. 컴포넌트가 최초에 마운트될 때 자동으로 모든 컴포넌트를 마운트 해제하고 두번째 마운트에서 이전 상태를 복원하게 된다. 이 기능은 오직 개발 모드에서만 제공된다.

### 10.2.6 Suspense 기능 강화

Suspense는 컴포넌트를 동적으로 가져올 수 있게 도와주는 기능이다.

- 리액트 18 버전 이전의 Suspense에는 몇 가지 문제점이 존재한다.

  - 기존의 Suspense는 컴포넌트가 아직 보이기도 전에 useEffect가 실행되는 문제가 존재한다.

    <Suspense> 컴포넌트로 감싼 컴포넌트 내부에 <Sibilings /> 컴포넌트 내부에 useEffect나 useLayoutEffect 등은 Suspense의 fallback이 실행중임에도 실행되는 문제가 있었다.

  - Suspense는 서버에서 사용할 수 없다.

- 리액트 18 버전에서는 Suspense가 정식으로 지원된다.

  - 앞에서 언급했던 아직 마운트되기 직전임에도 effect가 빠르게 실행되는 문제가 수정되었다. 현재는 화면이 렌더링될 때 effect가 실행된다.

  - Suspense로 인해 컴포넌트가 보이거나 사라질 때도 effect가 정상적으로 실행된다.

  - Suspense를 이제 서버에서도 실행이 가능하다. 앞의 예제와 같이 CustomSuspense를 구현하지 않더라도 정상적으로 Suspense를 사용할 수 있다.

  - Suspense 내에 스로틀링이 추가됐다. 화면이 너무 자주 업데이트되어 시각적으로 방해받는 것을 방지하기 위해 리액트는 다음 렌더링을 보여주기 전에 잠시 대기한다. 중첩된 Suspense의 fallback이 있다면 자동으로 스로틀되어 최대한 자연스럽게 보여주기 위해 노력한다.

### 10.2.7 인터넷 익스플로러 지원 중단에 따른 추가 폴리필 필요

리액트는 리액트를 사용하는 코드에서 최신 자바스크립트 기능을 사용할 수 있다는 가정하에 배포되었다.

- Promise

- Symbol

- Object.assign

위의 세가지 기능을 지원하지 않는다면 폴리필을 반드시 추가해야 한다.
