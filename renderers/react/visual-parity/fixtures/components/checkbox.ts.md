# renderers/react/visual-parity/fixtures/components/checkbox.ts

## 개요

`CheckBox` 컴포넌트의 시각적 동등성 테스트를 위한 픽스처 정의 파일이다. 미체크 상태, 체크 상태, 긴 레이블 세 가지 케이스를 제공한다. 파일 주석에 따르면 A2UI CheckBox는 반드시 `path` 바인딩을 통해 `value`를 지정해야 하며 `literalBoolean`은 지원하지 않는다. 따라서 모든 픽스처는 `data` 필드로 데이터 모델 초기값을 포함한다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `ComponentFixture` (타입 import)

## Exports

- `checkboxUnchecked`: `ComponentFixture` — 초기값 `false`, 미체크 상태 픽스처
- `checkboxChecked`: `ComponentFixture` — 초기값 `true`, 체크 상태 픽스처
- `checkboxLongLabel`: `ComponentFixture` — 초기값 `false`, 긴 레이블 텍스트 픽스처
- `checkboxFixtures`: `{ checkboxUnchecked, checkboxChecked, checkboxLongLabel }` — 컬렉션 객체

## 상세 명세

### `checkboxUnchecked: ComponentFixture`

- `root`: `'cb-1'`
- `data`: `{'/checkbox/unchecked': false}`
- `components`:
  - `id: 'cb-1'`, `CheckBox`, `label: {literalString: 'Unchecked option'}`, `value: {path: '/checkbox/unchecked'}`

### `checkboxChecked: ComponentFixture`

- `root`: `'cb-2'`
- `data`: `{'/checkbox/checked': true}`
- `components`:
  - `id: 'cb-2'`, `CheckBox`, `label: {literalString: 'Checked option'}`, `value: {path: '/checkbox/checked'}`

### `checkboxLongLabel: ComponentFixture`

- `root`: `'cb-long'`
- `data`: `{'/checkbox/longLabel': false}`
- `components`:
  - `id: 'cb-long'`, `CheckBox`, `label: {literalString: 'I agree to the terms and conditions of service and privacy policy'}`, `value: {path: '/checkbox/longLabel'}`

### `checkboxFixtures`

세 픽스처를 하나의 객체로 묶어 내보내는 네임스페이스 상수.

## 동작 흐름

각 픽스처의 `data` 필드가 데이터 모델에 사전 주입되어 체크박스의 초기 시각적 상태를 결정한다. `value: {path: ...}` 바인딩으로 데이터 모델과 양방향 연결된다. `checkboxLongLabel`은 레이블이 매우 길 때의 줄바꿈·레이아웃 처리를 시각적으로 검증하기 위한 케이스이다.
