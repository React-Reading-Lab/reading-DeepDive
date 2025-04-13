## React 19에서는 무엇이 새로운가?

- 보류 상태, 에러, 폼, 그리고 낙관적 업데이트를 **자동으로** 처리하기 위해 트랜지션에서 `비동기 함수 사용에 대한 지원` 추가
- 아래는 `useTransition`을 사용해 보류 상태를 처리한 예제.
- 비동기 트랜지션은 즉각적으로 `isPending`을 true로 만들고, 모든 트랜지션이 끝난 후 `isPending`을 다시 false로 만든다. 이로서 트랜지션 수행 중에도 UI반응성과 상호작용성을 지킬 수 있다.

  ```jsx
  // Before Actions
  function UpdateName({}) {
    const [name, setName] = useState("");
    const [error, setError] = useState(null);
    const [isPending, setIsPending] = useState(false);

    const handleSubmit = async () => {
      setIsPending(true);
      const error = await updateName(name);
      setIsPending(false);
      if (error) {
        setError(error);
        return;
      }
      redirect("/path");
    };

    return (
      <div>
        <input value={name} onChange={(event) => setName(event.target.value)} />
        <button onClick={handleSubmit} disabled={isPending}>
          Update
        </button>
        {error && <p>{error}</p>}
      </div>
    );
  }
  ```

  ```jsx
  // Using <form> Actions and useActionState
  function ChangeName({ name, setName }) {
    const [error, submitAction, isPending] = useActionState(
      async (previousState, formData) => {
        const error = await updateName(formData.get("name"));
        if (error) {
          return error;
        }
        redirect("/path");
        return null;
      },
      null
    );

    return (
      <form action={submitAction}>
        <input type="text" name="name" />
        <button type="submit" disabled={isPending}>
          Update
        </button>
        {error && <p>{error}</p>}
      </form>
    );
  }
  ```

### New hook : `useActionState`

- useActionState 는 액션인 함수를 받아 호출을 위해 랩핑된 Action을 리턴합니다. 이 작업은 액션을 합성하기 위함입니다. 랩핑된 액션이 호출되엇을 때, useActionState는 액션의 마지막 결과를 data로, 액션의 마지막 보류 상태를 pending 으로 반환합니다.

  ```jsx
  const [error, submitAction, isPending] = useActionState(
    async (previousState, newName) => {
      const error = await updateName(newName);
      if (error) {
        // You can return any result of the action.
        // Here, we return only the error.
        return error;
      }

      // handle success
      return null;
    },
    null
  );
  ```

### React DOM: `<form>` Actions

- 액션은 react-DOM의 새로운 <form> 기능에 통합되었습니다. `<form action={actionFunction}>` 처럼 액션을 사용해 form을 자동으로 제출하기 위해 `<form> , <input> , 그리고 <button> 엘리먼트`들의 action과 formAction 프롭스로 함수를 전달하기 위한 지원을 추가했습니다.
- `<form>` 액션이 성공하면, React는 제어되지 않는 컴포넌트를 위해 자동으로 폼을 초기화합니다. 만약 `<from>` 을 수동으로 초기화하고 싶다면 새로운 `requestFormReset React DOM API`를 호출할 수 있습니다.

### React DOM: New hook: `useFormStatus`

- 디자인 시스템에서 <form> 데이터에 접근하기 위해 프롭스 드릴링 없이 컴포넌트를 디자인 하는 것은 일반적입니다. 이것은 컨텍스트를 통해 가능했으나 일반적인 사용 사례를 더 쉽게 만들기 위해 새로운 훅인 useFormStatus 를 추가했습니다.

  ```jsx
  import { useFormStatus } from "react-dom";

  function DesignButton() {
    const { pending } = useFormStatus();
    return <button type="submit" disabled={pending} />;
  }
  ```

- useFormStatus 는 Context provider인 부모 <form> 의 상태를 읽습니다.

### New hook: `useOptimistic`

- 데이터 뮤테이션을 수행할 때의 또다른 일반적인 UI 패턴은 비동기 요청을 진행하는 동안 자동으로 마지막 상태를 보여주는 것입니다. React 19에는 useOptimistic 이라 불리는 훅을 추가해 해당 작업을 더 쉽게 만들었습니다.

  ```jsx
  function ChangeName({ currentName, onUpdateName }) {
    const [optimisticName, setOptimisticName] = useOptimistic(currentName);
    const submitAction = async (formData) => {
      const newName = formData.get("name");
      setOptimisticName(newName);
      const updatedName = await updateName(newName);
      onUpdateName(updatedName);
    };

    return (
      <form action={submitAction}>
        <p>Your name is: {optimisticName}</p>
        <p>
          <label>Change Name:</label>
          <input
            type="text"
            name="name"
            disabled={currentName !== optimisticName}
          />
        </p>
      </form>
    );
  }
  ```

- useOptimistic 훅은 updateName 요청이 진행 중일 동안 즉각적으로 optimisticName 을 렌더합니다. 업데이트가 완료되거나 에러가 발생했을 때 React는 자동으로 currentName 값으로 돌아갑니다.

### New API: `use`

- `use` 를 사용해 프로미스를 읽으면 React는 프로미스가 resolve 될 때까지 유예됩니다.

  ```jsx
  import { use } from "react";

  function Comments({ commentsPromise }) {
    // `use` will suspend until the promise resolves.
    const comments = use(commentsPromise);
    return comments.map((comment) => <p key={comment.id}>{comment}</p>);
  }

  function Page({ commentsPromise }) {
    // When `use` suspends in Comments,
    // this Suspense boundary will be shown.

    return (
      <Suspense fallback={<div>Loading...</div>}>
        <Comments commentsPromise={commentsPromise} />
      </Suspense>
    );
  }
  ```

- use 를 사용해 컨텍스트를 읽을 수 있으므로, 아래와 같이 얼리 리턴으로 통해 컨텍스트를 조건부로 읽는 것도 가능합니다.

  ```jsx
  import { use } from "react";
  import ThemeContext from "./ThemeContext";

  function Heading({ children }) {
    if (children == null) {
      return null;
    }

    // This would not work with useContext
    // because of the early return.
    const theme = use(ThemeContext);
    return <h1 style={{ color: theme.color }}>{children}</h1>;
  }
  ```

- use API는 훅처럼 렌더링 중에만 호출될 수 있다. 훅과 다른 점이라면, use는 조건부로 호출할 수 있다는 점입니다. 추후 우리는 use를 사용해 렌더링 중 리소스를 사용할 수 있는 더 많은 방법을 지원할 계획입니다.

### New React DOM Static APIs

- 우리는 `react-dom/static` 에 정적 사이트 생성을 위한 두 가지 새로운 API를 추가했습니다.

  - [`prerender`](https://ko.react.dev/reference/react-dom/static/prerender)
  - [`prerenderToNodeStream`](https://ko.react.dev/reference/react-dom/static/prerenderToNodeStream)

- 이 새로운 API들은 정적 HTML 생성 로드를 위해 데이터를 기다리면서 renderToString을 개선합니다. Web Stream이나 Node.js Stream같은 스트리밍 환경에서 작동하도록 설계되었습니다. 예를 들어, Web Stream 환경에서, 당신은 prerender를 사용해 정적 HTML에 리액트 트리를 pre-render 할 수 있습니다.

### `ref` as a prop

- React 19를 시작하면, 당신은 함수 컴포넌트에서 ref 프롭스에 접근할 수 있습니다.

  ```jsx
  function MyInput({ placeholder, ref }) {
    return <input placeholder={placeholder} ref={ref} />;
  }

  //...
  <MyInput ref={ref} />;
  ```

- 새로운 함수 컴포넌트는 더이상 forwardRef 가 필요하지 않으며, 새로운 ref 프롭을 사용하여 컴포넌트를 자동으로 업데이트할 수 있는 코드 모드를 게시할 예정입니다. 미래 버전에서는 forwardRef 를 사용하지 않고 제거할 것입니다. _(원래는 ref를 프롭스로 넘기려면 forwardRef를 사용해야 했는데, 이제 그냥 ref로 가능하는 의미인 것 같음)_

### `<Context>` as a provider

- 리액트 19에서는 `<Context.Provider>` 대신 `<Context>` 를 Provider로 사용할 수 있습니다.

  ```jsx
  const ThemeContext = createContext("");

  function App({ children }) {
    return <ThemeContext value="dark">{children}</ThemeContext>;
  }
  ```

- 새로운 Context Provider는 `<Context>` 를 사용할 수 있고, 우리는 기존 Provider를 변환하는 코드 모드를 출시할 예정입니다. 그 미래 버전에서 우리는 `<Context.Provider>`를 더이상 사용하지 않을 예정입니다.

### Cleanup functions for refs

- 우리는 이제 ref 콜백에서 정리 함수를 반환하는 기능을 지원합니다.

  ```jsx
  <input
    ref={(ref) => {
      // ref created

      // NEW: return a cleanup function to reset
      // the ref when element is removed from DOM.
      return () => {
        // ref cleanup
      };
    }}
  />
  ```

- 컴포넌트가 언마운트 될 때, 리액트는 ref 콜백에서 반환된 정리 함수를 호출합니다. 이 작업은 DOM refs, 클래스 컴포넌트 refs, 그리고 useImperativeHandle 에 적용됩니다.

### Support for Document Metadata

- HTML에서 `<title> , <link> , <meta>` 같은 document 메타데이터 태그들은 document의 `<head>` 태그 섹션에 배치하기 위해 예약되었습니다. React에서, 어떤 메타데이터가 앱에 적합한 지 결정하는 컴포넌트는 `<head>`를 렌더링한 위치에서 매우 떨어져있을 수 있으며, 리액트가 `<head>`를 렌더하지 않을 수 있습니다.
  ```jsx
  function BlogPost({ post }) {
    return (
      <article>
        <h1>{post.title}</h1>
        <title>{post.title}</title>
        <meta name="author" content="Josh" />
        <link rel="author" href="https://twitter.com/joshcstory/" />
        <meta name="keywords" content={post.keywords} />
        <p>Eee equals em-see-squared...</p>
      </article>
    );
  }
  ```
- React는 위 컴포넌트를 렌더링할 때 `<title>, <link>, <meta>` 태그를 인지하고, 자동으로 document의 `<head>` 섹션으로 호이스팅합니다. 이 메타데이터 태그들을 기본적으로 지원함으로써, 우리는 해당 태그들이 client-only 앱, 스트리밍 SSR, 그리고 서버 컴포넌트에서도 작동함을 보장할 수 있습니다.
