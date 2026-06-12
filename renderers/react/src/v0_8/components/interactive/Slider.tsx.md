# renderers/react/src/v0_8/components/interactive/Slider.tsx

## 개요

`Slider` 컴포넌트는 `min`~`max` 범위 내에서 숫자 값을 선택하는 HTML5 `<input type="range">` 요소를 렌더링한다. 현재 값은 슬라이더 옆 `<span>`에 실시간으로 표시된다. Lit 렌더러의 `Slider` 커스텀 엘리먼트와 동일한 DOM 구조를 유지하며, 양방향 데이터 바인딩과 서버 주도 값 갱신을 모두 지원한다.

## 의존성

### 외부 패키지
- `react`: `useState`, `useCallback`, `useEffect`, `useId`, `memo`
- `@a2ui/web_core/types/types`: `Types.SliderNode` (타입 전용 import)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md): `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md): `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md): `classMapToString`, `stylesToObject` 유틸

## Exports

- `Slider` — memo로 감싼 named export (React 함수 컴포넌트)
- `default` — `Slider`를 default export로도 제공

## 상세 명세

### `Slider` (컴포넌트)

**시그니처:** `memo(function Slider({ node, surfaceId }: A2UIComponentProps<Types.SliderNode>): JSX.Element)`

**내부 훅 및 변수 초기화:**

1. `useA2UIComponent(node, surfaceId)`로 `theme`, `resolveNumber`, `resolveString`, `setValue`, `getValue`를 가져온다.
2. `props.value?.path`를 `valuePath`에 저장한다.
3. `resolveNumber(props.value) ?? 0`으로 초기 숫자 값을 결정하고 `useState`로 `value` 상태를 초기화한다 (setter 이름은 `setLocalValue`).
4. `props.minValue ?? 0`으로 최솟값을 결정한다 (Lit 기본값 `0`에 맞춤).
5. `props.maxValue ?? 0`으로 최댓값을 결정한다 (Lit 기본값 `0`에 맞춤).
6. `useId()`로 고유 `id`를 생성해 `<label>`과 `<input>`을 연결한다.

**useEffect — 외부 데이터 모델 동기화 (path 바인딩):**

`valuePath`와 `getValue`를 의존성으로 갖는다. `valuePath`가 존재할 때 `getValue(valuePath)`를 호출하여 반환값이 `null`이 아니고 현재 `value`와 다르면 `setLocalValue(Number(externalValue))`로 동기화한다.

**useEffect — literalNumber 변경 동기화:**

`props.value?.literalNumber`가 `undefined`가 아닌 경우 해당 값으로 `setLocalValue`를 호출한다. 서버 주도 갱신(`surfaceUpdate`) 시 리터럴 숫자가 바뀔 때 반응한다. 의존성은 `[props.value?.literalNumber]`이다.

**`handleChange` 콜백:**

`React.ChangeEvent<HTMLInputElement>`를 받아 `Number(e.target.value)`로 숫자로 변환한 뒤 `setLocalValue`로 로컬 상태를 갱신하고, `valuePath`가 있으면 `setValue(valuePath, newValue)`로 데이터 모델을 갱신한다. 의존성은 `[valuePath, setValue]`이다.

**`label` 결정:**

`props`를 `any`로 캐스팅해 `label` 속성을 가져온다. TypeScript 타입에 미정의된 속성이며 eslint 경고를 비활성화한다. 값이 있으면 `resolveString(labelValue)`로 문자열로 해석하고, 없으면 빈 문자열 `''`을 사용한다.

**hostStyle 계산:**

`node.weight`가 정의된 경우 `{'--weight': node.weight}` CSS 변수 객체를 생성한다.

**렌더링 구조:**

최상위 `<div className="a2ui-slider">`에 `hostStyle`을 적용한다. 내부 `<section>`에 `theme.components.Slider.container` 클래스를 적용한다. `<section>` 안에는 세 요소가 들어간다.

1. `label`이 truthy이면 `<label htmlFor={id}>`을 렌더링하며 `theme.components.Slider.label` 클래스를 적용한다.
2. `<input type="range">`: `id`, `name="data"`, `value`, `min={minValue}`, `max={maxValue}`, `onChange`, `theme.components.Slider.element` 클래스, `stylesToObject(theme.additionalStyles?.Slider)` 스타일을 적용한다.
3. `<span className={classMapToString(theme.components.Slider.label)}>{value}</span>`: 현재 숫자 값을 표시한다. `label`과 동일한 theme 클래스를 공유한다는 점에 주의한다.

## 동작 흐름

마운트 시 props에서 초기 값과 범위를 읽어 슬라이더를 초기화한다. 사용자가 슬라이더를 조작하면 `handleChange`가 로컬 상태와 외부 데이터 모델을 동시에 갱신하며, 값은 `<span>`을 통해 즉시 화면에 반영된다. 외부 데이터 모델 변경 시 path 동기화 effect가, 서버 갱신 시 literalNumber 동기화 effect가 내부 상태를 업데이트한다.
