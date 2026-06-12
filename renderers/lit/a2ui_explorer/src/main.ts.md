# renderers/lit/a2ui_explorer/src/main.ts

## 개요

`a2ui_explorer` 애플리케이션의 진입점(entry point) 파일이다. `local-gallery.js`를 side-effect import하여 `local-gallery` 커스텀 엘리먼트를 브라우저 Custom Elements 레지스트리에 등록하는 역할만 수행한다. 이 파일 자체에는 어떠한 로직도 없다.

## 의존성

### 저장소 내부 모듈
- [`./local-gallery.js`](./local-gallery.ts.md) — `local-gallery` 커스텀 엘리먼트 정의 (side-effect import; `.js` 확장자는 TypeScript에서 ES 모듈 해석을 위한 관례)

### 외부 패키지
없음.

## Exports

없음.

## 동작 흐름

번들러 또는 브라우저가 이 파일을 실행하면 `local-gallery.js`가 평가되고 그 안의 `@customElement('local-gallery')` 데코레이터가 실행되어 커스텀 엘리먼트가 등록된다. HTML 파일(`index.html`)에서 `<local-gallery>` 태그를 사용하기 전에 이 모듈이 로드되어야 한다.
