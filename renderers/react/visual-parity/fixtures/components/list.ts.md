# renderers/react/visual-parity/fixtures/components/list.ts

## 개요

List 컴포넌트의 비주얼 패리티 테스트용 픽스처 모음이다. 세로 방향 텍스트 목록, 가로 방향 아이콘 목록, 카드를 아이템으로 갖는 목록, 헤더와 아이콘+텍스트 행이 혼합된 복합 목록의 네 가지 시나리오를 포함한다. 모든 픽스처는 플랫한 컴포넌트 배열 구조로 자식 컴포넌트를 먼저 정의하고 부모 List 컴포넌트가 `explicitList`로 참조한다.

## 의존성

### 저장소 내부 모듈

- [`../types`](../types.ts.md) — `ComponentFixture` 타입을 type-only import

### 외부 패키지

없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `listVertical` | 상수 (`ComponentFixture`) | 세로 방향 Text 아이템 4개를 갖는 List |
| `listHorizontal` | 상수 (`ComponentFixture`) | 가로 방향 Icon 아이템 3개를 갖는 List |
| `listWithCards` | 상수 (`ComponentFixture`) | Card 아이템 3개를 갖는 List |
| `listMixed` | 상수 (`ComponentFixture`) | 헤더 + 아이콘/텍스트 Row 아이템 혼합 List |
| `listFixtures` | 상수 (객체) | 위 4개 픽스처를 묶은 그룹 객체 |

## 상세 명세

### `listVertical: ComponentFixture`

- `root`: `'list-v'`
- 자식 컴포넌트: `id`가 `'list-v-1'`~`'list-v-4'`인 Text 컴포넌트 4개, 각 텍스트는 `'Item 1'`~`'Item 4'`
- 루트 컴포넌트: `id: 'list-v'`, `List.children.explicitList: ['list-v-1', 'list-v-2', 'list-v-3', 'list-v-4']`
- `direction` 미설정 → 렌더러 기본값(세로)으로 동작

### `listHorizontal: ComponentFixture`

- `root`: `'list-h'`
- 자식 컴포넌트: `id: 'list-h-1'` Icon(`'home'`), `id: 'list-h-2'` Icon(`'search'`), `id: 'list-h-3'` Icon(`'settings'`)
- 루트 컴포넌트: `id: 'list-h'`, `List.children.explicitList: ['list-h-1', 'list-h-2', 'list-h-3']`, `direction: 'horizontal'`

### `listWithCards: ComponentFixture`

- `root`: `'list-cards'`
- 세 개의 Card 아이템(`'card1'`, `'card2'`, `'card3'`)으로 구성되며, 각 Card는 Column을 child로 가지고 Column은 h3 Text와 본문 Text를 포함한다.
  - Card 1: 타이틀 `'Card One'` (usageHint: `'h3'`), 본문 `'First card content'`
  - Card 2: 타이틀 `'Card Two'` (usageHint: `'h3'`), 본문 `'Second card content'`
  - Card 3: 타이틀 `'Card Three'` (usageHint: `'h3'`), 본문 `'Third card content'`
- 루트 컴포넌트: `id: 'list-cards'`, `List.children.explicitList: ['card1', 'card2', 'card3']`

컴포넌트 배열 내 선언 순서: 각 Card의 title → body → column → card 순으로 정의 후 다음 Card로 진행. 마지막에 `'list-cards'` List 컴포넌트.

### `listMixed: ComponentFixture`

- `root`: `'list-mixed'`
- 헤더 Text `'Feature List'` (usageHint: `'h2'`, id: `'list-mixed-h'`)와 아이콘+텍스트 Row 3개(`'feat1'`, `'feat2'`, `'feat3'`)로 구성
  - 각 feat 항목: Icon(`'check'`) + Text(`'Feature One'`/`'Feature Two'`/`'Feature Three'`)를 Row로 묶음
- 루트 컴포넌트: `id: 'list-mixed'`, `List.children.explicitList: ['list-mixed-h', 'feat1', 'feat2', 'feat3']`

### `listFixtures: object`

`listVertical`, `listHorizontal`, `listWithCards`, `listMixed` 네 픽스처를 키-값으로 묶은 집계 객체.

## 동작 흐름

모든 픽스처는 동일한 패턴을 따른다: 리프 컴포넌트(Text, Icon 등)를 먼저 components 배열에 나열하고, 컨테이너 컴포넌트(Row, Column, Card)가 그 ID를 `explicitList`로 참조하며, 최종 List 컴포넌트가 최상위 자식 ID들을 참조한다. `data` 필드는 사용되지 않으며 런타임 데이터 모델 없이도 정적으로 렌더링된다.
