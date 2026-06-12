# renderers/react/src/v0_8/components/interactive/CheckBox.tsx

## 개요

`CheckBox` 컴포넌트는 불리언 값을 토글하는 체크박스 입력 요소와 레이블을 함께 렌더링한다. Lit 렌더러의 `CheckBox` 커스텀 엘리먼트와 동일한 DOM 구조를 유지하며, 외부 데이터 모델과의 양방향 바인딩을 지원한다. 서버에서 전달된 `surfaceUpdate`로 인한 props 변경 시에도 내부 상태를 동기화한다.

## 의존성

### 외부 패키지
- `react`: `useState`, `useCallback`, `useEffect`, `useId`, `memo`
- `@a2ui/web_core/types/types`: `Types.CheckboxNode` (타입 전용 import)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md): `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md): `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md): `classMapToString`, `stylesToObject` 유틸

## Exports

- `CheckBox` — memo로 감싼 named export (React 함수 컴포넌트)
- `default` — `CheckBox`를 default export로도 제공

## 상세 명세

### `CheckBox` (컴포넌트)

**시그니처:** `memo(function CheckBox({ node, surfaceId }: A2UIComponentProps<Types.CheckboxNode>): JSX.Element)`

**내부 훅 및 변수 초기화:**

1. `useA2UIComponent(node, surfaceId)`를 호출해 `theme`, `resolveString`, `resolveBoolean`, `setValue`, `getValue`를 가져온다.
2. `node.properties`를 `props`에 할당한다.
3. `useId()`로 고유 `id` 문자열을 생성한다 (`<input>`과 `<label>`을 연결하는 데 사용).
4. `resolveString(props.label)`로 레이블 문자열을 결정한다.
5. `props.value?.path`를 `valuePath`에 저장한다 (경로 바인딩 여부 판별).
6. `resolveBoolean(props.value) ?? false`로 초기 체크 상태를 결정하고 `useState`로 `checked` 상태를 초기화한다.

**useEffect — 외부 데이터 모델 동기화 (path 바인딩):**

`valuePath`와 `getValue`를 의존성으로 갖는다. `valuePath`가 존재할 때 `getValue(valuePath)`를 호출하여 반환값이 `null`이 아니고 현재 `checked` 상태와 다르면 `setChecked(Boolean(externalValue))`로 동기화한다. eslint exhaustive-deps 규칙을 의도적으로 비활성화한다.

**useEffect — literalBoolean 변경 동기화:**

`props.value?.literalBoolean`이 `undefined`가 아닌 경우 해당 값으로 `setChecked`를 호출한다. 서버 주도 갱신(`surfaceUpdate`) 시 리터럴 값이 변경될 때 반응한다. 의존성은 `[props.value?.literalBoolean]`이다.

**`handleChange` 콜백:**

`React.ChangeEvent<HTMLInputElement>`를 받아 `e.target.checked`를 읽은 뒤 `setChecked`로 로컬 상태를 갱신하고, `valuePath`가 있으면 `setValue(valuePath, newValue)`로 데이터 모델도 갱신한다. 의존성은 `[valuePath, setValue]`이다.

**hostStyle 계산:**

`node.weight`가 `undefined`가 아니면 `{'--weight': node.weight}` 형태의 CSS 변수 객체를 생성한다. flex 레이아웃에서 자식 크기 비율을 조정하는 데 쓰인다.

**렌더링 구조:**

최상위 `<div className="a2ui-checkbox">` (`:host` 역할)에 `hostStyle`을 적용한다. 내부에 `<section>`을 두며, `theme.components.CheckBox.container` 클래스맵과 `theme.additionalStyles?.CheckBox` 스타일을 적용한다. `<section>` 안에는 `type="checkbox"` `<input>`과 조건부 `<label>`이 들어간다. `<input>`에는 `id`, `checked`, `onChange`, `theme.components.CheckBox.element` 클래스가 지정된다. `label` 값이 falsy이면 `<label>`은 렌더링하지 않으며, 렌더링 시 `htmlFor={id}`와 `theme.components.CheckBox.label` 클래스를 적용한다.

## 동작 흐름

컴포넌트 마운트 시 `resolveBoolean`으로 초기 체크 상태를 결정하고, 이후 사용자 입력에 의한 변경은 `handleChange`가 처리해 로컬 상태와 외부 데이터 모델을 동시에 업데이트한다. 외부에서 데이터 모델이 변경되면 path 동기화 `useEffect`가 감지해 내부 상태를 갱신한다. 서버 갱신으로 literalBoolean이 바뀌면 두 번째 `useEffect`가 이를 반영한다.
