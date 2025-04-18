---
title: Next.js gif.js로 gif 이미지 만들기 (gif.js.optimized)
description: Next.js에서 gif.js (gif.js.optimized)로 gif 이미지를 만들어 보겠습니다.
author: hyno
date: 2024-04-19 00:00:00 +/-TTTT
categories: [프론트엔드, Next.js | React]
tags: [nextjs, frontend, image, img, convert, typescript, gifjs, gif, htmltoimage]
toc: true
comments: true
---

입력 폼 데이터와 이미지 데이터가 반응형으로 변경되는 DOM을 순차적으로 이어서 gif로 만들어야할 일이 있었습니다.

이전에 사용했던 `html-to-image`모듈에 이어서 `gif.js`를 사용하여 gif를 생성하도록 하겠습니다.

## 프로젝트 환경

> next 15.1.5, react 19.0.0, typescript
{: .prompt-info }

## npm 모듈 다운로드

```bash
npm i --save html-to-image gif.js.optimized
```

`gif.js`와 `gif.js.optimized`의 사용법은 똑같으며 optimized가 최적화 된 모듈입니다.

<https://www.npmjs.com/package/gif.js.optimized>

## gif worker 추가

브라우저의 웹워커가 돌아가도록 도와주는 모듈 내의 코드가 실행 되지 않는 버그가 존재하고 있습니다.

동적으로 실행 될 수 있도록 모듈의 worker 코드를 복사하여 넣어주었습니다.

> `node_modules > gif.js.optimized > dist > gif.worker.js`
파일 내 코드를 따옴표 안에 넣어주면 됩니다.
{: .prompt-warning }

```tsx
const workerStr = `!function(t){function e(r){if(i[r])return i[r].exports;
...중략
//# sourceMappingURL=gif.worker.js.map`;

export default workerStr;
```
{: file='@js/gifWorker.js'}

## 사용법

1. html-to-image를 통해 DOM을 이미지로 변환 (Promise로 진행)
2. 이미지를 Array에 push
3. new GIF 생성 후 이미지 프레임 추가
4. base64 형태의 gif 이미지 생성 완료

### 컴포넌트

```tsx
<div id="first-section">
    <Image id="first-image" src={image} alt="이미지"/>
</div>
<div id="second-section">
    <Image id="second-image" src={image} alt="이미지"/>
</div>
<button onClick={() => renderGifImage()}>생성</button>

<img src={resultGif} alt="결과 gif"/>
```

DOM을 불러오는 방법으로 useRef를 사용하는 방법도 있지만 여기서는 `document`로 불러오는 방법을 선택하도록 하겠습니다. `getElementById`로 불러올 id를 지정하여 줍니다.
`first-image`와 `second-image`는 이미지가 넘어갈 때 페이드 효과를 주기 위함으로 id 지정했습니다.

생성된 이미지는 결과 gif에 나타납니다.

### 함수

```tsx
let nodeFirst = document.getElementById('first-section');
let nodeSecond = document.getElementById('second-section');
let nodeFirstImg = document.getElementById('first-image');
let nodeSecondImg = document.getElementById('second-image');

let imgArr: string[] = []
let firstUrl = ""
let secondUrl = ""
for (let i = 0; i <= {staticCount}; i++) {
    firstUrl = await toPng(nodeFirst, {width: 420, height: 420})
    imgArr.push(firstUrl)
}
for (let i = {fadeCount}; i >= 0; i--) {
    nodeFirstImg.style.opacity = (i * 0.33).toString()
    firstUrl = await toPng(nodeFirst, {width: 420, height: 420})
    imgArr.push(firstUrl)
}
for (let i = 0; i <= {fadeCount}; i++) {
    nodeSecondImg.style.opacity = (i * 0.33).toString()
    secondUrl = await toPng(nodeSecond, {width: 420, height: 420})
    imgArr.push(secondUrl)
}
for (let i = 0; i <= {staticCount}; i++) {
    secondUrl = await toPng(nodeSecond, {width: 420, height: 420})
    imgArr.push(secondUrl)
}

nodeFirstImg.style.opacity = "1"
nodeSecondImg.style.opacity = "1"
```

DOM 두개를 기준으로 반복을 4번 실행합니다.

`staticCount`만큼 정적인 이미지, `fadeCount`만큼 동적인 이미지로 반복문 조건을 만들어줍니다.

DOM내 이미지의 `opacity`값을 변화시켜주면서 fade in, fade out 효과를 준 것 같은 장면 전환 효과를 만들어주고 이미지 배열에 추가해줍니다.

```tsx
const workerBlob = new Blob([workerStr], {
    type: "application/javascript"
});

const gif = new GIF({
    workers: 2,
    workerScript: URL.createObjectURL(workerBlob),
    quality: 30,
    width: 420,
    height: 420,
})
```

따로 저장해 둔 gif worker 파일을 임포트 후 blob 형태로 지정해 줍니다. new GIF에 알맞은 인자를 넣어줍니다.

```tsx
imgArr.forEach((imgUrl) => {
    const img = new Image()
    img.src = imgUrl as string
    gif.addFrame(img, { delay: 100 })
})

console.log('프레임 추가 완료');
```

이미지 배열을 `forEach`로 반복문으로 돌려주면서 base64 형태의 url을 Image 타입으로 변환 후 `gif.addFrame`으로 이미지 넣어줍니다.

```tsx
gif.on('finished', function(blob: Blob) {
    const url = URL.createObjectURL(blob)
    let resultGif = new Image()
    resultGif.src = url
    setResultGif(url)
    console.log('GIF 생성 완료');
})

gif.on('progress', (progress: number) => {
    console.log('GIF 생성 진행중 : ' + progress); //0.1 ~ 1.0
});

setTimeout(() => {
    console.log('GIF 렌더링 시작');
    gif.render()
}, 5000)
```

`gif.render()`가 실행 되면서 `gif.on(’progress’)`의 이벤트 리스너가 실행되고 렌더링이 종료가 되면 `gif.on(’finished’)` 이벤트 리스너가 실행됩니다.

`setTimeout`을 걸어준 이유는 각 이미지가 정상적으로 로드가 되는 시간을 벌이기 위함입니다.

`gif.on(’progress’)`의 `progress`는 종료되기 까지의 진행도를 나타내며 0.1 부터 1까지 나타냅니다.

`gif.on(’finished’)`에서 blob을 url의 형태로 바꿔준 후 이미지 타입으로 변환해줍니다.

> Error occurred prerendering page "/". Read more: https://nextjs.org/docs/messages/prerender-error
ReferenceError: navigator is not defined
…
> 
> 
> Export encountered an error on /page: /, exiting the build.
> ⨯ Static worker exited with code: 1 and signal: null
{: .prompt-danger }

그냥 app router 내 에서 빌드하게 되면 다음과 같은 에러가 발생합니다.

gif.js가 클라이언트에서만 읽혀야 하기 때문에 컴포넌트 분리 후 dynamic으로 ssr을 꺼놓은 상태로 빌드시켜주면 됩니다.

```tsx
const RenderGif = dynamic(() =>
		import('@components/ssr/RenderGif') as Promise<{ default: React.ComponentType }>,
		{ ssr: false })
```

## 전체 코드

```tsx
import dynamic from "next/dynamic";
const RenderGif = dynamic(() =>
		import('@components/ssr/RenderGif') as Promise<{ default: React.ComponentType }>,
		{ ssr: false })

export default function Home() {
		return (
				<RenderGif/>
		)
}
```

```tsx
import {toPng} from "html-to-image"
import GIF from "gif.js.optimized/dist/gif"
import workerStr from "@js/gifWorker";

const RenderGif = () => {
		const [resultGif, setResultGif] = useState<string>("");
		const renderGifImage = async () => {
        let nodeFirst = document.getElementById('first-section');
        let nodeSecond = document.getElementById('second-section');
        let nodeFirstImg = document.getElementById('first-image');
        let nodeSecondImg = document.getElementById('second-image');

        let imgArr: string[] = []
        let firstUrl = ""
        let secondUrl = ""
        
        for (let i = 0; i <= 5; i++) {
            firstUrl = await toPng(nodeFirst, { width: 420, height: 420 })
            imgArr.push(firstUrl)
        }
        for (let i = 3; i >= 0; i--) {
            nodeFirstImg.style.opacity = (i * 0.33).toString()
            firstUrl = await toPng(nodeFirst, { width: 420, height: 420 })
            imgArr.push(firstUrl)
        }
        for (let i = 0; i <= 3; i++) {
            nodeSecondImg.style.opacity = (i * 0.33).toString()
            secondUrl = await toPng(nodeSecond, { width: 420, height: 420 })
            imgArr.push(secondUrl)
        }
        for (let i = 0; i <= 5; i++) {
            secondUrl = await toPng(nodeSecond, { width: 420, height: 420 })
            imgArr.push(secondUrl)
        }

        nodeFirstImg.style.opacity = "1"
        nodeSecondImg.style.opacity = "1"

        const workerBlob = new Blob([workerStr], {
            type: "application/javascript"
        });

        const gif = new GIF({
            workers: 2,
            workerScript: URL.createObjectURL(workerBlob),
            quality: 30,
            width: 420,
            height: 420,
        })

        imgArr.forEach((imgUrl) => {
            const img = new Image()
            img.src = imgUrl as string
            gif.addFrame(img, { delay: 100 })
        })

        console.log('프레임 추가 완료');

        gif.on('finished', function(blob: Blob) {
            const url = URL.createObjectURL(blob)

            console.log(blob.size / (1024 * 1024))

            let resultGif = new Image()
            resultGif.src = url
            setResultGif(url)
            console.log('GIF 생성 완료');
        })

        gif.on('progress', (progress: number) => {
            console.log('GIF 생성 진행중 : ' + progress); //0.1 ~ 1.0
        });

        setTimeout(() => {
            console.log('GIF 렌더링 시작');
            gif.render()
        }, 5000)
		}
		
		return (
				<div id="first-section">
				    <Image id="first-image" src={image} alt="이미지"/>
				</div>
				<div id="second-section">
				    <Image id="second-image" src={image} alt="이미지"/>
				</div>
				<button onClick={() => renderGifImage()}>생성</button>

				<img src={resultGif} alt="결과 gif"/>
		)
}

export default RenderGif
```

## 참고자료

<https://mirrous9.medium.com/html-태그를-모아-gif-만들기-a0f63fffbc80>
