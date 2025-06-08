## EsLint

EsLint는 대표적으로 많이 쓰이는 정적 코드 분석 도구로 `expree`를 사용해서 코드를 파싱해 분석한 뒤 잠재적인 문제를 발견하고 알리거나 수정을 도와준다.

> typescript는 @typescript-eslint/typescript-estree라고 하는 espree기반 파서가 따로 있다.
> 

### 코드 분석 과정

1. JS 코드를 문자열로 읽는다
    
    ```jsx
    const a = 1;
    ```
    
2. JS 코드를 분석할 수 있는 파서로 코드를 트리(AST)로 구조화한다
(코드의 정확한 위치와 같은 세세한 정보가 담겨 있음)
    
    ```json
    {
      "type": "Program",
      "start": 0,
      "end": 12,
      "range": [
        0,
        12
      ],
      "body": [
        {
          "type": "VariableDeclaration",
          "start": 0,
          "end": 12,
          "range": [
            0,
            12
          ],
          "declarations": [
            {
              "type": "VariableDeclarator",
              "start": 6,
              "end": 11,
              "range": [
                6,
                11
              ],
              "id": {
                "type": "Identifier",
                "start": 6,
                "end": 7,
                "range": [
                  6,
                  7
                ],
                "name": "a"
              },
              "init": {
                "type": "Literal",
                "start": 10,
                "end": 11,
                "range": [
                  10,
                  11
                ],
                "value": 1,
                "raw": "1"
              }
            }
          ],
          "kind": "const"
        }
      ],
      "sourceType": "module"
    }
    ```
    
3. AST를 기준으로 각종 규칙과 대조한다
4. 위반한 코드는 알리거나 수정한다

### EsLint 규칙

- **rule**: 어떤 코드가 잘못된 코드인지, 어떻게 수정해야 하는지 알려줌
- **plugin**: rule의 모음
    
    ex) eslint-plugin-import: 모듈을 불러오는 것과 관련된 다양한 규칙 제공
    
- **config**: plugin의 모음
    - eslint-config-airbnb: 제일 유명한 eslint-config
    - @titicaca/triple-config-kit: 한국 커뮤니티에서 운영되는 config 중 유지보수가 활발한 config. airbnb를 기반으로 룰 몇 개만 수정하는 다른 패키지와 다르게 자체적으로 정의한 규칙으로 운영되고 있다.
    - eslint-config-next: JS뿐만 아니라 JSX 구문, _app, _document에 작성되어 있는 HTML 코드 또한 정적 분석을 제공한다. core web vitals를 분석해 제공하는 기능도 포함되어 있다.

> plugin과 config를 만들 때는 eslint-plugin, eslint-config 접두사를 붙여야 하며, 반드시 뒤에 한 단어만 붙을 수 있다.
> 

### ESLint 규칙 커스텀 or 직접 만들기

> https://eslint.org/docs/latest/extend/custom-rules를 참고해서 직접 규칙을 만들 수도 있다.
> 

### 주의할 점

- **Prettier와의 충돌**
    
    prettier는 유명한 코드 포멧팅 도구다. 정적 분석해서 문제를 해결한다는 점에서 eslint와 비슷하다. 그러나 문제는 prettier과 eslint 모두 들여쓰기, 줄바꿈 등을 처리할 수 있기 때문에 두 가지에 서로 충돌하는 규칙이 있다면 에러가 발생할 수 있다. 이럴 때 해결 방법은 크게 2가지가 있다.
    
    1. prettier에 해당하는 규칙은 EsLint에서 끄기
    2. JS, TS는 EsLint에서, 나머지는 prettier가 하기
- **규칙에 대한 예외 처리**
    
    필요없는 규칙이라면 끄는 것이 맞지만, 그냥 무시하기 전에 점검해보기
    
    ```jsx
    // eslint-disable-line no-console
    
    // eslint-disable-next-line no-console
    
    /* eslint-disable no-console */
    ```
    

## React Testing Library

### 테스트의 이점

- 설계한대로 프로그램이 작동하는지 확인할 수 있다.
- 버그를 사전에 방지해 안정적인 서비스를 제공할 수 있다.
- 수정한 이후에도 예외없이 잘 동작하는지 확인할 수 있다.

### 프론트 테스트 VS 백엔드 테스트

| 프론트 테스트 | 백엔드 테스트 |
| --- | --- |
| 일반적인 사용자와 동일한 환경에서 테스트를 수행하며 프로그램이 의도한대로 동작하는지 확인 | 서버나 데이터베이스에서 원하는 데이터를 올바르게 가져올 수 있는지, 데이터 수정 간 교착 상태나 경쟁 상태가 발생하지 않는지, 데이터 손실은 없는지 등등을 확인 |
| 블랙박스 테스트 | 화이트박스 테스트 |
| GUI | AUI |

### React Testing Library란?

DOM Testing Library를 기반으로 만들어진 테스팅 라이브러리. Dom Testing Library는  jsdom을 기반으로 하며 HTML이 없는 자바스크립트만 존재하는 환경에서 HTML과 DOM을 사용할 수 있게 해주는 라이브러리다. jsdom을 활용해 실제로 리액트 컴포넌트를 렌더링하지 않고도 리액트 컴포넌트가 원하는 대로 렌더링되고 있는지 확인할 수 있다.

```jsx
const jsdom = require('jsdom')

const { JSDOM } = jsdom

const dom = new JSDOM(`<!DOCTYPE html><p>Hellow World</p>`)

console.log(dom.window.document.querySelector('p').textContent)
```

### 자바스크립트 테스트

1. 테스트할 함수나 모듈을 선정한다.
2. 함수나 모듈이 반환하길 기대하는 값을 적는다.
3. 함수나 모듈의 실제 반환 값을 적는다.
4. 기대값과 실제 값이 일치하는지 확인한다. 결과가 같으면 성공, 틀리면 실패다.

```jsx
let actual = sum(1, 2) 

let expected = 3

if (expected !== actual) throw new Error();
```

테스트 결과를 확인할 수 있도록 도와주는 라이브러리를 Assertion라이브러리라고 한다.

```jsx
const assert = require('assert')

assert.equal(sum(1, 2), 3)
```

좋은 테스트 코드는 다양한 테스트 코드를 작성하고 통과하는 것뿐만 아니라 어떤 테스트가 무엇을 테스트하는지 알기 쉽게 해주어야 한다. 그래서 assertion 라이브러리를 기반으로 테스트를 수행하며 추가로 테스트를 알아보기 쉽게 정보를 작성할 수 있는 라이브러리를 같이 사용할 수 있는 테스팅 프레임워크를 사용한다. 대표적으로 Jest, Mocha 등이 있다. (Jest 같은 경우에는 expect 패키지를 사용해 assertion을 수행한다.)

```jsx
test('두 인수가 덧셈이 되어야 한다.', () => {
	expect(sum(1,2)).toBe(3)
})

// npm run test로 실행

/*
test, expect 모두 global 객체에 존재하지 않는 메소드다. 
그래도 오류없이 실행할 수 있는 이유는 Jest CLI 에 있다. Jest를 비롯한 테스트 프레임워크에는 
global이라고 해서 실행 시 전역 스코프에 기본으로 넣어주는 값이 있다.
그래서 node가 아닌 jest로 실행하는 것이다.
*/
```

### 리액트 컴포넌트 테스트

1. 컴포넌트를 렌더링한다.
2. 필요하면 컴포넌트에서 특정 액션을 수행한다.
3. 컴포넌트 렌더링과 액션을 통해 기대하는 결과와 실제 결과를 비교한다.

```jsx
test('renders learn react link', () => {
	render(<App />) // App 렌더링
	const linkElement = screen.getByText(/learn react/i) // 렌더링 컴포넌트 내부에 learn react가 있는지 확인
	expect(linkElement).toBeInTheDocument() // 어설션을 사용해 2번에서 찾은 요소가 document 내부에 있는지 확인
})
```

- HTML 요소가 있는지 확인하는 방법
    1. getBy: 인수의 조건에 맞는 요소를 반환
    2. findBy: Promise 반환. 1000ms 타임아웃을 가지고 있다.
    3. queryBy: 조건에 해당하는 요소를 찾지 못하면 에러를 반환하는 1,2와 다르게 null 반환.
- 다른 jest 메소드들
    1. beforeEach: 각 테스트를 수행하기 전에 실행하는 함수
    2. describe: 비슷한 속성을 가진 테스트를 묶는 역할
    3. it: test의 축악어.
    4. testId: HTML의 DOM 요소에 testId 데이터셋을 선언해 두면 이후 테스트 시에 getTestId, findTestId 등으로 선택할 수 있다.

### 동적으로 렌더링되는 컴포넌트 테스트하기

- setup으로 렌더링과 테스트에 필요한 컴포넌트 반환
- userEvent: 사용자가 행동하는 것을 흉내내는 메서드. fireEvent라고 라이브러리에서 제공하는 단일 이벤트를 묶어놓은 것으로 생각하면 됨.
- spyOn: 특정 메서드를 오염시키지 않고 실행이 됐는지, 어떤 인수로 실행이 되었는지 등 관련된 정보만 얻고 싶을 때 사용
    
    > 특정 메서드를 오염시키지 않는다는 게 무슨말이지?
    > 

### 비동기 이벤트 발생하는 컴포넌트 테스트하기

- MSW를 활용한다. MSW는 서비스 워커를 사용해 실제 네트워크 요청을 가로채는 방식으로 모킹을 구현한다. Node.js에서는 https나 XMLHttpRequest의 요청을 가로채는 방식으로 동작한다.

```jsx
beforeAll(() => server.listen()) // 서버 기동
afterEach(() => server.resetHandlers()) // setupServer를 되돌리는 역할
afterAll(() => server.close()) // 서버 종료
```

### 사용자 정의 훅 테스트

react-hooks-testing-library를 사용해서 테스트를 할 수 있다.

### 테스트를 작성할 때 주의할 점

테스트를 맹신하는 것이 아닌 애플리케이션에서 가장 취약하거나 중요한 부분을 파악하는 것이다.

가장 중요한 것은 애플리케이션이 비즈니스 요구사항을 충족하는지 확인하는 것이다.

### 그 밖의 여러 가지 테스트

- 유닛 테스트: 코드나 컴포넌트가 독립적으로 분리된 환경에서 테스트
- 통합 테스트: 유닛 테스트를 통과한 여러 컴포넌트를 묶어서 하나의 기능으로 정상 작동하는지 테스트
- 엔드 투 엔드: 실제 사용자처럼 작동하는 로봇을 활용해 app의 전체적인 기능을 테스트
