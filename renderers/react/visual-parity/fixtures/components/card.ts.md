# renderers/react/visual-parity/fixtures/components/card.ts

## 개요

`Card` 컴포넌트의 시각적 동등성 테스트를 위한 픽스처 정의 파일이다. 텍스트만 포함된 단순 카드, 헤더 이미지가 있는 카드, 아바타·이름·역할·구분선·본문을 포함한 복합 프로필 카드 세 가지 케이스를 제공한다. 모든 픽스처는 Card를 루트로 하고 Column/Row를 이용해 내부 레이아웃을 구성한다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `ComponentFixture` (타입 import)

## Exports

- `card`: `ComponentFixture` — 제목과 본문 텍스트를 가진 기본 카드
- `cardWithImage`: `ComponentFixture` — 헤더 이미지, 제목, 본문이 있는 카드
- `cardComplex`: `ComponentFixture` — 아바타, 이름, 역할, 구분선, 본문을 포함한 프로필 카드
- `cardFixtures`: `{ card, cardWithImage, cardComplex }` — 컬렉션 객체

## 상세 명세

### `card: ComponentFixture`

- `root`: `'card-1'`
- `components` (4개):
  1. `id: 'card-title'` — `Text`, `text: {literalString: 'Card Title'}`, `usageHint: 'h2'`
  2. `id: 'card-body'` — `Text`, `text: {literalString: 'Card body content goes here.'}`
  3. `id: 'card-content'` — `Column`, `children: {explicitList: ['card-title', 'card-body']}`
  4. `id: 'card-1'` — `Card`, `child: 'card-content'`

### `cardWithImage: ComponentFixture`

- `root`: `'card-img'`
- `components` (5개):
  1. `id: 'card-img-image'` — `Image`, `url: {literalString: 'https://picsum.photos/seed/card/300/150'}`, `usageHint: 'header'`
  2. `id: 'card-img-title'` — `Text`, `text: {literalString: 'Card with Image'}`, `usageHint: 'h2'`
  3. `id: 'card-img-body'` — `Text`, `text: {literalString: 'This card has a header image above the title.'}`
  4. `id: 'card-img-content'` — `Column`, `children: {explicitList: ['card-img-image', 'card-img-title', 'card-img-body']}`
  5. `id: 'card-img'` — `Card`, `child: 'card-img-content'`

### `cardComplex: ComponentFixture`

- `root`: `'card-complex'`
- `components` (9개):
  1. `id: 'card-complex-avatar'` — `Image`, `url: {literalString: 'https://picsum.photos/seed/avatar/48/48'}`, `usageHint: 'avatar'`
  2. `id: 'card-complex-name'` — `Text`, `text: {literalString: 'John Doe'}`, `usageHint: 'h3'`
  3. `id: 'card-complex-role'` — `Text`, `text: {literalString: 'Software Engineer'}`, `usageHint: 'caption'`
  4. `id: 'card-complex-info'` — `Column`, 자식: `['card-complex-name', 'card-complex-role']`
  5. `id: 'card-complex-header'` — `Row`, 자식: `['card-complex-avatar', 'card-complex-info']`
  6. `id: 'card-complex-divider'` — `Divider`, props 없음 (`{}`)
  7. `id: 'card-complex-body'` — `Text`, `text: {literalString: 'Building amazing user interfaces with A2UI.'}`
  8. `id: 'card-complex-content'` — `Column`, 자식: `['card-complex-header', 'card-complex-divider', 'card-complex-body']`
  9. `id: 'card-complex'` — `Card`, `child: 'card-complex-content'`

### `cardFixtures`

세 픽스처를 하나의 객체로 묶어 내보내는 네임스페이스 상수.

## 동작 흐름

모든 픽스처는 `Card → Column → [자식들]` 패턴으로 구성된다. `cardComplex`는 추가로 Row를 사용하여 수평 헤더 섹션(아바타 + 텍스트 정보)을 구성하고 Divider로 헤더와 본문을 분리한다. 이미지 URL은 모두 외부 picsum.photos를 사용하며 `usageHint`로 렌더러에게 시각적 힌트를 제공한다.
