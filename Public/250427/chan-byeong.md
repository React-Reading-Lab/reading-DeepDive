# Actions

이전에 API를 통해서 데이터를 주고 받을 때 상태를 다룰 경우에는 직접 `useState`를 사용하여 `error`, `loading`과 같은 상태를 관리해야 했습니다.

추가적으로 `낙관적 업데이트`나 일련의 요청들을 수동으로 처리해야 하는 불편함이 존재했습니다.

이러한 번거로움을 줄이기 위해서 v19 부터 새로운 훅을 도입하였습니다.

## useActionState

```javascript
const [state, formAction, isPending] = useActionState(fn, initialState, permalink?);
```

`form`의 `action` 함수가 호출될 때 업데이트되는 `state`를 가지려면 `useActionState` 훅을 사용하면 됩니다.

`useActionState`의 인자로는 기존의 `action` 함수와 "호출될 때 업데이트되는 `state`"의 초기값이 필요합니다.

이때 `useActionState`는 반환값으로

- "호출될 때 업데이트되는 `state`"인 `state` 인자
- 새로운 `action` 함수인 `formAction`
- `action` 함수의 peding 여부를 알려주는 `isPending`을 갖습니다.

이제 `form`에서는 기존의 `action` 함수 대신 `formAction` 함수를 사용하면 `action` 함수 호출 시 업데이트 되는 상태와 이를 폼의 pending 여부를 알려주는 `isPeding`을 사용할 수 있습니다.

`action` 함수의 인자는 `previousState`와 마지막으로 제출한 폼의 데이터를 담고 있는 `formData`가 있습니다.

```javascript
import { useActionState } from "react";

async function increment(previousState, formData) {
  return previousState + 1;
}

function StatefulForm({}) {
  const [state, formAction] = useActionState(increment, 0);
  return (
    <form>
      {state}
      <button formAction={formAction}>Increment</button>
    </form>
  );
}
```

추가로 v19가 되면서 `form` 태그에만 action props를 넣을 수 있는 것이 아니라 `form` 태그 내부의 `button`, `input` 태그에도 action props를 넣을 수 있도록 변경되었습니다.

## useFormStatus

```javascript
const { pending, data, method, action } = useFormStatus();
```

`useFormStatus` 훅은 마지막 제출에 대한 상태 정보를 알려줍니다.

```javascript
import { useFormStatus } from "react-dom";
import action from "./actions";

function Submit() {
  const status = useFormStatus();
  return <button disabled={status.pending}>Submit</button>;
}

export default function App() {
  return (
    <form action={action}>
      <Submit />
    </form>
  );
}
```

단 `useFormStatus` 훅은 `form` 태그 **하위 컴포넌트**에서 사용되어야 합니다.

만약 `useFormStatus`가 `form` 태그와 동일한 컴포넌트 코드에 존재한다면 해당 훅은 제대로 동작하지 않을 것입니다.

## useOptimistic

데이터 mutation 시 하나의 일반적인 패턴은 데이터 mutation 요청을 보낸 뒤 서버의 응답에 관계없이 미리 상태를 업데이트 해두는 낙관적 업데이트입니다.

이를 더 쉽게 하기 위해서 `useOptimistic` 훅을 도입하였습니다.

```javascript
const [optimisticState, addOptimistic] = useOptimistic(state, updateFn);
```

```javascript
import { useOptimistic } from "react";

function AppContainer() {
  const [optimisticState, addOptimistic] = useOptimistic(
    state,
    // updateFn
    (currentState, optimisticValue) => {
      // merge and return new state
      // with optimistic value
    }
  );
}
```

해당 훅을 사용하면 peding 상태일 때 optimisticState를 새로운 상태값으로 업데이트하고 이를 보여줍니다.

만약 요청이 끝나거나 에러가 발생한 경우 원래 상태값으로 돌려놓습니다. (요청이 끝났다는 의미는 원래 상태가 새로운 상태로 업데이트 되었음을 의미합니다.)

아래 예시를 보면 더 쉽게 이해가 가능합니다.

```javascript
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
          type='text'
          name='name'
          disabled={currentName !== optimisticName}
        />
      </p>
    </form>
  );
}
```

## use

`Promise`나 `Context` 같은 자원을 읽을 수 있도록 해주는 것이 `use` 훅이다.

```javascript
const value = use(resource);
```
