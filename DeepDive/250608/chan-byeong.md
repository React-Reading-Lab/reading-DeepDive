# 8 좋은 리액트 코드 작성을 위한 환경 구축하기

ESLint와 테스트 코드 작성법을 알아보자

## 8.1 ESLint를 활용한 정적 코드 분석

디버깅을 위해 정적 코드 분석.

**정적 코드 분석이란?**

코드의 실행과는 별개로 코드 그 자체만으로 코드 스멜(잠재적 버그를 야기할 수 있는 코드)을 찾아내어 문제의 소지가 있는 코드를 사전에 수정하는 것이다.

JS 생태계에서 가장 많이 사용되는 정적 코드 분석도구가 바로 `ESLint`이다.

### 8.1.1 ESLint 살펴보기

**ESLint는 어떨게 코드를 분석할까?**

1. 자바스크립트 코드를 문자열로 읽는다.

2. 자바스크립트 코드를 분석할 수 있는 파서로 코드를 구조화한다.

3. 2번에서 구조화한 코드 트리를 AST라고 하며 이 구조화된 트리를 기준으로 각종 규칙과 대조한다.

4. 규칙과 대조했을 때 이를 위반한 코드를 알리거나 수정한다.

자바스크립트를 분석하는 파서에는 여러가지가 존재한다.

그 중 ESLint는 `espree`를 사용한다.

코드를 `espree`를 통해서 분석하면 `json` 형태로 보여준다.

### 8.1.2 eslint-plugin과 eslint-config

`eslint-plugin`과 `eslint-config`는 모두 ESLint와 관련된 패키지지만 각각의 역할이 다르다.

**eslint-plugin**

`eslint-plugin`이라는 접두사로 시작하는 플러그인은 앞서 언급했던 규칙을 모아놓은 패키지이다.

예를 들어 `eslint-plugin-import` 패키지는 다른 모듈을 불러오는 `import`와 관련된 다양한 규칙을 제공한다.

`eslint-plugin` 특정 프레임워크나 도메인과 관련된 규칙을 묶어서 제공하는 패키지이다.

**eslint-config**

`eslint-config`는 `eslint-plugin`을 한데 묶어서 완벽하게 한 세트로 제공하는 패키지라 할 수 있다.

여러 프로젝트에 결처 동일하게 사용할 수 있는 ESLint 관련 설정을 제공하는 패키지가 바로 `eslint-config`이다.

`eslint-config`를 통해 빠르게 적용하는 것이 일반적이다.

대표적인 `eslint-config`를 알아보자.

`eslint-config-airbnb` , `@titicaca/triple-config-kit`, `eslint-config-next`

### 8.1.3 나만의 ESLint 규칙 만들기

ESLint 규칙을 생성해 관리하면 개발자가 일일이 수정하는 것보다 훨씬 더 빠르고 쉽게 수정할 수 있고, 이후 반복되는 실수 또한 방지할 수 있어 유용하다.

**이미 존재하는 규칙을 커스터마이징해서 적용하기: import React를 제거하기 위한 ESLint 규칙 만들기**

```javascript
module.exports = {
  rules: {
    "no-restricted-imports": [
      "error",
      {
        //paths에 금지시킬 모듈을 추가한다.
        paths: [
          {
            name: "react", // 모듈명
            importNames: ["default"], // 모듈의 이름
            message: "warning message", // 경고 메시지
          },
        ],
      },
    ],
  },
};
```

**완전히 새로운 규칙 만들기: new Date를 금지시키는 규칙**

### 8.1.4 주의할 점

**Prettier와의 충돌**

- 서로의 규칙이 충돌이 나지 않게끔 규칙을 선언해두어야한다. Prettier에서 제공하는 규칙을 어기지 않도록 ESLint에서는 해당 규칙을 끈다.

- js/ts 관련은 모두 eslint에게 나머지 파일들은 prettier에게 맡기는 것이다. 대신 js/ts에서는 `eslint-plugin-prettier`를 사용한다.

**규칙에 대한 예외 처리, 그리고 react-hooks/no-exhaustive-deps**

`eslint-disable-`주석을 사용하면 특정 규칙을 임시로 제외시킬 수 있다.

```javascript
//eslint-disable-line 현재 줄 무시

//eslint-disable-next-line 다음 줄 무시

//eslint-disable 파일 전체 무시
```

리액트 개발자가 많이 사용하는 곳이 `useEffect`나 `useMemo`의 의존성 배열이 비우거나 너무 길 경우 예외처리를 많이 사용한다.

하지만 의존성 배열을 임의로 판단해서 비우거나 모두 채우지 않는 경우 위험할 수 있다.

- 괜찮다고 임의로 판단한 경우: 위험한 케이스. _실제로 괜찮은 경우는 해당 변수가 컴포넌트의 상태와 별개로 동작하는 것을 의미한다._ 이 경우는 해당 변수를 어디서 어떻게 선언할지 다시 고민해 봐야 한다.

- 의존성 배열이 너무 긴 경우: 의존성 배열이 너무 길다는 것은 `useEffect` 내부의 함수가 너무 길다는 말과 동일하다. 그렇다면 내부 코드를 여러 개의 `useEffect`로 쪼개어 관리해야하는 것이 우선이다.

- 마운트 시점에 한 번만 실행하고 싶은 경우: 가장 흔히 볼 수 있는 케이스. 이러한 접근 방법은 과거 클래스 컴포넌트에서 사용되던 생명주기 형태의 접근 방법. 함수 컴포넌트의 패러다임과는 맞지 않을 가능성이 있다. _또한 [] 배열이 있다는 것은 컴포넌트의 상태값과 별개의 부수 효과가 되어 컴포넌트의 상태와 불일치가 일어날 수 있게 된다. 마지막으로 상태와 관계없이 한번만 실행되어야 하는 것이 있다면 해당 컴포넌트에 존재할 이유가 없다._

**ESLint 버전 충돌**

_충돌을 방지하기 위해 ESLint 공식 문서에서는 ESLint를 `peerDependencies`로 설정해두라고 권장하고 있다._

### 8.1.5 정리

이미 ESLint를 사용하고 있는 개발자라면 설치되어 있는 `eslint-config`는 무엇인지 왜 이것을 문제가 있는 코드로 간주하는지 반드시 한번 확인해보길 바란다.

## 8.2 리액트 팀이 권장하는 리액트 테스트 라이브러리

프론트엔드와 백엔드 모두 테스팅이 중요하지만 테스트하는 방법과 방법론은 사뭇 다르다.

백엔드의 테스트는 일반적으로 서버나 데이터베이스에서 원하는 데이터를 올바르게 가져올 수 있는지, 데이터 수정 간 교차 상태나 경쟁 상태가 발생하지는 않는지, 데이터 손실은 없는지. 특정 상황에서 장애가 발생하지 않는지 등을 확인하는 과정이 주를 이룬다.

이러한 백엔드 테스트는 일반적으로 화이트박스 테스트로, 작성된 코드가 의도대로 작동하는지 확인하고 GUI가 아닌 AUI에서 주로 수행해야하기 때문에 어느 정도 백엔드에 대한 이해가 있는 사람만 가능하다.

반면, 프론트엔드는 일반적인 사용자와 동일하거나 유사한 환경에서 수행된다. 주요 비지니스 로직이나 모든 경우의 수를 고려해야 하며 사용자는 프론트엔드 코드를 이해할 필요가 없기 때문에 블랙박스 형태로 테스트가 이루어진다.

백엔드의 경우 시나리오가 어느정도 정해져 있지만 프론트엔드는 사용자에게 완전히 노출된 영역이기 때문에 최대학 예측해서 확인해야 한다.

### 8.2.1 React Testing Library란?

React Testing Library는 DOM Testing Library를 기반으로 만들어졌고, DOM Testing Library는 jsdom을 기반으로 하고 있다.

jsdom은 순수한 자바스크립트 환경에서도 HTML과 DOM을 사용할 수 있도록 해주는 라이브러리이다.

jsdom을 사용하면 마치 HTML이 있는 것처럼 DOM을 불러오고 조작할 수 있다.

jsdom을 사용해 자바스크립트 환경에서 HTML을 사용할 수 있는 DOM Testing Library를 기반으로 동일한 원리로 리액트 기반 환경에서 리액트 컴포넌트를 테스팅할 수 있는 라이브러리가 바로 React Testing Library이다.

해당 라이브러리를 활용하면 직접 컴포넌트를 브라우저 환경에서 렌더링해보지 않고도 원하는 대로 렌더링되는지 확인할 수 있다.

### 8.2.2 자바스크립트 테스트의 기초

테스트 코드를 작성하는 방식은 다음과 같은 과정을 거친다.

1. 테스트할 함수나 모듈을 선정한다.

2. 함수나 모듈이 반환하길 기대하는 값을 적는다.

3. 함수나 모듈의 실제 반환 값을 적는다.

4. 3번의 기대에 따라 2번의 결과가 일치하는지 확인한다.

5. 기대하는 결과를 반환한다면 테스트는 성공이며, 만약 기대와 다른 결과를 반환하면 에러를 던진다.

5번 과정에서는 nodejs의 `assert`를 사용하여 처리할 수 있다.

위와 같은 과정을 도와주는 라이브러리가 바로 테스팅 라이브러리이다.

대표적인 자바스크립트 테스팅 라이브러리로는 `Jest`, `Mocha`, `Karma` 등이 존재한다.

### 8.2.3 리액트 컴포넌트 테스트 코드 작성하기

리액트에서 컴포넌트 테스트는 다음과 같은 순서로 진행된다.

1. 컴포넌트를 렌더링한다.

2. 필요하다면 컴포넌트에서 특정 액션을 수행한다.

3. 컴포넌트 렌더링과 2번의 액션을 통해 기대하는 결과와 실제 결과를 비교한다.

**실제로 해보기**

```javascript

import { render, screen } from '@testing-library/react`

test('renders learn react link', () => {
  render(<App />)
  const linkElement = screen.getByText(/learn react/i)
  expect(linkElement).toBeInTheDocument()
})

// App 컴포넌트 내부에 learn react 텍스트를 포함하고 있는 컴포넌트가 존재하는지 테스트하는 코드이다.

```

**정적 컴포넌트**

별도의 상태가 존재하지 않아 항상 같은 결과를 반환하는 컴포넌트를 테스트하는 방법은 크게 어렵지 않다.

```javascript
import { render, screen } from '@testing-library/react`

import StaticComponent from './index'

beforeEach(() => {
  render(<StaticComponent />)
})

describe('check link', () => {
  it('there are three links', () => {
    const ul = screen.getByTestId('ul')
    expect(ul.children.length).toBe(3)
  })

  it('link list style is square', () => {
    const ul = screen.getByTestId('ul')
    expect(ul).toHaveStyle('list-style-type: square')
  })
})

describe('react link test', () => {
  it('there is link', () => {
    const reactLink = screen.getByText('React')
    expect(reactLink).toBeVisible()
  })

  ...
})

```

- beforEach: `it`를 수행하기 전에 실행하는 함수이다. 위의 코드에서는 <StaticComponent>를 렌더링한다.

- describe: 비슷한 속성을 가진 테스트를 하나의 그룹으로 묶는 역할을 한다.

- it: test와 완전히 동일하며 test의 축약어이다.

- testId: testId는 리액트 테스팅 라이브러리의 예약어로 get 등의 선택자로 선택하기 어려운 요소를 선택하기 위하여 사용할 수 있다.

**동적 컴포넌트**

[사용자가 useState를 통해 입력을 변경하는 컴포넌트]

```javascript

import { fireEvent, render } from '@testing-library/react'
import userEvent from '@testing-library/user-event'

import InputComponent from '.'

describe('Test Input Component', () => {
  const setup = () => {
    const screen = render(<InputComponent />)
    const input = screen.getByLabelText('input') as HTMLInputElement
    const button = screen.getByText(/submit/i) as HTMLButtonElement

    return {
      input,
      button,
      ...screen,
    }
  }

  it('영문과 숫자만 입력된다.', () => {
    const { input } = setup()
    const inputValue = '안녕하세요123'
    userEvent.type(input, inputValue)
    expect(input.value).toEqual(123)
  })

  it('아이디를 입력하면 버튼이 활성화된다.', () => {
    const { button, input } = setup()

    const inputValue = 'hello'
    userEvent.type(input, inputValue)

    expect(input.value).toEqual(inputValue)
    expect(button).toBeEnabled()
  })

  it('버튼을 클릭하면 alert가 해당 아이디로 표시된다.', () => {
    const alertMock = jest.spyOn(window, 'alert').mockImplementation((_: string) => undefined)

    const { button, input } = setup()

    const inputValue = 'hello'
    userEvent.type(input, inputValue)
    fireEvent.click(button)

    expect(alertMock).toHaveBeenCalledTimes(1)
    expect(alertMock).toHaveBeenCalledWith(inputValue)
  })
})

```

- setup: setup 함수는 내부에서 컴포넌트를 렌더링하고 테스트에 필요한 button과 input를 반환한다.

- useEvent.type: 사용자가 타이핑하는 것을 흉내 내는 메서드. userEvent는 fireEvent의 여러 이벤트를 순차적으로 실행해주어서 좀 더 자세하게 사용자의 동작을 흉내낸다.

  예를 들어 userEvent.click을 실행하면 아래와 같은 이벤트가 순차적으로 실행된다.

  - fireEvent.mouseOver
  - fireEvent.mouseMove
  - fireEvent.mouseDown
  - fireEvent.mouseUp
  - fireEvent.click

- jest,spyO(window, 'alert').mockImplementation():

- jest.spyOn: 어떤 특정 메서드를 오염시키지 않고 실행됐는지 또 어떤 인수로 실행됐는지 등 실행과 관련된 정보만 얻고 싶을 때 사용한다.

- mockImplementation: 해당 메서드에 대한 모킹 구현을 도와준다. 현재 jest를 실행하는 Nodejs 환경에서는 window.alert가 존재하지 않으므로 해당 메서드를 모의 함수로 구현해야 하는데, 이것이 바로 `mockImplementation`의 역할이다.

**비동기 이벤트가 발생하는 컴포넌트**

fetch를 테스트할 수 있는 방법은 무엇일까?

jest.spyOn을 활용해서 fetch를 모킹해보자.

```javascript
jest.spyOn(window, "fetch").mockImplementation(
  jest.fn(() =>
    Promise.resolve({
      ok: true,
      statue: 200,
      json: () => Promise.resolve(MOCK_TODO_RESPONSE),
    })
  )
);
```

위의 케이스는 단순히 성공 시에만 모킹한 것이고 에러가 발생할 경우 또 다시 모킹함수를 구현해야한다.

이러한 문제를 해결하기 위해 등장한 것이 MSW(Mock Service Worker)이다.

MSW는 브라우저와 node.js에서 모두 사용할 수 있는 라이브러리로 서비스워커를 활용해 실제 네트워크 요청을 가로채서 모킹을 구현한다.

```javascript
import { rest } from "msw";

import { setupServer } from "msw/server";

const server = setupServer(
  rest.get(`/todos/:id`, (req, res, ctx) => {
    const todoId = req.params.id;

    if (Number(todoId)) {
      return res(ctx.json({ ...MOCK_RESPONSE, id: Number(todoId) }));
    } else {
      return res(ctx.status(404));
    }
  })
);

beforeAll(() => server.listen());

afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

- setupServer: MSW에서 제공하는 메서드로 이름 그대로 서버를 만드는 역할을 한다.

- server.resetHandlers: 해당 코드는 setupServer의 기본 설정으로 되돌리는 역할을 한다.

**비동기 코드 테스트**

```javascript
it('버튼을 클릭하면 데이터를 불러온다.', () => {
  const button = screen.getByRole('button', { name: /1번/})
  fireEvent.click(button)

  const data = await screen.findByText(MOCK_RESPONSE.title)
  expect(data).toBeInDocument()
})
```

동기 방식으로 요소를 즉시 찾는 `get`대신 요소가 렌더링될 때까지 일정 시간 동안 기다리는 `find`메서드를 사용하여 요소를 검색한다.

- server.use: 기존의 setupServer의 내용을 새롭게 덮어쓰는 기능

```javascript
server.use(
  rest.get("/todos/:id", (req, res, ctx) => {
    return res(ctx.status(503));
  })
);
```

해당 기능을 통해 데이터 페칭 시 오류가 발생한 경우에 대한 테스트도 가능하다.

### 8.2.4 사용자 정의 훅 테스트하기

`react-hooks-testing-library`를 활용하면 간단하게 훅을 테스트할 수 있다. 훅을 테스트하기 위해 훅을 포함하는 컴포넌트에 대한 테스트 코드를 작성하지 않아도 된다.

리액트 18버전 부터 @testing-library/react에 통합되었다.

예시

`useEffectDebugger` 훅의 기능

- 최초 컴포넌트 렌더링 시에는 호출되지 않는다.
- 이전 props를 useRef에 저장해두고 새로운 props를 넘겨받을 때마다 이전 props와 비교해 무엇이 렌더링을 발생시켰는지 확인한다.
- 이전 props와 신규 props의 비교는 리액트의 원리와 동일하게 Object.is를 활용해 얕은 비교를 수행한다.
- process.env.NODE_ENV === 'production' 환경에선 로깅을 하지 않는다.

해당 훅을 테스트하는 코드를 작성해보자

```javascript
import { renderHooks } from "@testing-library/react";

import useEffectDebugger, { CONSOLE_PREFIX } from "./useEffectDebuggr";

const consoleSpy = jest.spyOn(console, "log");
const componentName = "TestComponent";

descrbie("useEffectDebugger", () => {
  afterAll(() => {
    process.env.NODE_ENV === "development";
  });

  it("props가 없으면 호출되지 않는다", () => {
    renderHook(() => useEffectDebugger(componentName));

    expect(consoleSpy).not.toHaveBeenCalled();
  });

  it("props가 변경되지 않은 경우 호출되지 않는다.", () => {
    const props = { hello: "world" };

    const { rerender } = renderHooks(() =>
      useEffectDebuger(componentName, props)
    );

    expect(consoleSpy).not.toHaveBeenCalled();

    rerender();

    expect(consoleSpy).not.toHaveBeenCalled();
  });

  it("props가 변경된 경우 호출된다.", () => {
    const props = { hello: "world" };

    const { rerender } = renderHooks(
      () => useEffectDebuger(componentName, props),
      {
        initialProps: {
          componentName,
          props,
        },
      }
    );

    const newProps = { hello: "world2" };

    rerender({ componentName, newProps });

    expect(consoleSpy).toHaveBeenCalled();
  });
});
```

`react-hooks-testing-library`를 활용하면 훅의 테스트를 위한 컴포넌트를 따로 만들지 않고 훅만을 테스트할 수 있다.

### 8.2.5 테스트를 작성하기에 앞서 고려해야 할 점

- 테스트 커버리지를 맹신하지 말 것

- 테스트를 작성하기 전에 어플리케이션에서 가장 취약하거나 중요한 부분을 파악하는 것이 중요하다.

### 8.2.6 그 밖에 해볼 만한 여러 가지 테스트

- 유닛 테스트: 코드나 컴포넌트가 독립적으로 분리된 환경에서 의도대로 작동하는지 검증

- 통합 테스트: 유닛 테스트를 통과한 컴포넌트를 묶어 하나의 기능이 정상적으로 동작하는지 검증

- 엔드 투 엔드 테스트: 실제 사용자처럼 작동하는 로봇을 활용해 어플리케이션의 전체적인 기능을 테스트

### 8.2.7 정리

테스트가 이루어야 할 목표는 어플리케이션이 비지니스 요구사항을 충족하는지 확인하는 것.

이를 위해 처음부터 끝까지 테스트 코드를 작성하는 것보다는 핵심 로직부터 테스트 코드를 채워나가는 것이 좋다.
