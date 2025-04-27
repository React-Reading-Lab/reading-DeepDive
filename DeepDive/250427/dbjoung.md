# 3-1. 리액트의 모든 훅 파헤치기

함수 컴포넌트가 상태를 사용하거나 클래스 컴포넌트의 생명주기 메소드를 대체하는 등 다양한 작업을 하기 위해 `훅(hook)` 이라는 것이 추가 됐다.

훅은 함수 컴포넌트에서 가장 중요한 개념이다.

## useState

리액트에서 훅을 언급할 때 가장 먼저 떠올리는 훅이다. `useState`는 함수 컴포넌트 내부에서 상태를 정의하고, 이 상태를 관리할 수 있게 해준다.

`useState`의 기본적인 사용법은 다음과 같다.

```jsx
import { useState } from "react";

const [state, setState] = useState(initialState);
```

- 인수로 사용할 `state` 의 초깃값을 넘겨준다.
- 아무런 값을 넘겨주지 않으면 초깃값은 `undifined` 가 된다.
- `useState`의 반환값은 배열이다.
  - 첫번째 원소 : state 값 자체
  - 두번째 원소 : state 값을 변경할 수 있는 함수

`useState` 의 내부 구조를 상상해보자. 직관적으로 떠오르는 구조는 아래와 같다. 하지만 아래 구조로는 useState가 의도한 동작으로 실행되지 못한다. 컴포넌트에서 setState를 호출한다 해도, useState가 변경된 새로운 값을 반환하지 못하기 때문이다.

```jsx
function useState(initialValue) {
  let internalState = initialValue;

  function setState(newValue) {
    internalState = newValue;
  }

  return [internalState, setState];
}
```

이를 해결하기 위해서는 클로저를 사용해야 한다. 다음은 useState의 작동방식을 대략적으로 흉내낸 코드다. (**실제 React 코드가 아니다**.)

- MyReact라고 불리는 클로저 내부에 `useState`와 관련된 정보를 저장해두고, 이를 필요할 때마다 꺼내놓는 형식으로 구성돼있다.

```jsx
const MyReact = (function () {
  const global = {};
  let index = 0;

  function useState(initialState) {
    if (!global.states) {
      // 애플리케이션 전체의 states 배열을 초기화
      // 최초 접근이라면 빈 배열로 초기화
      global.states = [];
    }

    // states 정보를 조회해서 현재 상태값이 있는 지 확인하고,
    // 없다면 초깃값으로 설정.
    const currentState = global.states[index] || initialState;
    //states의 값을 위에서 조회한 현재 값으로 업데이트
    global.states[index] = currentState;

    // 즉시 실행 함수로 setter를 만든다.
    const setState = (function () {
      // 현재 index를 클로저로 가둬놔서 이후에도 계속해서 동일한
      // index에 접근할 수 있도록 한다.
      let currentIndex = index;
      return function (value) {
        global.states[currentIndex] = value;
        // 컴포넌트를 렌더링한다. 실제로 컴포넌트를 렌더링하는 코드는 생략했다.
      };
    })();

    // useState를 쓸 때마다 index를 하나씩 추가한다. 이 index는 setState에서 사용된다.
    // 즉, 하나의 state마다 index가 할당돼 있어 그 index가 배열의 값(global.states)을
    // 가리키고 필요할 때마다 그 값을 가져오게 한다.
    index = index + 1;

    return [currentState, setState];
  }

  // 실제 useState를 사용하는 컴포넌트.
  function Component() {
    const [value, setValue] = useState(0);
    // ...
  }
})();
```

실제 React 코드에서는 `useReducer`를 이용해 구현돼있다.

> [!Note] 실제 React에서 Hook은 어떻게 구성돼 있을까?
>
> - 리액트 깃허브에서 훅에 대한 구현체를 타고 올라가다 보면 `__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED` 라는 문구를 마주하게 된다.
> - 이 변수에는 `ReactSharedInternals`라 불리는 내부 객체가 저장돼 있는 것으로 추정된다.
> - 해당 변수는 외부 사용자가 사용하거나 참고하는 것을 권장하지 않기 때문에 정확한 구현을 알기 어렵다. 대신 이번 장에서 HOOK에 대한 예제는 `Preact` 구현을 기준으로 한다.
> - `Preact`는 리액트의 경량화 버전으로, 대부분의 리액트 API를 지원하며 가볍고 빠르다. 그리고 무엇보다 **모든 코드를 명확하게** 볼 수 있다.

### 게으른 초기화

`useState` 의 인수로는 특정한 값 대신 **함수**를 인수로 넣을 수 있다.

- 이렇게 `useState` 에 변수 대신 **함수**를 넘기는 것을 `게으른 초기화(lazy initialzation)`이라고 한다.
- 공식 문서에서 게으른 초기화는 `useState의 초기값이 복잡하거 나무거운 연산을 포함하고 있을 때 사용`하라고 돼 있다.
- 이 `게으른 초기화` 함수는 오로지 state가 처음 만들어질 때만 사용된다.
- 만약 이후에 리렌더링이 발생한다면 이 함수의 실행은 무시된다.
  - 리액트는 렌더링이 실행될 때마다 함수 컴포넌트의 함수가 다시 실행된다는 점을 명심해야 한다. 따라서 초기화 함수가 매 렌더링 때마다 실행된다면 매우 무거울 것이다.
  - 예를 들면 `Number.parseInt(window.localStorage.getItem(cacheKey))` 와 같이 어느정도 비용이 드는 연산을 useState의 인수로 넘겨준다면, 초깃값이 필요한 최초 렌더링과 초깃값이 있어 더이상 필요 없는 리렌더링 시에도 동일하게 계속 해당 값에 접근해서 낭비가 발생한다.
    - 왜냐? 리랜더링 = 컴포넌트 재실행이기 때문에, 매 랜더링 때마다 `Number.parseInt~` 가 실행될 것이기 때문이다. (초기 렌더링 이후부터는 어차피 무시될 값임에도.)
  - 따라서 이런 경우에는 함수 형태로 인수를 넘겨주는 편이 훨씬 경제적일 것이다. 초기값이 없다면 함수를 실행해 무거운 연산을 시도할 것이고, 이미 **초깃값이 존재한다면 함수 실행을 하지 않고 기존 값을 사용할 것이다.**
    ```jsx
    // 이렇게 화살표 함수로 넘겨주면, useState는 해당 함수를 첫 렌더링 시에만
    // 값을 계산하여 저장하고, 이후부터는 무시한다.
    const [value, setValue] = useState(() =>
      Number.parseInt(localStorage.getItem("cacheKey"))
    );
    ```
- 게으른 초기화를 사용하기에 적당한 시점은 대략 다음과 같다.
  - localStorage나 sessionStorage에 대한 접근
  - map, filter, find 같은 배열에 대한 접근
  - 초깃값 계산을 위해 함수 호출이 필요할 때 등

## useEffect

`useEffect`는 자주 쓰지만 사용하기 쉬운 훅은 아니다. 알려진 것처럼 **생명주기 메소드를 대체하기 위해 만들어진 훅도 아니다.**

useEffect의 정의를 정확하게 내리자면, **`useEffect`는 애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 매커니즘이다.**

그리고 이 부수 효과가 **언제** 일어나는 지 보다, **어떤 상태값과 함께 실행되는지** 살펴보는 것이 중요하다.

### useEffect의 형태

```jsx
function Component() {
	// ...
	useEffect(()=> {
		//do something
	}, [props, state])'

	// ...
}
```

- 첫 번째 인수 : **실행할 부수효과가 포함된 함수**를 전달
- 두 번째 인수 : **의존성 배열**을 전달
  - 어느 정도 길이를 가진 배열일 수도, 아무런 값이 없는 빈 배열일 수도 있다. 생략도 가능하다.
- 의존성 배열이 변경될 때마다 `useEffect`의 첫 번째 인수인 콜백을 실행한다는 것은 널리 알려져 있다. 그렇다면, useEffect는 어떻게 **의존성 배열이 변경된 것을 감지**할까?
  - 여기서도 역시나, 함수 컴포넌트는 매번 함수를 실행해 렌더링을 수행한다는 점이 중요하다.
  - 함수 컴포넌트는 렌더링 시마다 고유의 state와 props 값을 갖고 있다. useEffect는 JS의 프록시나 데이터 바인딩, 옵저버 같은 특별한 기능을 통해 갓의 변화를 관찰하는 것이 아닌, **`매 렌더링 시마다 의존성에 있는 값을 보면서 이 의존성의 값이 이전과 다른 게 하나라도 있으면 부수 효과를 실행하는 평범한 함수`**라 볼 수 있다.
- `useEffect`는 결국 state와 props의 변화 속에서 일어나는 렌더링 과정에서의 **부수 효과 함수**라 볼 수 있다.

### 클린업 함수의 목적

일반적으로 클린업 함수는 이벤트를 등록하고 지울 때 사용해야 한다고 알려져 있다.

아래 코드는 +버튼을 클릭했을 때 counter 상태를 1씩 증가시키는 App 컴포넌트인데, useEffect로 로그를 찍거나 이벤트 리스너를 붙이는 등의 부수 효과를 발생시키고 있다.

```jsx
import { useState, useEffect } from "react";

export default function App() {
  const [counter, setCounter] = useState(0);

  function handleClick() {
    setCounter((prev) => prev + 1);
  }

  useEffect(() => {
    function addMouseEvent() {
      console.log(counter);
    }

    window.addEventListener("click", addMouseEvent);

    // 클린업 함수
    return () => {
      console.log("클린업 함수 실행!", counter);
      window.removeEventListener("click", addMouseEvent);
    };
  }, [counter]);

  return (
    <>
      <h1>{counter}</h1>
      <button onClick={handleClick}>+</button>
    </>
  );
}
```

위 코드의 실행 결과는 다음과 같다. 이를 통해 클린업 함수는 **이전 counter 값, 즉 이전 state를 참조해 실행된다는 것**을 알 수 있다. 클린업 함수는 새로운 값과 함께 렌더링된 뒤에 실행되기 때문에 위와 같은 메시지가 나타난다.

여기서 중요한 것은, **클린업 함수는 비록 새로운 값을 기반으로 렌더링 된 뒤에 실행되지만, 이 변경된 값을 읽는 것이 아니라 함수가 정의됐을 당시에 선언됐던 이전 값을 보고 실행된다는 것**이다.

```jsx
클린업 함수 실행! 0
1

클린업 함수 실행! 1
2

클린업 함수 실행! 2
3

클린업 함수 실행! 3
4

//...
```

이 사실을 종합해보면 왜 useEffect에 이벤트를 추가했을 때 그 클린업 함수에서 지워야 하는 지 알 수 있다. 함수 컴포넌트의 `useEffect는 그 콜백이 실행될 때마다 이전의 클린업 함수가 존재한다면 그 클린업 함수를 실행한 뒤에 콜백을 실행`한다.

따라서 이벤트를 추가하기 전에 이전에 등록했던 이벤트 핸들러를 삭제하는 코드를 클린업 함수에 추가하는 것이다.

- 이렇게 함으로써 특정 핸들러가 무한히 추가되는 것을 방지할 수 있다.

이처럼 useEffect의 클린업 함수와 생명주기 메소드의 언마운트는 차이가 있다는 것을 볼 수 있다. **언마운트는 특정 컴포넌트가 DOM에서 사라진다는 것을 의미하는 클래스 컴포넌트 용어**다. 클린업 함수는 언마운트 함수라기보다는, 함수 컴포넌트가 리렌더링 됐을 때 **의존성 변화가 있었을 당시 이전의 값을 기준으로 실행되는, 말 그대로 이전 상태를 청소해 주는 개념으로 보는 것이 옳다.**

따라서 useEffect 관련 실행 순서를 정리하면 다음과 같다. (GPT 피셜)

1. 컴포넌트가 새로운 props/state로 **리렌더링 시작**
2. 새로운 **가상 DOM 생성** (렌더 함수 실행)
3. **가상 DOM diff & 실제 DOM 커밋** (DOM 업데이트 및 화면 반영 준비)
4. **`useLayoutEffect`의 이전 클린업 함수 실행**
5. **`useLayoutEffect`의 새로운 콜백 실행**
6. **브라우저가 화면을 repaint**
7. **`useEffect`의 이전 클린업 함수 실행**
8. **`useEffect`의 새로운 콜백 실행**

> [!Note] 아래와 같은 코드에서, 이벤트 리스너는 어떻게 계속 유지되는 걸까?
>
> ```jsx
> useEffect(() => {
>   const handler = () => {
>     setState((prev) => prev + 1);
>   };
>   window.addEventListener("click", handler);
>
>   return () => {
>     window.removeEventListener("click", handler);
>   };
> }, []);
> ```
>
> - 클린업 함수가 다음 렌더링 이후, 다음 effect 이후 실행되는거라면, 위 코드에서의 클린업은 첫 번째 리렌더링 때 작동해 이벤트 리스너가 삭제돼야 하는거 아닌가? 하지만 실제로 위 코드를 실행시켜보면, 매 렌더링 때마다 클릭 이벤트가 잘 실행된다.
> - 이는 책에 설명된 것 말고도 클린업 함수 호출 조건이 몇 가지 더 있기 때문이다.
>   1. 이전 클린업 함수는 컴포넌트 리렌더링 시 의존성 변화가 생겼을 때만 실행된다.
>      - 생각해보니 그럴 수 밖에 없다. 클린업 함수는 effect의 반환값이기 때문이다. 즉, effect가 실행되는 시점의 의존성 배열을 비교해 실행 여부를 판단하게 된다는 것.
>   2. 클린업 함수는 새로운 effect 실행 없이는 호출되지 않는다. (렌더링 후 실행할 effect가 있어야 그 전에 **이전 effect 부수효과를 청소한다는 의미**로 클린업이 발생하는 것.
>   3. 모든 클린업 함수는 컴포넌트 언마운트 시점에 전부 실행된다.
> - 따라서 위 코드의 실행 순서는 다음과 같다.
>
>   1. 사용자가 홈페이지에 접속해 컴포넌트가 **초기 마운트**된다.
>   2. 컴포넌트 함수가 실행되며, `useEffect`가 호출된다.
>   3. `useEffect`는 의존성 배열 `[]`을 보고, **최초 effect이므로 실행 대상에 등록한다.**
>   4. **가상 DOM diff가 끝나고, 변경사항이 실제 DOM에 커밋된다.**
>   5. 커밋 후, React는 수집했던 `useEffect` 콜백을 실행한다.
>   6. 콜백이 실행되며 `addEventListener("click", handler)`가 등록되고, 동시에 `return () => {...}` 클린업 함수도 **React 내부에 저장**된다.
>   7. 사용자 클릭 이벤트가 발생하면, 등록된 핸들러가 실행되어 `setState`가 호출된다.
>   8. 상태가 바뀌면서 컴포넌트가 **리렌더링**된다.
>   9. 컴포넌트 함수가 다시 실행되지만, `useEffect([])`는 의존성 배열이 동일하므로 **새 effect는 수집되지 않는다.**
>   10. React는 변경 사항을 다시 **커밋하고 DOM을 반영**한다.
>   11. 이번 리렌더에서는 `useEffect`가 다시 실행되지 않았기 때문에, 기존 클린업 함수도 **실행되지 않는다.**
>   12. 이후 사용자가 브라우저를 종료하거나 다른 페이지로 이동하여 컴포넌트가 **언마운트**되면, React는 내부에 저장되어 있던 클린업 함수를 **즉시 실행**한다.
>
>   → 즉, `removeEventListener("click", handler)`가 호출되어 리스너가 정리됨 ✅

### 의존성 배열

의존성 배열에는 빈 배열로 두거나, 배열 자체를 넘기지 않거나, 원하는 값을 추가할 수가 있다.

- 빈 배열로 두는 경우
  - useEffect는 비교할 의존성이 없다고 판단해 최초 렌더링 직후에 실행된 다음부터는 더 이상 실행되지 않는다.
- 배열 자체를 넘겨주지 않는다.

  - 의존성을 비교할 필요 없이 렌더링할 때마다 실행이 필요하다고 판단함. (따라서 매 렌더링 시마다 실행됨)
  - 보통 컴포넌트가 렌더링됐는 지 확인하기 위한 방법으로 사용됨.
  - 그런데, 여기서 의문이 한가지 든다. 의존성 배열이 없는 useEffect가 매 렌더링마다 실행된다면 그냥 useEffect 없이 써도 되는거 아닐까? 아래 예지의 1번 코드 처럼.

    ```jsx
    // 1번
    function Component() {
      console.log("렌더링 됨");
    }

    // 2번
    function Component() {
      useEffect(() => {
        console.log("렌더링 됨");
      });
    }
    ```

    - 위 두 코드는 명백하게 차이가 있다.
      - SSR 관점에서 `useEffect`는 클라이언트 사이드에서 실행되는 것을 보장해준다. useEffect 내부에서는 window 객체의 접근에 의존하는 코드를 사용해도 된다.
      - `useEffect`는 컴포넌트 렌더링의 부수 효과, 즉 컴포넌트의 렌더링이 완료된 이후에 실행된다. **반면 1번과 같이 함수 내부에서의 직접 실행은 컴포넌트가 렌더링되는 도중에 실행된**다. 따라서 2번과는 달리 SSR의 경우 서버에서도 실행된다.
      - 그리고 1번은 함수 컴포넌트의 반환을 지연시킨다. 즉, 무거운 작업일 경우 렌더링을 방해하므로 성능에 악영향을 미칠 수 있다. (생각해보면, 렌더링 과정 중 실행되므로 당연한 말이다.)

- 배열에 원하는 값을 추가한다.
  - 배열 속 값에 대해 이전 값과 비교하여, 변경 사항이 있으면 실행한다.
  - 의존성 배열의 (얕은)비교는 컴포넌트 렌더링 단계에서 실행된다. (가상 DOM 생성 과정)

### useEffect의 구현

useState와 마찬가지로 리액트 코드를 직접 구현할 수는 없지만 대략적인 모습은 다음과 같이 상상해볼 수 있다. 핵심은 의존성 배열의 이전 값과 현재 값의 얕은 비교다.

```jsx
const MyReact = (function () {
  const global = {};
  let index = 0;
  function useEffect(callback, dependencies) {
    const hooks = global.hooks;

    // 이전 훅 정보가 있는 지 확인
    let previousDependencies = hooks[index];

    //변경됐는지 확인
    //이전 값이 있다면 이전 값을 얕은 비교해 변경이 일어났는 지 확인
    //이전 값이 없다면 최초 실행이므로 변겨잉 일어난 것으로 간주해 실행 유도
    let isDependenciesChanged = previousDependencies
      ? dependencies.some(
          (value, idx) => !Object.is(value, previousDependencies[idx])
        )
      : true;
  }

  //변경이 일어났다면 첫 번째 인수인 콜백 함수를 실행
  if (isDependenciesChanged) {
    callback();

    // 다음 훅이 일어날 때를 대비하기 위해 index 추가
    index++;

    // 현재 의존성을 훅에 다시 저장
    hooks[index] = dependencies;
  }

  return { useEffect };
})();
```

React는 기본적으로 얕은 비교를 할 때 `Object.is`를 기반으로 하는 얕은 비교를 수행한다. 이전 의존성 배열과 현재 의존성 배열의 값에 하나라도 변경 사항이 있으면 callback으로 선언한 부수 효과를 실행한다. 이것이 useEffect의 본질이다.

### useEffect를 사용할 때 주의할 점

- `eslint-disable-line react-hooks/exhaustive-deps` 주석은 최대한 자제하라
  - 해당 ESLint 룰은 useEffect 인수 내부에서 사용하는 값 중 의존성 배열에 포함돼 있지 않는 값이 있을 때 경고를 발생시킨다.
  - 대부분의 경우에는 의도치 못한 버그를 만들 가능성이 큰 코드다. 이 코드를 사용하는 대부분의 예제가 빈 배열 []을 의존성으로 할 때, 즉 컴포넌트를 마운트하는 시점에만 무언가를 하고 싶다라는 의도로 작성하곤 한다. 하지만 이는 클래스 컴포넌트의 componentDidMount에 기반한 접근법으로, 가급적 사용해선 안 된다.
  - useEffect는 반드시 의존성 배열로 전달한 값의 변경에 의해 실행돼야 하는 훅이다. 하지만 의존성 배열을 넘기지 않은 채 콜백 함수 내부에서 특정 값을 사용한다는 것은, 이 부수 효과가 실제로 관찰해서 실행돼야 하는 값과는 별개로 작동한다는 것을 의미한다.
  - 즉, 컴포넌트의 state, props와 같은 어떤 값의 변경과 useEffect의 부수 효과가 별개로 작동하게 된다는 것이다. useEffect에서 사용한 콜백 함수의 실행과 내부에서 사용한 값의 실제 변경 사이에 연결 고리가 끊어져 있는 것이다.
  - 따라서 useEffect에 빈 배열을 넘기기 전에는 정말로 useEffect의 부수 효과가 컴포넌트의 상태와 별개로 작동해야만 하는 지, 혹은 여기서 호출하는 게 최선인지 한 번 더 검토해 봐야 한다.
  - 빈 배열이 아닐 때도 마찬가지다. 만약 특정 값을 사용하지만 해당 값의 변경 시점을 피할 목적이라면, 메모이제이션을 적절이 활용해 해당 값의 변화를 막거나 적당한 실행 위치를 다시 고민해보는 게 좋다.
- useEffect의 첫 번째 인수에 함수명을 부여하라
  - useEffect를 사용하는 많은 코드에서 useEffect의 첫 번째 인수로 익명 함수를 넘겨준다. 이는 리액트 공식 문서도 마찬가지다.
  - useEffect의 수가 적거나 복잡성이 낮다면 이런 구조가 상관 없다. 하지만 useEffect의 수가 많고 복잡해질 수록 무슨 일을 수행하는 useEffect 코드인 지 파악하기 어려워진다.
  ```jsx
  useEffect(
    function logActiveUser() {
      logging(user.id);
    },
    [user.id]
  );
  ```
- 거대한 useEffect를 만들지 마라
  - useEffect는 의존성 배열을 바탕으로 렌더링 시 의존성이 변경될 때마다 부수 효과를 실행한다. 이 부수 효과의 크기가 클 수록 애플리케이션 성능에 악영향을 준다.
  - 물론 useEffect가 컴포넌트 렌더링 이후 실행되기 때문에 렌더링 작업에는 영향을 적게 미치겠지만, 여전히 JS 실행 성능에 영향을 미친다는 사실은 변함 없다.
  - 가능한 한 useEffect를 가볍게 쓰려고 노력해야하며, 부득이하게 큰 useEffect를 만들어야 한다면 여러개의 useEffect로 분리하는 것이 좋다.
  - 만약 불가피하게 의존성 배열에 여러개의 값을 넣어야 하는 경우, `useCallback` 과 `useMemo` 등으로 사전에 정제한 내용들만 useEffect에 담아두는 것이 좋다. 이렇게 하면 언제 useEffect가 실행되는지 좀 더 명확하게 알 수 있다.

### 불필요한 외부 함수를 만들지 마라

- useEffect가 실행하는 콜백 또한 불필요하게 존재해서는 안된다.
- useEffect 내에서 사용할 부수 효과라면 내부에서 만들어 정의해 사용하는 편이 훨씬 도움 된다.

> **[!Note] 왜 useEffect** 콜백 인수로 비동기 함수를 바로 넣을 수 없을까?
>
> 만약 useEffect 인수로 비동기 함수가 사용 가능하다면, 함수의 응답 속도에 따라 결과가 이상하게 나올 수 있다. 극단적 예로 이전 state 기반 응답에 10초, 이후 바뀐 state 기반 응답에 1초가 걸린다면, 이전 응답 결과로 state가 반영될 수도 있는 것이다.
>
> - 이러한 문제를 useEffect의 `경쟁 상태(race condition)`이라고 한다.
> - 하지만 어디까지나 useEffect의 인수로 비동기가 안된다는 것이지, 비동기 함수 실행 자체가 불가능한 것은 아니다. useEffect 내부에서 비동기 함수를 선언해 실행하거나 즉시 실행 비동기 함수를 만들어서 사용하는 것은 가능하다.
> - 다만, 비동기 함수가 내부에 존재하게 되면 useEffect 내부에서 비동기 함수가 생성되고 실행되는 것을 반복하므로, 클린업 함수에서 이전 비동기 함수에 대한 처리를 추가해주는 것이 좋다.
>   - fetch의 경우 abortController 등으로 이전 요청을 취소.
>
> 즉, 비동기 useEffect는 state의 경쟁 상태를 야기할 수 있고, cleanup 함수의 실행 순서도 보장할 수 없기 때문에 막혀있다고 보면 된다.

## useLayoutEffect

공식 문서에 따르면 `useLayoutEffect` 를 다음과 같이 정의하고 있다.

- 이 함수의 시그니처는 useEffect와 동일하나, 모든 DOM의 변경 후에 동기적으로 발생한다.
  - 이 함수의 시그니처는 useEffect와 동일 : 두 훅의 형태나 사용 예제가 동일함을 뜻함
  - DOM의 변경 후 동기적 : 여기서 말하는 `DOM 변경` 은 컴포넌트 렌더링을 의미한다.
- 즉, 실행 순서는 다음과 같다.
  1. 리액트가 DOM을 업데이트
  2. useLayoutEffect의 콜백을 실행
  3. 브라우저에 변경사항을 반영 (브라우저 렌더링)
  4. useEffect의 콜백을 실행
- `useLayoutEffect`는 **DOM은 계산됐지만 이것이 화면에 반영되기 전에 하고 싶은 작업이 있을 때** 사용하는 것이 좋다.
  - DOM요소를 기반으로 한 애니메이션, 스크롤 위치 제어, 화면 크기 조절 등
- 더 자세한 내용은 위 `useEffect` 와 함께 정리하였다.

## useMemo

`useMemo`는 비용이 큰 연산에 대한 결과를 저장(메모이제이션)해 두고, 이 저장된 값을 반환하는 훅이다. 흔히 리액트에서 최적화를 떠올릴 때 가장 먼저 언급되는 훅이기도 하다.

- 첫 번째 인수 : 어떠한 값을 반환하는 생성 함수
- 두 번째 인수 : 해당 함수가 의존하는 값의 배열

useMemo는 렌더링 발생 시 의존성 배열의 값이 변경되지 않았으면 이전에 기억해 둔 해당 값을 반환하고, 의존성 배열 값이 변경됐다면 첫 번째 인수의 함수를 실행한 후 그 값을 반환한다. (그리고 그 값을 기억한다.)

이러한 메모이제이션은 단순히 값 뿐만 아니라 컴포넌트도 가능하다. 아래 코드는 useMemo를 사용해 컴포넌트를 메모이제이션 한 예다.

```jsx
function ExpensivComponent({ value }) {
  useEffect(() => {
    console.log("rendering!");
  });

  return <span>{value + 1000}</span>;
}

function App() {
  const [value, setValue] = useState(10);
  const [, triggerRendering] = useState(false);

  // 컴포넌트의 props를 기준으로 컴포넌트 자체를 메모이제이션 한다.
  const MemoizedComponent = useMemo(
    () => <ExpensiveComponent value={value} />,
    [value]
  );

  function handleChange(e) {
    setValue(Number(e.target.value));
  }

  function handleClick(e) {
    triggerRendering((prev) => !prev);
  }

  return (
    <>
      <input value={value} onChange={handleChange} />
      <button onClick={handleClick}>렌더링 발생!</button>
      {MemoizedComponent}
    </>
  );
}
```

- 위 코드에서는 useMemo로 컴포넌트를 감쌌다. **물론 `React.memo`를 쓰는 것이 더 현명할 것**이다.

## useCallback

useMemo가 값을 기억한다면, `useCallback`은 인수로 넘겨받은 콜백 자체를 기억한다. 쉽게 말해 `useCallback`은 특정 함수를 새로 만들지 않고 다시 재사용한다는 의미다.

아래는 memo를 사용했음에도 전체 자식 컴포넌트가 리렌더링되는 예제다. memo를 사용해서 ChildComponent를 감쌌지만, 의도한 것과 다르게 두 ChildComponent 중 어느 쪽의 버튼을 누르더라도 두 컴포넌트 전부 리렌더링된다.

```jsx
const ChildComponent = memo(({ name, value, onChange }) => {
  useEffect(() => {
    console.log("rendering!", name);
  });

  return (
    <>
      <h1>
        {name} {value ? "켜짐" : "꺼짐"}
      </h1>
      <button onClick={onChange}>toggle</button>
    </>
  );
});

function App() {
  const [status1, setStatus1] = useState(false);
  const [status2, setStatus2] = useState(false);

  const toggle1 = () => {
    setStatus1(!status1);
  };

  const toggle2 = () => {
    setStatus2(!status2);
  };
  return (
    <>
      <ChildComponent name="1" value={status1} onChange={toggle1} />
      <ChildComponent name="2" value={status2} onChange={toggle2} />
    </>
  );
}

export default App;
```

이유는 `setStatus1, 2` 발생으로 App 컴포넌트라 리렌더링되고, 그때마다 onChange로 넘기는 함수가 재생성되고 있기 때문이다. 크롬 개발자도구 메모리 탭에서도 해당 내용을 확인할 수 있다.

이런 상황에서 함수의 메모이제이션을 위해 사용하는 것이 `useCallback`이다.

- 첫 번째 인수 : 함수
- 두 번째 인수 : 의존성 배열

위 두 가지 인수를 넘겨주면, `useMemo`와 마찬가지로 의존성 배열이 변경되지 않는 한 함수를 재생성하지 않는다. 위 코드를 useCallback을 사용해 아래 처럼 바꿔주면, 두 ChildComponent는 자신의 버튼이 눌렸을 때만 리렌더링되게 된다.

```tsx
const ChildComponent = memo(({ name, value, onChange }) => {
  useEffect(() => {
    console.log("rendering!", name);
  });

  return (
    <>
      <h1>
        {name} {value ? "켜짐" : "꺼짐"}
      </h1>
      <button onClick={onChange}>toggle</button>
    </>
  );
});

function App() {
  const [status1, setStatus1] = useState(false);
  const [status2, setStatus2] = useState(false);

  const toggle1 = useCallback(
    function toggleA() {
      setStatus1(!status1);
    },
    [status1]
  );

  console.log(toggle1);

  const toggle2 = useCallback(
    function toggleB() {
      setStatus2(!status2);
    },
    [status2]
  );
  return (
    <>
      <ChildComponent name="1" value={status1} onChange={toggle1} />
      <ChildComponent name="2" value={status2} onChange={toggle2} />
    </>
  );
}
```

실행 순서는 다음과 같다.

1. 초기 렌더링

- `status1 === false`, `status2 === false`
- `useCallback(() => setStatus1(!status1), [status1])`
  - `status1 === false` → 새로운 `toggle1` 함수 생성됨 (call it `toggleA_v1`)
- `toggle1 = toggleA_v1`
- `ChildComponent 1`, `ChildComponent 2` 최초 렌더링 → 모두 `console.log("rendering!", name)` 출력됨

2. 첫 번째 버튼 클릭

- `toggle1()` 실행 → 내부에서 `setStatus1(!status1)`
  → `!false === true` → `setStatus1(true)`
- 상태 업데이트 "예약"
- React는 `App()` 컴포넌트의 리렌더링을 예약함

3. 리렌더링 시작 (setState 적용됨)

- `App()` 다시 실행됨
- 이번 렌더링에서는 `status1 === true`, `status2 === false`
- `useCallback(() => setStatus1(!status1), [status1])`
  - `status1 === true` → 이전 deps `[false]` 와 다름
    → 새로운 `toggle1` 함수 생성됨 (call it `toggleA_v2`)
- `toggle1 = toggleA_v2`

4. memo 비교

- 첫 번째 컴포넌트는 toggle1 의 메소드가 바뀌었기에 리렌더링되고, 두 번째 컴포넌트는 값이 바뀐 props가 없어 유지된다.

### useCallback 대신 useMemo 쓰기

기본적으로 `useCallback` 은 `useMemo`를 사용해 구현할 수 있다. 이는 Preact에서도, 리액트 공식 문서에서도 확인해 볼 수 있는 사실이다.

- useMemo와 useCallback의 유일한 차이는 메모이제이션을 하는 대상이 벼수냐 함수냐일 뿐이다. JS에서는 함수 또한 값으로 표현될 수 있으므로 이러한 코드는 매우 자연스럽다.
- 다만, `useMemo`로 `useCallback`을 구현하는 경우 다음과 같이 불필요하게 코드가 길어지고 혼동을 야기할 수 있다. (리액트에서 두 함수를 나눠놓은 이유로 추측된다.)

  ```jsx
  import { useState, useCallback, useMemo } from "react";

  export default function App() {
    const [counter, setCounter] = useState(0);

    /**
     * 아래 두 함수의 작동은 동일하다.
     */
    const handleClick1 = useCallback(() => {
      setCounter((prev) => prev + 1);
    }, []);

    const handleClick2 = useMemo(() => {
      return () => setCounter((prev) => prev + 1);
    }, []);
  }
  ```

## useRef

useRef는 useState와 동일하게 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장한다는 공통점이 있다. 하지만 useState와 구별되는 큰 차이점 두 가지가 있다.

1. useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있다.
2. useRef는 그 값이 변하더라도 렌더링을 발생시키지 않는다.

### useRef의 필요성

렌덜이에 영항을 미치지 않는 고정된 값을 관리하기 위해 useRef를 사용한다면, useRef를 사용하지 않고 그냥 함수 외부에서 값을 선언해 관리하는 것도 동일한 기능을 수행할 수 있다. 하지만 이 경우, 몇 가지 단점이 있다.

1. 컴포넌트가 실행되어 렌더링되지 않았음에도 value라는 값이 기본적으로 존재하게 된다. 이는 메모리에 불필요한 값을 갖게 하는 악영향이다.
2. 만약 Component가 여러번 생성된다면, 각 컴포넌트에서 가리키는 값이 전부 value로 동일할 것이다. (일일이 value를 매핑해주지 않는 이상) 대부분의 경우에는 컴포넌트 인스턴스 하나 당 하나의 값을 필요로 하는 게 일반적이다.

useRef는 위 두 가지 불편함을 극복할 수 있는 리액트식 접근법이다. **컴포넌트가 렌더링될 때만 생성되며, 컴포넌트 인스턴스가 여러 개라도 각각 별개의 값을 바라본다.**

### useRef 사용 예 & 구현

- useRef의 가장 일반적인 사용 예는 아래와 같이 DOM에 접근하고 싶을 때일 것이다.

  ```jsx
  function RefComponent() {
    const inputRef = useRef();

    // 이때는 미처 렌더링이 실행되기 전(반환되기 전)이므로 undifined를 반환한다.
    console.log(inputRef.current); //undifined

    useEffect(() => {
      console.log(inputRef.current); // <input type="text" />
    }, [inputRef]);

    return <input ref={inputRef} type="text" />;
  }
  ```

  - `useRef`는 최초에 넘겨받은 기본 값을 가지고 있다.
  - useRef가 선언된 당시에는 아직 컴포넌트가 렌더링되기 전이라 return으로 컴포넌트의 DOM이 반환되기 전이므로 undifined다.

- `useRef`를 사용할 수 있는 유용한 경우는 또 있다.

  - useState의 이전 값을 저장하는 usePrevious() 같은 훅을 구현할 때다.

    ```jsx
    function usePrevious(value) {
      const ref = useRef();
      useEffect(() => {
        ref.current = value;
      }, [value]);
      return ref.current;
    }

    function SomeComponent() {
      const [counter, setCounter] = useState(0);
      const previousCounter = usePrevious(counter);

      function handleClick() {
        setCounter((prev) => prev + 1);
      }

      // 0 (undifined)
      // 1 0
      // 2 1
      // 3 2
      return (
        <button onClick={handleClick}>
          {counter} {previousCounter}
        </button>
      );
    }
    ```

- 그렇다면 useRef는 어떻게 구현돼 있을까? 리액트에서의 구현은 다르겠지만, Preact에서 구현에 대한 힌트를 얻을 수 있다. 의외로 매우 간단하다.
  ```jsx
  export function useRef() {
    currentHook = 5;
    return useMemo(() => ({ current: initialValue }), []);
  }
  ```
  - 값이 변경돼도 렌더링되면 안된다는 점, 실제 값은 { current : value } 와 같은 객체 형태로 있다는 점을 떠올려보자.
  - 자바스크립트의 특징, 객체의 값을 변경해도 객체를 가리키는 주소가 변경되지 않는다는 것을 떠올리면 useMemo로 useRef를 구현할 수 있다.

## useContext

### Context란?

리액트는 프롭스 드릴링이 기본적이지만 전달 거리가 멀어질 수록 코드가 복잡해진다. 이러한 props 내려주기를 극복하기 위해 등장한 개념이 바로 컨텍스트(Context)다.

컨텍스트를 사용하면 명시적인 props 전달 없이도 선언한 하위 컴포넌트 모드에서 자유롭게 원하는 값을 사용할 수 있다.

### Context를 함수 컴포넌트에서 사용할 수 있게 해주는 useContext 훅

useContext 사용 예시는 다음과 같다.

```jsx
const Context = (createContext < { hello: string }) | (undifined > undifined);

function ParentComponent() {
  return (
    <>
      <Context.Provider value={{ hello: "react" }}>
        <Context.Provider value={{ hello: "javascript" }}>
          <ChildComponent />
        </Context.Provider>
      </Context.Provider>
    </>
  );
}

function ChildComponent() {
  const value = useContext(Context);

  //react가 아닌 javascript가 반환된다.
  return <>{value ? value.hello : ""}</>;
}
```

`useContext`는 상위 컴포넌트에서 만들어진 Context를 함수 컴포넌트에서 사용할 수 있도록 만들어졌다. `useContext` 를 사용하면 상위 컴포넌트 어딘가에서 선언된 `<Context.Provider />`에서 제공한 값을 사용할 수 있게 된다.

만약 여러개의 Provider가 있다면 가장 가까운 Provider의 값을 가져오게 된다. 위 예제에서는 더 가까운 ‘javascript’가 출력될 것이다.

하지만 컴포넌트가 복잡해질 수록 컨텍스트를 활용하는 것도 만만치 않아진다. `useContext` 로 원하는 값을 얻으려 했지만 정작 컴포넌트가 실행될 때 이 컨텍스트가 존재하지 않아 예상치 못하게 에러가 발생하는 경우도 종종 있다.

- 이러한 에러를 방지하려면 `useContext` 내부에 해당 컨텍스트가 존재하는 환경인지, 즉 컨텍스트가 한 번이라도 초기화되어 값을 내려주고 있는 지 확인해보면 된다.

  ```jsx
  const MyContext =
    (createContext < { hello: string }) | (undefined > undefined);

  function ContextProvider({
    children,
    text,
  }: PropsWithChildren<{ text: string }>) {
    return (
      <MyContext.Provider value={{ hello: text }}>
        {children}
      </MyContext.Provider>
    );
  }

  function useMyContext() {
    const context = useContext(MyContext);
    if (context == undefined) {
      throw new Error(
        "useMyContext는 ContextProvider 내부에서만 사용할 수 있습니다."
      );
    }

    return context;
  }

  function ChildComponent() {
    const { hello } = useMyContext();

    return <>{hello}</>;
  }

  function App() {
    return (
      <>
        <ContextProvider text="hello">
          <ChildComponent />
        </ContextProvider>
      </>
    );
  }
  ```

  - 다수의 Provider와 useContext를 사용할 때, 특히 타입스크립트를 사용하고 있다면 위와 같이 별도 함수로 감싸서 사용하는게 좋다. **타입 추론에도 유용**하고, 상위에 Provider가 없는 경우에도 **사전에 쉽게 에러를 찾을 수 있다**.

### useContext를 사용할 때 주의할 점

1. `useContext` 를 함수 컴포넌트 내부에서 사용할 때는, 항상 **컴포넌트 재활용이 어려워진다는 점**을 염두에 둬야 한다. `useContext`가 선언돼 있으면 **Provider에 의존성을 가지고 있는 셈이 되기 때문**이다.
   - 이러한 상황을 방지하기 위해 모든 컨텍스트를 최상위 루트 컨텍스트에 넣으면 어떨까, 하는 아이디어를 떠올릴 수 있지만, 컨텍스트가 많을 수록 루트가 많은 컨텍스트에 둘러쌓일 것이고 해당 props를 다수의 컴포넌트에서 쓸ㅇ 수 있게 해야 하므로 리소스 낭비도 있다.
   - 따라서, **컨텍스트가 미치는 범위는 필요한 환경에서 최대한 좁게 만들어야 한다.**
2. 일부 리액트 개발자들은 컨텍스트와 `useContext`를 상태관리를 위한 리액트 API로 오해하고 있다. 하지만 컨텍스트는 **상태를 주입해주는 API**일 뿐이지 상태관리 API가 아니다. 상태관리 라이브러리이기 위해서는 다음 두 조건을 만족시켜야만 한다.

   - 어떠한 상태를 기반으로 다른 상태를 만들어낼 수 있어야 한다.
   - 필요에 따라 이러한 상태 변화를 최적화할 수 있어야 한다.

   컨텍스트는 이 둘 중 어느것도 하지 못한다. 그저 props를 선언된 곳에서 하위 컴포넌트로 전달해줄 뿐이다. **필요에 따라 상태 변화를 최적화할 수 있어야 한다**가 어떤 의미인 지는 아래 코드로 설명할 수 있다.

   ```jsx
   const MyContext =
     (createContext < { hello: string }) | (undefined > undefined);

   function ContextProvider({
     children,
     text,
   }: PropsWithChildren<{ text: string }>) {
     return (
       <MyContext.Provider value={{ hello: text }}>
         {children}
       </MyContext.Provider>
     );
   }

   function useMyContext() {
     const context = useContext(MyContext);

     if (context == undefined) {
       throw new Error(
         "useMyContext는 ContextProvider 내부에서만 사용할 수 있습니다."
       );
     }

     return context;
   }

   function GrandChildComponent() {
     const { hello } = useMyContext();
     useEffect(() => {
       console.log("렌더링 GrandChildComponent");
     });

     return <h3>{hello}</h3>;
   }

   function ChildComponent() {
     useEffect(() => {
       console.log("렌더링 ChildComponent");
     });

     return <GrandChildComponent />;
   }

   function App() {
     const [text, setText] = useState("");

     function handleChange(e: ChangeEvent<HTMLInputElement>) {
       setText(e.target.value);
     }

     useEffect(() => {
       console.log("렌더링 ParentComponent");
     });

     return (
       <>
         <ContextProvider text="react">
           <input value={text} onChange={handleChange} />
           <ChildComponent />
         </ContextProvider>
       </>
     );
   }
   ```

   - 위 코드는 React 렌더링 원리에 따라 App 컴포넌트에서 setText가 발생하면 리액트 렌더링 규칙에 의해 하위 컴포넌트인 ChildComponent와 GrandChildComponent까지 전부 리렌더링 된다. useContext에는 이러한 상황을 최적화 할 수 있는 방법이 존재하지 않는다. (수동으로 React.memo를 붙여주는 수 밖에 없다.)
   - useContext로는 주입된 상태를 사용할 수 있을 뿐, 그 자체로 렌더링 최적화에 아무런 도움이 되지 않는다.

> [!Note] 공식 문서의 useContext 페이지에서는 아래와 같이 설명한다.
> React는 다른 `value`을 받는 Provider로부터 시작해서 특정 Context를 사용하는 모든 자식들을 **자동으로 리렌더링**합니다. 이전 값과 다음 값은 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)를 통해 비교합니다. [`memo`](https://ko.react.dev/reference/react/memo)로 리렌더링을 건너뛰어도 자식들이 새로운 Context 값을 받는 것을 막지는 못합니다.
>
> - 이 의미는 이전 렌더링과 다음 렌더링의 Context 가 변경됐을 때 해당 Context를 구독한 모든 자식 요소를 리렌더링한다는 뜻이다. 변경 감지는 다른 비교들과 마찬가지로 Object.is의 얕은 복사로 이루어진다.

## useReducer

`useReducer` 는 `useState`의 심화 버전으로 볼 수 있다. useState와 비슷한 형태를 띠지만 더 복잡한 상태값을 미리 정의해 놓은 시나리오에 따라 관리할 수 있다.

useReducer에서 사용하는 용어를 먼저 살펴보자.

- 반환값은 `useState`와 동일하게 길이가 2인 배열이다.
  - state : 현재 `useReducer`가 가지고 있는 값을 의미. useState와 마찬가지로 배열을 반환하는데, 동일하게 첫 번째 요소가 이 값이다.
  - dispatcher : state를 업데이트하는 함수. `useReducer`가 반환하는 배열의 두 번째 요소다. setState는 단순히 값을 넘겨주지만 여기서는 action을 넘겨준다는 점이 다르다. 이 action은 state를 변경할 수 있는 액션을 의미한다.
- `useState`의 인수와 달리 2개에서 3개의 인수를 필요로 한다.
  - reducer : useReducer의 기본 action을 정의하는 함수. 이 reducer는 useReducer의 첫 번째 인수로 넘겨주어야 한다.
  - initialState : 두 번째 인수로, useReducer의 초깃값을 의미한다.
  - init : `useState`의 인수로 함수를 넘겨줄 때처럼 초깃값을 지연해서 생성시키고 싶을 때 사용하는 함수. 이 함수는 필수값이 아니며, 만약 여기에 인수로 넘겨주는 함수가 존재한다면 `useState`와 동일하게 게으른 초기화가 일어나며 `initialState` 를 인수로 init 함수가 실행된다.

본격적인 사용법은 아래와 같다.

```jsx
// useReducer가 사용할 state 정의
type State = {
  count: number,
};

// state의 변화를 발생시킬 action의 타입과 넘겨줄 값(payload)을 정의
// 꼭 type과 payload라는 네이밍을 지킬 필요도 없으며, 굳이 객체일 필요도 없다.
// 다만 이러한 네이밍이 가장 널리 쓰인다.
type Action = {
  type: "up" | "down" | "reset",
  payload?: State,
};

// 무거운 연산이 포함된 게으른 초기화 함수
function init(count: State): State {
  // count : State를 받아서 초깃값을 어떻게 정의할지 연산하면 된다.
  return count;
}

//초깃값
const initialState: State = { count: 0 };

// 앞서 선언한 state와 action을 기반으로 state가 어떻게 변경될지 정의
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "up":
      return { count: state.count + 1 };
    case "down":
      return { count: state.count - 1 > 0 ? state.count - 1 : 0 };
    case "reset":
      return init(action.payload || { count: 0 });
    default:
      throw new Error(`Unexpected action type ${action.type}`);
  }
}

export default function App() {
  const [state, dispatcher] = useReducer(reducer, initialState, init);

  function handleUpButtonClick() {
    dispatcher({ type: "up" });
  }

  function handleDownButtonClick() {
    dispatcher({ type: "down" });
  }

  function handleResetButtonClick() {
    dispatcher({ type: "reset" });
  }

  return (
    <div className="App">
      <h1>{state.count}</h1>
      <button onClick={handleUpButtonClick}>+</button>
      <button onClick={handleDownButtonClick}>-</button>
      <button onClick={handleResetButtonClick}>reset</button>
    </div>
  );
}
```

- 언뜻 복잡해 보일 수 있지만, `useReducer`의 목적은 간단하다. 복잡한 형태의 state를 사전에 정의된 `dispatcher`로만 수정할 수 있게 만들어 줌으로써 state 값에 대한 접근은 컴포넌트에서만 가능하게 하고, 이를 업데이트하는 방법에 대한 상세 정의는 컴포넌트 밖에다 둔 다음, state의 업데이트를 미리 정의해 둔 `dispatcher`로만 제한하는 것이다.
- **state 값을 변경하는 시나리오를 제한적으로 두고 이에 대한 변경을 확인할 수 있게끔 하는 것**이 `useReducer`의 목적이다.
- 단순한 값은 useState로 충분하지만, state 하나가 가져야 할 값이 복잡하고 이를 수정하는 경우의 수가 많아진다면 state를 관리하는 것이 어려워진다. 때로는 성격이 비슷한 여러개의 state를 묶어 useReducer로 관리하는게 더 효율적일 수도 있다.
- 또한, useReducer를 사용해 state를 관리하면 state를 사용하는 로직과, 변경하는 로직을 분리할 수 있어 state 관리가 한결 편해진다.
- 세 번째 인수인 `게으른 초기화 함수`는 굳이 사용하지 않아도 된다. 이 함수가 없다면 두 번째 인수로 넘겨받은 기본값을 사용하게 된다.
  - 다만 해당 인수를 사용하면 useState 인자에 함수를 넣어준 것과 동일한 효과를 볼 수 있다.
  - 또, 초기화가 필요할 때 reducer에서 해당 함수를 재사용할 수도 있다.

## useImperativeHandle

해당 훅은 그리 널리 사용되는 훅은 아니지만, 일부 사용 사례에서 유용할 수 있다. `useImperativeHandle`을 이해하기 위해서는 먼저 React.forwardRef에 대해 알아야 한다.

### forwardRef 살펴보기

> [!Note] forwardRef는 React v19부터 사장되었다.
>
> → ref를 일반 프롭스처럼 넘길 수 있게 됨

- forwardRef는 ref 객체를 상위 컴포넌트에서 하위 컴포넌트로 전달하고 싶을 때 사용한다.
- 물론, ref라는 프롭스 이름으로 넘기는게 불가능하지, parentRef 등의 속성명으로 프롭스를 넘기면 잘 넘어간다. 그럼에도 forwardRef가 등장한 이유는 ref를 전달하는 데 있어 일관성을 제공하기 위해서다.

### useImperativeHandle이란?

`useImperativeHandle`은 부모에게서 넘겨받은 `ref`를 원하는 대로 수정할 수 있는 훅이다.

```jsx
const Input = (props) => {
  //useImperativeHandle을 사용하면 ref의 동작을 추가로 정의 가능.
  useImperativeHandle(
    props.ref,
    () => ({
      alert: () => alert(props.value),
    }),
    // useEffect의 deps와 같다.
    [props.value]
  );

  return <input {...props} />;
};

function App() {
  // input에 사용할 ref
  const inputRef = useRef(null);
  // input의 value
  const [text, setText] = useState("");

  function handleClick() {
    // inputRef에 추가한 alert라는 동작을 사용할 수 있다.
    inputRef.current.alert();
  }

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <>
      <Input ref={inputRef} value={text} onChange={handleChange} />
      <button onClick={handleClick}>Focus</button>
    </>
  );
}
```

- 위 코드에서 원래 `ref`에는 `{current : <HTMLElement>}` 와 같은 형태로 HTMLElement만 주입할 수 있는 객체였다. 하지만 위 코드에서는 `useImperativeHandle`을 사용해 추가적인 동작을 정의해다.
- 이로써 부모는 단순히 HTMLElement 뿐만 아니라 자식 컴포넌트에서 새롭게 설정한 객체의 키와 값에 대해서도 접근할 수 있게 됐다.

## useDebugValue

이 훅은 프로덕션 웹서비스가 아닌, 리액트 애플리케이션을 개발하는 과정에서 사용된다. 디버깅하고 싶은 정보를 이 훅에다 사용하면 리액트 개발자 도구에서 볼 수 있다.

```jsx
function useDate() {
  const date = new Date();
  //useDebugValue로 디버깅 정보를 기록
  useDebugValue(date, (data) => `현재 시간: ${date.toISOString()}`);
  return date;
}

export default function App() {
  const date = useDate();
  const [counter, setCounter] = useState(0);

  function handleClick() {
    setCounter((prev) => prev + 1);
  }

  return (
    <div className="App">
      <h1>
        {counter} {date.toISOString()}
      </h1>
      <button onClick={handleClick}>+</button>
    </div>
  );
}
```

위 코드를 실행한 후 개발자 도구 Components 탭에서 다음과 같은 결과를 볼 수 있다.

<img src="https://github.com/user-attachments/assets/b80267af-1eff-475b-8c08-fcc0a5993038" width="600px" />

- `useDebugValue`는 사용자 정의 훅 내부의 내용에 대한 정보를 남길 수 있는 훅이다.
- 두 번째 인수로 포매팅 함수를 전달하면 **이에 대한 값이 변경됐을 때만 호출되어 포매팅된 값을 노출**한다.
- `useDebugValue`는 오직 다른 훅 내부에서만 실행할 수 있다는 점에 유의하자. 컴포넌트 레벨에서 실행한다면 작동하지 않을 것이다.
  - 따라서 공통 훅을 제공하는 라이브러리나 대규모 웹 애플리케이션에서 디버깅 관련 정보를 제공하고 싶을 때 유용할 수 있다.

## 훅의 규칙

리액트에서 제공하는 훅은 사용하는 데 몇 가지 규칙이 존재한다. 이러한 규칙을 `rules-of-hooks`라고 하며 이와 관련된 ESLint 규칙인 `react-hooks/rules-of-hooks`도 존재한다.

리액트 공식 문서에는 훅을 사용할 때의 규칙에 대해 정리해 뒀다.

1. 최상위에서만 훅을 호출해야 한다. 반복문이나 조건문, 중첩된 함수 내에서 훅을 실행할 수 없다. 이 규칙을 따라야만 컴포넌트가 렌더링될 때마다 항상 동일한 순서로 훅이 호출되는 것을 보장할 수 있다.
2. 훅을 호출할 수 있는 것은 리액트 함수 컴포넌트, 혹은 사용자 정의 훅의 두 가지 경우 뿐이다. 일반 자바스크립트 함수에서는 훅을 사용할 수 없다.

앞서 `useState` 구현에서 보여줬던 것처럼 훅에 대한 정보 저장은 리액트 어딘가에 있는 `index`와 같은 키를 기반으로 구현돼 있다. (실제로는 객체 기반 링크드 리스트에 더 가깝다.)

- 즉, `useState`나 `useEffect` 는 모두 순서에 아주 큰 영향을 받는다.

  ```jsx
  function Component() {
    const [count, setCount] = useState(0);
    const [require, setRequired] = useState(false);

    useEffect(() => {
      // do something...
    }, [count, setCount]);
  }
  ```

- 해당 컴포넌트는 Fiber에서 다음과 같이 저장된다. 이 결과에서 볼 수 있듯이 리액트 훅은 파이버 객체의 링크드 리스트의 호출 순서에 따라 저장된다. 그 이유는 각 훅이 파이버 객체 내에서 순서에 의존해 state나 effect의 결과에 대한 값을 저장하고 있기 때문이다.
- 이렇게 고정된 순서에 의존해 훅과 관련된 정보를 저장함으로써, 이전 값에 대한 비교와 실행이 가능해진다.

  ```jsx
  {
      memoizedState: 0, //setCount 훅
      baseState: 0,
      queue: {/* ... */},
      baseUpdate: null,
      next: { // setRequired 훅
          memoizedState: false,
          baseState: false,
          queue: {/* ... */},
          baseUpdate: null,
          next: {
              memoizedState: {
                  tag: 192,
                  create: ()=>{},
                  destroy: undefined,
                  deps: [0, false],
                  next: {/* ... */}
              },
              baseState: null,
              queue: null,
              baseUpdate: null
          }
      }
  }
  ```

따라서 훅은 절대 조건문, 반복문 등에 의해 리액트에서 예측 불가능한 순서로 실행되게 해서는 안 된다. 항상 훅은 실행 순서를 보장받을 수 있는 컴포넌트 최상단에 선언돼 있어야 한다. 조건문이 필요하다면 반드시 훅 내부에서 수행해야 한다.

# 3-2. 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

리액트에서는 재사용할 수 있는 로직을 관리할 수 있는 두 가지 방법이 있다. `사용자 훅(custom hook)` 과 `고차 컴포넌트(higher order component)`다.

## 사용자 정의 훅

서로 다른 컴포넌트 내부에서 같은 로직을 공유하고자 할 때 주로 사용한다.

고차 컴포넌트는 리액트가 아니어도 사용할 수 있지만, 사용자 정의 훅은 오직 리액트

사용자 정의 훅이란 React Hook 들을 반으로 필요한 훅을 만드는 기법니다. 리액트 훅은 use로 시작한다는 규칙이 있어, 사용자 정의 훅도 이러한 규칙을 준수함으로써 개발 시 해당 함수가 리액트 훅이라는 것을 바로 인식할 수 있다.

다음은 fetch를 수행하는 `useFetch` 예제와 실제 사용 사례다.

```jsx
import { useEffect, useState } from "react";

// HTTP 요청을 하는 사용자 정의 훅
function useFetch<T>(
  url: string,
  { method, body }: { method: string; body?: XMLHttpRequestBodyInit }
) {
  // 응답 결과
  const [result, setResult] = useState<T | undefined>();

  // 요청 중 여부
  const [isLoading, setIsLoading] = useState<boolean>(false);

  // 2xx 3xx로 정상 응답인지 여부
  const [ok, setOk] = useState<boolean | undefined>();

  // HTTP status
  const [status, setStatus] = useState<number | undefined>();

  useEffect(() => {
    const abortController = new AbortController();

    (async () => {
      setIsLoading(true);

      const response = await fetch(url, {
        method,
        body,
        signal: abortController.signal,
      });

      setOk(response.ok);
      setStatus(response.status);

      if (response.ok) {
        const apiResult = await response.json();
        setResult(apiResult);
      }

      setIsLoading(false);
    })();

    return () => {
      abortController.abort();
    };
  }, [url, method, body]);

  return { ok, result, isLoading, status };
}

interface Todo {
  userId: number;
  id: number;
  title: string;
  completed: boolean;
}

export default function App() {
  // 사용자 지정 훅
  const { isLoading, result, status, ok } = useFetch<Array<Todo>>(
    "https://jsonplaceholder.typicode.com/todos",
    {
      method: "GET",
    }
  );

  useEffect(() => {
    if (!isLoading) {
      console.log("fetchResult >>", status);
    }
  }, [status, isLoading]);

  return (
    <div>
      {ok
        ? (result || []).map(({ userId, title }, index) => (
            <div key={index}>
              <p>{userId}</p>
              <p>{title}</p>
            </div>
          ))
        : null}
    </div>
  );
}

```

- 이 코드는 `fetch`를 이용해 API를 호출하는 로직을 사용자 정의 훅으로 분리한 예제다. 만약 훅으로 분리하지 않았다면 fetch는 API 호출을 해야 하는 모든 컴포넌트 내에서 공통적으로 관리되지 않는 최소 4개의 state를 선언해서 각각 구현해야 했을 것이다.
- 이렇게 **복잡하고 반복되는 로직은 사용자 정의 훅으로 간단하게 만들 수 있다.**
- 또한, 이 코드를 통해 use라는 이름을 왜 지켜야 하는 지 알 수 있다. 사용자 정의 훅은 내부에 useState와 useEffect 등을 가지고 자신만의 원하는 훅을 만드는 기법으로, 내부에서 useState와 같은 리액트 훅을 사용하고 있기 때문에 당연히 앞서 언급한 리액트 훅의 규칙을 따라야 한다.
- 만약 해당 네이밍 규칙을 따르지 않으면, 애러가 발생한다.
- 이런 사용자 정의 훅은 리액트 커뮤니티에서 다양하게 찾아볼 수 있다. 유명한 저장소로는 `use-Hooks`, `react-use`, `ahooks` 등이 있다.
  - 해당 훅들의 구성을 살펴보는 것도 많은 공부가 될 것이다.

## 고차 컴포넌트

코차 컴포넌트(HOC, Higher Order Component)는 컴포넌트 자체의 로직을 재사용하기 위한 방법이다. 사용자 정의 훅은 리액트 훅을 기반으로 하므로 리액트에서만 사용할 수 있는 기술이지만, 고차 컴포넌트는 `고차 함수(Higher Order Function)` 의 일종으로 JS 일급 객체, 함수의 특징을 이용하므로 굳이 리액트가 아니더라도 JS환경에서 널리 쓰일 수 있다.

리액트에서는 고차 컴포넌트 기법으로 다양한 최적화나 중복 로직을 관리할 수 있다. 리액트에서 가장 유명한 고차 컴포넌트는 리액트에서 제공하는 AP 중 하나인 `React.memo`다.

### React.memo란?

부모 컴포넌트가 새롭게 렌더링될 때 그 자식 컴포넌트는 프롭스 변경 여부 관계 없이 무조건 리렌더링 된다. `React.memo`는 그런 자식 컴포넌트의 불필요한 렌더링을 막기 위해 만들어졌다. 렌더링하기 앞서 props를 비교해, 이전과 props가 같다면 렌더링 자체를 생략하고 이전에 기억해 둔(memoization) 컴포넌트를 반환한다.

### 고차 함수 만들어보기

**고차 함수**란, `함수를 인수로 받거나 결과로 반환하는 함수`를 뜻한다.

- 가장 대표적인 고차 함수는 리액트에서 배열을 렌더링할 때 자주 사용하는 `Array.prototype.map` 이다.
  ```jsx
  const list = [1, 2, 3];
  const doubledList = list.map((item) => item * 2);
  ```
  위 예제에서 map 메소드는 인자로 `(item)=>item*2` 라는 함수를 인자로 받고 있다. map을 비롯해 `forEach`나 `reduce` 등도 고차 함수임을 알 수 있다.
- 아래는 한단계 나아가서, 고차함수로 `setState`를 만드는 예다. 이 역시 마찬가지로 **함수를 결과로 반환하는**이라는 조건을 만족하므로 고차 함수다.
  ```jsx
  // 즉시 실행 함수로 setter를 만든다.
  const setState = (function () {
    // 현재 index를 클로저로 가둬놔서 이후에도 계속 돌인한 index에 접근할 수 있도록 함
    let currentIndex = index;
    return function (value) {
      global.states[currentIndex] = value;
      // 컴포넌트를 렌더링한다. 실제로 컴포넌트를 렌더링하는 코드는 생략했다.
    };
  })();
  ```

### 고차 함수를 활용한 리액트 고차 컴포넌트 만들어보기

사용자 인증 정보에 따라서 인증된 사용자에게는 개인화된 컴포넌트를, 그렇지 않은 사용자에게는 별도로 정의된 공통 컴포넌트를 보여주는 시나리오를 떠올려보자.

고차 함수의 특징에 따라 개발자가 만든 또 다른 함수를 반환할 수 있다는 점에서 고차 컴포넌트를 사용하면 매우 유용하다. 다음 예제를 보자.

```jsx
interface LoginProps {
	loginRequired? : boolean
}

function withLoginComponent<T>(Component: ComponentType<T>) {
	return function (props : T & LoginProps) {
		const { loginRequired, ...restProps } = props;

		if (loginRequired) {
			return <>로그인이 필요합니다.</>
		}

		return <Component {...(restProps as T)} />
	}
}

// 원래 구현하고자 하는 컴포넌트를 만들고, withLoginComponent로 감싸기만 하면 끝
// 로그인 여부, 로그인이 안 되면 다른 컴포넌트를 렌더링하는 책임은 모두
// 고차 컴포넌트인 withLoginComponent에 맡길 수 있어 매우 편리하다.

const component = withLoginComponent((props : { value : string })=>{
	return <h3>{props.value}</h3>
});

export default function App() {
	// 로그인 관련 정보를 가져온다.
	const isLogin = true;
	return <Component value="text" loginRequired={isLogin} />
	// return <Component value="text" />;
}
```

Component는 평범한 컴포넌트지만, 이 함수 자체를 withLoginComponent라 불리는 고차 컴포넌트로 감쌌다. withLogicomponent는 함수(함수 컴포넌트)를 인수로 받으며, 컴포넌트를 반환하는 고차 컴포넌트다.

이 컴포넌트는 props에 loginRequired가 있다면 넘겨받은 함수를 반환하는 것이 아니라 “로그인이 필요합니다.”라는 전혀 다른 결과를 반환하게 돼있다. 물론 이러한 인증 처리는 서버나 NGINX와 같이 JS 이전 단계에서 처리하는 편이 훨씬 효율적이다. (위 예제는 고차 컴포넌트에 대한 이해를 돕기 위해 만든 것이다.)

이처럼, 고차 컴포넌트는 컴포넌트 전체를 감쌀 수 있다는 점에서 사용자 정의 훅보다 더욱 큰 영향력을 컴포넌트에 미칠 수 있다.

### 고차 컴포넌트 사용 시 주의사항

- 고차 컴포넌트는 이름 앞에 무조건 `with`이라는 단어를 붙여줘야 한다. 커스텀 훅 이름에 `use`를 붙여야 하는 것과 같은 이유다.
  - `use`처럼 ESLint 규칙 등으로 강제되는 사항은 아니지만 리액트 라우터의 `withRouter`와 같이 리액트 커뮤니티에 널리 퍼진 일종의 관습이다.
  - `with` 이 접두사로 붙어있으면 고차 컴포넌트임을 손쉽게 알아채어 개발자 스스로가 컴포넌트 사용에 주의를 기울일 수 있으므로 반드시 `with`으로 시작하는 접두사로 고차 컴포넌트를 만들자.
- 고차 컴포넌트를 사용할 때는 부수 효과를 최소화해야 한다. 고차 컴포넌트는 반드시 컴포넌트를 인수로 받게 되는데, 반드시 컴포넌트의 props를 임의로 수정, 추가, 삭제하는 일은 없어야 한다.
  - 앞의 예제에서도 `loginRequired`라는 props를 추가했을 뿐, 기존에 인수로 받는 컴포넌트의 props는 건드리지 않았다. 만약 기존 컴포넌트에서 사용하는 props를 수정하거나 삭제한다면 고차 컴포넌트를 사용하는 쪽에서 언제 props가 수정될 지 모른다는 우려를 가지고 개발해야하는 불편함이 생긴다.
- 여러개의 고차 컴포넌트로 컴포넌트를 감쌀 경우 복잡성이 커진다. 따라서 고차 컴포넌트는 최소한으로 사용하는게 좋다.

## 사용자 정의 훅 vs 고차 컴포넌트, 무엇을 써야 할까?

### 사용자 정의 훅이 필요한 경우

단순히 useEffect, useState와 같이 리액트에서 제공하는 훅으로만 공통 로직을 격리할 수 있다면 사용자 정의 훅을 사용하는 것이 좋다.

- 커스텀 훅은 훅의 결과를 컴포넌트 내에서 개발자가 원하는대로 사용하는 방식이기 때문에 부수 효과가 비교적 제한적이다.
- 대부분의 고차 컴포넌트는 렌더링에 영향을 미치는 로직이 존재하므로 사용자 정의 훅에 비해 예측하기가 어렵다.
- 따라서 단순히 컴포넌트 전반에 걸쳐 동일한 로직으로 값을 제공하거나 특정한 훅의 작동을 취하게 하고 싶다면 사용자 정의 훅을 사용하는 게 좋다.

### 고차 컴포넌트를 사용해야 하는 경우

앞선 예제였던 로그인되지 않은 사용자가 컴포넌트에 접근했을 때 로그인을 요구하는 공통 컴포넌트를 노출한다든가, 에러 바운더리 비슷하게 어떠한 특정 에러가 발생했을 때 현재 컴포넌트 대신 에러가 발생했음을 알릴 수 있는 컴포넌트 등에 쓰기 좋다.

```jsx
function HookComponent() {
  const { loggedIn } = useLogin();

  if (!loggedIn) {
    return <LoginComponent />;
  }

  return <>안녕하세요</>;
}

const HOCComponent = withLoginComponent(() => {
  // loggedIn state의 값을 신경 쓰지 않고 그냥 컴포넌트에 필요한 로직만
  // 추가해서 간단해졌다. loggedIn state에 따른 제어는 고차 컴포넌트에서 해줄 것이다.
  return <>안녕하세요.</>;
});
```

위 코드는 앞선 로그인 예제를 조금 변경한 것이다.

- 이러한 작업을 사용자 정의 훅으로 표현해야 한다고 가정한다면, 어차피 loggedIn이 false인 경우에 렌더링해야 하는 컴포넌트는 동일하지만 사용자 정의 훅만으로는 이를 표현하기 어렵다.
- 사용자 정의 훅은 해당 컴포넌트가 반환하는 랜더링 결과물에까지 영향을 미치기 어렵기 ㄸ문이다.
- 그리고 이런 중복 처리가 해당 사용자 정의 훅을 사용하는 앱 전반에 걸쳐 나타나게 될 것이므로, 사용자 정의 훅보다는 고차 컴포넌트를 사용하는게 좋다.

⇒ 결론적으로, 함수 컴포넌트의 반환값, 즉 렌더링의 결과물에도 영향을 미치는 공통 로직이라면 고차 컴포넌트를 사용하자. 고차 컴포넌트는 공통화된 렌더링 로직을 처리하기에 매우 훌륭한 방법이다.
