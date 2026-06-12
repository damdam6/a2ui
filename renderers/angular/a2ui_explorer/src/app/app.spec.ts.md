# renderers/angular/a2ui_explorer/src/app/app.spec.ts

## 개요

`App` 루트 컴포넌트의 기본 렌더링을 검증하는 단위 테스트 파일이다. 컴포넌트 생성 가능 여부와 핵심 DOM 요소의 존재를 확인한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `TestBed`

### 저장소 내부 모듈
- [`./app`](./app.ts.md) — `App`
- [`src/v0_9/core/markdown`](../../../../../src/v0_9/core/markdown.ts) — `provideMarkdownRenderer`

## 테스트 케이스 명세

### describe: `'App'`

#### 픽스처 및 모킹

**beforeEach 설정** (비동기):
- `TestBed.configureTestingModule`에 `App`을 `imports`로, `provideMarkdownRenderer()`를 `providers`로 등록한다.
- `compileComponents()`를 await해 템플릿을 컴파일한다.

#### 테스트 케이스 1: `'should create the app'`
- **검증 동작**: `TestBed.createComponent(App)`으로 픽스처를 생성하고, `fixture.componentInstance`가 truthy임을 확인한다. `fixture.detectChanges()`를 호출해 `ngOnInit`을 트리거한 후, `fixture.nativeElement`에서 `.canvas-frame` CSS 클래스를 가진 요소가 존재하는지 확인한다.
- **픽스처**: `TestBed.createComponent(App)` 반환값.

#### 테스트 케이스 2: `'should render title'`
- **검증 동작**: `fixture.detectChanges()` 후, `fixture.nativeElement`에서 `h3` 요소의 `textContent`가 `'A2UI Examples'` 문자열을 포함하는지 확인한다.
- **픽스처**: `TestBed.createComponent(App)` 반환값.
