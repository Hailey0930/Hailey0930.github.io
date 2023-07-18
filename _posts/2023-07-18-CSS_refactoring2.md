---
layout: post
title: "[CSS] CSS breakpoint refactoring2"
summary: "breakpoint 복붙 없애기 with styled-component"
author: hyerin
date: "2023-07-18 10:20:00 +0900"
category: CSS
thumbnail: /assets/img/posts/230718_thumbnail.jpg
keywords: CSS, breakpoint, refactoring
permalink:
usemathjax: true
---

> 이전 글에선 emotion을 사용할 때 breakpoint 복붙 없애기를 살펴보았다. 기간이 좀 지났지만 styled-component에 대해서도 살펴보자!

# legacy code

마찬가지로 styled-component에서도 진절머리가 났던 breakpoint 복붙 구문을 가져와보자
palette 파일에 breakpoint를 따로 빼서 관리를 한다고 해도, media query 구문이 있는 부분을 굳이 찾아 가서 복사해 온 후 필요한 구문에 붙여넣기를 했었다.

#### [styles.ts 파일]

```javascript
import styled from "styled-components";
import { breakPoints } from "../Palette";

export const LiveCardWrapper = styled.div`
  display: flex;
  align-items: flex-end;

  width: 100vw;
  height: 318px;
  padding-right: 100px;
  position: relative;
  overflow-x: auto;
  white-space: nowrap;

  @media screen and (max-width: ${breakPoints.tablet}px) {
    padding-right: 60px;
  }

  @media screen and (max-width: ${breakPoints.mobile}px) {
    height: 40.3646vw;
    min-height: 225px;
    padding-right: 32px;
  }
`;
```

#### [palette.ts 파일]

```javascript
export const breakPoints = {
  deskTop: 1920,
  tablet: 1439,
  mobile: 767,
};
```

# styled-component 공부하기

styled-component는 emotion처럼 composition 같은 것을 지원해주진 않는다. <br />
그 대신 자바스크립트 method를 사용해서 모듈화 하는 방식이 있어 한 [블로그](https://velog.io/@ye-ji/React%EC%97%90%EC%84%9C-Media-Query-%EC%9E%91%EC%84%B1%ED%95%98%EB%8A%94-%ED%8E%B8-styled-componentsJS%EC%99%80-TS)를 참조해보았다.

## media query 공통 함수 만들기

#### [mediaQuery.ts 파일]

```javascript
import { CSSProp } from 'styled-components';
import { breakPoints } from "./palette";

export const mediaQuery = (Object.keys(breakPoints) as Array<keyof typeof breakPoints>).reduce(
  (acc, key) => {
    acc[key] = (style: string) => `@media screen and (max-width: ${breakPoints[key]}px) {
      ${style}
    }`
    return acc;
  },
  {} as { [key in keyof typeof breakPoints] : (style: string) => CSSProp}
);
```

typescript랑 코드가 합쳐지면서 해석에 어려움을 겪었다. 결국 chat GPT의 도움을 받았고..ㅎㅎ 위 코드는 다음과 같은 의미이다.<br />
breakPoints의 key들을 mediaQuery 구문에 전달해줌으로써 mediaQuery에서는 해당 키로 breakPoint의 픽셀 값을 가져온다.<br />
reduce 함수에서 해당 mediaQuery 구문을 반환해주고, reduce 함수의 초기 누산값으로 breakPoints를 키값으로 가지고 styled-component의 CSSProp을 value로 갖는 빈배열을 전달해줌으로써 CSS 부분에 mediaQuery 함수를 불러와 mediaQuery 복붙을 없앨 수 있다.

#### [styles.ts 파일]

```javascript
import { css } from "@emotion/react";
import styled from "@emotion/styled";
import { mediaQuery } from "commons/styles/mediaQuery";

export const Wrapper = styled.div`
  padding: 0 1.25vw;

  ${mediaQuery.tablet`
    padding-right: 60px;
 `}

  ${mediaQuery.mobile`
    height: 40.3646vw;
    min-height: 225px;
    padding-right: 32px;
 `}
`;
```

mediaQuery.ts 파일에서 정의한 mediaQuery 함수를 import하여 각 breakPoint마다 정의함으로써 복붙 문제 해결!

> 추가적으로 이번에 공부하면서 새로 알게된 사실!
> airbnb css-in-javascript 네이밍 가이드에서는 특정 디바이스를 칭하는 이름 사용을 지양한다. 기기별 정해진 크기와 실제 기기의 크기가 다를 수 있기 때문이다. 따라서 아래와 같이 사이즈로 네이밍하도록 권장하고 있다.
>
> ```javascript
> const breakPoints = {
>   large: 1920,
>   medium: 1439,
>   small: 767,
> };
> ```

[ 참고 : <br />
https://velog.io/@ye-ji/React%EC%97%90%EC%84%9C-Media-Query-%EC%9E%91%EC%84%B1%ED%95%98%EB%8A%94-%ED%8E%B8-styled-componentsJS%EC%99%80-TS <br />
https://velog.io/@syoung125/CSS-%EB%B0%98%EC%9D%91%ED%98%95-%EC%9B%B9-%EB%94%94%EC%9E%90%EC%9D%B8-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0 ]
