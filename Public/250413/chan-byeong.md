## React v19가 되면서 새롭게 업데이트된 내용.

### 1. Actions

_비동기 전환을 사용하는 함수를 관례적으로 "Action"이라고 부른다._

기존에는 데이터 페칭 수행 시 `pending`, `error`와 같은 상태들을 손수 `useState`를 통해 관리했어야 했다.

```javascript
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

하지만 v19로 들어오면서 이 같은 상태를 관리할 수 있는 훅이 새롭게 등장했다.

v18 버전에도 상태를 관리하기 위한 훅이 존재했다 예를 들어

v18 버전에의 `useTransition` 훅은 `pending` 상태를 자동으로 관리해준다.

```javascript
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

`startTransition`이 호출되는 즉시 `isPending` 값은 `true`를 갖게되고 해당 함수의 동작이 끝나면 다시 `false`값을 가지게된다. 이제 우리는 `pending`을 관리하는 상태를 하나 줄일 수 있게되었다.

하지만 여기서드는 의문점은 `pending` 상태가 하나 주는 대신 `useTransition`이라는 훅이 하나 추가된 것이다.
이러면 똑같은 복잡도를 지닌게 아닌가 하는 의문이 든다.

이러한 의문을 해소해주듯 v19 버전에서는 좀 더 통합적으로 상태를 관리할 수 있는 새로운 훅이 등장했다.

#### `useActionState`
