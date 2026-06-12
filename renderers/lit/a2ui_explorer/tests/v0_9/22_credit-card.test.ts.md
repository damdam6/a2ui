# renderers/lit/a2ui_explorer/tests/v0_9/22_credit-card.test.ts

## 개요

`22_credit-card.json` 예제 파일을 로드하여 "Credit Card" 신용카드 UI의 정적 렌더링을 검증하는 브라우저 통합 테스트 파일이다. 카드 소유자 레이블, 만료일 레이블, 카드 브랜드, 마스킹된 카드 번호, 소유자 이름, 만료 날짜, 그리고 결제 아이콘을 확인한다. 버튼 상호작용은 없다.

## 의존성

### 외부 패키지
- `jasmine` (전역 테스트 프레임워크)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep` 유틸리티 함수
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스

## Exports

이 파일은 아무것도 export하지 않는다. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 동작 흐름

### 픽스처 및 상태 변수

- `gallery: LocalGallery` — 각 테스트마다 생성되고 `afterEach`에서 제거된다.
- `surface: HTMLElement` — 렌더링 표면 요소.
- `textContent: string` — Shadow DOM을 포함한 전체 텍스트 콘텐츠.

### beforeEach

`loadExample('22_credit-card.json')`을 비동기로 호출하여 `gallery`를 초기화한 후 `surface`와 `textContent`를 준비한다.

### afterEach

`gallery?.remove()`로 DOM을 정리한다.

## 테스트 케이스

### `should render text content`

- **검증 동작**: `textContent`에 다음 문자열들이 포함되어 있는지 확인한다: 카드 소유자 레이블 `'CARD HOLDER'`, 만료일 레이블 `'EXPIRES'`, 카드 브랜드 `'VISA'`, 마스킹된 카드 번호 `'•••• •••• •••• 4242'`, 소유자 이름 `'SARAH JOHNSON'`, 만료 날짜 `'09/27'`.
- **픽스처/모킹**: `beforeEach`에서 로드된 `22_credit-card.json` 예제 데이터.

### `should render icon`

- **검증 동작**: `querySelectorAllDeep(surface, 'a2ui-icon')[0]`이 truthy인지 확인하고, `textContent`에 아이콘 이름 `'payment'`가 포함되어 있는지 검증한다.
- **픽스처/모킹**: `querySelectorAllDeep` 유틸리티.
