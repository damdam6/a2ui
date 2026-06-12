# renderers/angular/a2ui_explorer/src/app/app.ts

## 개요

`App`은 A2UI Angular 탐색기 앱의 루트 컴포넌트다. `DemoComponent`를 직접 임포트하여 `<a2ui-v0-9-demo>` 태그 하나만 렌더링하는 얇은 쉘 역할을 한다. 모든 실질적인 레이아웃과 에이전트 렌더링 로직은 `DemoComponent` 내부에 위임된다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`

### 저장소 내부 모듈
- `./demo.component` — `DemoComponent` (이 파일은 문서화 대상 외이지만, 동일 디렉토리에 존재함)

## Exports

| 이름 | 종류 |
|---|---|
| `App` | 클래스 (Angular 컴포넌트) |

## 상세 명세

### 클래스 `App`

**데코레이터**: `@Component`
- `selector`: `'app-root'`
- `imports`: `[DemoComponent]`
- `template`: `'<a2ui-v0-9-demo></a2ui-v0-9-demo>'`

클래스 본문에는 필드나 메서드가 없다. 컴포넌트 자체는 순수한 컨테이너로서 `DemoComponent`의 셀렉터만 렌더링한다.

## 동작 흐름

`bootstrapApplication(App, appConfig)` 호출 시 이 컴포넌트가 DOM에 마운트되어 `<a2ui-v0-9-demo>` 태그를 통해 `DemoComponent`를 활성화한다.
