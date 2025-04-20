## useState

상태를 정의하고 상태를 관리할 수 있게 해주는 훅이다. 실제로는 useReducer를 이용해 구현되어 있다.

```jsx
import { useState } from 'react'

const [state, useState] = useState(initialState)
```


> 👉 **게으른 초기화란?**<br/>
> useState에 원시값이 아닌 특정한 값을 return하는 함수를 넣어주면 state가 처음 만들어질 때만 사용되고 이후 리렌더링이 발생할 때 함수의 실행이 무시된다. localStorage, sessionStorage에 접근하거나 map, filter, find, 혹은 초기값 계산을 위해 함수를 호출할 때 게으른 초기화를 사용하는 게 좋다고 한다.

> ❓ p.193 질문<br/>
> index를 클로저로 가둬두는 것 같지 않은데 useState 훅별로 index 값이 유지가 되는 것인가?

```jsx
// const currentIndex = index; 이런 줄이 없어도 되나?
const currentState = global.states[index] || initialState;
```

## useEffect

useEffect로 클래스 컴포넌트의 생명주기와 비슷한 작동을 구현할 수 있다고 하지만 사실 useEffect는 단순히 렌더링을할 때마다 의존성 배열 내의 값을 얕은 비교했을 때 변경 사항이 발생하면 부수 효과를 실행하는 함수다.

```jsx
useEffect(callback, [의존성 배열]);
```

### 클린업 함수란?

흔하게들 useEffect에서 반환되는 함수는 클래스 컴포넌트처럼 컴포넌트가 언마운트될 때 실행하는 함수로 알고 있는 경우가 많다. 

그러나 클린업 함수는 단순히 존재하면 함수 컴포넌트가 리렌더링되었을 때 콜백을 실행하기 전에 이전 클린업 함수를 실행한 뒤 콜백을 실행한다. useEffect도 결국 의존성에 따라 반복적으로 수행되기 때문에 이벤트 핸들러를 추가해두면 메모리 어쩌고 하기 위해 다음 콜백이 실행되기 전에 이벤트 헨들러를 삭제하는 것이다.

```jsx
useEffect(() => {
	function addMouseEvent() {}
	window.addEventListener('click', addMouseEvent);
	
	return () => {
		window.removeEventListener('click', addMouseEvent)
	}
}, []);
```

### 의존성 배열

의존성 배열을 작성하지 않으면 매 렌더링마다 실행된다. 그러면 useEffect를 왜 쓰는가 싶지만 그래도 useEffect를 써야하는 경우는 서버사이드 렌더링을 할 때 useEffect는 클라이언트 사이드에서 실행되는 것을 보장하기 때문이다. 그래서 useEffect내부에서 window객체에 접근해도 되고, useEffect는 렌더링 이후에 실행되기 때문에 무거운 연산 작업의 경우 useEffect를 사용하는게 컴포넌트 반환을 지연시키지 않을 것이다.

> ❓ p.203 질문
> index++ 왜 더하는거지?

컴포넌트 첫 렌더링 때 실행할 코드를 useEffect 콜백으로 넘기고 의존성 배열에 빈 배열을 넘길 수도 있지만 이는 지양하는 것이 좋다. 왜냐하면 useEffect의 흐름과 컴포넌트 props나 기타 속성의 흐름이 맞지 않을 수 있기 때문이다. 따라서 useEffect의 부수 효과가 정말 컴포넌트 상태와 별개로 동작하는지 검토할 필요가 있다.

- 또한 useEffect함수에 기명 함수를 넘기면 코드 가독성을 높일 수 있다.
- 그리고 useEffect는 간결하게 유지하는 것이 좋다. 의존성 배열에 여러 값을 넣을 바엔 여러 useEffect를 만드는 것이 좋다.
- 그리고 useEffect 내부에서만 실행될 함수는 외부 함수로 만들지 말고 훅 내부에서 선언해서 사용하는 게 가독성 측면에서 좋다.

또한 useEffect는 부수 효과를 일으키며 리렌더링을 일으키기 때문에 비동기 함수를 콜백으로 넣을 수 없다. 왜냐하면 자칫하단 useEffect로 리렌더링을 요청한 게 이전 요청이 더 오래걸리면 다음 요청에 따른 리렌더링이 일어나도 이전 요청에 따라 리렌더링 되면서 경쟁 상태가 되기 때문이다.

## useMemo / useCallback

useMemo는 비용이 큰 연산을 저장해두고 저장된 값을 반환하는 훅이다. useCallback은 인수로 넘겨받은 콜백을 기억한다. 사실 둘다 값을 저장해둔다는 동일한 기능을 한다.

```jsx
import { useMemo } from 'react'

const memoizedValue = useMemo(() => expensiveComputation(a, b), [a, b])
```

## useRef

useRef는 state와 마찬가지로 컴포넌트 내부에 값을 저장해둘 수 있지만 값이 변경해도 렌더링을 발생시키지 않는다. 주로 dom 요소에 접근할 때 사용한다. 내부적으로는 useMemo로 구현되어 있을 수 있다.

## useContext

props drilling을 방지할 수 있는 훅이다. 상위 컴포넌트에서 만들어진 Context를 함수 컴포넌트에서 사용할 수 있도록 만들어진 훅이다. useContext를 사용하면 상위 컴포넌트에서 선언된 context.provider에서 제공한 값을 사용할 수 있게 된다. 만약 provider가 여러 개라면 가장 가까운 provider 값을 가져오게 된다.

> 👉 useContext vs 상태 관리 라이브러리?
> 상태 관리 라이브러리는 상태 변화를 최적화할 수 있어야하고 어떠한 상태를 기반으로 다른 상태를 만들어낼 수 있어야 한다.

## useReducer

useReducer는 useReact의 심화버전이다. 복잡한 state를 사전에 정의하고 dispatcher로만 수정할 수 있게 하는 것이다.

```jsx
const [state, dispatcher] = useReducer(reducer, initialState, init)
```
