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
- 아래의 예제 코드처럼 쓰인다. 클라이언트 단에서 호출한 createPost는 서버에서 실행된다. 
  - 서버를 통해 응답을 가져오는 과정에서 `fetch`를 사용하지 않아도 되어 간결하고 편하다.
```javascript
// action.ts 
'use server'
export async function createPost(data : FormData) {
  // 서버에서 실행되는 코드
  await db.post.create({ ... });
}
```
```javascript
'use client'
import { createPost } from './actions';

export default function PostForm() {
  const action = async (formData: FormData) => {
    await createPost(formData); 
  };

  return <form action={action}>...</form>;
}
```

## hydration errors 
- React-Dom에서 hydration 오류를 리포팅하는 방법을 개선하였다. 
- 19버전 부터는 불일치에 대한 정보 없이 여러개의 로그를 표시하는 대신, 불일치의 차이점을 담은 하나의 메시지를 기록한다.
```
Warning: Text content did not match. Server: “Server” Client: “Client”
  at span
  at App
Warning: An error occurred during hydration. The server HTML was replaced with client content in <div>.
Warning: Text content did not match. Server: “Server” Client: “Client”
  at span
  at App
Warning: An error occurred during hydration. The server HTML was replaced with client content in <div>.
Uncaught Error: Text content does not match server-rendered HTML.
  at checkForUnmatchedText

//---------- 위에서 아래로 변경 ------------ //

Uncaught Error: Hydration failed because the server rendered HTML didn’t match the client. As a result this tree will be regenerated on the client. This can happen if an SSR-ed Client Component used:

- A server/client branch if (typeof window !== 'undefined').
- Variable input such as Date.now() or Math.random() which changes each time it’s called.
- Date formatting in a user’s locale which doesn’t match the server.
- External changing data without sending a snapshot of it along with the HTML.
- Invalid HTML tag nesting.

It can also happen if the client has a browser extension installed which messes with the HTML before React loaded.

https://react.dev/link/hydration-mismatch 

  <App>
    <span>
+    Client
-    Server

  at throwOnHydrationMismatch
  …
```

## MetaData에 대한 지원
- React가 컴포넌트를 렌더링할 때 메타 태그(`<title>, <link>, <meta> 등`)을 만나면 자동으로 인식하여 document의 `<head>` 섹션으로 자동으로 호이스팅한다.
- 이 지원은 클라이언트 전용 앱, 스트리밍 SSR, 서버 컴포넌트에서 모두 작동한다. 
  - 즉, SSR 개발 시의 SEO 관리 측면의 약점을 보완할 수 있게 된 것이다.

