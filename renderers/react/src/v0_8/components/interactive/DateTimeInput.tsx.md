# renderers/react/src/v0_8/components/interactive/DateTimeInput.tsx

## 개요

`DateTimeInput` 컴포넌트는 날짜, 시간, 또는 날짜+시간을 함께 입력받는 HTML5 네이티브 입력 요소를 렌더링한다. `enableDate`와 `enableTime` 속성 조합에 따라 `date`, `time`, `datetime-local` 중 하나의 `input type`을 선택하며, Lit 렌더러와 동일한 DOM 구조를 유지한다. 외부 데이터 모델과의 양방향 바인딩을 지원한다.

## 의존성

### 외부 패키지
- `react`: `useState`, `useCallback`, `useEffect`, `useId`, `memo`
- `@a2ui/web_core/types/types`: `Types.DateTimeInputNode` (타입 전용 import)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md): `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md): `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md): `classMapToString`, `stylesToObject` 유틸

## Exports

- `DateTimeInput` — memo로 감싼 named export (React 함수 컴포넌트)
- `default` — `DateTimeInput`을 default export로도 제공

## 상세 명세

### `DateTimeInput` (컴포넌트)

**시그니처:** `memo(function DateTimeInput({ node, surfaceId }: A2UIComponentProps<Types.DateTimeInputNode>): JSX.Element)`

**내부 훅 및 변수 초기화:**

1. `useA2UIComponent(node, surfaceId)`로 `theme`, `resolveString`, `setValue`, `getValue`를 가져온다.
2. `props.value?.path`를 `valuePath`에 저장한다.
3. `resolveString(props.value) ?? ''`로 초기 값을 결정하고 `useState`로 `value` 상태를 초기화한다 (setter 이름은 `setLocalValue`).
4. `props.enableDate ?? true`로 날짜 활성화 여부를 결정한다 (기본값 `true`).
5. `props.enableTime ?? false`로 시간 활성화 여부를 결정한다 (기본값 `false`).
6. `useId()`로 고유 `id`를 생성해 `<label>`과 `<input>`을 연결한다.

**useEffect — 외부 데이터 모델 동기화:**

`valuePath`와 `getValue`를 의존성으로 갖는다. `valuePath`가 존재할 때 `getValue(valuePath)`를 호출하여 반환값이 `null`이 아니고 현재 `value`와 다르면 `setLocalValue(String(externalValue))`로 동기화한다.

**`handleChange` 콜백:**

`React.ChangeEvent<HTMLInputElement>`를 받아 `e.target.value`를 읽은 뒤 `setLocalValue`로 로컬 상태를 갱신하고, `valuePath`가 있으면 `setValue(valuePath, newValue)`로 데이터 모델도 갱신한다. 의존성은 `[valuePath, setValue]`이다.

**`inputType` 결정 로직:**

- `enableDate && enableTime`이면 `'datetime-local'`
- `enableTime && !enableDate`이면 `'time'`
- 그 외 (기본, enableDate만 true)이면 `'date'`

**`getPlaceholderText` 내부 함수 (인라인 arrow function):**

`enableDate && enableTime`이면 `'Date & Time'`, `enableTime`만 true이면 `'Time'`, 그 외는 `'Date'`를 반환한다. 이 문자열은 `<label>` 내용으로 사용된다.

**hostStyle 계산:**

`node.weight`가 정의된 경우 `{'--weight': node.weight}` CSS 변수 객체를 생성한다.

**렌더링 구조:**

최상위 `<div className="a2ui-datetime-input">`에 `hostStyle`을 적용한다. 내부 `<section>`에 `theme.components.DateTimeInput.container` 클래스를 적용한다. `<section>` 안에는 `<label>`(내용: `getPlaceholderText()` 반환값, `htmlFor={id}`, `theme.components.DateTimeInput.label` 클래스)과 `<input>`(`type={inputType}`, `id`, `value`, `onChange`, `theme.components.DateTimeInput.element` 클래스, `stylesToObject(theme.additionalStyles?.DateTimeInput)` 스타일)이 들어간다.

## 동작 흐름

마운트 시 props에서 초기 값과 모드(`enableDate`/`enableTime`)를 읽어 입력 타입과 레이블 텍스트를 확정한다. 사용자 입력은 `handleChange`가 처리하며 로컬 상태와 외부 데이터 모델을 동시에 갱신한다. 외부에서 데이터 모델이 변경되면 path 동기화 `useEffect`가 내부 상태를 갱신한다.
