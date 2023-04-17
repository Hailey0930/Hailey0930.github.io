---
layout: post
title: "[CSS] CSS breakpoint refactoring"
summary: "breakpoint 복붙 없애기"
author: hyerin
date: "2023-04-17 19:20:00 +0900"
category: CSS
thumbnail: /assets/img/posts/230417_thumbnail.jpg
keywords: CSS, breakpoint, refactoring
permalink:
usemathjax: true
---

> 회사에서 web Player 작업을 하면서 breakpoint 복붙에 진절머리가 나기 시작했었다. 새롭게 사이드 프로젝트를 시작하면서 CSS 작업을 시작하니 역시나 또 breakpoint 구문 복붙 지옥의 시작... '이렇게 복붙을 할 수 밖에 없는건가'란 생각을 계속 하다가, 프로젝트 시작 초반에 새롭게 잡고 가는게 맞는 것 같아 관련 내용을 찾아보고 바로 refactoring 해봤다.

# legacy code

진절머리가 났던 breakpoint 복붙 구문을 가져와보자
palette 파일에 breakpoint를 따로 빼서 관리를 한다고 해도, media query 구문이 있는 부분을 굳이 찾아 가서 복사해 온 후 필요한 구문에 붙여넣기를 했었다.

#### [styles.ts 파일]

```javascript
import styled from "@emotion/styled";
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

# Emotion 공부하기

구글링을 하다가 나와 같은 고민을 한 글을 발견했고, 그 블로그를 참고했다.
(기록을 안해둬서 출처가 없다...)

'media query 부분을 따로 컴포넌트처럼 분리를 해서 props로 넘기면 공통 모듈로 사용할 수 있지 않을까?' 란 막연한 생각을 했었는데, 역시 Emotion이 props로 넘긴 부분을 받아 CSS로 적용할 수 있도록 제공하는 기능이 있었다.

**[Emotion Composition](https://emotion.sh/docs/composition)** 이다!

## Emotion Composition

Emotion Composition은 두 개 이상의 스타일 객체를 결합하여 새로운 객체를 생성하는 것을 지원하는 패턴으로, composition을 사용해서 CSS 스타일을 넘겨주면 media query로 감싼 스타일을 내보내는 공통 모듈 역할을 하는 함수를 만들 수 있다.

## media query 공통 함수 만들기

#### [mediaQuery.ts 파일]

```javascript
import { SerializedStyles, css } from "@emotion/react";
import { breakPoints } from "./palette";

export const setTabletStyle = (style: SerializedStyles) => css`
  @media screen and (max-width: ${breakPoints.tablet}px) {
    ${style}
  }
`;

export const setMobileStyle = (style: SerializedStyles) => css`
  @media screen and (max-width: ${breakPoints.mobile}px) {
    ${style}
  }
`;
```

Emotion composition을 활용하여 style이라는 props를 넘겨줌으로써 media query가 적용된 CSS를 해당 JSX에 적용할 수 있다.

#### [styles.ts 파일]

```javascript
import { css } from "@emotion/react";
import styled from "@emotion/styled";
import { setMobileStyle, setTabletStyle } from "commons/styles/mediaQuery";

export const Wrapper = styled.div`
  padding: 0 1.25vw;

  ${setTabletStyle(css`
    padding: 0 1.668vw;
  `)}

  ${setMobileStyle(css`
    padding: 0 4.444vw;
  `)}
`;
```

mediaQuery.ts 파일에서 정의한 media query 구문을 setTabletStyle, setMobileStyle와 css를 import 함으로써 복붙 문제 해결!

media query를 감싸고 있는 함수 이름으로 불러오면 되기 때문에 더 이상 breakpoint 부분을 굳이 찾지 않아도 된다!
