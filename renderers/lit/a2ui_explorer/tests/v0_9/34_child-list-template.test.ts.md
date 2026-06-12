# renderers/lit/a2ui_explorer/tests/v0_9/34_child-list-template.test.ts

## 개요

`34_child-list-template.json` 예제를 로드하여 ChildList Template Expansion 컴포넌트의 렌더링을 검증하는 통합 테스트 파일이다. 동적으로 생성된 아이템 목록(제목·수량)과 공통 레이블이 올바르게 렌더링되는지 세 개의 `it` 블록으로 확인한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent` 임포트
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스 임포트

## Exports

명시적 export 없음.

## 동작 흐름

1. `beforeEach`: `loadExample('34_child-list-template.json')` → `getSurface` → `getDeepTextContent`.
2. `afterEach`: `gallery.remove()`.

## 테스트 케이스

### `should render title`
- **검증 동작**: 텍스트에 컴포넌트 제목 `'Dynamic Item List'`가 포함되는지 확인한다.
- **픽스처**: `34_child-list-template.json`

### `should render list items`
- **검증 동작**: 텍스트에 아이템 이름 `'Apple'`·`'Banana'`·`'Cherry'`와 각각의 수량 `'10'`·`'5'`·`'20'`이 모두 포함되는지 확인한다.
- **픽스처**: 동일.

### `should render label`
- **검증 동작**: 수량 열 레이블 `'Qty'`가 텍스트에 포함되는지 확인한다.
- **픽스처**: 동일.
