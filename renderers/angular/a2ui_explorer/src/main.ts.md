# renderers/angular/a2ui_explorer/src/main.ts

## 개요

Angular 애플리케이션의 진입점 파일이다. `bootstrapApplication`을 사용하여 독립형(standalone) `App` 컴포넌트를 `appConfig` 설정과 함께 부트스트랩한다. 부트스트랩 실패 시 `console.error`로 에러를 출력한다. 런타임 로직 없이 단일 부트스트랩 호출만 포함한다.

## 의존성

### 외부 패키지
- `@angular/platform-browser` — `bootstrapApplication`

### 저장소 내부 모듈
- [`./app/app.config`](./app/app.config.ts.md) — `appConfig`
- `./app/app` — `App` (루트 컴포넌트)

## Exports

이 파일은 아무것도 export하지 않는다. 애플리케이션 부트스트랩 사이드이펙트만 실행한다.

## 동작 흐름

파일이 로드되는 즉시 `bootstrapApplication(App, appConfig)`를 호출한다. 이 호출은 `App` 컴포넌트를 DOM에 마운트하고 Angular DI 컨테이너와 변경 감지 메커니즘을 초기화한다. Promise가 거부되면 `.catch(err => console.error(err))`로 에러를 콘솔에 기록하며, 별도의 복구 로직은 없다.
