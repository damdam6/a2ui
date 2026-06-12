# renderers/angular/a2ui_explorer/src/app/tests/v0_8/1_simple_text.spec.ts

## 개요

v0.8 카탈로그의 "Simple Text (minimal)" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 파일이다. 캔버스 DOM에 기대하는 텍스트가 포함되어 있는지 단일 테스트 케이스로 확인한다. 외부 패키지나 `ComponentFixture`를 사용하지 않는 가장 단순한 형태의 렌더링 검증 테스트다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample` 임포트

## Exports

없음 (테스트 파일이므로 export 없음)

## 테스트 케이스 명세

### `describe`: `'Example: Simple Text (minimal) (v0.8)'`

#### 픽스처 / 설정
- `textContent: string` — 각 테스트 실행 전 캔버스 요소의 텍스트 내용을 담는 변수
- `beforeEach`: `loadExample('Simple Text (minimal)', Version.V0_8)`를 `await`하여 예제를 로드한 후, `getCanvas().textContent`를 `textContent`에 저장한다.

#### 테스트 케이스: `'should render expected text content'`
- **검증 동작**: 캔버스 텍스트에 `'Hello, Minimal Catalog!'` 문자열이 포함되어 있는지 확인한다.
- **사용 픽스처/모킹**: `beforeEach`에서 설정한 `textContent`, `loadExample`, `getCanvas`

## 동작 흐름

1. `beforeEach`에서 `loadExample`을 호출해 v0.8 카탈로그의 Simple Text 예제를 로드한다.
2. `getCanvas().textContent`로 렌더링된 캔버스 DOM 전체 텍스트를 추출한다.
3. 단일 `it` 블록에서 해당 텍스트에 `'Hello, Minimal Catalog!'`이 포함됨을 `toContain`으로 단언한다.
