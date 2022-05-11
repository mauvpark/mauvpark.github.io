---
layout: post
title: SSR(ServerSideRendering)과 Child Component에서의 적용 여부
date: 2022-05-11 19:02:00 +0900
parent: React
categories: react, ssr, nextJs, waterfall
nav_order: 5
---

*2022-05-11 19:02 작성*

# SSR(ServerSideRendering)과 Child Component에서의 적용 여부
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

[![HitCount](https://hits.dwyl.com/mauvpark/mauvparkgithubio/blob/gh-pages/docs/React/2022-05-11-about-ssr-and-child-ssrmd.svg?style=flat-square)](http://hits.dwyl.com/mauvpark/mauvparkgithubio/blob/gh-pages/docs/React/2022-05-11-about-ssr-and-child-ssrmd)

## SSR의 형태와 적용 이유

```js
export async function getServerSideProps() {
	const res = await Promise.all([
		fetchData(posts, 0),
		fetchData(userList, 0),
	]);
	return {
		props: {
			postsData: res[0],
			usersData: res[1],
		},
	};
}
```

SSR은 위와 같은 형태를 가지고 있다. 보통 SSR을 적용하지 않은 페이지의 경우, 렌더링이 모두 진행된 후에 api의 데이터를 페이지에 업데이트하게 되고, 사용자는 api에서 받아온 데이터를 보기 위해서 api가 load 되기를 기다려야 한다. 만약, 서버에서 *사용자와 상호작용하지 않고* 보여주는데에만 필요한 데이터를 미리 계산하고, 계산된 데이터를 통해 화면을 보여줄 수 있다면 사용자 입장에서는 기본적으로 표현된 UI 상에서 가만히 기다릴 필요는 없을 것이고 때에 따라 만족도가 높을 것이다. 또한, 정적 데이터를 업데이트 하는데 있어 불필요한 업데이트를 줄이고 빠르게 초기 페이지를 구성할 수 있을 것이다. 이 때, SSR을 사용한다.

## SSR은 하위 컴포넌트에서도 동작할까?

작동방식에 대한 논리적 판단을 떠나 눈으로 작동여부를 직접 보는 것은 중요하기에 다음과 같은 코드를 구성했다.

1. SSR로 가져오고자 하는 데이터는 `posts` 데이터와 `userList` 데이터다. 

2. SSR을 이용해 메인 페이지에서 `posts` 데이터를 가져오고 하위 컴포넌트에서 `userList` 데이터를 가져온다. (보통의 경우 `userList`를 가져오고 `posts`를 가져오지만, 테스트 용도라는 점을 참고했으면 한다.)

3. SSR에서 가져온 데이터가 화면에 정상적으로 표시되는지 확인한다.

### SSR로 가져오고자 하는 데이터는 `posts` 데이터와 `userList` 데이터다.

가설: 아래의 코드는 데이터를 가져오고 페이지 혹은 컴포넌트에 props 형태로 계산된 데이터를 넘길 것이다.

```js
// https://github.com/mauvpark/next-js-playground/blob/main/pages/ssr.tsx
export async function getServerSideProps() {
	const data = await fetchData(userList, 0);
	return {
		props: {
			data,
		},
	};
}

// https://github.com/mauvpark/next-js-playground/blob/main/components/users.tsx
export async function getServerSideProps() {
	const data = await fetchData(userList, 0);
	return {
		props: {
			data,
		},
	};
}
```

### SSR을 이용해 메인 페이지에서 `posts` 데이터를 가져오고 하위 컴포넌트에서 `userList` 데이터를 가져온다.

메인 페이지인 `ssr.tsx`은 아래와 같이 구성한다. `Ssr.tsx`에서 `posts` 데이터를 SSR로 가져오고, `Users` 컴포넌트에서 `userList` 데이터를 SSR로 가져올 것이다.

```js
// ssr.tsx
export async function getServerSideProps() {
	const data = await fetchData(posts, 0);
	return {
		props: {
			data,
		},
	};
}

const Ssr = ({ data }: ssrProps) => {
	const router = useRouter();
	const [length, setLength] = useState(0);

	useEffect(() => {
		setLength(data.length);
	}, [data.length]);

	return (
		<Page>
			<Head>
				<title>SSR Test Page</title>
				<meta
					name="viewport"
					content="initial-scale=1.0, width=device-width"
				/>
			</Head>
			<h1 onClick={() => navigateHandler(router, "/")}>
				Users
			</h1>
      {/* Users 컴포넌트에서 userList 데이터를 SSR로 불러올 예정 */}
			<Users />
			<h1 onClick={() => navigateHandler(router, "/")}>
				Posts({length})
			</h1>
      {/* SSR로 서버에서 미리 불러온 posts 데이터에 따라 매핑 */}
			{data?.map((p, i) => (
				<Post post={p} key={i} />
			))}
			<Footer />
		</Page>
	);
};

export default Ssr;
```

```js
// users.tsx
export async function getServerSideProps() {
	const data = await fetchData(userList, 0);
	return {
		props: {
			data,
		},
	};
}

const Users = ({ data }: usersProps) => {
	return (
		<div>
			{data?.map((u, i) => (
				<User data={u} key={i} />
			))}
		</div>
	);
};

export default Users;
```

### SSR에서 가져온 데이터가 화면에 정상적으로 표시되는지 확인한다.

검증: `ssr.tsx`에서 데이터는 정상적으로 표시됐고, 하위 컴포넌트인 `users.tsx`의 데이터는 정상적으로 표시되지 않았다. 노드에서 SSR이 처리되고, 렌더링이 진행된 후에 SSR을 처리하는 것은 SSR이 지향하는 바가 아니므로 작동이 정상적으로 되지 않아야 할 것이다. 그리고 이러한 처리는 waterfall로 인해 데이터 처리가 늦어지므로(ssr.tsx -> users.tsx) 이 방식보다는 다른 방식을 통하는 것이 좋다.

### 컴포넌트들의 상위인 페이지 단위에서 SSR을 사용한다.

결론: `ssr.tsx`에서 초기 업데이트에 필요한 데이터를 불러오고 하위 컴포넌트에 불러온 데이터를 drilling 한다면 waterfall에 대한 문제를 해결하고, `users.tsx`에 데이터를 정상적으로 초기 로드할 수 있다.

```js
export async function getServerSideProps() {
  // fetch all or nothing.
	const res = await Promise.all([
		fetchData(posts, 0),
		fetchData(userList, 0),
	]);
	return {
		props: {
			postsData: res[0],
			usersData: res[1],
		},
	};
}

const Ssr = ({ postsData, usersData }: ssrProps) => {
	const router = useRouter();
	const [postsLength, setPostsLength] = useState(0);
	const [usersLength, setUsersLength] = useState(0);

	useEffect(() => {
		setPostsLength(postsData.length);
	}, [postsData.length]);

	useEffect(() => {
		setUsersLength(usersData.length);
	}, [usersData.length]);

	return (
		<Page>
			<Head>
				<title>SSR Test Page</title>
				<meta
					name="viewport"
					content="initial-scale=1.0, width=device-width"
				/>
			</Head>
			<h1 onClick={() => navigateHandler(router, "/")}>
				Users({usersLength})
			</h1>
      {/* SSR로 가져온 데이터를 injecting */}
			<Users data={usersData} />
			<h1 onClick={() => navigateHandler(router, "/")}>
				Posts({postsLength})
			</h1>
			{postsData?.map((p, i) => (
				<Post post={p} key={i} />
			))}
			<Footer />
		</Page>
	);
};

export default Ssr;
```

위와 같이 SSR 데이터를 가져오게 되면, 하위 컴포넌트에 미리 로드한 데이터를 적용할 수 있으므로 순차적 로드를 하지 않고도 한 번에 모든 페이지를 로드할 수 있게 된다(waterfall 문제 해결).

## 참고

[Mauv's Next Js Playground](https://github.com/mauvpark/next-js-playground)

## 라이센스

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="크리에이티브 커먼즈 라이선스" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />이 저작물은 <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">크리에이티브 커먼즈 저작자표시-동일조건변경허락 4.0 국제 라이선스</a>에 따라 이용할 수 있습니다.

<script src="https://utteranc.es/client.js"
        repo="mauvpark/mauvpark.github.io" 
        issue-term="pathname"
        theme="github-light"
        label="comment"
        crossorigin="anonymous"
        async>
</script>
