# Intercepting Routes

Intercepting Routes는 현재 레이아웃 내에서 어플리케이션의 다른 부분에서 라우트를 로드할 수 있습니다. 이 라우트 패러다임은 사용자가 다른 컨텍스트로 전환하지 않고 라우트의 내용을 표시하기를 원할 때에 유용할 수 있습니다.

예를 들어 피드의 사진을 클릭했을 때 피드에 오버레이되어 모달에서 사진을 표시할 수 있습니다. 이 경우 넥스트는 `/photo/123` 경로를 가로채고 URL을 만들어 `/feed`위에 겹칩니다.

![](https://velog.velcdn.com/images/hyorimm/post/ee6feafa-027b-405d-a0ed-de78784e18a5/image.png)

하지만 공유 가능한 URL을 클릭하거나 페이지를 새로고침해서 사진으로 이동할 때, 모달 대신 전체 사진 페이지는가 렌더되어야 합니다. route interception은 일어나지 않아야 합니다.

![](https://velog.velcdn.com/images/hyorimm/post/de6459b4-e868-4749-be71-424ef50bff11/image.png)

## 컨벤션

Intercepting routes는 `(...)` 컨벤션으로 정의되어야 합니다. 이것은 경로 컨벤션인 `../` 와 닮았지만 세그먼트를 위한 것입니다.

아래와 같이 사용할 수 있습니다:

- `(.)` 는 **같은 레벨**의 세그먼트와 일치시킵니다.
- `(..)` **한 단계 위 레벨**의 세그먼트와 일치시킵니다.
- `(..)(..)` **두 단계 위 레벨**의 세그먼트와 일치시킵니다.
- `(...)` **루트** `app` 디렉토리에서 세그먼트와 일치시킵니다.

예를 들어, `(..)photo` 디렉토리를 생성함으로써 `feed` 세그먼트 내부의 `photo` 세그먼트를 가로챌 수 있습니다.

![](https://velog.velcdn.com/images/hyorimm/post/2a4408f8-bd2b-498d-9746-f8654d8c8968/image.png)

> (..) 컨벤션은 파일 시스템이 아닌 *라우트 세그먼트*에 기반합니다.

## 예

### 모달

Intercepting Routes는 모달을 만들기 위해 병렬 라우트와 함께 사용할 수 있습니다.

이 패턴을 사용하여 모달을 만드는 것은 다음과 같은 몇 가지 일반적인 문제를 해결할 수 있습니다:

- URL을 통해서 공유 가능한 모달 내용을 만드는 것
- 페이지가 새로고침 되었을 때 모달을 닫는 것 대신 내용을 유지하는 것
- 뒤로 가기를 했을 때 이전 라우트로 가는 것이 아니라 모달을 닫는 것
- 앞으로 가기를 했을 때 모달이 다시 열리는 것

![](https://velog.velcdn.com/images/hyorimm/post/6ecb592c-2fa7-4306-b6ec-05f40214ff09/image.png)

> `@modal` 은 세그먼트가 아닌 slot이기 때문에 위의 예시에서 `photo` 세그먼트로의 경로는 `(..)` matcher를 사용할 수 있습니다. 이것은 `photo`라우트가 두 개의 파일 시스템 레벨이 더 높음에도 불구하고 한 단계 위의 세그먼트 레벨만 높다라는 의미입니다.

다른 예로는 상단 내브바에서 로그인 모달을 여는 동시에 `/login`페이지를 갖거나 사이드 모달에서 쇼핑 카트를 여는 것이 있습니다.
