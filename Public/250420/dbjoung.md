## Action 이란?

React Action은 React 19 버전에 새롭게 도입된 기술이다. 공식 문서에서는 액션에 대해 다음과 같이 정의하고 있다.

- 관례적으로, 비동기 트랜지션을 사용하는 함수를 `Action` 이라고 합니다. 액션은 데이터 제출을 자동으로 관리합니다.

### 트랜지션(Transition)이란?

React 18에 도입된 개념으로, “긴급하지 않은 **상태 업데이트**”를 의미한다. 이를 통해 React는 중요한 업데이트에 우선순위를 두고, 사용자와의 상호작용을 방해하지 않으면서 백그라운드에서 상태 업데이트를 수행할 수 있다.

즉, `Action` 이란 트랜지션 작업을 **자동**으로 수행할 수 있게 해주는 **비동기 콜백**으로 이해할 수 있을 것이다.

## Action의 종류

React 19에서 액션은 데이터 제출을 자동으로 관리하는데, 그 대상은 다음과 같다.

### Pending State (보류 상태)

- 액션은 요청의 시작 시 `isPending` 상태를 `true`로 만들고, 요청이 완료되면 자동으로 `false`로 되돌려준다.
- 즉, 요청이 완료 되었는 지 체크하는 State의 상태 변화를 자동으로 해준다는 의미다. React 18에서는 마지막 요청이 완료됐을 때 수동으로 state를 변경해줘야 했지만, 이 작업을 Action을 통해 자동으로 관리할 수 있다.

### Optimistic updates (낙관적 업데이트)

- 새로운 `useOptimistic` Hook을 지원해서 요청들이 처리되는 동안에도 사용자에게 즉각적인 피드백을 보여줄 수 있도록 해준다.

### Error handling

- Action 함수 내부에서 에러가 발생하면, 해당 에러는 자동으로 `ErrorBoundary`에 포착된다.
- 동시에 `useOptimistic`으로 낙관적으로 반영했던 변경 사항도 자동 rollback 된다.
- React 18에서는 에러 핸들링과 rollback을 모두 개발자가 수동으로 처리해야 했다.

### Forms

- React 19에서는 `action` 프롭스에 Action을 전달할 수 있게 되었다.
- `useActionState` 훅과 함께 사용하면, 폼 제출 흐름을 React가 감지하고 pending 상태나 에러 처리를 자동으로 관리할 수 있다.
- 단, 폼 리셋은 uncontrolled input일 경우 브라우저 기본 동작으로 자동 처리되며, React가 직접 리셋을 수행하는 것은 아니다.

## 사용 사례

아래는 공식 문서에서 제공하는 Pending State, 에러 핸들링, form, 그리고 낙관적 업데이트를 자동으로 처리하기 위해 트랜지션에 비동기 함수(Action)를 사용한 예제들이다.

React 19 이전 버전에서는 위의 사항들을 전부 수동으로 처리해줘야했지만, Action의 등장으로 그럴 필요가 없어졌다.

### useTransition에서 Action을 사용한 예

```jsx
// Using pending state from Actions
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    // startTransition에 비동기 콜백 전달 (React 19부터 async 지원)
    // isPending에는 즉각적으로 true가 할당된다.
    startTransition(async () => {
      //undateName 함수의 처리 결과를 받음
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      }
      redirect("/path");

      // React 19에서는 async 콜백의 종료를 감지하여,
      // 트랜지션 완료 시 isPending을 자동으로 false로 변경해준다.
    });
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

### 새로운 훅 `useActionState`를 사용한 예

- 동작은 위 첫번째 코드와 같으나, useActionState hook을 통해 훨씬 간결해졌다.

```jsx
// Using <form> Actions and useActionState
function ChangeName({ name, setName }) {
  // useActionState에 비동기 콜백 전달
  // form action 속성에 submitAction을 넣었으므로,
  // form이 제출됐을 때 React가 pending 상태를 자동으로
  // 감지. isPending에는 즉각적으로 true가 할당된다.
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      // form 데이터가 자동으로 인자로 전달된다.
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

# 참고

- [ReactReact-19-Actions](https://velog.io/@njt6419/ReactReact-19-Actionshttps://velog.io/@njt6419/ReactReact-19-Actions)
- [React 18 - Transition](https://blue-tang.tistory.com/95)
