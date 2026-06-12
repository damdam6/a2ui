# renderers/react/tests/v0_9/weight.test.tsx

## 개요

`getWeightStyle` 유틸리티 함수의 순수 로직과, 기본 카탈로그 컴포넌트들이 `weight` prop을 올바르게 렌더링에 반영하는지를 검증하는 테스트 파일이다. `weight`는 A2UI spec에서 CSS `flex-grow`와 유사한 레이아웃 속성으로, 설정 시 `flex`, `minWidth`, `minHeight`를 함께 적용해야 한다. 컴포넌트별 파라미터화 루프를 사용하여 Image, Text, Card, Row, Column 다섯 개 컴포넌트를 일괄 검증한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`

### 저장소 내부 모듈
- [`../utils`](../utils.tsx.md) — `renderA2uiComponent` 헬퍼
- [`../../src/v0_9/catalog/basic/utils`](../../src/v0_9/catalog/basic/utils.md) — `getWeightStyle`
- [`../../src/v0_9/catalog/basic`](../../src/v0_9/catalog/basic/index.md) — `Image`, `Text`, `Card`, `Row`, `Column`

## Exports

없음 (테스트 전용 파일).

## 상세 명세 — 테스트 케이스

### `describe('getWeightStyle')`

`getWeightStyle` 함수 자체의 반환값을 직접 검증하는 단위 테스트 4개.

- **`returns empty object when weight is undefined`**: `getWeightStyle(undefined)`이 `{}`를 반환함을 확인한다.

- **`returns flex, minWidth, and minHeight when weight is set`**: `getWeightStyle(2)`가 `{flex: '2', minWidth: 0, minHeight: 0}`을 반환함을 확인한다. `flex` 값은 숫자가 아닌 문자열 `'2'`임에 주의.

- **`handles fractional weights`**: `getWeightStyle(1.5)`가 `{flex: '1.5', minWidth: 0, minHeight: 0}`을 반환함을 확인한다.

- **`treats weight: 0 as a valid value (a child that does not grow)`**: `getWeightStyle(0)`이 `{flex: '0', minWidth: 0, minHeight: 0}`을 반환함을 확인한다. spec 주석에 따르면 `weight`는 `flex-grow`와 유사하며 `0`은 의미 있는 값이다.

### `describe('weight property is honored on basic catalog components')`

파라미터화된 루프. `cases` 배열에 5개 항목이 정의된다:

```
cases = [
  {name: 'Image', impl: Image, props: {url: 'https://example.com/x.png'}},
  {name: 'Text',  impl: Text,  props: {text: 'hello'}},
  {name: 'Card',  impl: Card,  props: {child: 'unknown-child'}},
  {name: 'Row',   impl: Row,   props: {children: []}},
  {name: 'Column',impl: Column,props: {children: []}},
]
```

각 항목에 대해 두 개의 `it` 테스트를 생성한다:

- **`${name} applies flex when weight is set`**: `{...props, weight: 2}`로 렌더링하고 `view.container.firstChild`(루트 DOM 요소)의 `style.flexGrow`가 `'2'`이고 `style.minWidth`가 `'0px'`임을 확인한다.

- **`${name} does not apply flex when weight is unset`**: `props`만으로 렌더링하고 루트 요소의 `style.flex`가 빈 문자열 `''`임을 확인한다.

## 동작 흐름

테스트는 두 단계로 구성된다. 먼저 `getWeightStyle` 순수 함수를 독립적으로 검증하고, 이후 실제 컴포넌트 렌더링에서 그 결과가 DOM 스타일에 반영되는지를 `renderA2uiComponent`를 통해 확인한다. 모든 컴포넌트가 `weight` prop 처리에 동일한 로직(`getWeightStyle`)을 사용해야 함을 보장하는 회귀 방지 테스트이다.
