## 참고 페이지

[react-19 공식 문서](https://ko.react.dev/blog/2024/12/05/react-19)

## Actions

데이터 변경 후 UI 상태 업데이트를 더 간단히 처리할 수 있도록 도입된 개념이다.

```jsx
// action 등장 전
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
// action 등장 후
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      }
      redirect("/path");
    })
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
// useActionState 사용 예시
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
    null,
  );

  return (
    <form action={submitAction}>
      <input type="text" name="name" />
      <button type="submit" disabled={isPending}>Update</button>
      {error && <p>{error}</p>}
    </form>
  );
}
```

## useOptimistic

React19에서는 낙관적 업데이트를 쉽게 구현할 수 있도록 `useOptimstic` 훅을 추가했다.

### 낙관적 업데이트란?

서버에 반영되기 전에 요청이 성공했을 거라고 낙관적으로 생각하고 UI에 변경사항을 먼저 반영하는 것이다. 응답이 늦어질 것 같을 때, 그리고 실패할 확률이 낮은 요청일 때 사용하면 좋다.

인스타그램에서 하트 누르기는 상황을 예시로 생각해보면 다음과 같다. 

| **예전 방식** | **낙관적 업데이트** |
| --- | --- |
| 하트 누르기 | 하트 누르기 |
| 서버에 좋아요 요청 | **하트 빨갛게 변경** |
| 서버 응답 | 서버에 좋아요 요청 |
| **성공시 하트 빨갛게 변경**<br/>실패시 변경사항 없음 | 서버 응답 성공 시 유지<br/>서버 응답 실패 시 다시 회색 하트로 변경 |

### 코드 예시

```jsx
// useOptimistic 훅을 사용해 낙관적 업데이트를 구현할 경우(+action)
function ChangeName({ currentName, onUpdateName }) {
  const [optimisticName, setOptimisticName] = useOptimistic(currentName);

  const submitAction = async (formData) => {
    const newName = formData.get("name");

    // 낙관적 업데이트: UI에 먼저 반영
    setOptimisticName(newName);

    // 실제 서버 요청
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

```jsx
// 훅없이 낙관적 업데이트를 구현할 경우
function ChangeName({ currentName, onUpdateName }) {
  const [nameInput, setNameInput] = useState(currentName);
  const [displayName, setDisplayName] = useState(currentName);
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsPending(true);

    // 1. optimistic update
    setDisplayName(nameInput);

    try {
      // 2. 실제 서버 요청
      const updatedName = await updateName(nameInput);
      onUpdateName(updatedName);
    } catch (err) {
      // 3. 실패 시 되돌림
      setDisplayName(currentName);
      alert("업데이트 실패!");
    } finally {
      setIsPending(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <p>Your name is: {displayName}</p>
      <input
        value={nameInput}
        onChange={(e) => setNameInput(e.target.value)}
        disabled={isPending}
      />
      <button disabled={isPending}>Submit</button>
    </form>
  );
}
```
