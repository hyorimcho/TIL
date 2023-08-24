# Fetching, Caching, and Revalidating

데이터 페칭은 어떤 어플리케이션에서든 핵심 부분입니다. 이 페이지를 통해 리액트와 넥스트에서 데이터를 어떻게 페치, 캐시, 재검증하는지 알아볼 것입니다.

데이터를 페치할 수 있는 방법에는 네 가지가 있습니다:

1. 서버에서 fetch로
2. 서버에서 third-party 라이브러리를 사용해서
3. 클라이언트에서 라우트 핸들러를 통해
4. 클라이언트에서 third-party 라이브러리를 사용해서

### 서버에서 `fetch`를 사용하여 데이터 페칭

넥스트는 기존의 `fetch` Web API를 확장하여 서버의 각 페치 요청에 대해 캐싱하고 재검증하는 행동을 구성할 수 있도록 합니다. 리액트는 `fetch`를 리액트 컴포넌트 트리가 렌더링 되는 동안 페치 리퀘스트를 자동적으로 메모되도록 확장했습니다.

서버 컴포넌트, 라우트 핸들러, 서버 액션에서`fetch`를 `async`/`await` 와 함께 사용할 수 있습니다.

```tsx
// @app/page.tsx

async function getData() {
  const res = await fetch('https://api.example.com/...');
  // The return value is *not* serialized
  // You can return Date, Map, Set, etc.

  if (!res.ok) {
    // This will activate the closest `error.js` Error Boundary
    throw new Error('Failed to fetch data');
  }

  return res.json();
}

export default async function Page() {
  const data = await getData();

  return <main></main>;
}
```

> **Good to know**:
>
> - `cookies`, `headers`와 같은 서버 컴포넌트에서 데이터 페칭을 할 때 필요할지도 모르는 유용한 함수들을 넥스트는 제공합니다. 요청 시간 정보에 의존하므로 경로가 동적으로 렌더링 됩니다.
> - 라우트 핸들러에서 `fetch` 요청은 메모되지 않습니다. 라우트 핸들러는 리액트 컴포넌트 트리의 구성이 아니기 때문입니다.
> - 서버 컴포넌트에서 타입 스크립트와 함께 `async`/`await` 을 사용하기 위해서는 타입스크립트 `5.1.3` 버전 또는 `@types/react` `18.2.8` 버전 이상을 사용해야 합니다.

### 데이터 캐싱

캐싱은 데이터를 저장합니다. 그래서 매 요청마다 데이터 소스를 다시 페치하지 않아도 됩니다.

기본적으로 넥스트는 서버의 데이터 캐시에 반환된 cache 값을 자동적으로 캐시합니다. 이것은 데이터가 빌드 타임이나 요청 시간에 페치, 캐시되고 매 데이터 요청마다 재 사용된다는 뜻입니다.

```tsx
// 'force-cache' is the default, and can be omitted
fetch('https://...', { cache: 'force-cache' });
```

`POST`메서드를 사용하는 `fetch`요청은 자동적으로 캐시됩니다. POST 방법을 사용하는 경로 핸들러 내부가 아닌 경우에는 캐시되지 않습니다.

> **데이터 캐시란 무엇일까요?**
>
> 데이터 캐시는 영구적인 HTTP 캐시입니다. 플랫폼에 따라 캐시를 자동으로 확장하고 여러 영역에서 공유할 수 있습니다.

### 데이터 재검증

재검증은 데이터 캐시를 삭제하고 최신 데이터를 다시 가져오는 프로세스입니다. 데이터가 변하고 최신의 정보를 보는 것을 확신하고 싶을 때 유용합니다.

캐시된 데이터는 두 가지 방법으로 재검증이 가능합니다.

- 시간에 기초한 재검증: 일정 시간이 지나면 자동으로 데이터를 재검증합니다. 이 기능은 자주 변경되지 않고 새로운 것이 중요하지 않은 데이터에 유용합니다.
- 즉시 재검증: 이벤트(예-폼 제출)를 기반으로 데이터를 수동으로 재검증합니다. 즉시 재검증 하는 것은 태그 기반이나 경로 기반 접근으로 한번에 데이터 그룹을 재검증할 수 있습니다. 최신 데이터를 최대한 빨리 보여주고 싶은 것을 보장 할 때에 사용합니다(예- 헤드리스 CMS의 내용이 업데이트될 때)

**시간 기반 재검증**

시간 간격으로 데이터를 재검증하기 위해 `fetch`의 `next.revalidate`옵션을 사용하여 리소스의 캐시 수명을 초 단위로 조정할 수 있습니다.

```bash
fetch('https://...', { next: { revalidate: 3600 } })
```

또는, 라우트 세그먼트의 모든 fetch요청을 재검증 하려면 Segment Config Option을 사용할 수 있습니다.

```tsx
// @layout.js / page.js

export const revalidate = 3600; // revalidate at most every hour
```

정적으로 렌더링되는 라우트에 여러개의 fetch 요청이 있다면 각각은 다른 재검증 간격을 가집니다. 가장 짧은 간격이 모든 요청에 적용될 것입니다. 정적으로 렌더링 되는 라우트에는 각 `fetch`요청은 독립적으로 재검증됩니다.

> Good to know: Node.js 런타임에서만 재검증됩니다.

**즉시 재검증**

데이터는 라우트 핸들러 또는 서버 액션 내부의 경로(revalidatePath) 혹은 캐시 태그(revalidateTag)로 즉시 재검증됩니다.

넥스트는 라우트에 거쳐 재검증 `fetch`요청에 대한 캐시 태깅 시스템을 가집니다.

1. `fetch`를 사용할 때, 캐시 항목에 하나 이상의 태그를 지정하는 옵션이 있습니다.
2. 그리고 그 태그와 관련이 있는 모든 항목을 재검증하기 위해 `revalidateTag`를 호출합니다.

예를 들어 다음 `fetch`요청은 collection이라는 캐시 태그를 추가합니다.:

```tsx
// app/page.tsx

export default async function Page() {
  const res = await fetch('https://...', { next: { tags: ['collection'] } });
  const data = await res.json();
  // ...
}
```

라우트 핸들러를 사용하면 넥스트 어플리케이션만 알 수 있는 비밀 토큰을 만들어야 합니다. 이 비밀 토큰은 인증되지 않은 재검증 시도를 방지하는 데에 사용합니다. 예를 들어 다음 URL 구조로 라우트에 접근할 수 있습니다 (수동 혹은 웹훅으로)

```bash
https://<your-site.com>/api/revalidate?tag=collection&secret=<token>
```

```tsx
// @app/api/revalidate/route.ts

import { NextRequest, NextResponse } from 'next/server';
import { revalidateTag } from 'next/cache';

// e.g a webhook to `your-website.com/api/revalidate?tag=collection&secret=<token>`
export async function POST(request: NextRequest) {
  const secret = request.nextUrl.searchParams.get('secret');
  const tag = request.nextUrl.searchParams.get('tag');

  if (secret !== process.env.MY_SECRET_TOKEN) {
    return NextResponse.json({ message: 'Invalid secret' }, { status: 401 });
  }

  if (!tag) {
    return NextResponse.json({ message: 'Missing tag param' }, { status: 400 });
  }

  revalidateTag(tag);

  return NextResponse.json({ revalidated: true, now: Date.now() });
}
```

또는 경로와 관련된 모든 데이터를 재검증하기 위해서는 `revalidatePath`를 사용할 수 있습니다.

```tsx
// @app/api/revalidate/route.ts

import { NextRequest, NextResponse } from 'next/server';
import { revalidatePath } from 'next/cache';

export async function POST(request: NextRequest) {
  const path = request.nextUrl.searchParams.get('path');

  if (!path) {
    return NextResponse.json(
      { message: 'Missing path param' },
      { status: 400 }
    );
  }

  revalidatePath(path);

  return NextResponse.json({ revalidated: true, now: Date.now() });
}
```

**에러 핸들링과 재검증**

데이터를 재검증하는 시도동안 에러가 생긴다면, 마지막으로 성공적으로 생성된 데이터가 캐시에서 제공됩니다. 다음 요청 시에 넥스트는 데이터 재검증을 다시 시도합니다.

## 데이터 캐싱에서 제외

아래의 상황에서 `fetch`요청은 캐시하지 않을 것입니다.:

- `cache: ‘no-store’` 이 `fetch`요청에 더해지면
- `revalidate:0`옵션이 독립적인 `fetch` 요청에 더해지면
- `POST`메서드를 사용하는 라우터 핸들러 내부의 `fetch`요청
- `headers`나 `cookies`사용 이후에 오는 `fetch`요청
- `const dynamic = 'force-dynamic'` 라우트 세그먼트 옵션이 사용
- `fetchCache` 라우트 세그먼트 옵션은 기본적으로 캐시를 건너뛰도록 구성되어 있습니다.
- `Authorization`나 `Cookie`헤더를 사용하며 컴포넌트 트리에 캐시되지 않은 리퀘스트가 있는 `fetch`요청

**독립적인 `fetch`요청**

독립적인 `fetch`요청에 대한 캐싱을 선택하지 않으려면 `fetch`요청의 캐시 옵션을 `no-store`로 설정할 수 있습니다.

```tsx
//@layout.js / page.js

fetch('https://...', { cache: 'no-store' });
```

**다중 `fetch`요청**

라우트 세그먼트(레이아웃이나 페이지)에서 다중 `fetch`요청이 있을 때, 세그먼트 구성 옵션을 사용하여 세그먼트의 모든 데이터 요청에 대한 캐싱 동작을 구성할 수 있습니다.

예를 들어 `const dynamic = 'force-dynamic'`를 사용하는 것은 모든 데이터가 요청 시간에 페치되도록 할 것이고 세그먼트는 동적으로 렌더링 될 것입니다.

```tsx
// @layout.js / page.js
// Add
export const dynamic = 'force-dynamic';
```

라우트 세그먼트의 동적 및 정적 동작을 세분화하여 제어할 수 있는 세그먼트 구성 옵션의 광범위한 목록이 있습니다.

## third-party 라이브러리를 이용해 서버에서 데이터 페칭

fetch 를 지원하지 않는 third-party 라이브러리를 사용하는 경우, (예 - 데이터베이스, CMS 또는 ORM 클라이언트) 라우트 세그먼트 구성 옵션과 리액트의 `cache`함수를 사용하여 이러한 요청의 캐싱과 재검증 동작을 구성할 수 있습니다.

캐시 되든 아니든 데이터는 라우트 세그먼트가 정적으로 렌더되는지 동적으로 렌더되는지에 의존합니다. 세그먼트가 정적이면(기본값), 요청의 결과는 라우트 세그먼트의 일부분으로 캐시되고 재검증됩니다. 세그먼트가 동적이면 요청의 결과는 캐시되지 않고 세그먼트가 렌더되는 매 요청시마다 다시 페치됩니다.

> Good to know: 넥스트는 독립적인 third-party 요청의 캐싱과 재검증 동작을 구성하기 위해 API인 `unstable_cashe`에서 작업합니다.

### 예

- `revalidate`옵션을 `3600`으로 설정하는 것은 데이터가 매 시간마다 캐시되고 재검증된다는 뜻입니다.
- 리액트 `cache`함수는 데이터 요청을 메모합니다.

```tsx
// @utils/get-item.ts

import { cache } from 'react';

export const revalidate = 3600; // revalidate the data at most every hour

export const getItem = cache(async (id: string) => {
  const item = await db.item.findUnique({ id });
  return item;
});
```

`getItem`함수는 두번 호출되더라도 데이터베이스에는 쿼리 하나만 만들어질 것입니다.

```tsx
// @app/item/layout.tsx

import { getItem } from '@/utils/get-item';

export default async function Layout({
  params: { id },
}: {
  params: { id: string };
}) {
  const item = await getItem(id);
  // ...
}
```

```tsx
// @app/item/[id]/page.tsx

import { getItem } from '@/utils/get-item';

export default async function Page({
  params: { id },
}: {
  params: { id: string };
}) {
  const item = await getItem(id);
  // ...
}
```

## 라우트 핸들러를 이용해 클라이언트에서 데이터 페칭

클라이언트 컴포넌트에서 데이터 페치가 필요할 때 클라이언트에서 라우트 핸들러를 호출할 수 있습니다.라우트 핸들러는 서버에서 실행하고 클라이언트에 데이터를 반환합니다. 이것은 API토큰과 같이, 클라이언트에 민감한 정보를 드러내지 않고 싶을 때에 사용합니다.

> **서버 컴포넌트와 라우트 핸들러**
> 서버 컴포넌트는 서버에서 렌더되기 때문에 데이터 페치를 위해 서버 컴포넌트에서 라우트 핸들러를 호출할 필요가 없습니다. 대신 서버 컴포넌트 내에서 직접 데이터를 페치합니다.

## third-party 라이브러리를 이용해 클라이언트에서 데이터 페칭

SWR이나 리액트 쿼리같은 third-party 라이브러리를 이용해 클라이언트에서 데이터 페칭을 할 수 있습니다. 이 라이브러리들은 요청을 메모하고 캐싱하고 데이터를 변화시키는 API를 제공합니다.

> **Future APIs:**
> `use`는 함수에 의해 반환된 promise를 수락하고 처리하는 리액트 함수입니다. `use` 안에서 감싸진 `fetch`는 클라이언트 컴포넌트 내에서는 권장되지 않으며 여러번의 리렌더가 발생할 수 있습니다.
