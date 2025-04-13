# JSX란?

보통 React를 통해 JSX를 많이 배우기 때문에, JSX가 리액트의 전유물이라고 오해하는 경우가 종종 있다. 물론 리액트가 등장하며 페이스북(현 메타)에서 소개한 새로운 구문이지만, 꼭 리액트에서만 쓰라는 법은 없다.

JSX는 XML과 유사한 내장형 구문이며, 리액트에 종속적이지 않은 독자적인 문법으로 보는 것이 옳다.

자바스크립트 표준의 일부가 아니며, V8이나 Deno와 같은 JS 엔진이나 브라우저에 의해 실행되거나 표현되도록 만들어진 구문이 아니다.

따라서 JSX 코드를 바로 실행하면 오류가 발생한다. 반드시 **트랜스파일러**를 거쳐야 한다. JSX의 설계 목적은 다양한 트랜스파일러에서 다양한 속성을 가진 **트리 구조를 토큰화해 ES로 변환하는 데 초점**을 두고 있다.

## JSX의 정의

JSX는 기본적으로 `JSXElement`, `JSXAttributes`, `JSXChildren`, `JSXStrings` 라는 네 가지 컴포넌트를 기반으로 구성돼 있다.

### JSXElement

JSX를 구성하는 가장 기본 요소. HTML과 비슷한 역할을 한다. JSXElement가 되기 위해서는 다음과 같은 형태 중 하나여야 한다.

- JSXOpeningElement : 일반적으로 볼 수 있는 요소. JSXClosingElement가 동일한 요소로 같은 단계에서 선언되어야 한다.
  - <JSXElement JSXAttributes(optional)>
- JSXClosingElement : JSXOpeningElement가 종료됐음을 알리는 요소. 반드시 Opening과 쌍을 이루어야 한다.
  - </JSXElement>
- JSXSelfClosingElement : 요소가 시작되고, 스스로 종료하는 형태.
  - <JSXElement JSXAttributes(optional) />
- JSXFragment : 아무런 요소가 없는 형태. Self 형태를 띨 수는 없다.
  - <></>

Tip : 엘리먼트 명을 대분자로 써야하는 규칙은 JSXElement 표준에는 포함되어있지 않다. 리액트에서 HTML과 사용자 컴포넌트를 구분하기 위해 둔 규칙으로 보인다.

### JSXElementName

JSXElement의 요소 이름으로 쓸 수 있는 것을 의미한다. 이름으로 가능한 것은 다음과 같다.

- JSXIdentifier : JSX 내부에서 사용할 수 있는 식별자르 의미. JS 식별자 규칙과 동일.
  - <$></$>, <_></_> 등은 가능하지만 JS와 마찬가지로 <1></1>은 허용 안된다.
- JSXNamespacedName :
  - JSXIdentifier:JSXIdentifier의 조합. 즉, `:` 을 통해 서로 다른 식별자를 이어주는 것도 하나의 식별자로 취급된다.
  - `:` 을 통해 묶을 수 있는 식별자는 하나 뿐이다. 두 개 이상은 올바른 식별자로 취급되지 않는다.
    - 가능 : <foo:bar></foo:bar>
    - 불가능 : <foo:bar:baz></foo:bar:baz>
- JSXMemberExpression :
  - JSXIdentifier.JSXIdentifier의 조합. 즉, `.` 을 통해 서로 다른 식별자를 이어주는 것도 하나의 식별자로 취급된다.
  - `.`을 통해서는 3개 이상의 식별자를 하나로 묶는 게 가능하다. 하지만 JSXNamespacedName과 섞어서 사용하면 안된다.
    - <foo.bar.baz></foo.bar.baz>

### JSXAttributes

JSXElement에 부여할 수 있는 속성을 의미한다.

- JSXSpreadAttributes
  - JS의 전개 연산자와 동일한 역할을 한다.
  - {…AssignmentExpression} : 여기에서의 AssignmentExpression에는 단순히 객체뿐만 아니라 JS에서 AssignmentExpression로 취급되는 모든 표현식이 존재할 수 있다. **`조건문 표현식, 화살표 함수, 할당식`** 등 당양한 것이 포함돼 있다.
- JSXAttribute
  - 속성을 나타내는 키와 값의 쌍으로 표현한다.
  - 키는 JSXAttributeName으로, 값은 JSXAttributeValue로 불린다.
  - JSXAttributeName에는 JSXIdentifier와 JSXNamespacedName이 가능하다.
  - JSXAttributeValue는 다음과 같은 규칙을 만족해야 한다.
    - `큰따옴표`로 감싼 문자열
    - `작은따옴표`로 감싼 문자열
    - { JS의 AssignmentExpresstion }
    - JSXElement : 자주 사용하는 형태는 아니지만, **속성의 값으로 다른 JSX 요소가 들어갈 수 있다는 뜻**이다.

### JSXChildren

JSXElement의 자식 값.

- JSXChild : JSXChildren을 이루는 기본 단위. JSXChildren은 JSXChild를 0개 이상 가질 수 있다. 아래와 같은 형태가 될 수 있다.

  - JSXText : {, <, >, } 을 제외한 문자열.
  - JSXElement : 값으로 다른 JSX 요소가 들어갈 수 있다.
  - JSXFragment : 값으로 빈 JSX 요소인 <></>가 들어갈 수 있다.
  - { JSXChildExpression (optional) }

    - JSXChildExpression은 JS의 AssignmentExpression을 의미한다. 다음과 같은 코드도 올바른 JSX 코드다.

      ```jsx
      export default function App() {
        return <>{(() => "foo")()}</>;
      }

      // 렌더링하면 foo라는 문자열이 출력됨
      ```

### JSXStrings

HTML에서 사용 가능한 문자열은 모두 JSXStrings에서도 사용 가능하다. 여기서 정의된 문자열이라 함은 `큰 따옴표로 구성된 문자열`, `작은 따옴표로 구성된 문자열` , `JSXText` 를 의미한다.

- JS의 문자열과 한 가지 다른 점은, JS에서 `\`+ `이스케이프 문자` 를 사용할 때 걸리는 제약 사항들이 JSX에서는 존재하지 않는다는 것이다.

  ```jsx
  <button>\</button>  // 문제 없음

  let escape1 = '\' // SyntaxError 발생

  let escape2 = '\\' // 정상 처리
  ```

## JSX가 JS로 변환되는 방법

JSX의 변환 방식을 알려면 리액트에서 해당 작업을 수행하는 `@babel/plugin-transform-react-jsx` 플러그인을 알아야 한다.

이 플러그인은 JSX 구문을 JS 컴파일러가 이해할 수 있는 형태로 변환한다.

- 아래와 같은 형식이다. 하지만 해당 예제의 트랜스파일 결과는 리액트 17, 바벨 7.9.0 이후 버전에서 추가된 자동 런타임(automatic runtime) 결과와는 다르다. (자세한 예제는 125P 참고)

  ```jsx
  // JSX
  const ComponentA = <A required={true}>Hello World</A>;
  const ComponentB = <>Hello World</>;
  const ComponentC = (
    <div>
      <span>hello world</span>
    </div>
  );

  // JS 변환
  var ComponentA = React.createElement(A, { required: true }, "Hello World");
  var ComponentB = React.createElement(React.Fragment, null, "Hello World");
  var ComponentC = React.createElement(
    "div",
    null,
    React.createElement("span", null, "hello world")
  );
  ```

- 자동 런타임 결과와 위 결과에는 두가지 공통점이 있다.
  - JSXElement를 첫 번째 인수로 선언해 요소를 정의한다.
  - 옵셔널인 JSXChildren, JSXAttributes, JSXStrings는 이후 인수로 넘겨주어 처리한다.
- 이러한 특성으로 인해 JSX는 아래와 같은 두 가지 장점을 가진다.

  - 경우에 따라 다른 JSXElement를 렌더링해야 할 때 **굳이 요소 전체를 감싸지 않더라도 처리**할 수 있다.
  - JSXElement만 다르고, JSXAttributes, JSXChildren이 완전히 동일한 상황에서 **중복 코드를 최소화**할 수 있어 유용하기도 하다.
  - 위 두 가지 장점을 활용한 예제는 아래와 같다. JSX 반환값은 결국 `React.createElement`로 귀결되므로 결과는 동일하다.

    ```jsx
    import { createElement, PropsWithChildren } from 'react';

    // props 여부에 따라 children 요소만 달라지는 경우,
    // 굳이 번거롭게 전체 내용을 삼항 연산자로 처리할 필요가 없다.

    // 아래는 불필요한 코드 중복이 일어나는 코드
    // class="text" 와 {children} 이 코드 내에서 두 번 적힘
    function TextOrHeading({
    	isHeading,
    	children,
    }:PropsWithChildren<{ isHeading: boolean }>) {
    	return isHeading ? (
    		<h1 className="text">{children}</h1>
    	) : (
    		<span className="text">{children}</span>
    	)
    }

    // 위 코드를 코드 중복 없이 리팩토링
    import { createElement } from 'react'

    function TextOrHeading({
    	isHeading,
    	children,
    }: PropsWithChildren< isHeading:boolean >) {
    	return createElement(
    		isHeading ? 'h1' : 'span',
    		{ className : 'text' },
    		children,
    	)
    }
    ```

## 정리

- JSX 문법에 있지만 React에서 사용하지 않는 문법은 다음과 같다.
  - JSXNamespacedName
  - JSXMemberExpression
- 하지만 Preact, SolidJS, Nano JSX 등 JSX를 채용하고 있는 다른 라이브러리에서는 React와 다르게 위 문법을 채용하고 있기도 하다.
- JSX는 JS 코드 내에 HTML과 같은 트리 구조를 가진 컴포넌트를 표현할 수 있다는 점에서 각광받는다. 하지만 JSX 내부에 JS 문법이 많아질 수록 가독서을 해칠 것이므로 주의해야 한다.
- 그러므로, 적어도 React 내부에서 JSX가 어떻게 트랜스파일링 되는 지, 어떤 결과물을 만드는 지 알아두면 향후 리액트 프로젝트를 진행하는 데 도움이 될 수 있다. JSX는 대부분의 경우 편리하고 간결하게 컴포넌트를 작성하도록 도와주지만, 앞선 예제처럼 직접 **createElement를 사용하는 편이 더 효율적일 수도** 있다.

# 2.2 가상 DOM과 리액트 파이버

리액트의 가장 유명한 특징 중 하나는 실제 DOM이 아닌 가상 DOM을 운영한다는 점이다. 이번 장에서는 가상 DOM이 무엇인지, 그리고 실제 DOM에 비해 어떤 이점이 있는 지 살펴보고 가상 DOM 사용 시의 유의점에 대해서도 알아본다.

## DOM과 브라우저 렌더링 과정

가상 DOM을 공부하기 앞서 DOM(Document Object Model)이 무엇인지부터 살펴본다.

DOM은 웹페이지에 대한 인터페이스로 브라우저가 웹페이지의 콘텐츠와 구조를 어떻게 보여줄지에 대한 정보를 담고 있다.

브라우저가 웹사이트에 접근 요청을 받으면 다음과 같은 과정을 거친다.

1. 브라우저가 사용자가 요청한 주소를 방문해 HTML 파일을 다운로드한다.
2. 브라우저의 렌더링 엔진은 HTML을 파싱해 DOM 노드로 구성된 트리(DOM)를 만든다.
3. 2번 과정에서 CSS 파일을 만나면 해당 CSS 파일도 다운로드한다.
4. 브라우저의 렌더링 엔진은 이 CSS도 파싱해 CSS 노드로 구성된 트리(CSSOM)를 만든다.
5. 브라우저는 2번에서 만든 DOM 노드를 순회하는데, 여기서 모든 노드를 방문하지는 않고, 사용자의 눈에 보이는 노드만 방문한다. 즉, `display : none` 과 같이 사용자 화면에 보이지 않는 요소는 방문해 작업하지 않는다.

   _(이 부분은 다르게 알고 있는게, DOM노드와 CSSOM을 순회하며 Render Tree를 만들고, 이 Render Tree에 보이지 않는 요소가 포함되어있지 않은 쪽일 것이다.)_

6. 눈에 보이는 노드를 대상으로 해당 노드에 대한 CSSOM 정보를 찾고 여기서 발견한 CSS 스타일 정보를 이 노드에 적용한다. 이 DOM 노드에 CSS를 적용하는 과정은 크게 두 가지로 나뉜다.
   - 레이아웃(layout, reflow) : 각 노드가 브라우저 화면의 어느 좌표에 정확히 나타나야 하는지 계산하는 과정. 레이아웃을 거치면 반드시 페인팅도 거치게 된다.
   - 페인팅(painting) : 레이아웃 단계를 거친 노드에 색과 같은 실제 유효한 모습을 그리는 과정.

개발자도구 성능탭 호출트리에서 다음과 같이 파싱, 레이아웃, 페인트 등이 차례대로 일어난 모습을 볼 수 있다.

![Image](https://github.com/user-attachments/assets/6a7e59ce-000d-47cc-b503-576610a2b61c)

## 가상 DOM의 탄생 배경

이전 파트에서 살펴봤듯이, 브라우저가 웹페이지를 렌더링하는 과정은 매우 복잡하고 많은 비용이 든다. 심지어 요즘 앱들은 렌더링이 완료된 후 추가적인 인터렉션 발생까지 고려해야 한다.

- 특정한 요소의 색상만 변경되는 경우는 렌더링 단계 중 **페인팅만 일어나므로 비교적 빠르게 처리**할 수 있다. 하지만 특정한 요소의 노출 여부나 사이즈가 변경되는 등 요소의 위치와 크기가 재계산되는 경우는 **레이아웃** 이 일어나기 때문에 더 많은 비용이 발생한다.
- 또한, 변경된 DOM 요소가 많은 자식을 갖고 있다면 그 **하위 자식 요소들까지 함께 변경돼야하기 때문에 더 많은 비용을 지불**할 수도 있게 된다.
- 이러한 추가 렌더링 작업은 하나의 페이지에서 모든 작업이 일어나는 SPA에서 더욱 많아진다.
  - 페이지가 변경되는 경우 다른 페이지로 가서 처음부터 HTML을 새로 받아서 다시금 렌더링 과정을 시작하는 일반적인 웹페이지와는 다르게 하나의 페이지에서 계속해서 요소의 위치를 재계산한다. 이러한 SPA의 특징 덕분에 사용자는 깜빡임없이 웹페이지를 이용할 수 있지만, 그만큼 DOM을 관리하는 과정에서 부담해야 할 비용이 커진다.
- 하나의 인터렉션으로 페이지 내부의 DOM에서 여러가지가 변경되는 시나리오는 요즘 웹에서 아주 흔하다. 하지만 이러한 사용자의 인터랙션에 따라 DOM의 모든 변경 사항을 추적하는 것은 개발자 입장에서 너무 수고스러운 일이다.

때문에 **`인터랙션에 따른 DOM 최종 결과물을 간편하게 제공하는 것은 브라우저 뿐만 아니라 개발자에게도 유용`**하다.

**이러한 문제점을 해결하기 위해 가상 DOM이 탄생**했다.

가상 DOM은 웹페이지가 표시해야 할 DOM을 메모리에 저장하고 리액트가 변경에 대한 준비를 완료했을 때 실제 DOM에 반영한다.

- 여기서 말하는 리액트는 정확히는 `react` 가 아닌 `react-dom` 이다.
- 이렇게 DOM 계산을 브라우저가 아닌 메모리에서 계산하는 과정을 한 번 거치면 여러번 발생할 렌더링을 최소화할 수 있고, 브라우저는 렌더링 횟수가 적어져, 개발자는 관리할 영역이 적어져 부담을 최소화할 수 있다.

그렇다면, 리액트는 이 가상 DOM을 어떻게 관리할까?

## 가상 DOM을 위한 아키텍쳐, 리액트 파이버 (React Fiber)

### 리액트 파이버란?

- 리액트 파이버는 리액트에서 관리하는 JS 객체다. _(아키텍쳐 명도 React Fiber다.)_
- 파이버는 `Fiber 재조정자(Fiber reconciler)`가 관리하는데, 이 `Reconciler`는 이는 앞서 이야기한 가상 DOM과 실제 DOM을 비교해 변경 사항을 수집하며, 만약 이 둘 사이에 차이가 있으면 변경에 대한 정보를 가진 파이버를 기준으로 렌더링을 요청한다.
- 요약하면, **React에서 어떤 부분을 새롭게 렌더링해야 하는지 가상 DOM과 실제 DOM을 비교하는 작업(알고리즘)**이라고 이해하면 된다.
- 리액트 파이버의 목표는 리액트 웹 애플리케이션에서 발생하는 애니메이션, 레이아웃, 그리고 사용자 인터랙션에 올바른 결과물을 만드는 반응성 문제를 해결하는 것이다. 이를 위해 파이버는 다음과 같은 일을 할 수 있다.
  - 작업을 작은 단위로 분할하고 쪼갠 다음, 우선순위를 매긴다.
  - 이러한 작업을 일시 중지하고 나중에 다시 시작할 수 있다.
  - 이전에 했던 작업을 다시 재사용하거나 필요하지 않은 경우에는 폐기할 수 있다.
  - **한 가지 더 중요한 점은, 위 과정이 모두 `비동기` 로 일어난다는 것이다.**
- 과거 React의 조정 알고리즘은 스택 알고리즘이었다. 하나의 스택에 렌더링에 필요한 작업들이 쌓이면 스택이 빌 때까지 동기적으로 작업이 이루어졌다. JS의 특징인 싱글스레드로 인해 동기 작업은 다른 작업을 수행하고 싶어도 중단할 수 없었다.
  - 예를 들어 `<input type="text" />` 의 입력값을 자동으로 검색하는 UI가 있다고 하면, 검색어에 따라 결과물이 input 뿐만 아니라 다른 UI나 내부 fetch에도 영향을 줄 수 있다.
  - fetch작업으로 인한 네트워크 요청, 로딩 스피너 렌더링 등 스택에 쌓이는 작업이 많아질 수록 작업에 많은 시간이 소요되고, 최악의 경우 글자 입력에까지 문제가 생길 것이다.
  - 사용자 인터렉션에 의한 동시다발적 이벤트와 애니메이션이 필수인 요즘 앱에서는 피할 수 없는 문제다.
- 그러한 기존 랜더링 스택의 비효율성을 타파하기 위해, React 팀은 스택 조정자 대신 **Fiber**를 탄생시켰다.

### 파이버의 구현

- 파이버는 기본적으로 하나의 작업 단위로 구성돼있다.
- React는 이러한 작업 단위를 하나씩 처리하고 `finishedWork()` 라는 작업으로 마무리한다.
- 그리고 이 작업을 커밋해 실제 브라우저 DOM에 가시적인 변경 사항을 만들어낸다.
- 위 단계를 **두 단계로 정리**하면 아래와 같다.
  1. Render Phase
     - 사용자에게 노출되지 않는 모든 비동기 작업을 수행한다.
     - 앞서 언급했던 파이버의 작업에 대해 우선순위를 지정하거나 중지시키거나 버리는 등의 작업이 일어난다.
  2. Commit Phase
     - DOM에 실제 변경사항을 반영하기 위한 작업.
     - `commitWork()` 가 실행되는데, 이 과정은 **동기식으로 일어나고 중단될 수도 없다.**
  - 위 두 단계에 대해서는 `2.4절에서 더 자세히 설명` 한다.
- 아래는 리액트 18.2.0 버전 코드에 작성돼있는 **파이버 객체** 이다. 보다시피 **단순한 JS 객체**임을 확인할 수 있다.

  ```jsx
  function FiberNode(tag, pendingProps, key, mode) {
    // Instance
    this.tag = tag;
    this.key = key;
    this.elementType = null;
    this.type = null;
    this.stateNode = null;

    //Fiber
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

    //Effects
    this.flags = NoFlags;
    this.subtreeFlags = NoFlags;
    this.deletions = null;

    this.lanes = NoLanes;
    this.childLanes = NoLanes;

    this.alternate = null;

    // 이하 프로파일러, __DEV__ 코드 생략
  }
  ```

### 파이버와 리액트 엘리먼트(가상 DOM 노드)와의 차이점

파이버는 리액트 엘리먼트(가상 DOM 노드)와 유사하다고 느낄 수 있지만 차이점이 있다.

- 리액트 엘리먼트는 렌더링이 발생할 때마다 새롭게 생성되지만, **파이버는 가급적이면 재사용**된다.
- 파이버는 **컴포넌트가 최초로 마운트되는 시점에 생성**되어, **이후에는 가급적 재사용**된다.
- _그리고 파이버에는 가상 DOM 노드와 달리 hook 관련 정보가 포함된다._

> **⚠️리액트 엘리먼트(React Element)란?**
>
> - React 엘리먼트는 UI를 구성하는 가장 작은 단위이며, 화면에 그릴 내용을 설명하는 JS 객체다.
> - `createElement` 로 만들어지는 **불변 객체**로, DOM 노드나 컴포넌트 인스턴스가 아니다.
>   - createElement로는 가상 DOM 노드(React Element)가 만들어진다.
>
> ```jsx
> const el = <h1>Hello</h1>;
> // 내부적으로는:
> // React.createElement('h1', null, 'Hello')
> ```
>
> **리액트 컴포넌트(함수, 클래스)**를 통해 만들어진다.

### 파이버를 만드는 함수 및 Fiber 객체의 속성들

리액트에는 파이버를 만드는 다양한 함수들이 있는데, 아래와 같다. (이 또한 18.2.0 버전 기준)

```jsx
var createFiber = function (tag, pendingProps, key, mode) {
	return new FiberNode(tag, pendingProps, key, mode)
}

// 생략...
function createFiberFromElement(element, mode, lanes) {
	var owner = null

	{
		owner = element._owner
	}

	var type = element.type;
	var key = element.key;
	var pendingProps = element.props
	var fiber = createFiberFromTypeAndProps(
		fiber._debugSource = element._source
		fiber._debugOwner = element._owner
	)

	return fiber
}

function createFiberFromFragment(elements, mode, lanes, key) {
	var fiber = createFiber(Fragment, elements, key, mode)
	fiber.lanes = lanes
	return fiber
}
```

위 코드에서는 앞서 언급했던 `element` 와 `Fiber` 의 공통점, 즉 `1:1` 관계를 확인할 수 있다.

주요 속성들은 다음과 같다.

- tag
  - 파이버와 엘리먼트는 하나의 element에 하나의 fiber가 생성되는 1:1 관계이다. 여기서 1:1 로 매칭된 정보를 갖고 있는 것이 바로 tag다.
  - 1:1로 연결되는 것은 리액트 컴포넌트일 수도, HTML DOM 노드일 수도, 혹은 다른 어떤 것일 수도 있다. Fiber와 tag를 통해 1:1로 매칭될 수 있을지도 모르는 객체*(객체가 아닌 것도 있을 지는 잘 모르겠음)* 들의 종류는 다음과 같다.
    ```jsx
    var FunctionComponent = 0;
    var ClassComponent = 1;
    var IndeterminateComponent = 2;
    var HostRoot = 3;
    var HostPortal = 4;
    var HostComponent = 5;
    var HostText = 6;
    var Fragment = 7;
    var Mode = 8;
    var ContextConsumer = 9;
    var ContextProvider = 10;
    var ForwardRef = 11;
    var Profiler = 12;
    var SuspenseComponent = 13;
    var MemoComponent = 14;
    var SimpleMemoComponent = 15;
    var LazyComponent = 16;
    var IncompleteClassComponent = 17;
    var DehydratedFragment = 18;
    var SuspenseListComponent = 19;
    var ScopeComponent = 21;
    var OffscreenComponent = 22;
    var LegacyHiddenComponent = 23;
    var CacheComponent = 24;
    var TracingMarkerComponent = 25;
    ```
    - 여기서 HostComponent가 바로 웹의 div 같은 요소를 의미한다.
- stateNode
  - 이 속성에는 파이버 자체에 대한 참조(reference) 정보를 갖고 있다. 이 참조를 바탕으로 리액트는 파이버와 관련된 상태에 접근한다.
- child, sibling, return
  - 파이버 간의 관계 개념을 나타내는 속성이다.
  - 리액트 컴포넌트 트리가 형성되는 것과 동일하게 파이버도 트리 형식을 갖는데, 이 트리 형식을 구성하는 데 필요한 정보가 이 속성 내부에 정의된다. **한 가지 리액트 컴포넌트 트리와 다른 점**은 children이 없다는 것이다. → 즉, **하나의 child만 존재**한다.
    > ⚠️ 파이버의 자식은 항상 첫 번째 자식의 참조로 구성된다. 아래 코드 구조가 파이버로 만들어지면, `<ul>` 파이버의 자식(child)은 첫 번째 `<li>` 파이버가 된다.
    > 그리고 나머지 두 개의 `<li>` 파이버는 형제, 즉 `sibling`으로 이어진다.
    > 마지막으로 `return` 은 부모 파이버를 의미한다.
    > 아래 예에서 모든 `<li>` 파이버는 `<ul>` 파이버를 `return` 으로 갖게 된다.
    >
    > ```jsx
    > <ul>
    >   <li>하나</li> // l1
    >   <li>둘</li> // l2
    >   <li>셋</li> // l3
    > </ul>;
    > // 해당 엘리먼트를 JS 로 파이버 노드 간의 관계를 정리하면 다음과 같다.
    > const l3 = {
    >   return: ul,
    >   index: 2,
    > };
    >
    > const l2 = {
    >   sibling: l3,
    >   return: ul,
    >   index: 1,
    > };
    >
    > const l1 = {
    >   sibling: l2,
    >   return: ul,
    >   index: 0,
    > };
    >
    > const ul = {
    >   // ...
    >   child: l1,
    > };
    > ```
- index : 여러 형제(sibling)들 사이에서 자신의 위치가 몇 번째인 지 숫자로 표현
- pendingProps : 아직 작업을 미처 처리하지 못한 props
- memoizedProps : pendingProps를 기준으로 렌더링이 완료된 이후에 pendingProps를 memoizedProps로 저장해 관리한다. _→ 작업 완료된 props라고 보면 되나?_
- updateQueue :
  - 상태 업데이트, 콜백 함수, DOM 업데이트 등 필요한 작업을 담아두는 큐.
  - 이 큐는 대략 다음과 같은 구조다.
    ```jsx
    type UpdateQueue = {
    	first : Update | null
    	last : Update | null
    	hasForceUpdate : boolean
    	callbackList : null | Array<Callback> //setState로 넘긴 콜백 목록
    }
    ```
- memoizedState
  - 함수 컴포넌트의 훅 목록이 저장된다.
  - 단순히 useState 뿐만 아니라 **모든 훅 리스트가 저장**된다.
- alternate
  - 뒤이어 설명할 리액트 파이버 트리와 이어질 개념.
  - **리액트의 트리는 두 개**인데, 이 alternate는 반대편 트리 파이버를 가리킨다.

### 리액트 파이버 정리

- 위에서 본 것과 같은 구조로 생성된 파이버는 **state가 변경**되거나, **생명주기 메소드가 실행**되거나, **DOM의 변경이 필요한 시점** 등에 실행된다.
- 중요한 점은 리액트가 파이버를 처리할 때마다 이러한 작업을 직접 처리하기도 하고 스케쥴링하기도 한다는 것이다. 즉, 이러한 작업들은 **작은 단위로 나눠서 처리할 수도, 애니메이션과 같이 우선순위가 높은 작업부터 처리하거나, 낮은 작업을 연기시키는 등 좀 더 유연하게 처리될 수 있다.**
  - 이런 식으로 우선순위를 따져 컴포넌트를 렌더링하는 작업을 `동시성 렌더링`이라고 부르는 것 같다.(P 178)
- 리액트 개발 팀은 사실 리액트는 가상 DOM이 아닌 **Value UI, 즉 값을 가지고 있는 UI 라이브러리**라는 내용을 피력한 바 있다.
  → 파이버 객체 구조에서도 알 수 있듯이, **리액트의 핵심 원칙은 UI를 문자열, 숫자, 배열과 같은 값으로 관리한다는 것**이다. 변수에 UI 관련 값을 보관하고, JS 코드 흐름에 따라 이를 관리하고 표현하는 것이 바로 React다.

## 리액트 파이버 트리 (React Fiber Tree)

파이버 트리는 리액트 내에 **두 개 존재**한다.

- Current Tree : 현재 모습을 담은 트리
- workInProgress Tree : 작업 중인 상태를 나타내는 트리

리액트 파이버의 작업이 끝나면 리액트는 **단순히 포인터만 변경해서 workInProgress 트리를 Current 트리로 바꿔**버린다.

이러한 기술을 **`더블 버퍼링`**이라고 한다.

> ### ⚠️더블 버퍼링(Double Buffering)
>
> 더블 버퍼링은 컴퓨터 그래픽 분야에서 사용하는 용어다.
>
> 화면에 표시할 것을 그래픽으로 그리기 위해서는 내부적으로 처리를 거쳐야 하는데, 그 과정에서 사용자에게 미처 다 그리지 못한 모습을 노출하게 되는 경우가 있다.
>
> (한 번에 모든 작업을 전부 마무리해 그릴 수 없기 때문이다.)
>
> 이런 상황을 방지하기 위해 보이지 않는 곳에서 그림을 미리 그려두고, 완성됐을 때 화면 요소를 (완성된) 새 그림으로 바꾸는 기법을 더블 버퍼링이라고 한다.
>
> 리액트의 경우는 아직 작업이 다 끝나지 않은 불완전한 트리를 보여주지 않기 위해 **더블 버퍼링**을 사용하며, 이 작업은 `Commit Phase` 에 수행된다.
>
> 더블 버퍼링 작업 순서는 다음과 같다.

1. 먼저 현재 UI 렌더링을 위해 존재하는 Current Tree를 기준으로 모든 작업이 시작된다.
2. 1번에서 만약 업데이트가 발생하면, 파이버는 새로 받은 데이터로 새로운 workInProgress Tree를 빌드하기 시작한다.
3. workInProgress Tree 빌드가 끝나면 다음 렌더링에 이 트리를 사용한다.
4. workInProgress Tree가 최종적으로 렌더링되어 실제 DOM에 반영이 완료되면 current Tree를 가리키는 참조가 workInProgress Tree를 가리켜 workInProgress Tree가 새로운 current Tree가 된다.

### 파이버의 작업 순서

일반적인 파이버 노드의 생성 흐름은 다음과 같다.

1. 리액트는 `beginWork()` 함수를 실행해 파이버 작업을 수행하는데, 더 이상 자식이 없는 파이버를 만날 때까지 트리 형식으로 시작된다.
2. 1번 작업이 끝나면 그다음 `completeWork()` 함수를 실행해 파이버 작업을 완료한다.
3. 형제가 있다면 형제로 넘어간다.
4. 2번, 3번이 모두 끝났다면 `return` 으로 돌아가 자신의 작업이 완료됐음을 알린다.

아래의 JSX 코드는 다음과 같은 순서로 수행된다.

```jsx
<A1>
  <B1></B1>
  <B2>
    <C1>
      <D1 />
      <D2 />
    </C1>
  </B2>
</A1>
```

1. A1의 beginWork() 가 수행된다.
2. A1은 자식이 있으므로, `child` 가 가리키는 B1로 이동해 beginWork()를 수행한다.
3. B1은 자식이 없으므로 completeWork() 가 수행된다. 이어 형제인 B2로 넘어간다.
4. B2의 beginWork()가 수행된다. 자식이 있으므로 C1로 넘어간다.
5. C1의 beginWork()가 수행된다. 자식이 있으므로 D1로 넘어간다.
6. D1의 beginWork()가 수행된다. 자식이 없으므로 completeWork()를 수행한다.
7. 형제인 D2로 넘어와 마찬가지로 completeWork()를 수행한다.
8. C1의 completeWork()가 수행된다.
9. B2의 completeWork()가 수행된다.
10. A1의 completeWork()가 수행된다.
11. 위와 같은 형식으로 루트 노드가 완성되는 순간, 최종족으로 `commitWork()` 가 수행되고 이 중 변경 사항을 비교해 업데이트가 필요한 변경사항이 DOM에 반영된다.

    _위 과정은 workInProgress 트리를 업데이트하는 작업으로 이해하였음. 해당 내용에서 말하는 ‘변경 사항을 비교해 업데이트가 필요한 변경 사항을 반영’ 한다는게, diff 알고리즘 외에 React 렌더링 파이프라인 상 workInProgress 트리와 실제 DOM을 비교하는 작업이 한 단계 더 있다는 뜻인 지, 아니면 브라우저 렌더링 파이프라인에서의 작업을 뜻하는 것인 지 헷갈림. 일단 후자로 추정 중…_

### setState 등으로 업데이트 발생

- 이미 리액트는 앞서 만든 current 트리가 있으므로, setState로 인한 업데이트 요청을 받아 `workInProgress` 트리를 다시 빌드하기 시작한다. 이 과정은 앞서 설명한 `가상DOM 생성~Fiber 작업` 까지 동일하다.
- 최초 렌더링 시에는 모든 파이버를 새롭게 만들어야 했지만, 두 번째부터는 파이버가 이미 존재하므로 되도록 새로 생성하지 않고 기존 파이버에서 업데이트된 props들을 받아 **`파이버 내부에서 처리`한다.**
  - 앞서 파이버를 설명할 때 언급한 `가급적 새로운 파이버를 생성하지 않는다` 의 이유다.
  - 과거 스택 구조였을 당시에는 해당 작업을 동기적으로 처리했다. 여태까지 설명한 **트리를 업데이트하는 과정**을 재귀적으로 **동기 처리**해가며 중단할 수 없었다.
  - 하지만 현재의 리액트는 그러한 작업들을 파이버 단위로 나눠 수행하므로, **우선순위에 따른 순서**로 작업을 수행하거나 **비동기 작업**이 가능하다.

## 파이버와 가상 돔

- 리액트 컴포넌트에 대한 정보를 1:1로 가지고 있는 것이 파이버이며, 이 **파이버는 리액트 아키텍쳐 내부에서 비동기**로 이뤄진다.
- 하지만 실제 브라우저 구조인 DOM에 반영하는 작업은 동기적으로 일어나야 하고, 또 처리하는 작업이 많아 화면에 불완전하게 표시될 수 있는 가능성이 높으므로 이러한 작업을 가상에서, 즉 메모리에서 먼저 수행해 최종적인 결과물만 실제 브라우저 DOM에 적용하는 것이다.
- 본문에서는 계속 `가상 DOM` 이라는 표현을 썼지만 엄밀히 말하자면 이는 오직 웹 어플리케이션에서만 통용되는 개념이다. 리액트 파이버는 리액트 네이티브처럼 브라우저가 아닌 환경에서도 사용할 수 있기 때문에, **파이버와 가상 DOM은 동일한 개념이 아니다**.
- 리액트와 리액트 네이티브의 렌더러가 서로 다르더라도 내부적으로 파이버를 통해서 조정되는 과정은 동일하기 때문에, 동일한 재조정자(Reconciler)를 사용할 수 있게 된다.

## 정리

가상 DOM과 파이버는 단순히 브라우저에서 DOM을 변경하는 작업보다 빠르다는 이유로만 만들어지지 않았다. 개발자가 일일의 DOM 요소를 수정하는 수고로움을 덜어 대규모 웹 애플리케이션을 효율적으로 유지보수하고 관리하기 위한 목적이 더 크다.

가상 DOM과 리액트의 핵심은 브라우저의 DOM을 더욱 빠르게 그리고 반영하는 것이 아닌, **값으로 UI를 표현하는 것** 이다.

- **화면에 표시되는 UI를 JS의 문자열, 배열 등과 마찬가지로 값으로 관리하고 이러한 흐름을 효율적으로 관리하기 위한 매커니즘이 바로 React의 핵심이다.**

# 클래스 컴포넌트와 함수 컴포넌트

함수 컴포는트는 의외로 리액트 0.14 버전부터 만들어진 역사가 깊은 컴포넌트 선언 방식이다. 당시에는 stateless functional component(무상태 함수 컴포넌트)라고 해서 별도의 상태 없이 단순히 어떠한 요소를 정적으로 렌더링하는게 목표였다.

함수 컴포넌트가 각광받기 시작한 것은 16.8 버전에서 `Hook` 이 소개된 이후였다.

## 클래스 컴포넌트

요즘은 대부분 함수 컴포넌트를 사용하지만, React 16.8 버전 미만으로 작성된 코드에서는 클래스 컴포넌트가 대다수일 것이다. 따라서 오래된 코드의 유지보수를 위해서는 클래스 컴포넌트를 알아둘 필요가 있다.

- 클래스 컴포넌트는 클래스를 선언한 후 `React.Compoent` 혹은 `React.PureComponent`를 상속받아야 했다.
  > ### ⚠️Component vs PureComponent
  >
  > - Component : setState가 호출될 때마다 업데이트 발생
  > - PureComponent : setState로 배정된 값과 기존 state를 얕은 비교 하여 달라졌을 때만 업데이트 발생 → 객체 같은 복잡한 구조의 데이터는 변경 감지를 하지 못해 신중하게 사용해야 함.
    </aside>

```jsx
import React from 'react'

//props 타입 선언
interface SampleProps {
	required? : boolean
	text : string
}

//state 타입을 선언
interface SampleState {
	count : number
	isLimited? : boolean
}

// Component에 제네릭으로 props, state를 순서대로 넣어준다.
class SampleComponent extends React.Component<SampleProps, SampleState> {
	//contructor에서 props를 넘겨주고, state의 기본값을 설정한다.
	private constructor(props : SampleProps) {
		super(props)
		this.state = {
			count : 0,
			isLimited : false,
		}
	}

	// render 내부에서 쓰일 함수를 선언한다.
	private handleClick = () => {
		const newValue = this.state.count+1
		this.setState({ count : newValue, isLimited : newValue >=10 })
	}

	// render에서 이 컴포넌트가 렌더링할 내용을 정의한다.
	public render() {
		// props와 state 값을 this, 즉 해당 클래스에서 꺼낸다.
		const {
			props: { required, text },
			state : { count, isLimited },
		} = this

		return (
			<h2>
				Sample Component
				<div>{ required ? '필수' : '필수아님' }</div>
				<div>문자: { text }</div>
				<div>count: { count }</div>
				<button onClick={this.handleClick} disabled={isLimited}>
					증가
				</button>
			</h2>
		)
	}
}
```

- 위 예제에서 props는 `{ required? : boolean; text : string }` 로 되어있으므로, 해당 컴포넌트를 호출하기 위해서는 `<SampleComponent text="안녕하세요" />` 같은 형태가 될 것이다.
- state는 클래스 컴포넌트 내에서 관리하는 갓ㅂ이며, 항상 객체여야 한다.
- 메소드를 만드는 방법은 크게 세 가지다.
  - contructor에서 this 바인드
  - 화살표 함수를 쓰는 방법
  - 렌더링 함수 내에서 함수를 새롭게 만들어 전달 (렌더링 시마다 새로 만들어지므로 지양)

### 클래스 컴포넌트의 생명주기 메소드

클래스 컴포넌트의 생명주기 메소드가 실행되는 시점은 크게 3가지다.

- 마운트(mount) : 컴포넌트가 마운팅(생성)되는 시점
- 업데이트(update) : 이미 생성된 컴포넌트의 내용이 변경(업데이트)되는 시점
- 언마운트(unmount) : 컴포넌트가 더 이상 존재하지 않는 시점

#### render()

- 클래스 컴포넌트의 유일한 필수값.
- 컴포넌트 UI를 렌더링하기 위해 쓰이며, 마운트아ㅗ 업데이트 과정에서 일어난다.
- 순수함수여야 하므로 render 내에서 `setState` 같이 상태를 변경시키는 작업을 해서는 안된다.

#### componentDidMount()

- 클래스 컴포넌트가 마운트되고 준비가 됐다면, 그 다음으로 호출되는 생명주기 메소드.
- 컴포넌트가 마운트되고 즉시 실행된다.
- setState로 state 변경 시 즉시 다시한번 렌더링이 시도된다. 하지만 브라우저 렌더링이 시작되기 전에 처리되므로 사용자에게는 보이지 않는다.
  - 하지만 성능 문제를 일으킬 수 있으므로 지양하자. 생성자에서 다룰 수 없는 state 작업만 처리하자.
    - 이벤트리스너 추가, API 호출 후 업데이트 등

#### componentDidUpdate()

- 컴포넌트 업데이트가 일어난 후 바로 실행된다.
- state나 props의 변화에 따라 DOM을 업데이트하는 등에 쓰인다.
- 적절한 조건문을 사용하지 않으면 컴포넌트 렌더링이 무한히 발생할 수 있다.
  - setState→componentDidUpdate→setState→componentDidUpdate의 반복…

#### componentWillUnmount()

- 컴포넌트가 언마운트되거나 더이상 사용되지 않기 직전에 호출된다.
- 메모리 누수나 불필요한 작동을 막기 위한 **클린업 함수**를 호출하기 위한 최적의 위치다.
- 당연하지만 setState 호출 불가능하다.

#### shouldComponentUpdate()

- state나 props의 변경으로 리액트 컴포넌트가 다시 리렌더링되는 것을 막고 싶을 때 사용하면 된다.
- 기본적으로 this.setState를 호출하면 컴포넌트는 리렌더링을 일으킨다. 하지만 해당 메소드를 사용하면 **컴포넌트에 영향을 받지 않는 변화에 대해 정의**할 수 있다.
- 하지만 state에 따라 컴포넌트가 변하는 것은 자연스러운 일이므로, 신중하게 사용하자.

#### static getDerivedStateFromProps()

- 가장 최근에 도입된 생명주기 메소드 중 하나로, 지금은 사라진 `componentWillReceiveProps`를 대체할 수 있는 메소드다.
- render() 호출 직전에 호출되며 static으로 선언돼있어 this로 접근 불가능하다.
- 해당 메소드에서 반환되는 객체는 해당 객체의 내용이 모두 state로 들어가게 된다.
- 다음에 올 props를 바탕으로 현재의 state를 변경하고 싶을 때 사용할 수 있다.

#### getSnapShotBeforeUpdate()

- 최근에 도입된 생명주기 메소드 중 하나로, `componentWillUpdate`를 대체할 수 있다.
- DOM 업데이트 직전에 호출되고, 여기서 반환하는 값은 `componentDidUpdate` 로 전달된다.
- DOM이 렌더링되기 전에 윈도우 크기를 조절하거나 스크롤 위치를 조정하는 등의 작업을 처리하는데 유용하다.
  - 이후 계산된 위치 따위를 `componentDidUpdate` 로 넘겨줘 활용하는 식

#### getDerivedStateFromError()

- 에러 상황에서 실행되는 메소드다.
- React 18 기준 아직 리액트 훅으로 구현돼있지 않다. (추가할 예정이라고 언급했지만, 아직 구체적 일정 미정)
- 자식 컴포넌트에서 에러가 발생했을 때 호출되는 에러 메소드다. 이 에러 메소드를 사용하면 적절한 에러 처리 로직을 구현할 수 있다.
- 반드시 state 값을 반환해야하는데, 하위 컴포넌트에서 에러가 발생했을 경우 어떻게 자식 리액트 컴포넌트를 렌더링할지 결정하는 용도이기 때문에, **반드시 미리 정의해둔 state를 반환**해야 자식 컴포넌트가 해당 state를 받아 제대로 렌더링된다.
- 렌더링 과정에서 호출되는 메소드이기 때문에 부수효과를 발생시켜서는 안된다. (console.error 등 에러 로깅 포함)

#### componentDidCatch

- 마찬가지로 에러 상황에서 실행되며, 자식 컴포넌트에 에러가 발생했을 때 실행된다.
- `getDerivedStateFromError()` 에서 에러를 잡고 state를 결정한 이후 실행된다.
- 두 개의 인수를 받는데, 첫 번째는 `getDerivedStateFromError()` 와 동일한 `error`, 그리고 정확히 어떤 컴포넌트가 에러를 발생시켰느지 정보를 갖고 있는 `info` 다.
- `getDerivedStateFromError()`에서 하지 못한 부수효과를 여기서 발생시키면 된다.
- info 에서 표시되는 컴포넌트 이름은 Function.name 또는 displayName을 따르기 때문에 memo로 감싸고 따로 이름 지정을 안해주면 어떤 컴포넌트인지 제대로 알기 어렵다.

### 클래스 컴포넌트의 한계

클래스 컴포넌트는 그 자체로 하나의 앱을 완성시키기에 무리가 없지만, 함수 컴포넌트에 훅을 도입한 새로운 패러다임이 나온 이유는 다음과 같이 추측된다.

- 데이터의 흐름을 추적하기 어렵다.
  - 생명주기 메소드마다 역할이 다 다르며, 작성 순서가 정해져있지 않기 때문에 개발자가 직접 정리해놓지 않는 이상 읽기가 어렵다. state 가 변경될 수 있는 위치도 많아서 숙련된 개발자라도 state가 어떤 흐름으로 변경돼서 렌더링이 일어나는 지 파악하기 어렵다.
- 애플리케이션 내부 로직의 재사용이 어렵다.
  - 컴포넌트 간에 중복되는 로직이 있고, 이를 재사용하고 싶을 경우 컴포넌트를 다른 고차 컴포넌트로 감싸거나 props로 넘겨줘야하는데, 공통로직이 많아질수록 레퍼 지옥(Wrapper hell)에 빠져들 위험성이 커진다.
- 기능이 많을 수록 컴포넌트 크기가 커진다.
  - 컴포넌트 로직이 많을 수록, 내부에서 처리하는 데이터 흐름이 복잡해져 생명주기 메소드 사용이 잦아지는 경우 컴포넌트 크기가 기하급수적으로 증가한다.
- 클래스는 함수에 비해 상대적으로 어렵다.
  - JS 사용자는 기본적으로 Class에 익숙하지 않은 경향이 있다.
- 코드를 최적화하기 어렵다.
  - 클래스 컴포넌트는 최종 결과물인 번들 크기를 줄이는데 어려움을 준다. (사용하지 않는 메소드에 대해 트리 쉐이킹이 안돼서 코드에 그대로 남는다.)
- 핫 리로딩에 상대적으로 불리하다.

## 함수 컴포넌트 vs 클래프 컴포넌트

함수 컴포넌트는 16.8 버전에서 훅이 등장하면서 리액트 개발자들에게 각광받고 있다. 클래스 컴포넌트에 비해 간결하고, this 바인딩을 조심할 필요가 없으며, state가 각각의 원시값으로 관리되어 훨씬 사용하기 편해졌다.

그 외에 두 컴포넌트 사이에는 또 어떤 차이점이 있을까?

### 생명주기 메소드의 부재

- 앞서 살펴본 생명주기 메소드가 함수 컴포넌트에는 존재하지 않는다.
  - 함수 컴포넌트는 props를 받아 단순히 리액트 요소만 반환하는 함수인 반면, 클래스 컴포넌트는 render 메소드가 있는 React.Component를 상속받아 구현하는 JS 클래스이기 때문이다.
  - 즉, 생명주기 메소드들은 React.Component에서 오는 것이기 때문에 이를 상속받을 수 없는 함수형 컴포넌트는 생명주기 메소드를 사용할 수 없다.
  - 대신 useEffect를 사용해 `componentDidMount`, `componentDidUpdate`, `componentWillUnmount` 를 비슷하게 구현할 수 있다.
    - 하지만 비슷할 뿐 절대 똑같지 않다. useEffect는 생명주기 관리르 위한 훅이 아니라, state를 활용해 동기적으로 부수효과를 만드는 매커니즘이다.

### 함수 컴포넌트와 렌더링된 값

- 함수 컴포넌트는 렌더링된 값을 고정하고, 클래스 컴포넌트는 고정하지 못한다. 이는 리액트 개발자 댄 아브라모프가 블로그에서 이야기한 유명한 내용이다.
- 클래스 컴포넌트는 props의 값을 항상 this로부터 가져온다. 클래스 컴포넌트의 props는 외부에서 변경되지 않는 이상 불변 값이지만 this가 가리키는 객체, 즉 컴포넌트의 인스턴스 멤버는 변경 가능한(mutable) 값이다.
- 따라서 render 메소드를 비롯한 리액트의 생명주기 메소드가 변경된 값을 읽을 수 있게 된다.
- 함수 컴포넌트는 렌더링이 일어날 때마다 그 순간의 값인 props와 state 기준으로 렌더링된다. 반면 클래스 컴포넌트는 시간의 흐름에 따라 변화하는 this를 기준으로 렌더링이 일어난다.

# 렌더링은 어떻게 일어나는가?

- 리액트의 렌더링에 소요되는 시간과 리소스는 전부 웹 애플리케이션에 방문한 사용자가 부담하기 때문에, 리액트의 렌더링 과정을 이해하는 것은 중요하다.

## 리엑트의 렌더링이란?

`렌더링`이라는 단어는 브라우저에서도 사용하므로 리액트의 렌더링과 브라우저의 렌더링을 혼동해서는 안된다.

- 리액트에서의 렌더링이란, `리액트 애플리케이션 트리 안에 있는 모든 컴포넌트들이 현재 자신들이 가지고 있는 props와 state 값을 기반으로 어떻게 UI를 구성하고 이를 바탕으로 **어떤 DOM 결과를 브라우저에 제공할 것인 지 계산하는 일련의 과정**` 이다.
- 요약하면, **`브라우저가 렌더링에 필요한 DOM 트리를 만드는 과정`** 이라고 할 수 있다.
  - 만약 컴포넌트가 props와 state같은 상태값을 가지고 있지 않다면 오직 해당 컴포넌트가 반환하는 JSX 값에 기반해 렌더링이 일어나게 된다.

### 리엑트에서 렌더링이 일어나는 이유

리액트에서 렌더링이 발생하는 시나리오는 다음과 같다.

1. 최초 렌더링

- 사용자가 처음 애플리케이션에 진입했을 때, 브라우저에 렌더링 결과물을 제공하기 위해 최초 렌더링 수행

2. 리렌더링

- 최초 렌더링이 발생한 이후 발생하는 모든 렌더링.
- 리액트에서 렌더링이 일어나는 시나리오는 아래의 목록들 뿐이다.
  - 클래스 컴포넌트에서 `setState` 가 실행되는 경우
    - state의 변화는 상태의 변화를 의미. 클래스 컴포넌트에서는 state 변화를 setState를 호출하여 수행하므로 리렌더링 발생
  - 클래스 컴포넌트에서 `forceUpdata` 가 실행되는 경우
  - 함수 컴포넌트에서 `useState()`의 두 번째 배열 요소인 `dispatch`가 실행되는 경우
  - 함수 컴포넌트에서 `useReducer()` 의 두 번째 배열 요소인 `dispatch`가 실행되는 경우
  - 컴포넌트의 `key props`가 변경되는 경우
    - 리애그에서 `key`는 명시적으로 선언돼 있지 않아도 모든 컴포넌트에서 사용할 수 있는 특수한 props다.
    - key는 왜 필요한가?
      → key는 리렌더링이 발생하는 동안 형제 요소들 사이에서 동일한 요소를 식별하는 값이다.
      → 앞서 살펴봤던 Fiber Tree에서 current 를 순회하며 변경된 workInProgress를 업데이트해야하는데, 형제 노드들은 sibling 참조로 이어져 있어 이들을 구분할 다른 키 값이 필요하며, 위의 key가 바로 그것이다.
  - `props`가 변경되는 경우
    - 부모로부터 전달받는 값인 `props` 가 달라지면 이를 사용하는 자식 컴포넌트에서도 변경이 필요하므로 리렌더링이 발생한다.
  - 부모 컴포넌트가 리렌더링 될 경우
    - 한 가지 주의할 점은, 부모 컴포넌트가 리렌더링된다면 **자식 컴포넌트도 무조건 리렌더링이 일어난다는 것**이다.
- MobX, Redux 같은 상태관리 라이브러리들도 위에서 언급한 방법 중 하나를 사용해 리렌더링을 발생시킨다. Recoil의 경우는 내부에서 useState 등을 사용해 리렌더링을 발생시킨다.

## 리액트의 렌더링 프로세스

- 렌더링 프로세스가 시작되면 리액트는 컴포넌트의 루트에서부터 아래쪽으로 내려가면서 업데이트가 필요하다고 지정돼 있는 모든 컴포넌트를 찾는다.
- 그 과정에서 업데이트가 필요하다고 지정돼 있는 컴포넌트를 발견하면 (함수 컴포넌트의 경우) FunctionComponent() 그 자체를 호출한 뒤에, 결과물을 저장한다.
- 일반적인 렌더링 결과물은 JSX 문법으로 구성돼있고, 자바스크립트로 컴파일되면서 React.createElement()를 호출하는 구문으로 변환된다.
- 이후 createElement()는 브라우저의 **UI 구조를 설명할 수 있는 일반적인 JS 객체**를 반환한다.

  ```jsx
  // JSX
  function Hello() {
  	return (
  		<TestComponent a={35} b="yceffort">
  			안녕하세요
  		</TestComponent>
  	)
  }

  // babel로 트랜스 파일링
  function Hello() {
  	return (
  		React.createElement(
  			TestComponent,
  			{a : 35, b : "yceffort"},
  			"안녕하세요"
  		)
  	)
  }

  // 반환된 객체
  {
  	type : TestComponent,
  	props : {
  		a : 35,
  		b : "yceffort"
  		children : "안녕하세요"
  	}
  }
  ```

- 위와 같은 과정을 거쳐 각 컴포넌트 렌더링의 결과물을 수집한 다음, 리액트의 새로운 트리인 가상 DOM과 비교해 실제 DOM에 반영하기 위한 모든 변경 사항을 차례대로 수집한다.
- 이렇게 계산하는 과정을 **`재조정(Reconciliation)`** 이라고 한다. 이러한 재조정 과정이 모두 끝나면 모든 **변경 사항을 하나의 동기 시퀀스로 DOM에 적용**해 변경된 결과물이 보이게 된다.

## 렌더와 커밋

**렌더 단계(Render Phase)**는 컴포넌트를 렌더링하고 변경 사항을 개선하는 모든 작업을 말한다.

- 렌더링 프로세스에서 컴포넌트를 실행해(return() 또는 return) 이 결과와 이전 가상 DOM을 비교하는 과정을 거쳐 변경이 필요한 컴포넌트를 체크하는 단계다.
- 여기서 비교하는 것은 크게 세 가지로, `type`, `props`, `key` 다.
  - 이 세 가지중 하나라도 변경된 것이 있으면 **변경이 필요한 컴포넌트로 체크**해둔다.

**커밋 단계(Commit Phase)**는 렌더 단계의 변경 사항을 실제 DOM에 적용해 사용자에게 보여주는 과정을 말한다.

- 리액트가 먼저 DOM을 커밋 단계에서 업데이트한다면 이렇게 만들어진 모든 DOM 노드 및 인스턴스를 가리키도록 리액트 내부의 참조를 업데이트한다.
  - _(아래 GPT 답변)_
  - _리액트가 먼저 DOM을 커밋 단계에서 업데이트한다 : 완성된 Fiber Tree 기반으로 실제 DOM을 변경하는 작업 수행_
  - _이렇게 만들어진 모든 DOM 노드 및 인스턴스를 가리키도록 리액트 내부의 참조를 업데이트 : 변경된 DOM의 참조를 Fiber 노드 stateNode 프로퍼티에 저장_
  - _업데이트된 Fiber는 flags 라는 변수에 저장한다고 함._
- 그 다음, 생명주기 개념이 있는 클래스 컴포넌트에서는 componentDidMount, componentDidUpdate 메소드를 호출하고, 함수 컴포넌트에서는 `useLayoutEffect 훅` 을 호출한다.
- 여기서 중요한 점은 리액트의 렌더링이 일어난다고 해서 **무조건 DOM 업데이트가 일어나지는 않는다는 것**이다.

  - 렌더링을 수행했으나 커밋 단계까지 갈 필요가 없다면, 즉 변경 사항을 계산했는데 아무런 변경 사항이 감지되지 않는다면 이 커밋 단계는 생략될 수 있다.
  - 즉, 렌더링 과정에서 **변경 사항을 감지할 수 없다면 커밋 단계가 전체 생략되어 브라우저의 DOM 업데이트가 일어나지 않을 수 있다**.

    ```jsx
    import { useState } from "react";

    export default function A() {
      return (
        <div className="App">
          <h1>Hello React!</h1>
          <B />
        </div>
      );
    }

    function B() {
      const [counter, setCounter] = useState(0);
      function handleButtonClick() {
        setCounter((previous) => previous + 1);
      }

      return (
        <>
          <label>
            <C number={counter} />
          </label>
          <button onClick={handleButtonClick}>+</button>
        </>
      );
    }

    function C({ number }: { number: number }) {
      return (
        <div>
          {number} <D />
        </div>
      );
    }

    function D() {
      return <>리액트 재밌다!</>;
    }
    ```

    - 위 코드의 경우 렌더링은 B, C, D 발생하지만, 실제 DOM 업데이트는 B와 C만 발생한다.
      - B는 setCounter가 호출되어서, C는 number 프롭스가 변경되어서 업데이트가 발생.
      - D는 memo로 감싸주면 리렌더링이 발생하지 않는다.

# 컴포넌트와 함수의 무거운 연산을 기억해 두는 메모이제이션

`useMemo`, `useCallback`, 그리고 고차 컴포넌트인 `memo`는 리액트에서 발생하는 렌더링을 최소로 줄이기 위해 제공된다.

메모이제이션 최적화는 리액트 커뮤니티에서 오랜 논쟁 주제 중 하나로, 메모이제이션 이야기가 나올 때마다 갑론을박이 이어지곤 한다.

### 주장 1. 섣부른 최적화는 독이다. 꼭 필요한 곳에만 메모이제이션을 해라.

- 메모이제이션도 어디까지나 비용이 드는 작업이므로 최적화에 대한 비용을 지불할 때는 항상 신중해야한다는 입장이다.
- 메모이제이션의 **비용**은 `값을 비교하고 렌더링 또는 재계산이 필요한지 확인하는 작업`, `결과물을 저장해 두었다가 다시 꺼내오는 것` 이다.
- 리렌더링을 다시 수행하는 비용과, 메모이제이션의 비용 중 어느 쪽이 더 비쌀 지는 상황에 따라 다를 것이다. 그렇기에 메모이제이션은 항상 신중하게 접근해야 하며 섣부른 최적화는 경계해야 한다.
  - 섣부른 최적화를 가리켜 영어권 커뮤니티에서는 `premature optimization` 또는 `premature memoization` 이라고 한다.

### 주장2. 렌더링 과정의 비용은 비싸다. 전부 메모이제이션 해버리자.

- memo를 필요한 곳에만 사용하는게 분명 좋겠지만, 프로젝트 규모가 커지다보면 이 기조를 꾸준히 유지하는 것도 일이다. 따라서 모든 컴포넌트를 memo로 감싸서, 최적화에 대해 고민하는 비용을 줄이자는 의견이다.
- 잘못된 memo로 지불하는 비용은 props에 대한 얕은 비교가 발생하면서 지불해야 하는 비용이다. 메모이제이션을 위해서는 CPU와 메모리를 사용해 이전 렌더링 결과물을 저장해둬야 하고, 리렌더링할 필요가 없다면 이전 결과물을 사용해야 한다.
- 하지만, 리액트는 이미 위의 구조를 가지고 있다. 어차피 리액트는 내부적으로 다음 렌더링을 위해 이전 DOM 결과를 저장해두기 때문이다. 따라서 `memo` 로 지불해야 하는 비용은 props에 대한 얕은 비교 알고리즘 뿐이다. (이 비용도 컴포넌트가 크고 복잡할 경우에는 무시할 수 없긴 하다.)
- `memo`를 하지 않았을 때 발생할 수 있는 문제는 다음과 같다.
  - 렌더링을 함으로써 발생하는 비용
  - 컴포넌트 내부의 복잡한 로직의 재실행
  - 위 두 가지 모두가 모든 자식 컴포넌트에서 반복해서 일어남
  - 리액트가 구 트리와 신규 트리를 비교
- 이처럼 `memo`를 하지 않았을 때 치러야 할 잠재적 위험이 더 크다. 따라서 비록 섣부른 초기화라 할지라도 했을 때 누릴 수 있는 이점, 그리고 이를 실수로 빠트렸을 때 치러야 할 위험 비용이 더 크기 때문에 **최적화에 대한 확신이 없다면 가능한 한 모든 곳에 메모이제이션을 활용한 최적화를 하는 것이 좋다**.

## 정리

- 이처럼 메모이제이션 활용법에 대한 주장에는 모두 그를 뒷받침할만한 이유가 있다. 저자가 추천하는 방식은, 리액트를 배우고 싶거나 깊게 이해하고 싶고, 이를 위한 여유가 있다면 1번 의견대로 섣부른 메모이제이션을 지양하며 성능상 이점을 체감해나가는 방식으로 사용할 것을 권장한다.
- 만약 현업에서 리액트를 사용하고 있거나 실제로 다룰 예정이지만 성능에 대해 깊게 연구해볼 여유가 없다면, 의심스러운 곳에는 다 적용해볼 것을 권장한다. lodash나 간단한 함수의 경우와는 다르게 일반적으로는 props에 대한 얕은 비교를 수행하는 것보다 리액트 컴포넌트의 결과물을 다시 계산하고 실제 DOM 까지 비교하는 작업이 너무 무겁고 비싸다.
- 성능에 대해 지속적으로 모니터링하고 관찰하는 것보다 섣부른 메모이제이션 최적화가 주는 이점이 더 클수 있다.
