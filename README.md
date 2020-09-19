## 웹 최적화 왜 할까?

1. **사용자가 떠나지 않도록 하기 위해 = 수익 증대를 위해**

   페이지 로딩 시간이 길어질 수록 사용는 서비스를 떠나기 마련, 반대로 성능이 좋아져서 머무는 시간이 늘어날 수록 그만큼 서비스에서 창출할 수 있는 수익이 증가

2. **프론트엔드 개발자로서 경쟁력 갖추기 위해**

---

## 배울것 들

- 브라우저의 렌더링 원리
- Performance 패널을 이용한 분석
- Lighthouse 패널을 이용한 분석
- Network 패널을 이용한 분석
- webpack-bundle-analyzer를 이용한 분석
- 텍스트 압축 \[loading\]
- 이미지 사이즈 최적화 \[loading\]
- 이미지 CDN을 통한 최적화 \[loading\]
- 리소스 캐싱
- 이미지 preload \[loading\]
- 컴포넌트 preload \[loading\]
- Component Lazy Load \[loading\]
- React Code Splitting \[loading\]
- Image Lazy Load
- Bottleneck(병목) 코드 제거 \[rendering\]
- 애니메이션 최적화(repaint, reflow 줄이기) \[rendering\]

---

## 웹 성능 결정 요소

### 로딩 성능

각 Resource를 불러오는 성능

### 렌더링 성능

불러온 Resource들을 화면에 보여주는 성능

---

## 분석 툴

- 크롬 Network 탭
- 크롬 Performance 탭
- 크롬 Audit 탭(Light house)
- webpack-bundle-analyzer

---

### CDN (Contents Delivery Network)

물리적 거리의 한계를 극복하기 위해 소비자(사용자)와 가까운 곳에 컨텐츠 서버를 두는 기술

### Image CDN (Image Processing CDN)

기본적인 CDN 개념과 이미지를 사용자에게 보내기전에 특정 형태로 가공하여(사이즈를 줄이거나, 포맷을 바꾸거나..) 전달한다.

> 이미지 사이지가 적절하지 않아, 이미지를 불러올때 적절한 사이즈로 불러올 수 있도록 개선할 수 있다.

---

## 크롬 Performance 탭

웹 페이지의 런타임 성능을 분석하는 툴

> 해당 프로젝트에서 List를 불러올 때 퍼포먼스에 영향을 주는 부분에 대해 개선

#### Minor GC

가비지 컬렉션의 작업으로 메모리의 여유가 없을 때 메모리를 정리해주는 역할

---

## Code Splitting

덩치가 큰 번들 파일을 쪼개서 작은 파일로 만드는 것

### Pattern

- 페이지 별로
- 모듈 별로

불필요한 코드 또는 중복되는 코드가 없이 적절한 사이즈의 코드가 적절한 타이밍에 로드될 수 있도록 하는것

[React 공식페이지 Code Spplitting](https://ko.reactjs.org/docs/code-splitting.html)

## React Suspense ,Lazy Loading

```javascript
// App.js
import React { Suspense, lazy } from "react";
import { Switch, Route } from "react-router-dom";

const ListPage = lazy(() => import("./pages/ListPage");
const ViewPage = lazy(() => import("./pages/ViewPage");

function App() {
  return (
    <div className="App">
      <Suspense fallback={<div>loading..</div>}>
        <Switch>
          {/* Routes */}
        </Switch>
      </Suspense>
    </div>
  )
}
```

> 번들 사이즈를 줄이기 위해 buddle analyzer를 통해 분석 후 해당 페이지에서 사용되지 않는 모듈을 불러올 필요가 없기에 Suspence와 lazy를 통해서 필요한 모듈만을 불러올 수 있도록 개선

---

## 텍스트 압축

server에서 보내는 resource를 압축하여 service하는 것

일반적으로 웹에서는 대게 `GZIP`, `Deflate(LZ77)`의 형태로 사용한다.

server 단에서 압축을 하여 client에 전달하고 이를 client에서 다시 압축 해체하여 사용하는데, 무분별하게 모든 파일을 압축하여 전달하게 되면 되려 성능이 떨어질 수 있다.

이를 방지하기 위해서는 2kB 이상만을 압축하고 미만은 압축하지 않도록 한다.

---

## 애니메이션 최적화(Reflow, Repaint)

### 쟁크 현상 (애니메이션이 버벅이는 현상)

모니터 / 웹 브라우져는 초당 60 Frame(60FPS)인 반면, `쟁크 발생` 발생하여 초당 30 Frame / 20 Frame 등의 버벅이는 현상이 일어난다.

### 브라우저 렌더링 과정

> DOM + CSSOM > Render Tree > Layout > Paint > Composite

- DOM + CSSOM : HTML, CSS 가공
- Render Tree : 위 HTML, CSS를 조합
- Layout : 위치, 크기 등 계산 (Reflow 발생)
- Paint : 색 채워 넣기 (Repaint 발생)
- Composite : 각 레이어 합성하여 최종적 화면 그리는 과정

이를 통틀어 `Critical Rendering Path` 또는 `Pixel Popeline`이라 칭한다.

> `transform`과 `opacity` 속성을 사용하여 Reflow, Repaint 과정을 생략할 수 있다.

---

## 컴포넌트 Preloading

```javascript
// Factory Pattern
function LazyWithPreload(importFunction) {
  const Component = React.lazy(importFunction);
  Component.preload = importFunction;
  return Component;
}

const LazyImageModal = LazyWithPreload(() => import("./components/ImageModal"));

function App() {
  const [showModal, setShowModal] = useState(false);

  /* Component Preload */
  useEffect(() => {
    LazyImageModal.preload();
  }, []);

  // ...
}
```

### 컴포넌트 Preloading 타이밍

모달에서의 이미지 Preloading의 타이밍으로

1. 버튼 위에 마우스를 올려 놨을 때
2. 최초 페이지 로드가 되고, 모든 컴포넌트의 마운트가 끝났을 때

---

## 이미지 Preloading

```javascript
// App.js

function App() {
  useEffect(() => {
    const img = new Image();
    img.src =
      "https://stillmed.olympic.org/media/Photos/2016/08/20/part-1/20-08-2016-Football-Men-01.jpg?interpolation=lanczos-none&resize=*:800";
  }, []);
}
```

위 코드의 의미는 App 컴포넌트가 마운트가 모두 되었을때 해당 모달의 첫 페이지 이미지를 Object 데이터 형태로 만들어 두어 미리 caching 해두는 방법이다.

### 이미지 Lazy Loading의 단점

모달(이미지 갤러리)과 같은 경우 Click시에 모달 컴포넌트를 로드해와야 하기에 이미지가 뜨기까지 딜레이가 생긴다.

이 부분을 보완하기 위해 사용하는것이 `이미지 Preloading` 이다.

---

[CSS Triggers](https://csstriggers.com/)

[CRA에서 eject 없이 webpack-bundle-analyzer 사용하기](https://velog.io/@jesop/CRA%EC%97%90%EC%84%9C-eject-%EC%97%86%EC%9D%B4-webpack-bundle-analyzer-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)

[리엑트 웹 성능 최적화](https://www.inflearn.com/course/%EC%9B%B9-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94-%EB%A6%AC%EC%95%A1%ED%8A%B8-1)
