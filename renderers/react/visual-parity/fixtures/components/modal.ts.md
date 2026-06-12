# renderers/react/visual-parity/fixtures/components/modal.ts

## 개요

Modal 컴포넌트의 비주얼 패리티 테스트용 픽스처 모음이다. 단순 텍스트 콘텐츠를 가진 기본 모달과, Card 안에 복합 콘텐츠를 담은 모달 두 가지 시나리오를 제공한다. 각 픽스처는 트리거 버튼과 모달 콘텐츠를 별도 컴포넌트로 분리하여 `Modal`의 `entryPointChild`와 `contentChild`로 연결하는 구조를 사용한다.

## 의존성

### 저장소 내부 모듈

- [`../types`](../types.ts.md) — `ComponentFixture` 타입을 type-only import

### 외부 패키지

없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `modalBasic` | 상수 (`ComponentFixture`) | 단순 텍스트 콘텐츠를 갖는 기본 Modal 픽스처 |
| `modalWithCard` | 상수 (`ComponentFixture`) | Card 내 복합 콘텐츠를 갖는 Modal 픽스처 |
| `modalFixtures` | 상수 (객체) | 위 2개 픽스처를 묶은 그룹 객체 |

## 상세 명세

### `modalBasic: ComponentFixture`

- `root`: `'modal-1'`
- 컴포넌트 구성 (배열 순서):
  1. `id: 'trigger-text'` — Text(`'Open Modal'`)
  2. `id: 'trigger-btn'` — Button(`child: 'trigger-text'`, `action: {name: 'open'}`)
  3. `id: 'modal-content-text'` — Text(`'This is the modal content.'`)
  4. `id: 'modal-1'` — Modal(`entryPointChild: 'trigger-btn'`, `contentChild: 'modal-content-text'`)
- `primary` 플래그 없이 기본 버튼 스타일 사용

### `modalWithCard: ComponentFixture`

- `root`: `'modal-2'`
- 컴포넌트 구성 (배열 순서):
  1. `id: 'trigger-text-2'` — Text(`'Show Details'`)
  2. `id: 'trigger-btn-2'` — Button(`child: 'trigger-text-2'`, `action: {name: 'open'}`, `primary: true`)
  3. `id: 'modal-title'` — Text(`'Details'`, `usageHint: 'h2'`)
  4. `id: 'modal-body'` — Text(`'Here are the details you requested. This modal contains a card with multiple text elements.'`)
  5. `id: 'modal-content-col'` — Column(`children.explicitList: ['modal-title', 'modal-body']`)
  6. `id: 'modal-card'` — Card(`child: 'modal-content-col'`)
  7. `id: 'modal-2'` — Modal(`entryPointChild: 'trigger-btn-2'`, `contentChild: 'modal-card'`)
- `primary: true` 플래그로 강조 스타일 버튼 사용

### `modalFixtures: object`

`modalBasic`과 `modalWithCard`를 키-값으로 묶은 집계 객체.

## 동작 흐름

두 픽스처 모두 같은 구조 패턴을 따른다: 트리거 역할의 버튼 컴포넌트(내부 Text 포함)를 먼저 정의하고, 모달 콘텐츠 컴포넌트를 그 다음에 정의한 뒤, 최종 Modal 컴포넌트가 `entryPointChild`로 버튼을, `contentChild`로 콘텐츠를 참조한다. `data` 필드는 사용되지 않는다.
