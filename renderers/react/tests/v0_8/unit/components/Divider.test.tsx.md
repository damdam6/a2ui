# renderers/react/tests/v0_8/unit/components/Divider.test.tsx

## 개요

A2UI 명세를 따르는 `Divider` 컴포넌트의 단위 테스트 모음이다. `Divider`는 필수 속성이 없고 선택 속성 `axis`(`'horizontal'` | `'vertical'`)만 가지는 단순한 구분선 컴포넌트다. `hr` 요소 렌더링, 래퍼 구조, axis 처리, 테마 클래스 적용, Column 내 다중 렌더링을 검증한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`
- `react` — `React`
- `@a2ui/web_core/types/types` — `Types` (타입 전용 import)

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSimpleMessages`, `createSurfaceUpdate`, `createBeginRendering`

## Exports

이 파일은 아무것도 export하지 않는다.

## 상세 명세 (테스트 케이스)

### describe: `Divider Component`

#### describe: `Basic Rendering`

- **`should render an hr element`**
  - 검증: 속성 없이(`{}`) 렌더링하면 `hr` 요소가 DOM에 존재한다.
  - 픽스처: `createSimpleMessages('div-1', 'Divider', {})`.

- **`should render with wrapper div`**
  - 검증: `.a2ui-divider` 클래스를 가진 래퍼 요소가 DOM에 존재한다.

- **`should render as self-closing element`**
  - 검증: `hr` 요소의 `children.length`가 `0`이다. `Divider`는 자식 노드를 가지지 않는다.

#### describe: `Axis`

- **`should render horizontal divider`**
  - 검증: `axis: 'horizontal'`을 전달해도 `hr` 요소가 정상 렌더링된다.

- **`should render vertical divider`**
  - 검증: `axis: 'vertical'`을 전달해도 `hr` 요소가 정상 렌더링된다.
  - 참고: 두 테스트 모두 `hr`의 존재 여부만 확인하며, axis에 따른 DOM 어트리뷰트 차이는 검사하지 않는다(스타일은 CSS 담당).

#### describe: `Theme Support`

- **`should render hr element with theme classes applied`**
  - 검증: `hr` 요소가 DOM에 존재한다. 주석에 따르면 기본 테마에서 Divider의 테마 클래스는 비어 있을 수 있다.

#### describe: `Structure`

- **`should have correct DOM structure`**
  - 검증: `.a2ui-divider`의 자식이 정확히 1개이고 그 자식의 태그명이 `'HR'`이다. DOM 구조는 `div.a2ui-divider > hr`.

#### describe: `Multiple Dividers`

- **`should render multiple dividers in a column`**
  - 검증: Column의 `explicitList`에 3개의 Divider ID(`'div-1'`, `'div-2'`, `'div-3'`)를 넣으면 `.a2ui-divider` 요소가 3개 렌더링된다.
  - 픽스처: `createSurfaceUpdate`로 `div-1`·`div-2`·`div-3`(각 `{Divider: {}}`) + `col-1`(Column, explicitList: `['div-1', 'div-2', 'div-3']`) 메시지를 구성하고 `createBeginRendering('col-1')`로 루트 설정. 이 테스트만 `Types.ServerToClientMessage[]` 타입을 명시적으로 사용한다.

## 동작 흐름

단순 렌더링 테스트는 `createSimpleMessages`를 사용해 단일 컴포넌트 메시지를 생성한다. 다중 Divider 테스트는 `createSurfaceUpdate` + `createBeginRendering` 조합으로 Column 컨텍스트를 구성한다. 모든 테스트에서 `TestWrapper`와 `TestRenderer`가 A2UI React 렌더링 환경을 제공하며, `render`의 반환 값 `container`를 통해 DOM을 쿼리하여 단언한다.
