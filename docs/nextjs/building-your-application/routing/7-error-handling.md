# Error Handling

`error.js` 파일 컨벤션은 예상되지 중첩 라우트에서 못한 런타임 에러를 정상적으로 처리할 수 있도록 합니다.

- 자동적으로 라우트 세그먼트와 에러 바운더리 내의 중첩된 자식을 감쌉니다.
- 파일 시스템 계층을 사용하여 특정 세그먼트에 맞게 조정된 에러 UI를 생성하여 세분화를 조정합니다.
- 나머지 어플리케이션은 기능을 유지하는 동안 영향을 받는 세그먼트에 오류를 격리합니다.
- 전체 페이지 리로드 없이 에러 복구를 시도하는 기능을 더합니다

`error.js` 파일을 라우트 세그먼트 안에 추가하고 리액트 컴포넌트로 내보내기 하여 에러 UI를 생성합니다.

![](https://velog.velcdn.com/images/hyorimm/post/0e87ace5-614f-40fd-8d79-7bed2bd3e82d/image.png)

```tsx
// @app/dashboard/error.tsx

'use client'; // Error components must be Client Components

import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  useEffect(() => {
    // Log the error to an error reporting service
    console.error(error);
  }, [error]);

  return (
    <div>
      <h2>Something went wrong!</h2>
      <button
        onClick={
          // Attempt to recover by trying to re-render the segment
          () => reset()
        }
      >
        Try again
      </button>
    </div>
  );
}
```

## `error.js` 의 작동 방법

![](https://velog.velcdn.com/images/hyorimm/post/1b9712a9-e630-4d58-bde5-f098e493a885/image.png)

- `error.js`는 중첩된 자식 세그먼트나 `page.js` 컴포넌트를 감싸는 React Error Boundary를 자동적으로 생성합니다.
- `error.js` 파일에서 내보내기 된 리액트 컴포넌트는 폴백 컴포넌트로 사용됩니다.
- 에러 바운더리 내에서 에러가 발생하면 에러가 포함되고 폴백 컴포넌트가 렌더링됩니다.
- 폴백 에러 컴포넌트가 활성화되면 에러 바운더리 위의 레이아웃은 상태와 상호작용을 유지하고, 에러 컴포넌트는 에러를 복구하기 위한 기능을 나타낼 수 있습니다.

## 에러 복구

에러의 이유는 때때로 일시적일 수 있습니다. 이런 경우는 이슈를 해결하기 위해 다시 시도해 볼 수 있습니다.

에러 컴포넌트는 사용자에게 오류 복구를 시도하라고 `reset()` 함수를 사용할 수 있습니다. 실행되면 함수는 에러 바운더리의 내용을 다시 렌더할 것입니다. 성공이라면 폴백 에러 컴포넌트는 리렌더된 결과로 교체됩니다.

```tsx
// @app/dashboard/error.tsx

'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

## 중첩 라우트

특별한 파일을 통해 생성된 컴포넌트는 특정 중첩 계층 구조로 렌더링 됩니다.

예를 들어 모두 `layout.js`및 `error.js`파일이 포함된 두 세그먼트가 있는 중첩 라우트는 다음과 같이 단순하된 컴포넌트 계층에서 렌더됩니다.

![](https://velog.velcdn.com/images/hyorimm/post/99399ec0-33ff-48ac-8f3e-d7528e052f96/image.png)

중첩 컴포넌트 계층은 중첩 라우트 간에 `error.js` 파일의 동작에 영향을 끼칩니다.

- 가장 가까운 부모 에러 바운더리까지 에러가 버블링됩니다. 이것은 `error.js`파일이 모든 중첩된 자식 세그먼트의 에러를 핸들링한다는 의미입니다. 라우트의 중첩된 폴더에 다른 수준의 `error.js` 파일을 배치하여 더욱 세분화된 에러 UI를 만들 수 있습니다.
- `error.js` 바운더리는 같은 세그먼트 내의 `layout.js` 컴포넌트에서 생긴 에러까지 다루지 않을 것입니다. 에러 바운더리는 레이아웃 컴포넌트 내에 중첩되어 있기 때문입니다.

## 레이아웃 에러 핸들링

`error.js`바운더리는 layout.js 나 template.js 컴포넌트의 같은 세그먼트에서 생긴 에러를 잡지 못합니다. 이거것은 의도적인 계층입니다. 이 의도적인 계층 구조는 오류가 발생할 때 형제 라우트(네비게이션과 같은) 간에 공유되는 중요한 UI를 표시하고 작동하도록 유지합니다.

루트 레이아웃이나 템플릿 내에서 에러를 다루려면 `global-error.js`라는 `error.js`의 변수를 사용하세요.

## 루트 레이아웃 에러 핸들링

루트 `app/error.js`바운더리는 `app/layout.js`나 `app/template.js` 컴포넌트에서 발생한 에러를 처리하지 못합니다.

특별히 이런 루트 컴포넌트의 에러를 다루기 위해서는 루트 `app`디렉토리에 위치한 `global-error.js`라는 `error.js`의 변수를 사용합니다.

`error.js`와 다르게 `global-error.js`에러 바운더리는 전체 어플리케이션을 감싸고, 활성화 되었을 때 폴백 컴포넌트는 루트 레이아웃을 대체합니다. 이러한 이유로 `global-error.js`는 `<html>` ,`<body>` 태그를 반드시 정의해야 하는 것이 중요합니다.

`global-error.js`는 최소 세분화된 에러 UI이며 전체 어플리케이션에 대한 “catch-all”에러 핸들링으로 고려될 수 있습니다. 루트 컴포넌트는 일반적으로 덜 동적이고 다른 `error.js` 바운더리가 대부분의 에러를 잡기 때문에 자주 트리거되지 않습니다.

만약 `global-error.js`가 정의되더라도, 전역적으로 공유되는 UI와 브랜딩을 포함하는 루트 레이아웃 내에서 렌더되는 폴백 컴포넌트인 루트 `error.js`를 정의하는 것을 추천합니다.

```tsx
// @app/global-error.tsx

'use client';

export default function GlobalError({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <h2>Something went wrong!</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  );
}
```

## 서버 에러 핸들링

서버 컴포넌트에서 오류가 발생했다면 넥스트는 `Error`객체를 가장 근처에 있는 `error.js` 파일에 `error` prop으로 전달할 것입니다.

### 민감한 에러 정보 보호

프로덕션동안 클라이언트로 전달된 Error 객체는 일반적인 `message`, `digest` 속성만 포함됩니다.

이것은 에러에 포함된 잠재적으로 민감한 정보가 클라이언트에게 유출되는 것을 피하기 위한 보안 예방 조치입니다.

이 `message`속성은 에러에 대한 일반적인 메세지를 포함하고 `digest`속성은 자동적으로 발생되는 에러의 해시를 포함합니다. 해시는 서버 사이드 로그에서 상응하는 에러를 맞춰보는 데에 사용할 수 있습니다.

개발동안 클라이언트에 전달된 `Error`객체는 직렬화되고 디버깅을 쉽게 하기 위해서 기존의 에러 메세지가 포함됩니다.
