# Dynamic Routes

정확한 세그먼트 이름을 모르거나 동적 데이터로 라우트를 만들고 싶다면 요청 시간이나 빌드 타임에 프리렌더로 채워지는 동적 세그먼트를 사용할 수 있습니다.

## 컨벤션

동적 세그먼트는 폴더의 이름을 [ ] 로 감싸서 만들 수 있습니다 : `[folderName]` 예를 들어, `[id]` 나 `[slug]` 처럼.

동적 세그먼트는 `params` prop으로 `layout`, `page`, `route` 그리고 `generateMatadata`에 전달됩니다.

## 예

예를 들어 블로그는 다음 `app/blog/[slug]/page.js` 를 포함할 수 있습니다. 여기서 `[slug]`는 블로그 포스트를 위한 동적 세그먼트입니다.

```tsx
// @app/blog/[slug]/page.tsx

export default function Page({ params }: { params: { slug: string } }) {
  return <div>My Post: {params.slug}</div>;
}
```

| Route                   | Example URL | params        |
| ----------------------- | ----------- | ------------- |
| app/blog/[slug]/page.js | /blog/a     | { slug: 'a' } |
| app/blog/[slug]/page.js | /blog/b     | { slug: 'b' } |
| app/blog/[slug]/page.js | /blog/c     | { slug: 'c' } |

generateStaticParams() 페이지에서는 세그먼트를 위해 어떻게 params를 생성하는지 배울 수 있습니다.

> **Good to know**: 동적 세그먼트는 `pages` 디렉토리의 동적 라우트와 같습니다.

## 정적 params 생성하기

`generateStaticParams` 함수는 동적 라우트 세그먼트와 함께 사용하여 요청 시 제공하는 것 대신 대신 빌드 시에 정적으로 라우트를 생성할 수 있습니다.

```tsx
// @app/blog/[slug]/page.tsx

export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json());

  return posts.map((post) => ({
    slug: post.slug,
  }));
}
```

`generateStaticParams` 함수의 주된 이점은 데이터를 스마트하게 검색하는 것입니다. fetch 요청을 사용하여 `generateStaticParams` 함수 내에서 내용이 페치된다면, 요청은 자동적으로 메모됩니다. 이것은 다중의 `generateStaticParams`, 레이아웃, 페이지에서 같은 인자를 이용한 페치 요청은 한번만 이루어지고 빌드 시간이 줄어듭니다.

페이지 디렉토리에서 마이그레이션 한다면 migration guide 를 사용하세요.

## 모든 세그먼트 검색

대괄호 안에 생략 표시로 동적 세그먼트는 모든 후속 세그먼트 검색으로 확장할 수 있습니다.`[...folderName]`.

예를 들어 `app/shop/[...slug]/page.js` 는 `/shop/clothes` 와 매치될 것입니다. 하지만 `/shop/clothes/tops`, `/shop/clothes/tops/t-shirts`등과도 매치됩니다.

| Route                      | Example URL | params                    |
| -------------------------- | ----------- | ------------------------- |
| app/shop/[...slug]/page.js | /shop/a     | { slug: ['a'] }           |
| app/shop/[...slug]/page.js | /shop/a/b   | { slug: ['a', 'b'] }      |
| app/shop/[...slug]/page.js | /shop/a/b/c | { slug: ['a', 'b', 'c'] } |

## 선택적 모든 세그먼트 검색

모든 세그먼트 검색은 두 번 쓴 대괄호 안에 파라미터를 넣어 선택적으로 만들 수 있습니다.`[[...folderName]]`.

예를 들어, `app/shop/[[...slug]]/page.js` 는`/shop/clothes`, `/shop/clothes/tops`, `/shop/clothes/tops/t-shirts` 에 더해서 `/shop` 과도 매치됩니다.

모든 세그먼트 검색과 선택적인 세그먼트 검색의 다른 점은 파라미터 없는 라우트 역시 매치된다는 점입니다. (위의 예시에서는 `/shop` 의 경우)

| Route                         | Example URL | params                    |
| ----------------------------- | ----------- | ------------------------- |
| app/shop/\[[...slug]]/page.js | /shop       | {}                        |
| app/shop/\[[...slug]]/page.js | /shop/a     | { slug: ['a'] }           |
| app/shop/\[[...slug]]/page.js | /shop/a/b   | { slug: ['a', 'b'] }      |
| app/shop/\[[...slug]]/page.js | /shop/a/b/c | { slug: ['a', 'b', 'c'] } |

## 타입스크립트

타입스크립트를 사용할 때, 설정된 경로 세그먼트에 따라 `params`를 위한 타입을 더할 수 있습니다.

```tsx
// @app/blog/[slug]/page.tsx

export default function Page({ params }: { params: { slug: string } }) {
  return <h1>My Page</h1>;
}
```

| Route                             | params Type Definition                 |
| --------------------------------- | -------------------------------------- |
| app/blog/[slug]/page.js           | { slug: string }                       |
| app/shop/[...slug]/page.js        | { slug: string[] }                     |
| app/[categoryId]/[itemId]/page.js | { categoryId: string, itemId: string } |

> **Good to know**: 이것은 미래에 [TypeScript plugin](https://nextjs.org/docs/app/building-your-application/configuring/typescript#typescript-plugin) 에서 자동적으로 해 줄 것입니다.
