# Loading UI and Streaming

특별한 파일인 `loading.js` 는 리액트 서스펜스로 의미있는 로딩 UI를 만드는 것을 도와줄 것입니다. 이 컨벤션을 사용해 라우트 세그먼트의 내용이 로드되는 동안 서버로부터 즉각적인 로딩 상태를 보여줄 수 있습니다. 새로운 내용은 자동적으로 렌더링이 끝나면 교체됩니다.

![](https://velog.velcdn.com/images/hyorimm/post/afbde03b-6efe-466b-a933-edb91bdc6354/image.png)

## 즉각적인 로딩 상태

즉각적인 로딩 상태는 탐색 즉시 보여지는 폴백 UI입니다. 스켈레톤이나 스피너 혹은 커버 사진, 제목 등 나중에 화면에 보여질 작지만 의미있는 부분을 미리 렌더링 할 수 있습니다. 이것은 사용자가 앱이 반응한다고 이해하는 것과 더 나은 사용자 경험을 제공하는 것을 도와줍니다.

폴더 안에 `loading.js` 을 추가하여 로딩 상태를 만듭니다.

![](https://velog.velcdn.com/images/hyorimm/post/87861609-c99f-4895-a2eb-89359c8c4636/image.png)

```tsx
// @app/dashboard/loading.tsx
export default function Loading() {
  // You can add any UI inside Loading, including a Skeleton.
  return <LoadingSkeleton />;
}
```

같은 폴더 내 `loading.js` 는 `layout.js`에 중첩되어 있을 것입니다. `page.js` 아래와 아래 모든 자식을 <Suspense> 경계에 자동으로 감쌉니다.

![](https://velog.velcdn.com/images/hyorimm/post/e5fc4070-5d6c-4222-8605-f862211a3aad/image.png)

> **Good to know**:
>
> - 서버 중앙 라우팅이라고 해도 탐색은 즉각적입니다.
> - 경로를 변경하면 다른 경로로 탐색하기 전에 경로의 내용이 완전히 로드될 때까지 기다릴 필요가 없다는 점에서 탐색은 중단할 수 있습니다.
> - 공유 레이아웃은 새로운 세그먼트가 로드되는 동안 상호작용 적이게 유지됩니다.

```tsx
추천: 라우트 세그먼트(레이아웃과 페이지)에는 loading.js 컨벤션을 사용하세요. 넥스트는 이 기능을 최적화합니다.
```

## 서스펜스가 있는 스트리밍

`loading.js` 외에도, 직접 서스펜스 경계를 만들 수 있습니다. 앱 라우터는 node.js 와 edge 런타임에서 서스펜스가 있는 스트리밍을 지원합니다.

### 서스펜스란?

리액트와 넥스트에서 스트리밍이 어떻게 동작하는지 배우기 전에 서버사이드 렌더링과 그 제한에 대해 이해하는 것은 도움이 될 것입니다.

SSR에서 사용자가 페이지를 보고 상호작용하기 전에 완성되어야 하는 몇 가지 단계가 있습니다.

1. 첫째, 페이지에게 주어지는 모든 데이터는 서버에서 페치됩니다.
2. 그리고 서버는 페이지의 HTML을 렌더합니다.
3. 페이지의 HTML, CSS, 자바스크립트는 클라이언트에게 전송됩니다.
4. 비 상호작용적인 사용자 인터페이스는 생성된 HTML, CSS를 사용해서 보여집니다.
5. 마지막으로, 리액트는 상호적이게 만들기 위해 사용자 인터페이스를 hydrate합니다.

![](https://velog.velcdn.com/images/hyorimm/post/248181a4-233c-481e-9421-f044953fe15a/image.png)

이 단계는 순차적이고, 차단적입니다. 즉, 서버는 모든 데이터를 페치한 후에만 페이지의 HTML을 렌더링 할 수 있습니다. 그리고 클라이언트에서 리액트는 페이지의 모든 컴포넌트의 코드가 다운로드 된 후에만 UI를 hydrate 할 수 있습니다.

리액트와 넥스트의 SSR은 비 상호작용적인 페이지를 되도록이면 빨리 유저에게 보여줌으로써 지연되는 로딩 성능을 향상시키는 데에 도움을 줍니다.

![](https://velog.velcdn.com/images/hyorimm/post/990f8f1a-27d0-4222-860d-051af90bd513/image.png)

하지만, 페이지가 사용자에게 보여지기 이전에 서버에서 모든 데이터 페칭이 완료되어야 하기 때문에 여전히 속도가 느릴 수 있습니다.

**스트리밍**은 페이지의 HTML을 작은 덩어리들로 쪼개고 점진적으로 그 덩어리를 서버에서 클라이언트로 보낼 수 있도록 합니다.

![](https://velog.velcdn.com/images/hyorimm/post/2cb66410-c6da-47bd-9807-c23c67a08f2f/image.png)

이것은 UI가 렌더되기 이전에 모든 데이터를 기다려야 할 필요 없이 페이지의 부분을 더 빨리 보여지게 합니다.

스트리밍은 리액트의 컴포넌트 모델과 잘 작동하는데, 각 컴포넌트가 덩어리로 간주되기 때문입니다. 우선순위가 높거나(예 - 상품 정보) 데이터에 의존하지 않는 컴포넌트를 먼저 전송할 수 있으며, 리액트는 hydration을 먼저 시작할 수 있습니다. 낮은 우선순위를 가진 컴포넌트(예 - 리뷰, 관련 품목)는 데이터가 페치된 이후에 같은 서버 요청에 전송될 수 있습니다.

![](https://velog.velcdn.com/images/hyorimm/post/62e16afe-70e0-49b3-94da-0e59b3a0f4bb/image.png)

스트리밍은 TTFB(Time To First Byte) 및 FCP(First Contentful Paint)를 줄일 수 있으므로 긴 데이터 요청으로 인해 페이지 렌더링이 차단되는 것을 방지할 때 특히 유용합니다. 특히 느린 기기에서 TTI(Time To Interactive)을 개선하는 데도 또한 도움이 됩니다.

## 예

`<Suspense>`는 비동기적인 액션(예 - 데이터 페치)을 수행하는 컴포넌트를 감싸고 실행 중에 폴백 UI(예 - 스켈레톤, 스피너)를 보여주고 다음 작업이 완료되면 컴포넌트에서 바꾸는 방식으로 작동합니다.

```tsx
// @app/dashboard/page.tsx

import { Suspense } from 'react';
import { PostFeed, Weather } from './Components';

export default function Posts() {
  return (
    <section>
      <Suspense fallback={<p>Loading feed...</p>}>
        <PostFeed />
      </Suspense>
      <Suspense fallback={<p>Loading weather...</p>}>
        <Weather />
      </Suspense>
    </section>
  );
}
```

서스펜스를 사용하여 아래의 장점을 얻을 수 있습니다:

1. 스트리밍 서버 렌더링 - 서버에서 클라이언트로 HTML을 점진적으로의 렌더링
2. 선택적인 hydration - 컴포넌트는 사용자 상호 작용을 기반으로 상호 작용적이게 먼저 만들 컴포넌트의 우선 순위를 결정합니다.

## SEO

- 넥스트는 클라이언트에 UI를 스트리밍 하기 전에 `generateMetadata` 내에서 데이터 페칭이 완료되기를 기다립니다. 이것은 `<head>` 태그를 포함해서 스트리밍되는 응답의 첫 번째 부분을 보장합니다.
- 스트리밍은 서버에서 렌더되기 때문에 SEO에 영향을 주지 않습니다. 구글에서 제공하는 Mobile Friendly Test 도구를 사용하여 구글의 웹 크롤러에 페이지가 어떻게 표시되는지 확인하고 직렬화된 HTML을 볼 수 있습니다.
