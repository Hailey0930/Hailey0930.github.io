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

> 어쩌다 리액트 Suspense까지 타고 오게 됐는지 모르겠지만, Suspense가 대체 뭔지 몰라서 찾아보다가 쓰게 된 github.io 블로그 첫 포스트! <br />
> Suspense에 대해 생각보다 잘 정리된 글이 없어 내가 찾아본 여러 자료를 참고하여 이해한 플로우대로 정리해보고자 한다.

# `Suspense` 란?

React `Suspense`는 코드 스플리팅을 위해 등장한 컴포넌트이다. <br />
React 16 버전부터 React.lazy와 연계해서 사용했다. <br />
(**코드 스플리팅**이란, 네트워크를 통해 받아오는 javascript를 앱 구동에 필요한 만큼만 다운로드 받아올 수 있게 잘게 나누는 것을 말한다.) <br />

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
