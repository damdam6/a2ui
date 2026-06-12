# renderers/react/visual-parity/fixtures/components/column.ts

## 개요

`Column` 레이아웃 컴포넌트의 시각적 동등성 테스트를 위한 픽스처 정의 파일이다. 기본 column, `distribution: 'start'`, `distribution: 'center'`, `distribution: 'end'`, 혼합 텍스트 스타일 다섯 가지 케이스를 제공하여 Column의 배치(distribution) 속성과 다양한 자식 유형을 검증한다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `ComponentFixture` (타입 import)

## Exports

- `column`: `ComponentFixture` — distribution 미지정 기본 Column
- `columnStart`: `ComponentFixture` — `distribution: 'start'` Column
- `columnCenter`: `ComponentFixture` — `distribution: 'center'` Column
- `columnEnd`: `ComponentFixture` — `distribution: 'end'` Column
- `columnMixed`: `ComponentFixture` — h3/body/caption 혼합 텍스트 스타일 Column
- `columnFixtures`: `{ column, columnStart, columnCenter, columnEnd, columnMixed }` — 컬렉션 객체

## 상세 명세

모든 픽스처는 3개의 Text 자식 컴포넌트를 Column으로 묶는 동일한 구조를 따른다. 공통 패턴:
- `data`: 없음 (정적 리터럴만 사용)
- Text 자식들은 `{text: {literalString: '...'}}`으로 정의
- Column은 `children: {explicitList: [id1, id2, id3]}`으로 자식 목록 지정

### `column: ComponentFixture`

- `root`: `'col-1'`
- 자식 텍스트: `'First item'`, `'Second item'`, `'Third item'`
- Column props: `distribution` 미지정 (기본값 적용)

### `columnStart: ComponentFixture`

- `root`: `'col-start'`
- 자식 텍스트: `'Top'`, `'Middle'`, `'Bottom'`
- Column props: `distribution: 'start'`

### `columnCenter: ComponentFixture`

- `root`: `'col-center'`
- 자식 텍스트: `'Top'`, `'Middle'`, `'Bottom'`
- Column props: `distribution: 'center'`

### `columnEnd: ComponentFixture`

- `root`: `'col-end'`
- 자식 텍스트: `'Top'`, `'Middle'`, `'Bottom'`
- Column props: `distribution: 'end'`

### `columnMixed: ComponentFixture`

- `root`: `'col-mixed'`
- 자식 컴포넌트 (3개, 각각 `usageHint` 다름):
  1. `id: 'col-mixed-h'` — `Text`, `'Section Title'`, `usageHint: 'h3'`
  2. `id: 'col-mixed-body'` — `Text`, `'Regular body text content.'` (usageHint 없음)
  3. `id: 'col-mixed-caption'` — `Text`, `'Caption text below'`, `usageHint: 'caption'`
- Column props: `distribution` 미지정

### `columnFixtures`

다섯 픽스처를 하나의 객체로 묶어 내보내는 네임스페이스 상수.

## 동작 흐름

각 픽스처는 Column의 `distribution` 속성이 자식 요소의 주축(세로 방향) 배치를 어떻게 결정하는지를 검증하기 위해 설계되었다. `columnMixed`는 distribution과 무관하게 다양한 텍스트 유형(`h3`, 기본, `caption`)이 Column 내에서 올바르게 렌더링되는지를 확인한다.
