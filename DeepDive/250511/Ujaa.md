## 서버 사이드 렌더링이란?

### SPA(Single Page Application)란?

- 하나의 HTML 파일에서 모든 UI 동작이 이루어지는 웹 애플리케이션 방식
- 라우팅 및 렌더링 로직을 서버가 아닌 브라우저와 자바스크립트가 담당
- 첫 페이지 로딩 시 모든 자바스크립트와 데이터를 불러온 후, 이후 페이지 전환은 JS와 `history` API를 통해 처리
- 서버로부터 HTML을 매번 새로 받지 않고, JS로 필요한 부분만 업데이트.

### SPA가 널리 퍼진 이유

- 과거에는 PHP, JSP 등 서버 사이드 렌더링(SSR)이 주류였음. 당시 자바스크립트는 한계가 있었고, 웹 앱도 서버 중심으로 동작.
- 그러나 SPA는 브라우저에서 많은 로직을 처리하면서 부분 렌더링 덕분에 페이지 전환 시 화면 깜박임 없이 부드러운 UX 제공. 또한, 백엔드(PHP 등)에 비해 프론트 중심 개발이 간단해짐. 그리고 프론트엔드 개발자 중심의 생산성과 유지보수성 향상.

### 서버 사이드 렌더링(SSR)이란?

- 초기 화면을 서버에서 HTML로 완성해 클라이언트에 전달.
- 빠른 초기 렌더링과 검색 엔진 최적화(SEO)에 유리.
- SPA의 단점을 보완하는 방식으로, Next.js 등에서 활용.

**장점**

- 최초 페이지 진입이 비교적 빠르다
- 검색 엔진과 SNS 공유 등 메타데이터 제공이 쉽다
- 누적 레이아웃 이동이 적다
- 사용자의 디바이스 성능에 비교적 자유롭다
- 보안에 좀 더 안전하다

**단점**

- 소스코드를 작성할 때 항상 서버를 고려해야 한다
- 적절한 서버가 구축돼 있어야 한다
- 서비스 지연에 따른 문제

### SPA vs MPA(일반적인 관점에서)

- 싱글 페이지가 멀티 페이지보다 사용자 경험이 좋다
- 싱글 페이지가 멀티 페이지보다 느리다.

## 서버 사이드 렌더링을 위한 리액트 API 살펴보기

### renderToString

- 리액트 컴포넌트를 HTML 문자열로 렌더링하는 가장 기본적인 함수.
- 리액트 컴포넌트의 초기 HTML을 생성
- 클라이언트에서 하이드레이션 가능 (이벤트 핸들러 연결 가능)
- 동기 방식으로 동작
- data-reactroot 속성 포함

```jsx
// server.js
const html = ReactDOMServer.renderToString(<App />);

// client.js
ReactDOM.hydrate(<App />, document.getElementById('root'));
```

### renderToStaticMarkup

- 정적 HTML만 생성하며, 하이드레이션이 필요하지 않은 경우에 사용
- 리액트 관련 속성 제거 (data-reactroot 등), `useEffect` 등 사용 불가
- 정적 페이지에 적합

```jsx
ReactDOMServer.renderToStaticMarkup(<Component/>);
```

### renderToNodeStream

- 스트림 방식을 사용해 청크 단위로 HTML을 전송
- 대용량 페이지에 적합
- 첫 바이트까지의 시간 단축
- Node.js 환경에서만 동작

### renderToStaticNodeStream

- 스트림 방식으로 정적 HTML을 생성.
- `renderToNodeStream`과 동일하지만 리액트 속성 제거

### 정리

| API | 용도 | React 속성(=Hydration) | 스트리밍 |
| --- | --- | --- | --- |
| `renderToString()` | SSR | ✅  | ❌ |
| `renderToStaticMarkup()` | 정적 콘텐츠 | ❌ | ❌ |
| `renderToNodeStream()` | 큰 SSR | ✅ | ✅ |
| `renderToStaticNodeStream()` | 큰 정적 콘텐츠 | ❌ | ✅ |
