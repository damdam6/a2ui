# renderers/angular/a2ui_explorer/src/app/tests/v0_9/35_markdown-text.spec.ts

## 개요

`Markdown Text Support` 예제의 브라우저 통합 테스트 파일이다. 마크다운 텍스트가 HTML 태그로 올바르게 변환되어 렌더링되는지 검증하며, 일반 텍스트 존재 여부와 실제 HTML 태그 생성 여부를 모두 확인한다. `textContent` 검사와 `innerHTML` 검사를 모두 수행하는 점이 특징이다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

이 파일은 테스트 스위트 정의만 포함하며, 외부로 내보내는 항목이 없다.

## 테스트 케이스 명세

### 픽스처 및 셋업

`describe('Example: Markdown Text Support')` 블록에서 `fixture: ComponentFixture<DemoComponent>`와 `textContent: string`을 공유 변수로 선언한다.

`beforeEach` — 각 테스트 전에 비동기로 실행된다.
1. `loadExample('Markdown Text Support')`를 호출하여 예제를 로드하고 fixture를 얻는다.
2. `getCanvas().textContent`를 `textContent`에 저장한다 (`wait()` 미사용).

### 테스트 케이스 1: `should render text content`

- **검증 대상**: 캔버스의 `textContent`에 `'Markdown Rendering'`이 포함되어야 한다.

### 테스트 케이스 2: `should render markdown HTML tags`

- **검증 대상**: `fixture.nativeElement.innerHTML`에 `'<h1>Heading 1</h1>'` HTML 문자열이 포함되어야 한다.
- **특이사항**: `textContent` 대신 `innerHTML`을 검사하여 마크다운(`# Heading 1` 구문)이 실제로 `<h1>` 요소로 파싱·렌더링되었음을 DOM 구조 수준에서 확인한다.

## 동작 흐름

`loadExample` → `getCanvas().textContent` 셋업 후, 텍스트 콘텐츠 존재 여부와 마크다운에서 변환된 HTML 태그(`<h1>`) 존재 여부를 순차적으로 검증한다. `innerHTML` 검사로 텍스트 수준이 아닌 DOM 구조 수준의 마크다운 파싱을 확인한다.
