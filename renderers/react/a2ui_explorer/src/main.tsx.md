# renderers/react/a2ui_explorer/src/main.tsx

## 개요

브라우저 진입점(entry point) 파일이다. React 18의 `createRoot` API를 사용하여 `<App>` 컴포넌트를 `<div id="root">` DOM 노드에 마운트한다. 전역 CSS와 StrictMode를 적용한다.

## 의존성

### 외부 패키지
- `react` — `StrictMode`
- `react-dom/client` — `createRoot`

### 저장소 내부 모듈
- `./index.css` — 전역 스타일 시트
- [`./App.tsx`](./App.tsx.md) — `App` 컴포넌트

## Exports

없음 (진입점 파일)

## 동작 흐름

`document.getElementById('root')!`로 루트 DOM 요소를 가져온 뒤 `createRoot`로 React 루트를 생성하고, `<StrictMode><App /></StrictMode>`를 렌더링한다. 이 파일은 HTML의 `<script type="module">` 태그를 통해 Vite가 직접 로드한다.
