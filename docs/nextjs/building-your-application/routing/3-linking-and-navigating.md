# Linking and Navigating

넥스트에서 라우트간 이동하는 데에는 두 가지 방법이 있습니다.

- <Link> 컴포넌트를 사용
- useRouter 훅을 사용

이 페이지에서는 `<Link>` , `useRouter()` 의 사용법과 어떻게 이동이 이루어지는지 알아 볼 것입니다.

## <Link> 컴포넌트

HTML `<a>` 태그에서 확장되어 프리페칭과 클라이언트 측 라우트 사이의 네비게이션을 제공하는 `<Link>` 는 내장된 컴포넌트입니다. 이것이 넥스트에서 라우트 사이를 이동하는 주된 방법입니다.

`next/link` 에서 불러오기 해서 컴포넌트에 `href` 를 prop으로 전달하여 사용할 수 있습니다.

```tsx
// @app/page.tsx

import Link from 'next/link';

export default function Page() {
  return <Link href="/dashboard">Dashboard</Link>;
}
```

`<Link>` 에 전달할 수 있는 다른 선택적인 props도 있습니다. API reference 에서 더 자세히 알 수 있습니다.

### 예시

**동적 세그먼트에 연결하기**

동적 세그먼트에 연결할 때, 링크의 목록을 생성하기 위해 템플릿 리터럴과 보간법을 사용할 수 있습니다. 예를 들어 블로그 포스트의 목록을 생성하려면 :

```tsx
// @app/blog/PostList.js

import Link from 'next/link';

export default function PostList({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  );
}
```

**활성화된 링크 확인하기**

링크가 활성화되었는지 결정하기 위해서 `usePathname()` 을 사용할 수 있습니다. 예를 들어 활성화 된 링크에 클래스를 추가하기 위해서 현재의 `pathname` 이 링크의 `href` 와 일치하는지 확인할 수 있습니다.

```tsx
// @app/ui/Navigation.js

'use client';

import { usePathname } from 'next/navigation';
import Link from 'next/link';

export function Navigation({ navLinks }) {
  const pathname = usePathname();

  return (
    <>
      {navLinks.map((link) => {
        const isActive = pathname === link.href;

        return (
          <Link
            className={isActive ? 'text-blue' : 'text-black'}
            href={link.href}
            key={link.name}
          >
            {link.name}
          </Link>
        );
      })}
    </>
  );
}
```

**`id`로 스크롤 하기**

넥스트 앱 라우터의 기본 동작은 새 경로의 상단으로 스크롤 하거나 앞 뒤로 이동할 때에 스크롤 위치를 유지하는 것입니다.

특정한 이동 중에 `id`로 스크롤 하고 싶다면, URL에 `#`해시 링크를 붙이거나 `href` prop에 해시 링크를 전달할 수 있습니다. 이것은 `<Link>` 가 `<a>`요소로 렌더링되기 때문에 가능합니다.

```tsx
<Link href="/dashboard#settings">Settings</Link>

// Output
<a href="/dashboard#settings">Settings</a>
```

**스크롤 복구를 불가능하게 하기**

넥스트 앱 라우터의 기본 동작은 새 경로의 상단으로 스크롤 하거나 앞 뒤로 이동할 때에 스크롤 위치를 유지하는 것입니다. 이 동작을 불가능하게 하려면 `<Link>`컴포넌트에 `scroll={false}` 을 전달하거나, `router.push()` 혹은 `router.replace()` 에 `scroll:false`을 전달합니다.

```tsx
// next/link
<Link href="/dashboard" scroll={false}>
  Dashboard
</Link>
```

```tsx
// useRouter
import { useRouter } from 'next/navigation';

router.push('/dashboard', { scroll: false });
```

## useRouter() 훅

`useRouter` 훅은 기계적으로 경로를 변경할 수 있게 해줍니다.

이 훅은 클라이언트 컴포넌트 내에서만 사용할 수 있고 `next/navigation`에서 불러오기 합니다.

```tsx
// @app/page.js

'use client';

import { useRouter } from 'next/navigation';

export default function Page() {
  const router = useRouter();

  return (
    <button type="button" onClick={() => router.push('/dashboard')}>
      Dashboard
    </button>
  );
}
```

useRouter 메서드의 전체 목록을 보려면 API reference를 참조하세요.

```tsx
추천: useRouter를 써야하는 특별한 이유가 없다면 경로간 이동에는 <Link>컴포넌트를 사용하는 것을 추천합니다.
```

## 라우팅과 이동이 이루어지는 방식

앱 라우터는 라우팅과 이동에 하이브리드 접근 방식을 사용합니다. 서버에서 어플리케이션 코드는 루트 세그먼트에 의해 자동적으로 코드가 분할됩니다. 그리고 클라이언트에서 넥스트는 라우트 세그먼트를 미리 페치하고 캐시합니다. 이것은 사용자가 새로운 경로로 이동할 때, 브라우저는 페이지를 리로드하지 않고 리렌더를 변경하는 라우트 세그먼트 로드하여 이동 경험과 성능을 향상시킵니다.

1. 프리페칭

프리페칭은 사용자가 경로를 방문하기 전에 백그라운드에서 미리 로드하는 방식입니다.

넥스트에는 프리페치되는 두가지 방법의 라우트가 있습니다.

- `<Link>` 컴포넌트: 라우트가 유저의 뷰포트에 나타날 때 자동으로 페치됩니다. 프리페칭은 페이지가 먼저 로드되거나 스크롤을 통해 페이지를 볼 수 있을 때 프리페칭 됩니다.
- `router.prefetch()`: `useRouter`훅은 프로그램적으로 라우트를 프리페치하게 됩니다.

`<Link>`의 프리페칭 방식은 정적 라우트와 동적 라우트 간에 차이점이 있습니다.

- 정적 라우트: `prefetch`가 기본적으로 `true`입니다. 모든 라우트는 프리페치되고 캐시됩니다.
- 동적 라우트: `prefetch`가 기본적으로 자동입니다. 첫 번째 `loading.js` 파일이 30초 동안 프리페치되고 캐시될 때까지 공유 레이아웃만 다운됩니다. 이것은 전체 동적 라우트의 프리페칭 비용을 줄이고 유저에게 더 나은 시각적인 피드백을 위해 즉각적인 로딩 상태를 보여줄 수 있음을 뜻합니다.

`prefetch` prop에 `false`를 전달하면 프리페칭을 불가능하게 만들 수 있습니다.

> Good to know: 프리페칭은 개발 단계에서는 불가능하고 프로덕션 단계에서만 가능합니다.

1. **캐싱**

넥스트는 Router Cashe 라고 하는 메모리 내 클라이언트 측 캐시가 있습니다. 사용자가 앱을 탐색할 때, 프리페치된 라우트 세그먼트와 방문한 경로의 리액트 서버 컴포넌트 페이로드가 캐시에 저장됩니다.

이는 탐색 시 서버로 새로운 요청을 보내는 대신 캐시가 가능한 많이 재사용되는 것을 뜻하며 데이터 요청 및 전송 수를 줄여 성능을 향상시킵니다.

1. **부분 렌더링**

부분 렌더링은 클라이언트에서 네비게이션 리렌더시 변경되는 라우트 세그먼트만 의미하며, 공유된 세그먼트는 보존됩니다.

예를 들어 `/dashboard/settings` 와 `/dashboard/analytics` 의 두 형제 라우트를 탐색 시 `setting`과 `analytics`페이지는 렌더 될 것이고 공유된 `dashboard`레이아웃은 유지될 것입니다.

부분적인 렌더링이 없다면 각 탐색은 서버에 전 페이지를 요청할 것입니다. 변경되는 세그먼트만 렌더링하면 전송되는 데이터의 양과 시간이 줄어들어 성능이 향상됩니다.

1. **부드러운 탐색**

기본적으로 브라우저는 페이지 간 하드한 탐색을 수행합니다. 이는 브라우저가 페이지를 리로드하고 앱의 `useState`훅과 같은 리액트 상태와 유저의 스크롤 위치나 요소 포커스와 같은 브라우저 상태를 리셋한다는 뜻입니다. 하지만 넥스트의 앱 라우터는 부드러운 탐색을 합니다. 리액트는 리액트 및 브라우저 상태를 유지하는 동안 변경된 세그먼트만 렌더링하며 전체 페이지 리로드는 없다는 뜻입니다.

1. **앞 뒤로 탐색**

기본적으로 넥스트는 앞 뒤 탐색에서 스크롤 위치를 유지하고 Router Cache의 라우트 세그먼트를 재사용합니다.
