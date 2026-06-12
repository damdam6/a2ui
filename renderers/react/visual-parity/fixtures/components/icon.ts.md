# renderers/react/visual-parity/fixtures/components/icon.ts

## 개요

`Icon` 컴포넌트의 시각적 동등성 테스트를 위한 픽스처 정의 파일이다. 단일 아이콘 케이스와 여러 아이콘을 Row로 나란히 배치한 복합 케이스 두 가지를 제공한다. 모든 아이콘 이름은 `literalString`으로 지정된다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `ComponentFixture` (타입 import)

## Exports

- `icon`: `ComponentFixture` — `home` 아이콘 단일 픽스처
- `iconMultiple`: `ComponentFixture` — 5개 아이콘을 Row로 배치한 픽스처
- `iconFixtures`: `{ icon, iconMultiple }` — 컬렉션 객체

## 상세 명세

### `icon: ComponentFixture`

- `root`: `'icon-1'`
- `data`: 없음
- `components`:
  - `id: 'icon-1'`, `Icon`, `name: {literalString: 'home'}`

### `iconMultiple: ComponentFixture`

- `root`: `'icons-row'`
- `data`: 없음
- `components` (6개):
  1. `id: 'icon-home'` — `Icon`, `name: {literalString: 'home'}`
  2. `id: 'icon-search'` — `Icon`, `name: {literalString: 'search'}`
  3. `id: 'icon-settings'` — `Icon`, `name: {literalString: 'settings'}`
  4. `id: 'icon-favorite'` — `Icon`, `name: {literalString: 'favorite'}`
  5. `id: 'icon-star'` — `Icon`, `name: {literalString: 'star'}`
  6. `id: 'icons-row'` — `Row`, `children: {explicitList: ['icon-home', 'icon-search', 'icon-settings', 'icon-favorite', 'icon-star']}`

### `iconFixtures`

두 픽스처를 하나의 객체로 묶어 내보내는 네임스페이스 상수.

## 동작 흐름

`icon`은 단일 Material Symbol 아이콘 렌더링을 최소 구성으로 검증한다. `iconMultiple`은 여러 아이콘이 Row 레이아웃 내에서 가로로 나열될 때의 시각적 결과를 검증하며, 아이콘 이름은 Material Symbols 라이브러리의 표준 이름(`home`, `search`, `settings`, `favorite`, `star`)을 사용한다.
