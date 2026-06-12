# renderers/react/visual-parity/fixtures/components/row.ts

## 개요

Row 컴포넌트의 비주얼 패리티 테스트용 픽스처 모음이다. 아이콘 3개를 나열하는 기본 Row, 텍스트 3개를 나열하는 Row, 그리고 `distribution` 속성의 네 가지 값(`start`, `center`, `end`, `spaceBetween`)을 각각 검증하는 Row 픽스처 총 6개를 포함한다.

## 의존성

### 저장소 내부 모듈

- [`../types`](../types.ts.md) — `ComponentFixture` 타입을 type-only import

### 외부 패키지

없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `row` | 상수 (`ComponentFixture`) | Icon 3개를 담은 기본 Row |
| `rowWithText` | 상수 (`ComponentFixture`) | Text 3개를 담은 Row |
| `rowStart` | 상수 (`ComponentFixture`) | `distribution: 'start'` Row |
| `rowCenter` | 상수 (`ComponentFixture`) | `distribution: 'center'` Row |
| `rowEnd` | 상수 (`ComponentFixture`) | `distribution: 'end'` Row |
| `rowSpaceBetween` | 상수 (`ComponentFixture`) | `distribution: 'spaceBetween'` Row |
| `rowFixtures` | 상수 (객체) | 위 6개 픽스처를 묶은 그룹 객체 |

## 상세 명세

### `row: ComponentFixture`

- `root`: `'row-1'`
- 자식: `id: 'row-icon-1'` Icon(`'home'`), `id: 'row-icon-2'` Icon(`'search'`), `id: 'row-icon-3'` Icon(`'settings'`)
- 루트: `id: 'row-1'`, `Row.children.explicitList: ['row-icon-1', 'row-icon-2', 'row-icon-3']`, `distribution` 미설정

### `rowWithText: ComponentFixture`

- `root`: `'row-text'`
- 자식: `id: 'row-text-1'` Text(`'First'`), `id: 'row-text-2'` Text(`'Second'`), `id: 'row-text-3'` Text(`'Third'`)
- 루트: `id: 'row-text'`, `Row.children.explicitList: ['row-text-1', 'row-text-2', 'row-text-3']`, `distribution` 미설정

### `rowStart: ComponentFixture`

- `root`: `'row-start'`
- 자식: Text `'A'`, `'B'`, `'C'` (id: `'row-start-1'`~`'row-start-3'`)
- 루트: `id: 'row-start'`, `Row.distribution: 'start'`

### `rowCenter: ComponentFixture`

- `root`: `'row-center'`
- 자식: Text `'A'`, `'B'`, `'C'` (id: `'row-center-1'`~`'row-center-3'`)
- 루트: `id: 'row-center'`, `Row.distribution: 'center'`

### `rowEnd: ComponentFixture`

- `root`: `'row-end'`
- 자식: Text `'A'`, `'B'`, `'C'` (id: `'row-end-1'`~`'row-end-3'`)
- 루트: `id: 'row-end'`, `Row.distribution: 'end'`

### `rowSpaceBetween: ComponentFixture`

- `root`: `'row-space'`
- 자식: Text `'Left'` (`id: 'row-space-1'`), `'Center'` (`id: 'row-space-2'`), `'Right'` (`id: 'row-space-3'`)
- 루트: `id: 'row-space'`, `Row.distribution: 'spaceBetween'`

### `rowFixtures: object`

`row`, `rowWithText`, `rowStart`, `rowCenter`, `rowEnd`, `rowSpaceBetween` 6개를 키-값으로 묶은 집계 객체.

## 동작 흐름

모든 픽스처는 자식 컴포넌트(Text 또는 Icon)를 먼저 배열에 나열하고, 마지막 엔트리의 Row 컴포넌트가 `explicitList`로 자식 ID를 참조한다. `distribution` 속성 검증이 이 파일의 핵심 목적으로, `start`·`center`·`end`·`spaceBetween` 네 가지 값 모두를 독립 픽스처로 커버한다. `data` 필드는 사용되지 않는다.
