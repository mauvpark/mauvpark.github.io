---
layout: post
title: (번역) Next 14 버전과 Next 15 버전의 차이 정리
description: Next 14 버전과 Next 15 버전의 차이에 알아보고 마이그레이션 시 고려할 사항에 대해 알아봅니다.
date: 2025-01-10 00:00:00 +0900
parent: React
categories: react,next-js,next15,next14,migration
nav_order: 3
comments: false
---

_2025-01-10_

# (번역) Next 14 버전과 Next 15 버전의 차이 정리

{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

[본문](https://nextjs.org/docs/app/building-your-application/upgrading/version-15)

## React 19

- `react`와 `react-dom`의 최소 버전은 19입니다.
- `useFormState`는 `useActionState`로 대체 되었습니다. `useFormState`는 React 19에서 여전히 사용할 수 있지만 미래에는 삭제 될 예정입니다. `pending`과 같은 추가적인 프로퍼티들을 포함하는 `useActionState` 사용을 권장합니다.
- 이제 `useFormStatus`는 `data`, `method` 그리고 `action`과 같은 추가적인 key들을 포함합니다. 만약 React 19를 사용하고 있지 않다면, `pending` key만 사용할 수 있습니다.
- [React 19 업그레이드 가이드](https://react.dev/blog/2024/04/25/react-19-upgrade-guide)에 대한 내용을 확인해 보세요.

> Tip: 만약 타입스크립트를 사용하고 있다면, `@types/react`와 `@types/react-dom`을 최신 버전으로 업그레이드 하세요.

## 비동기 Request APIs(Breaking change)

이전에 런타임 정보에 의존적이었던 동기적인 Dynamic APIs는 이제 **비동기화** 되었습니다:

- [`cookies`](https://nextjs.org/docs/app/api-reference/functions/cookies)
- [`headers`](https://nextjs.org/docs/app/api-reference/functions/headers)
- [`draftMode`](https://nextjs.org/docs/app/api-reference/functions/draft-mode)
- [`layout.js`](https://nextjs.org/docs/app/api-reference/file-conventions/layout), [`page.js`](https://nextjs.org/docs/app/api-reference/file-conventions/page), [`route.js`](https://nextjs.org/docs/app/api-reference/file-conventions/route), [`default.js`](https://nextjs.org/docs/app/api-reference/file-conventions/default), [`opengraph-image`](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/opengraph-image), [`twitter-image`](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/opengraph-image), [`icon`](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/app-icons) 그리고 [`apple-icon`](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/app-icons)에서의 `params`
- [`page.js`](https://nextjs.org/docs/app/api-reference/file-conventions/page)에서의 `searchParams`

### `cookies`

**비동기 사용을 권장**

```typescript
import { cookies } from 'next/headers';

// Before
const cookieStore = cookies();
const token = cookieStore.get('token');

// After
const cookieStore = await cookies();
const token = cookieStore.get('token');
```

**임시 동기 사용 시**

```typescript
import { cookies, type UnsafeUnwrappedCookies } from 'next/headers';

// Before
const cookieStore = cookies();
const token = cookieStore.get('token');

// After
const cookieStore = cookies() as unknown as UnsafeUnwrappedCookies
// Dev 환경에서 주의 알림이 뜸
const token = cookieStore.get('token');
```

### `headers`

**비동기 사용을 권장**

```typescript
import { headers } from 'next/headers';

// Before
const headersList = headers();
const userAgent = headersList.get('user-agent');

// After
const headersList = await headers();
const userAgent = headersList.get('user-agent');
```

**임시 동기 사용 시**

```typescript
import { headers, type UnsafeUnwrappedHeaders } from 'next/headers';

// Before
const headersList = headers();
const userAgent = headersList.get('user-agent');

// After
const headersList = headers() as unknown as UnsafeUnwrappedHeaders;
// Dev 환경에서 주의 알림이 뜸
const userAgent = headersList.get('user-agent');
```

### `draftMode`

**비동기 사용을 권장**

```typescript
import { draftMode } from 'next/headers';

// Before
const { isEnabled } = draftMode();

// After
const { isEnabled } = await draftMode();
```

**임시 동기 사용 시**

```typescript
import { draftMode, type UnsafeUnwrappedDraftMode } from 'next/headers';

// Before
const { isEnabled } = draftMode();

// After
// Dev 환경에서 주의 알림이 뜸
const { isEnabled } = draftMode() as unknown as UnsafeUnwrappedDraftMode;
```

### `params` & `searchParams`

**비동기 Layout**

```typescript
// Before
type Params = { slug: string };

export function generateMetadata({ params }: { params: Params }) {
  const { slug } = params;
};

export default async function Layout({
  children,
  params,
}: {
  children: React.ReactNode;
  params: Params;
}) {
  const { slug } = params;
}

// After
type Params = Promise<{ slug: string }>;

export async function generateMetadata({ params }: { params: Params }) {
  const { slug } = await params;
}

export default async function Layout({
  children,
  params,
}: {
  children: React.ReactNode;
  params: Params;
}) {
  const { slug } = await params;
}
```

**동기 Layout**

```typescript
// Before
type Params = { slug: string };

export default function Layout({
  children,
  params,
}: {
  children: React.ReactNode;
  params: Params;
}) {
  const { slug } = params;
}

// After
import { use } from 'react';

type Params = Promise<{ slug: string; }>;

export default function Layout(props: {
  children: React.ReactNode;
  params: Params;
}) {
  const params = use(props.params);
  const slug = params.slug;
}
```

**비동기 Page**

```typescript
// Before
type Params = { slug: string; };
type SearchParams = { [key: string]: string | string[] | undefined };

export function generateMetadata({
  params,
  searchParams,
}: {
  params: Params;
  searchParams: SearchParams;
}) {
  const { slug } = params;
  const { query } = searchParams;
}

export default async function Page({
  params,
  searchParams,
}: {
  params: Params;
  searchParams: SearchParams;
}) {
  const { slug } = params;
  const { query } = searchParams;
}

// After
type Params = Promise<{ slug: string; }>;
type SearchParams = Promise<{ [key: string]: string | string[] | undefined }>;

export async function generateMetadata(props: {
  params: Params;
  searchParams: SearchParams;
}) {
  const params = await props.params;
  const searchParams = await props.searchParams;
  const slug = params.slug;
  const query = searchParams.query;
}

export default async function Page(props: {
  params: Params;
  searchParams: SearchParams;
}) {
  const params = await props.params;
  const searchParams = await props.searchParams;
  const slug = params.slug;
  const query = searchParams.query;
}
```

**동기 Page**

```typescript
'use client'

// Before
type Params = { slug: string; };
type SearchParams = { [key: string]: string | string[] | undefined };

export default function Page({
  params,
  searchParams,
}: {
  params: Params;
  searchParams: SearchParams;
}) {
  const { slug } = params;
  const { query } = searchParams;
}

// After
import { use } from 'react';

type Params = Promise<{ slug: string }>;
type SearchParams = Promise<{ [key: string]: string | string[] | undefined }>;

export default function Page(props: {
  params: Params;
  searchParams: SearchParams;
}) {
  const params = use(props.params);
  const searchParams = use(props.searchParams);
  const slug = params.slug;
  const query = searchParams.query;
}
```

**Route 핸들러**

```typescript
// Before
type Params = { slug: string; };

export async function GET(request: Request, segmentData: { params: Params }) {
  const params = segmentData.params;
  const slug = params.slug;
}

// After
type Params = Promise<{ slug: string; }>;

export async function GET(request: Request, sementData: { params: Params }) {
  const params = await segmentData.params;
  const slug = params.slug;
}
```

### `runtime` 설정(Breaking change)

`runtime` 부분의 설정은 이전에 `edge`와 `experimental-edge`를 값으로서 지원해왔습니다. 두 설정값 모두 같은 것을 참조하고 있어 옵션 간소화를 위해 이제 `experimental-edge`을 사용하면 에러를 발생시킬 것입니다. 따라서 이 에러를 수정하기 위해서 `runtime`을 `edge`로 변경하기를 권장합니다.

### `fetch` requests

더이상 [`fetch` requests](https://nextjs.org/docs/app/api-reference/functions/fetch)는 기본적으로 캐시하지 않도록 변경됩니다.

특정 `fetch` requests를 캐싱 상태로 인입하기 위해 `cache: 'force-cache'` 옵션을 전달할 수 있습니다.

```typescript
export default async function RootLayout() {
  const a = await fetch('https://...'); // 캐시 되지 않음
  const b = await fetch('https://...', { cache: 'force-cache' }) // 캐시 됨

  // ...
}
```

한 layout 또는 한 페이지의 모든 `fetch` requests를 캐싱 상태로 인입하기 위해서는 `export const fetchCache = 'default-cache'` [segment config 옵션](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config)을 사용할 수 있습니다. 만약 개별 `fetch` requests에 `cache` 옵션을 지정하면, 그 옵션이 대신 사용됩니다.

```typescript
// 이 layout은 root layout이기 때문에
// 개별 캐시 옵션을 가지고 있지 않은 앱의 모든 fetch requests는 캐시됩니다.
export const fetchCache = 'default-cache';

export default async function RootLayout() {
  const a = await fetch('https://...'); // 캐시 됨
  const b = await fetch('https://...', { cache: 'no-store' }); // 캐시 되지 않음

  // ...
}
```

## Route 핸들러

더이상 [Route 핸들러](https://nextjs.org/docs/app/api-reference/file-conventions/route)에서 `GET` 함수는 기본적으로 캐시하지 않게 됩니다. `GET` 메서드들을 캐싱 상태로 인입하기 위해서는 `export const dynamic = 'force-static'`과 같은 [route config 옵션](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config)을 사용할 수 있습니다.

```typescript
export const dynamic = 'force-static';

export async function GET() {}
```

## Client-side Router 캐시

`<Link>` 또는 `useRouter`를 통한 pages 사이에서 navigating(페이지 변경)할 때, [page](https://nextjs.org/docs/app/api-reference/file-conventions/page) segments는 더이상 client-side router 캐시에서 사용되지 않습니다. 그러나, 브라우저 뒤로가기와 앞으로 가기 그리고 공유 layouts는 여전히 재사용 됩니다.

Page segments를 캐싱 상태로 인입하기 위해, [`staleTimes`](https://nextjs.org/docs/app/api-reference/config/next-config-js/staleTimes) config 옵션을 사용할 수 있습니다:

```typescript
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    staleTimes: {
      dynamic: 30,
      static: 180,
    },
  },
}

module.exports = nextConfig;
```

[Layouts](https://nextjs.org/docs/app/api-reference/file-conventions/layout)와 [loading states](https://nextjs.org/docs/app/api-reference/file-conventions/loading)는 여전히 캐시되고 navigation에서 재사용 됩니다.

## `next/font`

`@next/font` 패키지는 내장된 [`next/font`](https://nextjs.org/docs/app/api-reference/components/font)으로 인해 제거됩니다.

```typescript
// Before
import { Inter } from '@next/font/google';

// After
import { Inter } from 'next/font/google';
```

## bundlePagesRouterDependencies

`experimental.bundlePagesExternals`는 이제 안정화 되어 `bundlePagesRouterDependencies`로 명칭이 변경 되었습니다.

```typescript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Before
  experimental: {
    bundlePagesExternals: true,
  },

  // After
  bundlePagesRouterDependencies: true,
}

module.exports = nextConfig;
```

## serverExternalPackages

`experimental.serverComponentsExternalPackages`는 이제 안정화 되어 `serverExternalPackages`로 명칭이 변경 되었습니다.

```typescript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Before
  experimental: {
    serverComponentsExternalPackages: ['package-name'],
  },

  // After
  serverExternalPackages: ['package-name'],
}

module.exports = nextConfig;
```

## Speed Insights

Next 15에서는 Speed Insights를 위한 자동 장치들이 제거 되었습니다.

Speed Insights를 계속해서 사용하길 원한다면, [Vercel Speed Insights QuckStart](https://vercel.com/docs/speed-insights/quickstart) 문서를 참고 하세요.

## `NextRequest` Geolocation

이 프로퍼티들은 호스팅 제공자에 의해 얻을 수 있는 값들이기 때문에 `NextRequest`에서 `geo`와 `ip` 프로퍼티들이 삭제됩니다.

만약 Vercel을 사용하고 있다면 [`@vercel/function`](https://vercel.com/docs/functions/vercel-functions-package)에서 `geolocation`과 `ipAddress` 함수들로 대체할 수 있습니다:

```typescript
import { geolocation } from '@vercel/functions';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const { city } = geolocation(request);

  // ...
}
```

```typescript
import { ipAddress } from '@vercel/functions';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const ip = ipAddress(request);

  // ...
}
```

<script src="https://utteranc.es/client.js"
        repo="mauvpark/mauvpark.github.io" 
        issue-term="pathname"
        theme="github-light"
        label="comment"
        crossorigin="anonymous"
        async>
</script>