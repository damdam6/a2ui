# renderers/angular/a2ui_explorer/src/app/app.config.ts

## 개요

Angular 애플리케이션의 최상위 설정 객체(`ApplicationConfig`)를 정의한다. 브라우저 전역 에러 리스너와 마크다운 렌더러를 루트 provider로 등록하며, `main.ts`에서 `bootstrapApplication`에 전달된다.

## 의존성

### 외부 패키지
- `@angular/core` — `ApplicationConfig`, `provideBrowserGlobalErrorListeners`

### 저장소 내부 모듈
- [`src/v0_9/core/markdown`](../../../../../src/v0_9/core/markdown.ts) — `provideMarkdownRenderer`

## Exports

| 이름 | 종류 |
|---|---|
| `appConfig` | 상수 (`ApplicationConfig`) |

## 상세 명세

### 상수 `appConfig: ApplicationConfig`

`providers` 배열에 두 개의 provider를 포함한다:

1. `provideBrowserGlobalErrorListeners()` — Angular 내장 함수. 브라우저의 `window.onerror` 및 `unhandledrejection` 이벤트를 Angular 에러 핸들러로 연결한다.
2. `provideMarkdownRenderer()` — 저장소 내부 함수. v0.9 마크다운 렌더링에 필요한 서비스들을 등록한다.

## 동작 흐름

이 파일은 순수 설정 상수만 내보낸다. `bootstrapApplication(App, appConfig)` 호출 시 `providers` 배열이 Angular DI 컨테이너에 등록된다.
