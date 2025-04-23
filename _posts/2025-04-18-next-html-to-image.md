---
title: Next.js DOM 이미지로 변환하여 다운로드하기 (html-to-image)
description: Next.js에서 보여주는 DOM을 이미지로 다운로드 하겠습니다.
author: hyno
date: 2024-04-18 00:00:00 +/-TTTT
categories: [프론트엔드, Next.js | React]
tags: [nextjs, frontend, image, img, convert, dom, typescript, htmltoimage, html2canvas]
toc: true
comments: true
---

![hyno-profile](https://raw.githubusercontent.com/gusgh00/github_blog_img/refs/heads/main/hyno-profile.png){: width="200" height="200" .left}
스케줄표의 내용을 자율적으로 수정하고 테마를 꾸며주는 기능을 웹으로 제작하게 되었습니다.
처음에는 이 기능을 필요로 했던 사용자에게 빠르게 제공해주기 위해 바닐라js로 개발하였고 `html2canvas`를 사용하였습니다.
그러나 `html2canvas`에는 치명적이진 않지만 불편했던 단점들이 눈에 들어왔습니다. 몇 가지 css가 적용이 안되어 스타일 적용에 번거로움이 생긴 것입니다.
이를 일방적으로 해결하기 위해 모든 필터 등의 스타일을 지운 후 간단한 퍼블리싱만 해놓고 사용자에게 제공하였습니다.

이후 `html-to-image` 라는 다른 모듈을 찾았고 사용성을 높이기 위해 바닐라가 아닌 next.js로 프로젝트를 생성했습니다.

## 프로젝트 환경

> next 15.1.5, react 19.0.0, typescript
{: .prompt-info }

## npm 모듈 다운로드

```bash
npm i --save html-to-image
```

## 사용법

사용법은 매우 간단합니다. 이미지로 변환하고자 하는 DOM의 node를 불러오고 모듈로 변환한 다음 a 링크 다운로드를 실행하면 됩니다.

### 컴포넌트

```tsx
<div id="image-section">테스트</div>
<button onClick={() => saveConvertImage()}>테스트</button>
```

DOM을 불러오는 방법으로 useRef를 사용하는 방법도 있지만 여기서는 document로 불러오는 방법을 선택하도록 하겠습니다. getElementById로 불러올 id를 지정하여 줍니다.

### 함수

```tsx
import * as htmlToImage from 'html-to-image';

const saveConvertImage = () => {
    let node = document.getElementById('image-section')

    htmlToImage.toPng(node, {includeQueryParams: true})
        .then((dataUrl) => {
          const link = document.createElement('a')
          link.download = 'something.png'
          link.href = dataUrl
          link.click()
        })
  }
```

> Type error: Argument of type 'HTMLElement | null' is not assignable to parameter of type 'HTMLElement'.
Type 'null' is not assignable to type 'HTMLElement'.
{: .prompt-danger }

타입스크립트에서 주로 발생되는 타입 에러로 getElement로 불러오는 노드의 타입은 HTMLElement, null이 적용 가능하기 때문에 자체적으로 null이 아님을 선언해야합니다.

```tsx
let node = document.getElementById('image-section') as HTMLElement
```

또는

```tsx
let node = document.getElementById('image-section')
if (node !== null) {
		return false
}
```

## 전체 코드

```tsx
import * as htmlToImage from 'html-to-image';

export default function Home() {
		const saveConvertImage = () => {
		    let node = document.getElementById('image-section') as HTMLElement

		    htmlToImage.toPng(node, {includeQueryParams: true})
		        .then((dataUrl) => {
		          const link = document.createElement('a')
		          link.download = 'something.png'
		          link.href = dataUrl
		          link.click()
		        })
		}
		return (
				<div id="schedule-image">테스트</div>
				<button onClick={() => saveConvertImage()}>테스트</button>
		)
}
```
