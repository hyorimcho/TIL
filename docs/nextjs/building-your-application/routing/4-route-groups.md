# Route Groups

`app` 디렉토리에서 중첩된 폴더는 일반적으로 URL 경로에 매핑됩니다. 하지만 Route Group으로 표시한다면 URL 경로에 포함되는 것을 막을 수 있습니다.

URL 경로 구조에 영향을 주지 않고 라우트 세그먼트 및 프로젝트 파일을 논리적 그룹으로 구성할 수 있습니다.

Route Groups 는 아래와 같이 사용 가능합니다 :

- 경로를 그룹으로 구성할 수 있습니다. 예) 지역 섹션, 의도나 팀
- 같은 라우트 세그먼트 레벨에서 중첩된 레이아웃을 가능하게 합니다.
  - 다중 루트 레이아웃을 포함한 같은 세그먼트에서의 다중 중첩 레이아웃
  - 공통 세그먼트에서 라우트 부분 집합에 레이아웃 추가

## 컨벤션

폴더 이름을 괄호로 묶어서 route group 을 만들 수 있습니다. `(folderName)`

## 예시

### URL 경로에 영향을 주지 않고 라우트 구성하기

URL에 영향을 주지 않고 경로를 구성하려면 관련 경로를 함께 유지할 그룹을 생성합니다. 괄호 안의 폴더( `(marketing)` 나 `(shop)`)는 URL에서 생략됩니다.

![](https://velog.velcdn.com/images/hyorimm/post/7314c1b8-8199-47b0-9a5c-68c8c6815c8c/image.png)

`(marketing)` 나 `(shop)` 안의 라우트가 같은 URL을 공유한다고 하더라도 각 폴더 안에 `layout.js` 파일을 추가하여 다른 레이아웃을 만들 수 있습니다.

![](https://velog.velcdn.com/images/hyorimm/post/3f8f7818-5544-4f46-8886-bd98682caf07/image.png)

### 레이아웃에 특정 세그먼트 선택

레이아웃에 특정 라우트를 선택하기 위해서는 `(shop)` 과 같은 새 라우트 그룹을 만들고 그 그룹에 `account` 나 `cart` 처럼 같은 레이아웃을 공유하세요. `checkout` 과 같은 그룹의 바깥에 있는 라우트는 레이아웃으로 공유되지 않을 것입니다.

![](https://velog.velcdn.com/images/hyorimm/post/2708ea73-ca3a-4c6e-866c-3c5750d81539/image.png)

### 다중 루트 레이아웃 생성

다중 루트 레이아웃을 생성하기 위해서는 상단의 `layout.js` 파일을 제거하고 각 라우트 그룹에 `layout.js` 파일을 추가해야 합니다. 이 기능은 어플리케이션을 완전히 다른 UI 또는 경험을 가진 섹션으로 분할할 때 유용합니다. `html`과 `body`태그는 각 루트 레이아웃에 더해져야 합니다.

![](https://velog.velcdn.com/images/hyorimm/post/d80169bd-64b7-4569-b573-2c4eb42b82ed/image.png)

위의 예시에서 `(marketing)` 과 `(shop)` 은 그들 각각의 루트 레이아웃을 가집니다.

> **Good to know**:
>
> - 라우트 그룹의 이름은 다른 조직들에 비해 특별한 중요도가 없습니다. 그 이름은 URL 경로에 영향을 주지 못합니다.
> - 라우트 그룹을 포함하는 라우트는 다른 라우트와 같은 URL경로로 **확인되지 않아야 합니다.** 예를 들어, 라우트 그룹은 URL 구조에 영향을 주지 않기 때문에, `(marketing)/about/page.js` 와 `(shop)/about/page.js` 는 `/about` 으로 확인될 것이고 에러를 낼 것입니다.
> - 상단의 `layout.js` 파일 없이 다중 루트 레이아웃을 사용한다면 홈 `page.js` 파일은 라우트 그룹의 하나로 정의되어야 합니다. 예) `app/(marketing)/page.js`.
> - Navigating **across multiple root layouts** will cause a **full page load** (as opposed to a client-side navigation). For example, navigating from `/cart` that uses `app/(shop)/layout.js` to `/blog` that uses `app/(marketing)/layout.js` will cause a full page load. This **only** applies to multiple root layouts.
> - **다중 루트 레이아웃을 탐색**하는 것은 **풀 페이지 로드**를 일으킵니다. (클라이언트 탐색과 반대로) 예를 들어  `app/(shop)/layout.js` 를 사용하는 `/cart` 에서 `app/(marketing)/layout.js` 를 사용하는 `/blog` 로의 탐색은 풀페이지 로드를 일으킵니다. 이것은 다중 루트 레이아웃에**만** 적용됩니다.
