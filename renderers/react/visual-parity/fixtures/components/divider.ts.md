# renderers/react/visual-parity/fixtures/components/divider.ts

## 개요

`Divider` 컴포넌트의 시각적 동등성 테스트를 위한 픽스처 정의 파일이다. 수평(horizontal, 기본값) 구분선과 수직(vertical) 구분선 두 가지 케이스를 제공한다. 두 픽스처 모두 단일 컴포넌트로 구성된 최소한의 구조를 가진다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `ComponentFixture` (타입 import)

## Exports

- `dividerHorizontal`: `ComponentFixture` — 수평 구분선 (기본 props)
- `dividerVertical`: `ComponentFixture` — 수직 구분선 (`axis: 'vertical'`)
- `dividerFixtures`: `{ dividerHorizontal, dividerVertical }` — 컬렉션 객체

## 상세 명세

### `dividerHorizontal: ComponentFixture`

- `root`: `'div-h'`
- `data`: 없음
- `components`:
  - `id: 'div-h'`, `Divider`, `component` 값이 `{Divider: {}}` (빈 객체, 모든 prop 기본값 적용)
- 기본값으로 수평 방향 구분선이 렌더링된다.

### `dividerVertical: ComponentFixture`

- `root`: `'div-v'`
- `data`: 없음
- `components`:
  - `id: 'div-v'`, `Divider`, `axis: 'vertical'`

### `dividerFixtures`

두 픽스처를 하나의 객체로 묶어 내보내는 네임스페이스 상수.

## 동작 흐름

`dividerHorizontal`은 `axis` prop 없이 Divider를 렌더링하여 기본값(수평) 동작을 검증하고, `dividerVertical`은 `axis: 'vertical'`을 명시하여 수직 방향 구분선이 올바르게 렌더링됨을 확인한다. 두 케이스 모두 단일 컴포넌트로 구성된 가장 단순한 픽스처 구조를 가진다.
