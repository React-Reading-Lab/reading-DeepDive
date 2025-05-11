## React DOM의 정적 API

- 정적 사이트 생성을 위해 react-dom/static에 새로운 두 가지 API를 추가
  - prerender
  - prerenderToNodeStream
- 이 새로운 API들은 `renderToString`보다 더 나아가, 정적 HTML 생성을 위해 데이터가 로드될 때까지 기다린다.
- 기존의 `renderToString`은 API 데이터가 필요할 경우 fetch를 통해 기다려야 했다. 하지만 Prerender API는 내부적으로 `use(promise)`를 만나면 suspend가 작동해서, React가 직접 데이터가 로딩될 때까지 기다렸다가 HTML을 생성한다.
- 즉, **데이터 로딩 단계**까지 React가 전담하게 된 것이다.

```JSX
// 서버 전용 컴포넌트 (React 19 RSC 환경)
export default function App() {
  const data = use(fetchMessage());

  return <div>{data.message}</div>;
}

async function fetchMessage() {
  // 1초 뒤 메시지 반환
  return new Promise((resolve) =>
    setTimeout(() => resolve({ message: "안녕하세요, React 19!" }), 1000)
  );
}
```

```JSX
import { prerender } from 'react-dom/static';
import App from './App.server.js'; // 서버 컴포넌트

export async function handler(req, res) {
  const { prelude } = await prerender(<App />, {
    bootstrapScripts: ['/main.js'],
  });

  res.setHeader('Content-Type', 'text/html');
  res.end(prelude); // 모든 데이터 로딩 완료 후 HTML 응답
}

```

## 서버 컴포넌트

- React 19에서는 Canary 버전에 포함되어있던 모든 RSC 기능이 포함되어 있음. (즉, 공식적으로 RSC를 사용할 수 있도록 기반이 갖춰줬다는 뜻인 것 같음)

## 서버 액션

- 서버 액션은 서버에서 실행되는 비동기 함수를 클라이언트 컴포넌트에서 호출하는 것을 허용한다.
- 서버 액션이 `use server` 지시어로 정의될 때 프레임워크는 자동으로 서버 함수에 대한 참조를 생성하고 클라이언트 컴포넌트에 이를 전달한다.
