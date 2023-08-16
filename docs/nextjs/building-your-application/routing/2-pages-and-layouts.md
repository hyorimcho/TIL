# Pages and Layouts

페이지와 레이아웃

넥스트 13의 앱 라우터는 페이지, 공용 레이아웃, 템플릿을 쉽게 만드는 파일 컨벤션을 소개합니다. 이 페이지는 이런 특별한 파일들을 어떻게 사용해야 하는지 안내합니다.

## 페이지

페이지는 라우트의 고유한 UI입니다. page.js 파일에서 컴포넌트 내보내기로 정의할 수 있습니다. 중첩 폴더를 사용하여 라우트를 정의하고 라우트를 공개적으로 액세스할 수 있도록 합니다.

앱 디렉토리 안에 page.js 파일을 더해 첫번째 페이지를 만들어 보세요.

```tsx
// @app/page.tsx

// `app/page.tsx` is the UI for the `/` URL
export default function Page() {
  return <h1>Hello, Home page!</h1>;
}
```

```tsx
// @app/dashboard/page.tsx

// `app/dashboard/page.tsx` is the UI for the `/dashboard` URL
export default function Page() {
  return <h1>Hello, Dashboard Page!</h1>;
}
```

> **Good to know**:
>
> - 페이지는 라우트 서브 트리의 나뭇잎 입니다.
> - `.js`, `.jsx`, `.tsx` 파일 확장자는 페이지를 위해 쓰입니다.
> - `page.js` 파일은 공개적으로 접근 가능한 라우트 세그먼트를 만드는 데에 필수적입니다.
> - 페이지는 기본적으로 서버 컴포넌트이지만 클라이언트 컴포넌트로 설정할 수 있습니다.
> - 페이지는 데이터를 페칭할 수 있습니다.

## 레이아웃

레이아웃은 여러개의 페이지가 공유하는 UI입니다. 네비게이션에서 레이아웃은 상태를 보존하고 상호작용을 유지하고 리렌더를 하지 않습니다. 레이아웃은 역시 중첩될 수 있습니다.

레이아웃을 layout.js 파일에서 default exporting 으로 정의할 수 있습니다. 컴포넌트는 렌더링 중에 children 레이아웃(존재한다면) 또는 자식 페이지로 채워지는 children prop을 받아들여야 합니다

```tsx
// @app/dashboard/layout.tsx

export default function DashboardLayout({
  children, // will be a page or nested layout
}: {
  children: React.ReactNode;
}) {
  return (
    <section>
      {/* Include shared UI here e.g. a header or sidebar */}
      <nav></nav>

      {children}
    </section>
  );
}
```

> **Good to know**:
>
> - 맨 위에 있는 레이아웃을 루트 레이아웃이라고 합니다. 이 필요한 레이아웃은 어플리케이션에서 전 페이지를 통해 공유됩니다. 루트 레이아웃은 `html` 와 `body` 태그를 꼭 포함하고 있어야 합니다.
> - 어떠한 라우트 세그먼트라도 그 자신만의 레이아웃을 옵셔널하게 정의할 수 있습니다. 이 레이아웃들은 세그먼트 안의 모든 페이지에서 공유될 것입니다.
> - 라우트의 레이아웃들은 기본적으로 중첩됩니다. 각 부모 레이아웃들은 리액트의 children prop을 사용해서 하위 자식 레이아웃을 감쌉니다.
> - 라우트 그룹을 사용하여 공유 레이아웃의 특정 경로 세그먼트 및 레이아웃의 외부를 선택할 수 있습니다.
> - 레이아웃은 기본적으로 서버 컴포넌트이지만 클라이언트 컴포넌트로 설정할 수 있습니다.
> - 레이아웃은 데이터를 페치할 수 있다. [Data Fetching](https://nextjs.org/docs/app/building-your-application/data-fetching) 섹션에 더 많은 정보가 있습니다.
> - 부모 레이아웃과 자식 레이아웃간 데이터를 전달해주는 것은 가능하지 않습니다. 하지만, 같은 데이터를 라우트 안에서 한 번이상 페치할 수 있으며 리액트는 성능에 영향을 주지 않고 요청을 자동으로 추론합니다.
> - 레이아웃은 현재 라우트 세그먼트에 접근할 수 없스빈다. 라우트 세그먼트에 접근하기 위해서는 클라이언트 컴포넌트에서 `useSelectedLayoutSegment` 또는 `useSelectedLayoutSegments` 를 사용해야 합니다.
> - `.js`, `.jsx`, `.tsx` 파일 확장자명은 레이아웃으로 사용될 수 있습니다.
> - `layout.js` 과 `page.js` 파일은 같은 폴더에 정의될 수 있습니다. 레이아웃이 페이지를 래핑할 것입니다.

## 루트 레이아웃 (필수)

루트 레이아웃은 앱 디렉토리의 상단에서 정의되고 모든 라우트에 적용됩니다. 이 레이아웃은 서버에서 반환되는 초기 HTML을 수정할 수 있습니다.

```tsx
// @app/layout.tsx

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

> **Good to know**:
>
> - 앱 디렉토리는 **반드시** 루트 레이아웃에 포함되어야 합니다.
> - 루트 레이아웃은 반드시 html 과 body 태그를 정의해야하는데, 넥스트가 자동으로 만들어주지 않기 때문이다.
> - HTML 요소의 `<head>` 를 수정하여 내장된 SEO 서포트를 사용할 수 있다. 예를 들어 `<title>` 요소
> - 다중 루트 레이아웃을 만들기 위해서 라우트 그룹을 사용할 수 있다.
> - 루트 레이아웃은 기본적으로 서버 컴포넌트이고 클라이언트 컴포넌트로 **설정할 수 없다.**

page 디렉토리로 마이그레이션 : 루트 레이아웃은 `[_app.js](https://nextjs.org/docs/pages/building-your-application/routing/custom-app)` 과 `[_document.js](https://nextjs.org/docs/pages/building-your-application/routing/custom-document)` 을 대체합니다.

## 중첩 레이아웃

폴더(`app/dashboard/layout.js`) 내부에 정의된 레이아웃은 특정한 라우트 세그먼트([`acme.com/dashboard`](http://acme.com/dashboard)) 및 해당 세그먼트가 활성화된 경우에 렌더링 됩니다. 기본적으로 파일 계층안에 있는 레이아웃은 중첩되어있고, 이것은 자식 레이아웃들을 children prop으로 감싼다는 것을 의미합니다.

```tsx
// @app/dashboard/layout.tsx

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return <section>{children}</section>;
}
```

> **Good to know**:
>
> - 루트 레이아웃만이 `<html>` 와 `<body>` 태그를 가질 수 있습니다.

위의 두 레이아웃을 결합한다면 루트 레이아웃(`app/layout.js`)은 대시보드 레이아웃(`app/dashboard/layout.js`)을 감싸 `app/dashboard/*` 안의 라우트 세그먼트를 모두 감싸게 됩니다.

두 레이아웃은 이렇게 중첩되게 됩니다.

라우트 그룹을 사용하여 공유된 레이아웃의 특정 라우트 세그먼트의 내 외부를 선택할 수 있습니다.

## 템플릿

템플릿은 자식 레이아웃과 페이지를 감싼다는 점에서 레이아웃과 비슷합니다. 라우트 전체에 걸쳐 지속되고 상태를 유지하는 레이아웃과 달리 템플릿은 탐색 중인 각 자식에 대한 새 인스턴스를 만듭니다. 이것은 유저가 템플릿을 공유하는 라우트 사이를 탐색할 때, 컴포넌트의 새 인스턴스가 마운트되고 돔 엘리먼트가 다시 만들어지고 상태는 보존되지 않으며 이펙트가 다시 동기화됩니다.

이러한 구체적인 행동이 필요한 경우가 있을 수 있으며 템플릿은 레이아웃보다 적절한 옵션일 것입니다. 예를 들면 :

- css나 애니메이션 라이브러리를 사용해 애니메이션에 들어가고 나오는 경우
- useEffect(로그인 페이지 화면) 이나 useState(페이지 별 피드백 형식)에 의존하는 기능
- 기본 프레임워크 행동을 변화시키는 경우. 예를 들어 레이아웃 안의 Suspense Boundaries는 레이아웃이 처음 로드 될 때만 fallback을 보여주고 페이지가 변할 때는 보여주지 않습니다. 템플릿의 경우, fallback이 각 네비게이션에 보여집니다.

```
추천: 템플릿을 사용하는 데에 구체적인 이유가 있지 않은 이상, 레이아웃 사용을 추천합니다.
```

템플릿은 `template.js` 파일에서 default 리액트 컴포넌트 내보내기로 정의될 수 있습니다. 컴포넌트는 중첩 세그먼트가 될 children prop을 받아들여야만 합니다.

```tsx
// @app/template.tsx

export default function Template({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>;
}
```

레이아웃과 템플릿을 가진 라우트 세그먼트의 렌더된 아웃풋은 아래와 같을 것입니다.

```tsx
<Layout>
  {/* Note that the template is given a unique key. */}
  <Template key={routeParam}>{children}</Template>
</Layout>
```

## <head> 수정하기

내장된 SEO support를 이용하여 `app` 디렉토리에서 `title`이나 `meta` 와 같은 `<head>` HTML 요소를 수정할 수 있습니다.

메타데이터는 `layout.js` 또는 `page.js` 파일의 `metadata` 객체나 `generateMetadata` 함수를 내보내기 하여 정의할 수 있습니다.

```tsx
// @app/page.tsx

import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Next.js',
};

export default function Page() {
  return '...';
}
```

> Good to know : `<title>` , `<meta>` 와 같은 `<head>` 태그는 루트 레이아웃에 직접 추가하면 안됩니다. 대신, 스트리밍이나 `<head>` 요소 중복 요청을 자동으로 처리해주는 진화한 Metadata API 를 사용해야 합니다.
