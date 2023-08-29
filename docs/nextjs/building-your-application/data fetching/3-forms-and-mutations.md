# \***\*Forms and Mutations\*\***

폼은 웹 어플리케이션에서 데이터를 생성하고 업데이트 하도록 합니다. 넥스트는 서버 액션을 사용하여 폼 제출과 데이터 변이를 다루는 강력한 방법을 제공합니다.

## 서버 액션이 작동하는 방법

서버 액션을 사용하면 API endpoint를 수동으로 만들 필요가 없습니다. 대신, 비동기 서버 함수를 정의하고 그것이 컴포넌트로부터 직접 호출됩니다.

서버 액션은 서버 컴포넌트 내에서 정의되거나 클라이언트 컴포넌트로부터 호출됩니다. 서버 컴포넌트에서 정의되는 것은 폼이 자바스크립트 없이도 점진적인 강화 하도록 합니다.

`next.config.js`파일 내부에서 서버 액션을 가능하게 합니다:

```tsx
//next.config.js

module.exports = {
  experimental: {
    serverActions: true,
  },
};
```

> **Good to know:**
>
> - 서버 컴포넌트에서 서버 액션을 호출하는 폼은 자바스크립트 없이도 작동합니다.
> - 클라이언트에서 서버 액션을 호출하는 폼은 자바스크립트가 아직 로드되지 않은 경우, 제출을 queue로 지정하고 클라이언트 하이트레이션을 우선시합니다.
> - 서버 액션은 그들이 사용되는 페이지나 레이아웃의 런타임을 상속됩니다.
> - 최근, 라우트가 서버 액션을 사용하면, 동적으로 렌더링하는 것이 필요합니다.

## 캐시 데이터 재검증

서버 액션은 넥스트 캐싱 및 재검증 아키텍처와 긴밀하게 통합됩니다. 폼이 제출될 때 서버 액션은 캐시 데이터를 업데이트하고 변해야 하는 캐시 키를 재검증합니다.

기존의 어플리케이션같이 라우트당 하나의 폼에 제한되는 것보다, 서버 액션은 라우트당 여러개의 액션을 가지는 것이 가능합니다. 더욱이, 브라우저는 폼 제출에 새로고침할 필요가 없습니다. 왕복 싱글 네트워크에서 넥스트는 업데이트된 UI와 새로워진 데이터를 반환합니다.

## 예

### 서버 전용 폼

서버 전용 폼을 만들려면, 서버 컴포넌트에서 서버 액션을 정의합니다. 함수 최 상단에서 `“use server”` 지시어를 인라인으로 작성하거나, 파일 최상단에서 지시어가 있는 분리된 파일을 작성합니다.

```tsx
// app/page.tsx
export default function Page() {
  async function create(formData: FormData) {
    'use server';

    // mutate data
    // revalidate cache
  }

  return <form action={create}>...</form>;
}
```

### 데이터 재검증

서버 액션은 필요시에 즉시 넥스트 캐시 재검증을 하도록 합니다. `revalidateCache` 를 사용하여 전체 라우트 세그먼트를 재검증합니다.

```tsx
//app/actions.ts

'use server';

import { revalidatePath } from 'next/cache';

export default async function submit() {
  await submitForm();
  revalidatePath('/');
}
```

또는 `revalidateTag` 를 사용하여 캐시태그가 된 특정한 데이터만 재검증합니다.

```tsx
//app/actions.ts

'use server';

import { revalidateTag } from 'next/cache';

export default async function submit() {
  await addPost();
  revalidateTag('posts');
}
```

### 리다이렉팅

서버 액션이 완료된 후에 사용자를 다른 라우트로 리다이렉트 하고 싶다면, `redirect` 와 절대 혹은 상대 URL을 사용할 수 있습니다.

```tsx
//app/actions.ts

'use server';

import { redirect } from 'next/navigation';
import { revalidateTag } from 'next/cache';

export default async function submit() {
  const id = await addPost();
  revalidateTag('posts'); // Update cached posts
  redirect(`/post/${id}`); // Navigate to new route
}
```

### 폼 재검증

기본 폼 재검증은 `required`나 `type=”email”`과 같은 HTML 검증을 추천합니다

보다 고급 서버 사이드 검증을 위해서는 zod와 같은 schema 검증을 사용하여 구문 분석된 폼 데이터의 구조를 검증합니다.

```tsx
//app/actions.ts
import { z } from 'zod';

const schema = z.object({
  // ...
});

export default async function submit(formData: FormData) {
  const parsed = schema.parse({
    id: formData.get('id'),
  });
  // ...
}
```

### 로딩 스테이트 나타내기

서버에서 폼이 제출될 때 로딩 스테이트를 보여주기 위해 `useFormStatus`훅을 사용합니다.

```tsx
//app/page.tsx

'use client';

import { experimental_useFormStatus as useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button disabled={pending}>{pending ? 'Submitting...' : 'Submit'}</button>
  );
}
```

> Good to know: 현재 로딩, 에러 스테이트를 보여주려면 클라이언트 컴포넌트를 사용해야 합니다. 서버 작업에 대해 안정적으로 진행하면서 이러한 값을 검색하기 위해 서버 사이드 기능에 대한 옵션을 탐색하고 있습니다.

### 에러 핸들링

서버 액션은 직렬화된 객체를 반환합니다. 예를 들어, 서버 액션은 새로운 아이템을 만들어 성공 또는 에러 메세지를 반환합니다.

```tsx
//app/actions.ts

'use server';

export async function create(formData: FormData) {
  try {
    await createItem(formData.get('item'));
    revalidatePath('/');
    return { message: 'Success!' };
  } catch (e) {
    return { message: 'There was an error.' };
  }
}
```

이후 클라이언트 컴포넌트에서 이 값을 읽을 수 있고 상태에 저장하여 컴포넌트가 서버 액션의 결과를 뷰어에 나타낼 수 있습니다.

```tsx
//app/page.tsx

'use client';

import { create } from './actions';
import { useState } from 'react';

export default function Page() {
  const [message, setMessage] = useState<string>('');

  async function onCreate(formData: FormData) {
    const res = await create(formData);
    setMessage(res.message);
  }

  return (
    <form action={onCreate}>
      <input type="text" name="item" />
      <button type="submit">Add</button>
      <p>{message}</p>
    </form>
  );
}
```

> Good to know: 현재 로딩, 에러 스테이트를 보여주려면 클라이언트 컴포넌트를 사용해야 합니다. 서버 작업에 대해 안정적으로 진행하면서 이러한 값을 검색하기 위해 서버 사이드 기능에 대한 옵션을 탐색하고 있습니다.

### 최적 업데이트

서버 액션이 끝나기 전에 최적의 UI 업데이트를 하려면, 응답을 기다리기 보다 `useOptimistic` 을 사용하세요.

```tsx
//app/page.tsx

'use client';

import { experimental_useOptimistic as useOptimistic } from 'react';
import { send } from './actions';

type Message = {
  message: string;
};

export function Thread({ messages }: { messages: Message[] }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic<Message[]>(
    messages,
    (state: Message[], newMessage: string) => [
      ...state,
      { message: newMessage },
    ]
  );

  return (
    <div>
      {optimisticMessages.map((m) => (
        <div>{m.message}</div>
      ))}
      <form
        action={async (formData: FormData) => {
          const message = formData.get('message');
          addOptimisticMessage(message);
          await send(message);
        }}
      >
        <input type="text" name="message" />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

### 쿠키 세팅

서버 액션 내부에 `cookies`함수를 사용하여 쿠키를 설정할 수 있습니다.

```tsx
//app/actions.ts
'use server';

import { cookies } from 'next/headers';

export async function create() {
  const cart = await createCart();
  cookies().set('cartId', cart.id);
}
```

### 쿠키 읽기

서버 액션 내부에 `cookies`함수를 사용하여 쿠키를 읽을 수 있습니다.

```tsx
//app/actions.ts
'use server';

import { cookies } from 'next/headers';

export async function create() {
  const auth = cookies().get('authorization')?.value;
  // ...
}
```

### 쿠키 삭제

서버 액션 내부에 `cookies`함수를 사용하여 쿠키를 삭제할 수 있습니다.
