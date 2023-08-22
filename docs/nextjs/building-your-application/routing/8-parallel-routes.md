# Parallel Routes

병렬 라우팅은 같은 레이아웃을 가진 하나 이상의 페이지에 동시 혹은 조건부 렌더링을 가능하게 합니다. SNS의 대시보드나 피드와 같은 앱의 동적 섹션에 병렬 라우팅은 복잡한 라우팅 패턴을 수행할 수 있습니다.

예를 들어 동시에 팀 페이지와 분석 페이지를 렌더할 수 있습니다.

![](https://velog.velcdn.com/images/hyorimm/post/83a0b532-82bc-409e-a2a7-a969d5988213/image.png)

병렬 라우팅은 독립적으로 스트리밍되는 각각의 라우트에 독립적인 에러와 로딩 스테이트를 정의할 수 있습니다.

![](https://velog.velcdn.com/images/hyorimm/post/b838a69f-9123-4ec0-aef2-789d4d59d8ae/image.png)

또한 인증 상태와 같은 특정 조건에 따라 슬롯을 조건부로 렌더링 할 수 있습니다.

![](https://velog.velcdn.com/images/hyorimm/post/034efe37-5c6a-4fdd-8dc1-9dec2d1aefe9/image.png)

## 컨벤션

병렬 라우트는 **slots** 이름을 사용하여 만들어집니다. slots는 `@folder`컨벤션과 함께 정의되고 props로 같은 레벨의 레이아웃에 전달됩니다.

<aside>
💡 slots 는 라우트 세그먼트가 아니며 URL 구조에 영향을 주지 못합니다. 파일 경로 /@team/members 는 /member에 접근할 수 있습니다.

</aside>

예를 들어 다음 파일 구조는 `@analytics`과 `@team`라는 두개의 외부 Slots를 정의합니다.

![](https://velog.velcdn.com/images/hyorimm/post/4b59d49e-78c9-4add-8f6a-0665e9a9510d/image.png)

위의 폴더 구조는 `app/layout.js` 의 컴포넌트가 이제 `@analytics` 과 `@team` slot을 props로 받아들이고 `children` prop으로 병렬적으로 렌더가 가능하다는 것을 의미합니다.

```tsx
// @app/layout.tsx

export default function Layout(props: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  team: React.ReactNode;
}) {
  return (
    <>
      {props.children}
      {props.team}
      {props.analytics}
    </>
  );
}
```

> Good to know: `children`prop은 폴더에 매핑될 필요가 없는 암시적인 slot 입니다. 이것은 `app/page.js` 와 `app/@children/page.js` 가 동일하다는 것을 뜻합니다.

## 일치하지 않는 라우트

기본적으로 slot 안에서 렌더되는 내용은 정확한 URL과 일치합니다.

일치되지 않는 slot의 경우에는, 넥스트가 렌더하는 내용은 라우팅 기술과 폴더 구조에 따라 다릅니다.

## `default.js`

넥스트가 현재 URL에 기반하여 slot의 활성화 상태를 회복하지 못할 때 대체 UI를 렌더하기 위해 `default.js` 파일을 정의할 수 있습니다.

다음 폴더 구조를 고려해보세요. `@team` slot은 `setting`디렉토리를 가지고 있지만 `@analytics`는 아닙니다.

![](https://velog.velcdn.com/images/hyorimm/post/3128a134-aff3-4f0b-b37d-9364d4943d9e/image.png)

### 네비게이션

네비게이션 중에 넥스트는 현재 URL과 일치하지 않더라도 slot의 이전 활성화 상태를 렌더할 것입니다.

### 리로드

리로드 중에 넥스트는 우선 일치하지 않는 슬롯인 `default.js`파일을 렌더하려고 할 것입니다. 그것이 불가능하다면 404가 렌더됩니다.

> 일치하지 않는 라우트에 대한 404는 병렬로 렌더링해서는 안 되는 라우트를 실수로 렌더링하지 않도록 보장합니다.

## `useSeletedLayoutSegment(s)`

`useSelectedLayoutSegment` 와 `useSelectedLayoutSegments` 는 해당 slot 내에서 활성 경로 세그먼트를 읽을 수 있는 `parallelRoutesKey`를 사용할 수 있습니다.

```tsx
// @app/layout.tsx

'use client';

import { useSelectedLayoutSegment } from 'next/navigation';

export default async function Layout(props: {
  //...
  auth: React.ReactNode;
}) {
  const loginSegments = useSelectedLayoutSegment('auth');
  // ...
}
```

사용자가 주소창에서 `@auth/login`나 `/login` 로 접근하면 `loginSegments` 는 문자 `"login"` 과 같습니다.

## 예

### 모달

병렬 라우팅은 모달을 렌더링 해줍니다

![](https://velog.velcdn.com/images/hyorimm/post/ddd1a5f2-05d2-4328-a754-c074cc690baf/image.png)

`@auth` slot은 `/login`과 같은 일치하는 라우트에 접근할 때 보여지는 <Moda> 컴포넌트를 렌더링합니다.

```tsx
// @app/layout.tsx

export default async function Layout(props: {
  // ...
  auth: React.ReactNode;
}) {
  return (
    <>
      {/* ... */}
      {props.auth}
    </>
  );
}
```

```tsx
// @app/@auth/login/page.tsx

import { Modal } from 'components/modal';

export default function Login() {
  return (
    <Modal>
      <h1>Login</h1>
      {/* ... */}
    </Modal>
  );
}
```

모달이 비활성화 되었을 때는 모달의 내용이 렌더링 되지 않도록 `null`을 리턴하는 `default.js`파일을 생성합니다.

```tsx
// @app/@auth/default.tsx

export default function Default() {
  return null;
}
```

### 모달 해제

`<Link href="/login">` 을 사용하여 클라이언트 네비게이션을 통해 모달이 활성화된 경우 `router.back()`을 호출하거나 `Link`컴포넌트를 사용하여 모달을 해제할 수 있습니다.

```tsx
// @app/@auth/login/page.tsx

'use client';
import { useRouter } from 'next/navigation';
import { Modal } from 'components/modal';

export default async function Login() {
  const router = useRouter();
  return (
    <Modal>
      <span onClick={() => router.back()}>Close modal</span>
      <h1>Login</h1>
      ...
    </Modal>
  );
}
```

다른 곳을 탐색하고 모달을 무시하려는 경우 catch-all 라우트를 사용할 수도 있습니다.

![](https://velog.velcdn.com/images/hyorimm/post/3390e354-c083-4609-a960-caebb4c69c65/image.png)

```tsx
// @app/@auth/[...catchAll]/page.tsx

export default function CatchAll() {
  return null;
}
```

> catch-all 라우트는 `default.js`보다 우선합니다.

### 조건부 라우트

병렬 라우트는 암시적인 조건부 라우팅입니다. 예를 들어 인증 상태에 따라 `@dashboard`나 `@login` 라우트를 렌더링 할 수 있습니다.

![](https://velog.velcdn.com/images/hyorimm/post/535ec550-9b76-47fd-ad18-4936469f07fc/image.png)
