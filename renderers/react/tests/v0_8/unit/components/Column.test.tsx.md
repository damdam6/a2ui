# renderers/react/tests/v0_8/unit/components/Column.test.tsx

## 개요

A2UI 명세를 따르는 `Column` 컴포넌트의 단위 테스트 모음이다. `Column`의 DOM 렌더링 구조, 자식 컴포넌트 표시, `alignment`·`distribution` 속성 처리, 중첩 레이아웃, 테마 클래스 적용을 검증한다. 모든 테스트는 서버-클라이언트 메시지를 시뮬레이션하는 `TestRenderer`를 통해 실제 React 렌더링을 수행하고 결과 DOM을 단언한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`, `screen`
- `react` — `React`
- `@a2ui/web_core/types/types` — `Types` (타입 전용 import)

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSurfaceUpdate`, `createBeginRendering`

## Exports

이 파일은 아무것도 export하지 않는다. 테스트 실행 시 vitest가 직접 로드한다.

## 상세 명세 (테스트 케이스)

### describe: `Column Component`

#### describe: `Basic Rendering`

- **`should render a section element`**
  - 검증: `Column`을 루트 컴포넌트로 렌더링하면 DOM에 `section` 요소가 존재한다.
  - 픽스처: `text-1`(Text) + `col-1`(Column, explicitList: `['text-1']`) 메시지를 구성하고 `col-1`을 루트로 `BeginRendering`.

- **`should render with wrapper div`**
  - 검증: `.a2ui-column` 클래스를 가진 래퍼 요소가 DOM에 존재한다.
  - 픽스처: 위와 동일한 단일 Text 자식을 포함하는 Column 메시지.

#### describe: `Children Rendering`

- **`should render child Text components`**
  - 검증: `explicitList`에 두 개의 Text ID를 나열하면 각각의 텍스트 내용(`'Row 1'`, `'Row 2'`)이 화면에 렌더링된다.
  - 픽스처: `text-1`(`'Row 1'`), `text-2`(`'Row 2'`), `col-1`(explicitList: `['text-1', 'text-2']`).

- **`should render empty column with empty explicitList`**
  - 검증: `explicitList`가 빈 배열(`[]`)이어도 `.a2ui-column` 요소가 렌더링된다.
  - 픽스처: `col-1`(Column, explicitList: `[]`).

#### describe: `Alignment`

- **`should default to stretch alignment`**
  - 검증: `alignment` 속성을 지정하지 않으면 `.a2ui-column`의 `data-alignment` 어트리뷰트가 `'stretch'`이다.

- **`should set data-alignment="${alignment}"` (파라미터화)**
  - 대상 값: `['start', 'center', 'end', 'stretch']` (const 배열 `alignments`).
  - 검증: 각 값을 `alignment` 속성으로 전달하면 `.a2ui-column`의 `data-alignment`가 해당 값으로 설정된다.
  - `alignments.forEach`로 4개의 개별 테스트를 동적 생성한다.

#### describe: `Distribution`

- **`should default to start distribution`**
  - 검증: `distribution` 속성 미지정 시 `.a2ui-column`의 `data-distribution`이 `'start'`이다.

- **`should set data-distribution="${distribution}"` (파라미터화)**
  - 대상 값: `['start', 'center', 'end', 'spaceBetween', 'spaceAround', 'spaceEvenly']` (const 배열 `distributions`).
  - 검증: 각 값을 `distribution` 속성으로 전달하면 `data-distribution`이 해당 값이 된다.
  - `distributions.forEach`로 6개의 개별 테스트를 동적 생성한다.

#### describe: `Nested Layouts`

- **`should render Row inside Column`**
  - 검증: Column의 자식으로 Row를 중첩하면 `.a2ui-column`과 `.a2ui-row` 요소가 모두 DOM에 존재하고, Row 내부 Text의 내용(`'Nested text'`)도 화면에 출력된다.
  - 픽스처: `text-1`(Text), `row-1`(Row, explicitList: `['text-1']`), `col-1`(Column, explicitList: `['row-1']`).

#### describe: `Theme Support`

- **`should apply theme classes to section`**
  - 검증: `section` 요소의 `className`이 truthy(빈 문자열이 아닌 값)이다. 테마가 section에 클래스를 부여함을 확인.

#### describe: `Structure`

- **`should have correct DOM structure`**
  - 검증: `.a2ui-column` 내에 자식이 정확히 1개 있고, 그 자식의 태그명이 `'SECTION'`이다. 즉 DOM 구조는 `div.a2ui-column > section`이다.

## 동작 흐름

각 테스트는 `createSurfaceUpdate`로 컴포넌트 노드 맵(ID → 컴포넌트 정의)을 담은 메시지와 `createBeginRendering`으로 루트 노드 ID를 지정하는 메시지를 배열로 구성한다. `TestRenderer`가 이 메시지 배열을 소비하여 실제 A2UI React 렌더러를 구동하고, `TestWrapper`가 필요한 Context(테마 등)를 제공한다. 테스트는 `@testing-library/react`의 `render`·`screen`으로 결과 DOM을 쿼리해 단언한다. 파라미터화 테스트는 상수 배열에 `forEach`를 호출하여 동적으로 `it` 블록을 등록한다.
