# Data Fetching Patterns

몇 가지 추천하는 패턴과 리액트와 넥스트에서 데이터를 페칭하는 좋은 연습이 있습니다. 이 페이지에서는 가장 일반적인 패턴과 그것들을 어떻게 사용하는지에 대해 다룹니다.

## 서버에서 데이터 페칭

가능할 때는 언제라도 서버에서 데이터를 페칭하는 것을 추천합니다. 서버에서 데이터를 페칭하는 것은

- 백엔드 데이터 리소스에 직접 접근할 수 있습니다 (예 - 데이터베이스)
- 액세스 토큰과 API 키와 같은 민감한 정보를 클라이언트에 노출되지 않도록 하여 어플리케이션을 더욱 안전하게 합니다.
- 같은 환경에서 데이터를 페치하고 렌더합니다. 이것은 서버와 클라이언트 간의 back-and-forth 커뮤니케이션을 줄여주고 또한 클라이언트의 메인 스레드에 대한 작업도 줄어듭니다.
- 클라이언트에서 중복된 개별 요청 대신 단일 왕복으로 다중 데이터를 페치합니다.
- 클라이언트 서버 waterfall을 줄여줍니다.
- 지역에 따라 데이터 페칭은 데이터 소스에 더 가까운 곳에서 발생하여 지연을 줄여주고 성능을 높여줍니다.

서버 컴포넌트, 라우트 핸들러, 서버 액션을 사용하면 서버에서 데이터를 페치할 수 있습니다.

## 필요한 곳에서 데이터 페칭

트리의 여러개의 컴포넌트에서 같은 데이터(예 - 현재 사용자)를 사용하고 싶다면, 데이터를 전역으로 페치할 필요도, 컴포넌트간 prop으로 전달해 줄 필요도 없습니다. 대신 데이터가 필요한 컴포넌트 내에서 `fetch` 혹은 리액트 `cache`를 사용하여 동일한 데이터에 대해 여러 번 요청할 때 발생하는 성능 영향에 대해 걱정할 필요가 없습니다.

이것은 `fetch` 요청이 자동적으로 메모되기 때문에 가능합니다.

> Good to know: 부모 레이아웃과 자식 레이아웃 간 데이터 전달은 불가능하기 때문에 이것은 레이아웃에도 적용됩니다.

## 스트리밍

스트리밍과 서스펜스는 UI의 단위를 점진적으로 클라이언트에 렌더링하고 스트리밍 할 수 있는 리액트 기능입니다.

서버 컴포넌트와 중첩 레이아웃을 이용하면 특별히 데이터를 필요하지 않은 페이지의 부분을 즉시 렌더링 하고 데이터를 페칭하는 부분에만 로딩 스테이트를 보여줄 수 있습니다. 이것은 사용자가 상호작용을 시작하기 전에 전체 페이지가 로드되기를 기다리지 않아도 된다는 것을 의미합니다.

![](https://velog.velcdn.com/images/hyorimm/post/ef0b1f18-0edf-4046-b286-ce73c72cd437/image.png)

## 병렬 및 순차적 데이터 페칭

리액트 컴포넌트에서 데이터를 페칭할 때 두 가지 데이터 페칭 패턴에 주의해야 합니다. 병렬 그리고 순차입니다.

![](https://velog.velcdn.com/images/hyorimm/post/e69c1115-b049-40b7-b50a-4c6e905f0752/image.png)

- 순차적 데이터 페칭은 라우트 내의 요청이 각각 종속적이므로 waterfalls를 만듭니다. 한 페치가 다른 페치의 결과에 의존하므로 이 패턴을 원하는 경우가 있거나 다음 페치 전에 리소스를 저장하는 조건이 충족되기를 원하는 경우가 있습니다. 하지만 이 행동은 의도적이지 않고 더 긴 로딩 시간을 초래할 수 있습니다.
- 병렬 데이터 페칭은 라우트의 요청이 열심히 시작되고 동시에 데이터를 로드합니다. 이는 클라이언트-서버 waterfalls와 데이터 로드하는 데에 걸리는 총 시간이 줄어듭니다.

**순차적 데이터 페칭**

중첩된 컴포넌트가 있고 각 컴포넌트가 고유한 데이터를 페치하면, 데이터 페칭은 순차적으로 이루어집니다.(자동으로 메모되는 것과 같은 동일한 데이터 요청에는 적용되지 않습니다)

예를 들어 `Artist`컴포넌트가 데이터 페칭을 마치면 `Playlists`컴포넌트는 데이터 페칭을 시작할 것입니다. 왜냐하면 `Playlists`는 `artistID` prop에 의존하기 때문입니다.

```tsx
// @app/artist/page.tsx
// ...

async function Playlists({ artistID }: { artistID: string }) {
  // Wait for the playlists
  const playlists = await getArtistPlaylists(artistID);

  return (
    <ul>
      {playlists.map((playlist) => (
        <li key={playlist.id}>{playlist.name}</li>
      ))}
    </ul>
  );
}

export default async function Page({
  params: { username },
}: {
  params: { username: string };
}) {
  // Wait for the artist
  const artist = await getArtist(username);

  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<div>Loading...</div>}>
        <Playlists artistID={artist.id} />
      </Suspense>
    </>
  );
}
```

이런 케이스처럼, 리액트가 결과를 스트리밍할 때 `loading.js` (루트 세그먼트 용) 또는 리액트 `<Suspense>` (중첩 컴포넌트 용) 를 즉각적인 로딩 상태를 보여주기 위해 사용할 수 있습니다.

이것은 모든 라우트가 데이터 페칭에 의해 차단되는 것을 막을 수 있고, 사용자는 차단되지 않은 페이지의 부분과 상호 작용 할 수 있습니다.

<aside>
💡 **데이터 요청 차단:**
waterfalls를 막기 위한 대안적인 접근은 어플리케이션의 루트에서 데이터를 전역으로 페치하는 것입니다. 하지만 이것은 로딩이 끝날 때까지 그 아래의 모든 라우트 세그먼트의 렌더링을 막을 것입니다. 이것은 “모 아니면 도” 데이터 페칭으로 묘사될 수 있습니다. 페이지의 전체 데이터를 가지고 있거나 전혀 가지고 있지 않습니다.

`await`를 사용하는 모든 `fetch`요청들은 그것의 아래로 렌더링과 데이터 페칭을 막을 것입니다. `<Suspense>` 바운더리로 감싸지거나 `loading.js`가 사용되기 전까지. 다른 대안은 병렬 데이터 페칭이나 미리 로드하는 패턴을 사용하는 것입니다.

</aside>

### 병렬 데이터 페칭

병렬로 데이터를 페칭하기 위해서는 데이터를 사용하는 컴포넌트의 외부에서 정의한 다음, 컴포넌트 내부에서 호출하여 요청을 열렬히 시작할 수 있습니다. 이렇게 하면 두 요청을 병렬로 시작하여 시간을 절약할 수 있지만 사용자는 두 promises가 해결될 때까지 렌더링된 결과를 볼 수 없습니다.

아래 예시에서 `getArtist`와 `getArtistAlbums`함수는 `Page` \컴포넌트 바깥에서 정의되고 컴포넌트 내부에서 호출되어 두 promise가 해결되기를 기다립니다.

```tsx
// @app/artist/[username]/page.tsx

import Albums from './albums';

async function getArtist(username: string) {
  const res = await fetch(`https://api.example.com/artist/${username}`);
  return res.json();
}

async function getArtistAlbums(username: string) {
  const res = await fetch(`https://api.example.com/artist/${username}/albums`);
  return res.json();
}

export default async function Page({
  params: { username },
}: {
  params: { username: string };
}) {
  // Initiate both requests in parallel
  const artistData = getArtist(username);
  const albumsData = getArtistAlbums(username);

  // Wait for the promises to resolve
  const [artist, albums] = await Promise.all([artistData, albumsData]);

  return (
    <>
      <h1>{artist.name}</h1>
      <Albums list={albums}></Albums>
    </>
  );
}
```

사용자 경험을 향상시키려면 서스펜스 바운더리를 추가하여 렌더링 작업을 분할하고 가능한 빨리 결과의 일부를 보여줄 수 있습니다.

## preloading 데이터

waterfall을 막을 수 있는 다른 방법은 preload 패턴을 사용하는 것입니다. 병렬 데이터 페칭을 최적화 하기 위해 `preload`함수를 선택적으로 만들 수 있습니다. 이러한 접근은 props로 promise를 전달하지 않아도 되게합니다. `preload`함수는 API가 아닌 패턴이므로 어떠한 이름도 가질 수 있습니다.

```tsx
// @components/Item.tsx

import { getItem } from '@/utils/get-item';

export const preload = (id: string) => {
  // void evaluates the given expression and returns undefined
  // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/void
  void getItem(id);
};
export default async function Item({ id }: { id: string }) {
  const result = await getItem(id);
  // ...
}
```

```tsx
// @app/item/[id]/page.tsx

import Item, { preload, checkIsAvailable } from '@/components/Item';

export default async function Page({
  params: { id },
}: {
  params: { id: string };
}) {
  // starting loading item data
  preload(id);
  // perform another asynchronous task
  const isAvailable = await checkIsAvailable();

  return isAvailable ? <Item id={id} /> : null;
}
```

## `cache`, `server-only` 그리고 `preload` 패턴을 사용하기

`cache`함수와 `preload`패턴 그리고 `server-only`패키지를 합해 앱 전체에서 사용할 수 있는 데이터 페칭 유틸리티를 만들 수 있습니다.

```tsx
// @utils/get-item.ts

import { cache } from 'react';
import 'server-only';

export const preload = (id: string) => {
  void getItem(id);
};

export const getItem = cache(async (id: string) => {
  // ...
});
```

이러한 방식으로 데이터를 가져오고 응답을 캐시하고 데이터 페칭을 서버에서만 일어나도록 보장할 수 있습니다.

`utils/get-item`내보내기는 레이아웃, 페이지 또는 다른 컴포넌트에서 아이템의 데이터가 페치될 때 제어할 수 있도록 사용할 수 있습니다.

> Good to know: `server-only`패키지를 사용하여 서버 데이터 페칭 함수가 클라이언트에서 사용되지 않도록 하는 것이 좋습니다.
