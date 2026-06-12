# renderers/react/visual-parity/fixtures/components/text.ts

## 개요

Text 컴포넌트의 비주얼 패리티 테스트용 픽스처 모음이다. `usageHint` 없는 기본 텍스트부터 h1~h5 제목, body, caption까지 총 8가지 텍스트 변형을 각각 독립 픽스처로 정의한다.

## 의존성

### 저장소 내부 모듈

- [`../types`](../types.ts.md) — `ComponentFixture` 타입을 type-only import

### 외부 패키지

없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `textBasic` | 상수 (`ComponentFixture`) | `usageHint` 없는 기본 Text |
| `textH1` | 상수 (`ComponentFixture`) | `usageHint: 'h1'` Text |
| `textH2` | 상수 (`ComponentFixture`) | `usageHint: 'h2'` Text |
| `textH3` | 상수 (`ComponentFixture`) | `usageHint: 'h3'` Text |
| `textH4` | 상수 (`ComponentFixture`) | `usageHint: 'h4'` Text |
| `textH5` | 상수 (`ComponentFixture`) | `usageHint: 'h5'` Text |
| `textBody` | 상수 (`ComponentFixture`) | `usageHint: 'body'` Text |
| `textCaption` | 상수 (`ComponentFixture`) | `usageHint: 'caption'` Text |
| `textFixtures` | 상수 (객체) | 위 8개 픽스처를 묶은 그룹 객체 |

## 상세 명세

모든 픽스처는 동일한 구조를 따른다: 단일 컴포넌트 배열, `root`와 컴포넌트 `id`가 동일한 값, `Text.text.literalString`으로 텍스트 내용 지정.

### `textBasic: ComponentFixture`

- `root`: `'text-1'`, `id: 'text-1'`
- `Text.text.literalString`: `'Hello, this is basic text.'`
- `usageHint` 없음

### `textH1: ComponentFixture`

- `root`: `'text-h1'`, `id: 'text-h1'`
- `Text.text.literalString`: `'Heading 1'`, `usageHint: 'h1'`

### `textH2: ComponentFixture`

- `root`: `'text-h2'`, `id: 'text-h2'`
- `Text.text.literalString`: `'Heading 2'`, `usageHint: 'h2'`

### `textH3: ComponentFixture`

- `root`: `'text-h3'`, `id: 'text-h3'`
- `Text.text.literalString`: `'Heading 3'`, `usageHint: 'h3'`

### `textH4: ComponentFixture`

- `root`: `'text-h4'`, `id: 'text-h4'`
- `Text.text.literalString`: `'Heading 4'`, `usageHint: 'h4'`

### `textH5: ComponentFixture`

- `root`: `'text-h5'`, `id: 'text-h5'`
- `Text.text.literalString`: `'Heading 5'`, `usageHint: 'h5'`

### `textBody: ComponentFixture`

- `root`: `'text-body'`, `id: 'text-body'`
- `Text.text.literalString`: `'Body text content goes here.'`, `usageHint: 'body'`

### `textCaption: ComponentFixture`

- `root`: `'text-caption'`, `id: 'text-caption'`
- `Text.text.literalString`: `'Caption text'`, `usageHint: 'caption'`

### `textFixtures: object`

`textBasic`, `textH1`, `textH2`, `textH3`, `textH4`, `textH5`, `textBody`, `textCaption` 8개를 키-값으로 묶은 집계 객체.

## 동작 흐름

파일은 8개의 독립 픽스처를 순서대로 정의하고 마지막에 `textFixtures` 객체로 집계한다. 모든 픽스처는 `data` 필드 없이 `literalString`만 사용하므로 런타임 데이터 모델과 완전히 독립적으로 렌더링된다. 각 픽스처는 단 하나의 Text 컴포넌트만 포함하여 `usageHint` 값별 시각적 차이를 명확히 분리 검증한다.
