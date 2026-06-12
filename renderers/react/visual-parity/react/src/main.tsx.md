# renderers/react/visual-parity/react/src/main.tsx

## 개요

비주얼 패리티 React 앱의 진입점(entry point) 파일이다. A2UI 컴포넌트 카탈로그 초기화, 스타일 주입, URL에서 테마 파싱을 순서대로 수행한 후 `ReactDOM.createRoot`로 `React.StrictMode` 내에 `A2UIProvider`와 `FixturePage`를 마운트한다.

## 의존성

### 외부 패키지
- `react` — `React`
- `react-dom/client` — `ReactDOM`
- `@a2ui/react` — `A2UIProvider`, `initializeDefaultCatalog`
- `@a2ui/react/styles` — `injectStyles`

### 저장소 내부 모듈
- [`./FixturePage`](./FixturePage.tsx.md) — `FixturePage`
- [`../../fixtures/themes`](../../fixtures/themes/index.ts.md) — `getTheme`, `themeNames`

## Exports

없음 (모듈 레벨 사이드 이펙트만 수행하는 진입점 파일)

## 상세 명세

### 모듈 레벨 실행 순서

1. **`initializeDefaultCatalog()`** 호출 — A2UI 기본 컴포넌트 카탈로그를 등록한다.
2. **`injectStyles()`** 호출 — `litTheme` 유틸리티 클래스 등 A2UI 구조적 CSS를 `<head>`에 주입한다.
3. **URL 파싱** — `new URLSearchParams(window.location.search)`로 `theme` 파라미터를 읽어 `themeName`(string | null)에 저장한다. `getTheme(themeName)`을 호출하여 `selectedTheme` 객체를 얻는다.
4. **디버깅 속성 설정** — `themeName`이 truthy이면 `document.documentElement.setAttribute('data-theme', themeName)`으로 루트 요소에 현재 테마 이름을 기록한다.
5. **콘솔 로그** — `[Visual Parity - React] Available themes:` 접두사로 `themeNames` 배열을, `[Visual Parity - React] Selected theme:` 접두사로 `themeName || 'default'`를 출력한다.
6. **ReactDOM 마운트** — `document.getElementById('root')!`를 루트로 삼아 다음 트리를 렌더링한다:
   ```
   <React.StrictMode>
     <A2UIProvider theme={selectedTheme}>
       <FixturePage />
     </A2UIProvider>
   </React.StrictMode>
   ```

## 동작 흐름

앱 시작 시 한 번 실행된다. 카탈로그 초기화 → 스타일 주입 → 테마 결정 → DOM 속성 설정 → React 트리 마운트 순서로 진행되며, 이후 제어권은 React 렌더 사이클로 넘어간다.
