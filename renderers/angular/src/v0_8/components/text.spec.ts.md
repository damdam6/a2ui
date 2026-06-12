# renderers/angular/src/v0_8/components/text.spec.ts

## 개요

`Text` Angular 컴포넌트의 단위 테스트 파일이다. `MarkdownRenderer`를 비동기 스파이로 모의하여 마크다운 렌더링 경로를 제어하고, `usageHint`에 따른 마크다운 전처리, CSS 클래스 적용, 조건부 렌더링 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/platform-browser` — `By`

### 저장소 내부 모듈
- [`./text`](./text.ts.md) — 테스트 대상 컴포넌트 `Text`
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor` (mock, 빈 객체)
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme` (mock)
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog` (mock, 빈 객체)
- [`../data/markdown`](../data/markdown.ts.md) — `MarkdownRenderer` (spy mock)

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 및 모킹 설정 (`beforeEach`)

- `mockTheme.components.Text`를 다음 구조로 설정:
  - `all: {'base-all': true}`
  - `h1: {'style-h1': true}`
  - `h2: {'style-h2': true}`
  - `body: {'style-body': true}`
  - `caption: {'style-caption': true}`
- `mockMarkdownRenderer`: `jasmine.createSpyObj('MarkdownRenderer', ['render'])`. `render`는 `Promise.resolve('<div class="rendered">${markdown}</div>')` 반환.
- 컴포넌트 입력값: `surfaceId='surface-1'`, `component={id: 'text-1', type: 'Text', weight: 1}`, `weight=1`, `text={literalString: 'Hello World'}`, `usageHint='body'`.

---

### `should create`

컴포넌트 인스턴스가 생성되는지 확인한다.

---

### `should render content from MarkdownRenderer`

`fixture.whenStable()` 대기 후 `detectChanges()` 재호출. `By.css('section')` 조회 후:
- `innerHTML`에 `class="rendered"`가 포함되는지 확인.
- `textContent`에 `'Hello World'`가 포함되는지 확인.

---

### `should format text based on usageHint BEFORE calling MarkdownRenderer`

`usageHint`를 `'h1'`으로 변경하고 안정화 대기. `mockMarkdownRenderer.render`가 `'# Hello World'`를 첫 번째 인자로 받아 호출되었는지 검증. 이어서 `usageHint='caption'`으로 변경 후, `'*Hello World*'`로 호출되었는지 확인.

---

### `should apply correct classes based on usageHint`

- `section` 요소의 `className`에 `'base-all'`과 `'style-body'`가 포함되는지 검증 (초기 `usageHint='body'`).
- `usageHint='h1'`으로 변경 후 `'style-h1'`은 포함, `'style-body'`는 제외되는지 검증.

## 동작 흐름

`beforeEach`에서 컴포넌트를 마운트하고, 각 테스트는 `usageHint` 입력을 변경하거나 마크다운 렌더러의 호출 인자를 검사하여 `Text` 컴포넌트의 반응성과 렌더링 정확성을 확인한다. `async/await`와 `fixture.whenStable()`을 결합해 비동기 마크다운 렌더링 파이프라인을 동기적으로 검증한다.
