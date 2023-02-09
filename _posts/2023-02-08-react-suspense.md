---
layout: post
title: "[React]Suspense"
summary: "React 18 Suspense에 대해 살펴보자! (feat. CSR, SSR)"
author: hyerin
date: "2023-02-08 14:35:23 +0530"
category: React
thumbnail: /assets/img/posts/230208_thumbnail.jpg
keywords: React, suspense
permalink:
usemathjax: true
---

> 어쩌다 리액트 `Suspense`까지 타고 오게 됐는지 모르겠지만, `Suspense`가 대체 뭔지 몰라서 찾아보다가 쓰게 된 github.io 블로그 첫 포스트! <br /> `Suspense`에 대해 생각보다 잘 정리된 글이 없어 내가 찾아본 여러 자료를 참고하여 이해한 플로우대로 정리해보고자 한다.<br /> 또한, Suspense와 함께 유저의 웹사이트 경험 향상에 도움이 되는 `Error Boundary`도 함께 살펴보자!

# `Suspense` 란?

React `Suspense`는 코드 스플리팅을 위해 등장한 컴포넌트이다. <br />
React 16 버전부터 React.lazy와 연계해서 사용했다. <br />
(**코드 스플리팅**이란, 네트워크를 통해 받아오는 Javascript를 앱 구동에 필요한 만큼만 다운로드 받아올 수 있게 잘게 나누는 것을 말한다.) <br />

```javascript
import React, { lazy, Suspense } from "react";

const AvatarComponent = lazy(() => import("./AvatarComponent"));

const renderLoader = () => <p>Loading</p>;

const DetailsComponent = () => (
  <Suspense fallback={renderLoader()}>
    <AvatarComponent />
  </Suspense>
);
```

## React.lazy

React는 유저가 '/' 경로에서 요청할 경우 `App.js`에서 `Route`로 불러와지는 컴포넌트들을 `Route`에서 설정한 경로로 가지 않았음에도 모든 컴포넌트들을 로딩해온다.<br />
이럴 때 `React.lazy`를 쓸 수 있다.

[ App.js ]

```javascript
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Home from "./Home";
import Board from "./Board";

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Routes path="/board" element={<Board />} />
      </Routes>
    </BrowserRouter>
  );
}
```

[ Board.js ]

```javascript
const Huge = React.lazy(() => import("@huge-library"));

export default function Board() {
  return (
    <div>
      <Huge />
    </div>
  );
}
```

용량이 큰 라이브러리 등 많은 시간과 데이터가 소요되는 것은 `React.lazy`를 통해 비동기로 import 해옴으로써, 해당 컴포넌트에 유저가 진입했을 때만 import 되도록 설정할 수 있다.

해당 컴포넌트에 유저가 진입할 때 import를 해오기 때문에, 유저가 순간적으로 빈 UI를 볼 수 있다. <br />
따라서 `Suspense`와 함께 사용할 경우 큰 시너지 효과를 볼 수 있다.

# 16버전에서의 `Suspense`

16버전에서는 다음과 같은 한계점이 있었다.

- SSR에서는 Suspense를 지원하지 않았다.
- Suspense가 소개된 이후, Apollo, Relay 등을 이용해 data fetching 전 fallback UI를 보여주는 용도로 사용할 수 있었지만 캐싱 지원의 불안정함으로 인해 권장하지 않았다.

React 18 버전이 나오면서, SSR에서도 Suspense를 지원하게 되었다.

# CSR, SSR

여기서 다시 살펴보는 CSR, SSR!

## CSR

CSR은 `Client Side Rendering`의 약자로, 클라이언트에서 초기 화면 로드를 위해 서버에 요청을 보내면 `서버`는 요청을 받아 클라이언트에게 HTML과 Javascript를 보내준다. <br />
`클라이언트`는 그것(HTML과 React로 짜여진 Javascript 코드)을 받아 브라우저에서 React의 Javascript 코드를 실행하여 렌더링을 시작한다.

서버에서 HTML, Javascript를 전달해준 후 브라우저가 Javascript를 실행하기 전까지 브라우저에는 아무것도 없기 때문에, 초기 로딩 시간이 오래 걸린다. <br />
따라서 환경이 안좋을 경우 유저는 웹사이트의 빈 화면을 볼 확률이 높다.

## SSR

SSR은 `Server Side Rendering`의 약자로, 클라이언트에서 초기 화면 로드를 위해 서버에 요청을 보내면 `서버`는 React 코드를 실행하여 화면에 표시하는데 필요한 데이터와 CSS를 모두 삽입한 HTML을 클라이언트에게 보내준다. <br />
`클라이언트`는 흰 화면 대신 브라우저에서 꽉 채워져있는 HTML을 미리 보여주며 React의 Javascript를 로딩 및 실행시켜 인터랙션을 추가한다. <br/>
( `서버`에서 HTML에 넣어둔 React의 Javascript와 `클라이언트`에서 전달받은 React의 Javascript를 비교 및 그리는 과정을 `Hydration`이라고 한다. ) <br />

SSR도 서버에서 먼저 전체 어플리케이션을 렌더링하기 때문에, 어떤 하나의 컴포넌트가 로딩하는데 오래 걸릴 경우 서버에서 전체 렌더링을 하는 동안에는 유저가 빈 화면을 볼 수 있다는 문제가 있다.

```javascript
<App>
  <Header />
  <Posts />
</App>
```

위와 같은 코드가 있고, App 컴포넌트를 렌더링할 경우 유저는 `Posts` 컴포넌트가 렌더링이 끝나기 전까지 `Header` 컴포넌트 역시 볼 수 없다.

# Suspense를 활용하여 문제 해결

위에서 살펴본 바와 같이, CSR SSR 모두 렌더링 과정에서 문제가 발생할 수 있다는 단점을 가지고 있다.<br />
이는 React 18버전 `Suspense`를 통해 해결할 수 있다!

```javascript
import { Suspense } from "react";

<App>
  <Header />
  <Suspense fallback={<Loader />}>
    <Posts />
  </Suspense>
</App>;
```

- `Suspense`로 `Posts` 컴포넌트를 감싸줄 경우, `Posts`컴포넌트의 렌더링이 오래 걸리더라도 기존과 달리 유저는 `Header` 컴포넌트를 미리 볼 수 있다.
- `HTTP stream`을 통해 `Posts` 컴포넌트가 서버에서 렌더링이 완료될 경우, 브라우저에서 React가 로딩되기도 전에 기존에 보여주던 `Loader`컴포넌트 HTML을 `Posts`컴포넌트 HTML로 대체한다.
- 아직 브라우저에서 로딩되지 않은 컴포넌트를 유저가 인터랙션하길 원하는 경우 (=로딩되지 않은 컴포넌트를 클릭할 경우), 다른 컴포넌트의 hydrate를 멈추고 해당 컴포넌트의 hydrate를 먼저 실행한다.

# Error Boundary

`Error Boundary`는 하위 컴포넌트 트리의 어디에서는 Javascript 에러를 기록하여 깨진 컴포넌트 트리 대신 fallback UI를 보여주는 React 컴포넌트이다.

```javascript
import { ErrorBoundary } from "react-error-boundary";

const UserProfileFallback = ({ error, resetErrorBoundary }) => {
  <div>
    <p> 에러 : {error.message} </p>
    <button onClick={() => resetErrorBoundary()}> 다시 시도 </button>
  </div>;
};

const handleOnError = (error) => sendErrorToErrorTracker(error);

export default const User = () => {
  <ErrorBoundary
    FallbackComponent={UserProfileFallback}
    onError={handleOnError}>
    <UserProfile />
  </ErrorBoundary>
}
```

`Error Boundary`를 사용할 경우 컴포넌트가 제공하는 `FallbackComponent`나 `onError`와 같은 props를 사용하여 기능을 구현할 수 있다. <br />
`resetErrorBoundary` 함수를 `FallbackComponent` 컴포넌트의 props로 제공하므로 다시 시도 등의 UI 요소도 쉽게 추가할 수 있다.

# Concurrent UI Pattern 구현을 위한 Suspense, Error Boundary 사용

Pattern에 대해 무지한 나는, 최근 각종 Pattern에 대해 관심이 생기기 시작했고 아직까지 개념은 잡혀 있지 않은 상태다.
`Suspense`에 대해 찾아보던 중, `Suspense`가 `Concurrent UI Pattern` 구현에 적합하다는 카카오페이 블로그 글을 보고 `Concurrent UI Pattern`에 대해 공부를 해봤다.

## Concurrent UI Pattern 이란?

`Concurrent`는 '동시의'라는 뜻을 갖고 있다. <br />
Javascript나 Javascript를 기반으로 만들어진 React는 다중 코어를 이용하여 병렬적으로 실생시키는 언어 또는 라이브러리가 아니기 때문에, **Concurrent 모드**는 최대한 동시성을 추구할 수 있는 방법을 도입하겠다는 의미를 담고 있다.

DOM 트리가 복잡해질수록 반응성 저하 등의 문제가 발생할 수 있는데, Concurrent 모드를 사용하면 앱이 빠른 반응 속도를 유지하도록 해주고 사용자 기기의 성능과 네트워크 속도에 맞춰 동작할 수 있게끔 만들 수 있다.<br />
이를 위해, `우선 순위에 따른 화면 렌더`, `컴포넌트의 지연 렌더`, `로딩 화면의 유연한 구성` 등을 쉽게 구성할 수 있도록 특성화된 기능들을 제공하고 있는데 이러한 기능들을 사용한 UI 개발 패턴을 React 팀에서 `Concurrent UI Pattern`이라고 부른다.

`Concurrent UI Pattern` 작성 방식은 개발론에서의 `선언적 프로그래밍`을 설명하는 방식과 비슷하다.<br />
일반적으로 개발할 때 컴포넌트를 `어떻게` 애플리케이션 상태에 따라 화면을 보여줄지에 집중하지만, `Concurrent UI Pattern`을 사용할 경우 사용자 경험 향상을 위해 컴포넌트가 `무엇을` 애플리케이션에서 보여줄지에 집중한다.

## Suspense, Error Boundary with React Query

React에서 비동기 데이터 관리를 위해 사용되는 라이브러리 `React Query`에서는 비동기 데이터 요청 시 `Suspense`와 `Error Boundary`를 활용할 수 있는 옵션을 제공한다.

```javascript
import { useQuery } from 'react-query';
import { ErrorBoundary } from "react-error-boundary";
import { Suspense } from 'react';

const queryKey = 'user';
const queryFn = () => axios('/user').then((res) => res.data);

const UserProfile = () => {
  const {data} = useQuery(queryKey, queryFn, {
    // 데이터 불러오기를 위한 Suspense를 활성화하는 옵션
    suspense: true
    // Error Boundary 사용을 위한 옵션으로, suspense 옵션이 true인 경우 기본값이 true로 설정된다.
    useErrorBoundary: true
  })

  return (
    <span>
      {data.name} / {data.birthDay}
    </span>
  )
};

const UserProfileFallback = ({error, resetErrorBoundary}) => {
  <div>
    <p> 에러: {error.message} </p>
    <button onClick={() => resetErrorBoundary()}> 다시 시도 </button>
  </div>
};

const UserProfileLoading = () => {
  <div>사용자 정보를 불러오는 중입니다.</div>
};

export default const User = () => {
  <ErrorBoundary FallbackComponent={UserProfileFallback}>
    <Suspense fallback={<UserProfileLoading />}>
      <UserProfile />
    </Suspense>
  </ErrorBoundary>
};
```

< 참고 : <br /> https://www.youtube.com/watch?v=7mkQi0TlJQo <br />
https://tech.kakaopay.com/post/react-query-2/#%EC%84%A0%EC%96%B8%ED%98%95-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%9C-react-component >
