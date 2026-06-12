# renderers/react/visual-parity/fixtures/components/button.ts

## 개요

`Button` 컴포넌트의 시각적 동등성 테스트를 위한 픽스처 정의 파일이다. primary 버튼, secondary 버튼, 아이콘+텍스트를 포함한 복합 버튼 세 가지 케이스를 제공한다. Button은 항상 `child`로 자식 컴포넌트 ID를 받으므로 각 픽스처는 Text 또는 Row 등의 자식 컴포넌트를 함께 정의한다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `ComponentFixture` (타입 import)

## Exports

- `buttonPrimary`: `ComponentFixture` — `primary: true` 버튼 픽스처
- `buttonSecondary`: `ComponentFixture` — `primary: false` 버튼 픽스처
- `buttonWithIcon`: `ComponentFixture` — 아이콘과 텍스트를 Row로 묶은 복합 버튼 픽스처
- `buttonFixtures`: `{ buttonPrimary, buttonSecondary, buttonWithIcon }` — 컬렉션 객체

## 상세 명세

### `buttonPrimary: ComponentFixture`

- `root`: `'btn-1'`
- `components`:
  1. `id: 'btn-text'` — `Text` 컴포넌트, `text: {literalString: 'Primary Button'}`
  2. `id: 'btn-1'` — `Button` 컴포넌트, `child: 'btn-text'`, `action: {name: 'click'}`, `primary: true`

### `buttonSecondary: ComponentFixture`

- `root`: `'btn-2'`
- `components`:
  1. `id: 'btn-text-2'` — `Text` 컴포넌트, `text: {literalString: 'Secondary Button'}`
  2. `id: 'btn-2'` — `Button` 컴포넌트, `child: 'btn-text-2'`, `action: {name: 'click'}`, `primary: false`

### `buttonWithIcon: ComponentFixture`

- `root`: `'btn-icon'`
- `components` (4개):
  1. `id: 'btn-icon-icon'` — `Icon` 컴포넌트, `name: {literalString: 'add'}`
  2. `id: 'btn-icon-text'` — `Text` 컴포넌트, `text: {literalString: 'Add Item'}`
  3. `id: 'btn-icon-row'` — `Row` 컴포넌트, `children: {explicitList: ['btn-icon-icon', 'btn-icon-text']}`
  4. `id: 'btn-icon'` — `Button` 컴포넌트, `child: 'btn-icon-row'`, `action: {name: 'add'}`, `primary: true`

### `buttonFixtures`

세 픽스처를 하나의 객체로 묶어 내보내는 네임스페이스 상수.

## 동작 흐름

각 픽스처는 `root` ID에서 시작하여 `child` 참조로 연결된 컴포넌트 트리를 구성한다. `buttonWithIcon`은 가장 복잡한 형태로, Button → Row → [Icon, Text] 3단계 계층을 가진다. 시각적 parity 테스트 러너는 이 트리를 React와 Lit 각각으로 렌더링하여 스크린샷 또는 DOM 구조를 비교한다.
