---
created: 2025-05-06 12:03:11
modified: 2025-05-06 12:03:30
source: https://nextjs-ko.org/docs/app/building-your-application/routing/parallel-routes#modals
dg-publish: true
---
## Modals

Parallel Routes는 [Intercepting Routes](https://nextjs-ko.org/docs/app/building-your-application/routing/intercepting-routes)와 함께 사용하여 딥 링크를 지원하는 모달을 생성할 수 있습니다. 이를 통해 모달을 구축할 때 일반적으로 발생하는 문제를 해결할 수 있습니다:

- 모달 콘텐츠를 **URL을 통해 공유 가능**하게 만들기.
- 페이지를 새로 고침할 때 **컨텍스트를 유지**하여 모달을 닫지 않기.
- 이전 탐색(뒤로가기)하면 **모달 닫기**.
- 앞으로 탐색하면 **모달 다시 열기**.

사용자가 클라이언트 측 탐색을 사용하여 레이아웃에서 로그인 모달을 열거나 별도의 `/login` 페이지에 접근할 수 있는 다음 UI 패턴을 고려해보세요:

![Parallel Routes Diagram](https://i.imgur.com/kZrEUOw.png)

이 패턴을 구현하려면, 먼저 **메인** 로그인 페이지를 렌더링하는 `/login` 라우트를 생성합니다.

![Parallel Routes Diagram](https://i.imgur.com/he3H2ko.png)


`app/login/page.tsx`
```ts
import { Login } from '@/app/ui/login'
 
export default function Page() {
  return <Login />
}
```

그런 다음, `@auth` slot 내에 [`default.js`](https://nextjs-ko.org/docs/app/api-reference/file-conventions/default) 파일을 추가하여 모달이 활성 상태가 아닐 때 렌더링되지 않도록 합니다.

`app/@auth/default.tsx`
```ts
export default function Default() {
  return '...'
}
```

`@auth` slot 내에서 `/(.)login` 폴더를 업데이트하여 `/login` 라우트를 가로챕니다. `/(.)login/page.tsx` 파일에 `<Modal>` 컴포넌트와 자식을 가져옵니다:

`app/@auth/(.)login/page.tsx`
```ts
import { Modal } from '@/app/ui/modal'
import { Login } from '@/app/ui/login'
 
export default function Page() {
  return (
    <Modal>
      <Login />
    </Modal>
  )
}
```

> [!TIP]
> - 라우트를 가로채기 위해 사용하는 규칙, 예: `(.)`는 파일 시스템 구조에 따라 다릅니다. [Intercepting Routes convention](https://nextjs-ko.org/docs/app/building-your-application/routing/intercepting-routes#convention)을 참조하세요.
> - 모달 콘텐츠 (`<Login>`)에서 `<Modal>` 기능을 분리함으로써 모달 내의 모든 콘텐츠, 예: [forms](https://nextjs-ko.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#forms),가 서버 컴포넌트가 되도록 할 수 있습니다. 자세한 내용은 [Interleaving Client and Server Components](https://nextjs-ko.org/docs/app/building-your-application/rendering/composition-patterns#supported-pattern-passing-server-components-to-client-components-as-props)를 참조하세요.

### 모달 열기

이제 Next.js 라우터를 활용하여 모달을 열고 닫을 수 있습니다. 이를 통해 모달이 열릴 때와 이전 및 이후 탐색 시 URL이 올바르게 업데이트되도록 보장할 수 있습니다.

모달을 열려면, 부모 레이아웃에 `@auth` slot을 prop으로 전달하고 `children` prop과 함께 렌더링합니다.

`app/layout.tsx`
```ts
import Link from 'next/link'
 
export default function Layout({
  auth,
  children,
}: {
  auth: React.ReactNode
  children: React.ReactNode
}) {
  return (
    <>
      <nav>
        <Link href="/login">Open modal</Link>
      </nav>
      <div>{auth}</div>
      <div>{children}</div>
    </>
  )
}
```

사용자가 `<Link>`를 클릭하면 `/login` 페이지로 이동하는 대신 모달이 열립니다. 그러나 새로 고침 또는 초기 로드 시 `/login`으로 이동하면 주요 로그인 페이지로 이동합니다.

### 모달 닫기

모달을 닫으려면 `router.back()`을 호출하거나 `Link` 컴포넌트를 사용할 수 있습니다.

`app/ui/modal.tsx`
```ts
'use client'
 
import { useRouter } from 'next/navigation'
 
export function Modal({ children }: { children: React.ReactNode }) {
  const router = useRouter()
 
  return (
    <>
      <button
        onClick={() => {
          router.back()
        }}
      >
        Close modal
      </button>
      <div>{children}</div>
    </>
  )
}
```

`Link` 컴포넌트를 사용하여 더 이상 `@auth` slot을 렌더링하지 않아야 하는 페이지로 이동할 때는 평행 라우트가 `null`을 반환하는 컴포넌트와 일치하도록 해야 합니다. 예를 들어, 루트 페이지로 돌아갈 때 `@auth/page.tsx` 컴포넌트를 생성합니다:

`app/ui/modal.tsx`
```ts
import Link from 'next/link'
 
export function Modal({ children }: { children: React.ReactNode }) {
  return (
    <>
      <Link href="/">Close modal</Link>
      <div>{children}</div>
    </>
  )
}
```

`app/@auth/page.tsx`
```ts
export default function Page() {
  return '...'
}
```

또는 다른 페이지(예: `/foo`, `/foo/bar` 등)로 이동할 때 catch-all slot을 사용할 수 있습니다:

`app/@auth/[...catchAll]/page.tsx`
```ts
export default function CatchAll() {
  return '...'
}
```

> [!TIP]
> - [Active state and navigation](https://nextjs-ko.org/docs/app/building-your-application/routing/parallel-routes#active-state-and-navigation)에서 설명한 동작으로 인해 slot과 더 이상 일치하지 않는 라우트로 클라이언트 측 탐색 시 slot이 계속 표시되므로 모달을 닫기 위해 slot이 `null`을 반환하는 라우트와 일치시켜야 합니다.
> - 다른 예로는 전용 `/photo/[id]` 페이지가 있는 갤러리에서 사진 모달을 열거나, 사이드 모달에서 쇼핑 카트를 여는 것이 포함될 수 있습니다.
> - Intercepted 및 Parallel Routes가 포함된 모달의 [예제 보기(opens in a new tab)](https://github.com/vercel-labs/nextgram).

## Loading and Error UI

Parallel Routes는 독립적으로 스트리밍될 수 있으므로 각 라우트에 대해 독립적인 오류 및 로딩 상태를 정의할 수 있습니다:

![Parallel routes enable custom error and loading states](https://i.imgur.com/aEBQZpP.png)

[Loading UI](https://nextjs-ko.org/docs/app/building-your-application/routing/loading-ui-and-streaming) 및 [Error Handling](https://nextjs-ko.org/docs/app/building-your-application/routing/error-handling) 문서를 참조하여 자세한 정보를 확인하세요.

Next: [Dynamic Routes](https://nextjs-ko.org/docs/app/building-your-application/routing/dynamic-routes "Dynamic Routes")