# renderers/react/src/v0_8/components/interactive/MultipleChoice.tsx

## 개요

`MultipleChoice` 컴포넌트는 드롭다운 `<select>` 요소를 렌더링하며, 여러 선택지 중 하나를 고르는 UI를 제공한다. Lit 렌더러의 `MultipleChoice` 커스텀 엘리먼트와 동일한 DOM 구조를 유지하며, 선택 값은 배열 형태로 외부 데이터 모델에 기록된다. 내부 선택 상태를 관리하지 않는 비제어 방식 컴포넌트다.

## 의존성

### 외부 패키지
- `react`: `useCallback`, `useId`, `memo`
- `@a2ui/web_core/types/types`: `Types.MultipleChoiceNode` (타입 전용 import)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md): `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md): `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md): `classMapToString`, `stylesToObject` 유틸

## Exports

- `MultipleChoice` — memo로 감싼 named export (React 함수 컴포넌트)
- `default` — `MultipleChoice`를 default export로도 제공

## 상세 명세

### `MultipleChoice` (컴포넌트)

**시그니처:** `memo(function MultipleChoice({ node, surfaceId }: A2UIComponentProps<Types.MultipleChoiceNode>): JSX.Element)`

**내부 훅 및 변수 초기화:**

1. `useA2UIComponent(node, surfaceId)`로 `theme`, `resolveString`, `setValue`를 가져온다.
2. `props.options`를 `{label: {literalString?: string; path?: string}; value: string}[]` 타입으로 캐스팅하며, 없으면 빈 배열 `[]`을 사용한다.
3. `props.selections?.path`를 `selectionsPath`에 저장한다.
4. `props`를 `any`로 캐스팅한 뒤 `description` 속성을 `resolveString`으로 해석하고, 없으면 기본값 `'Select an item'`을 사용한다. TypeScript 타입에 정의되지 않은 속성이므로 `any` 캐스팅이 필요하며, eslint 경고를 비활성화한다.
5. `useId()`로 고유 `id`를 생성해 `<label>`과 `<select>`를 연결한다.

**`handleChange` 콜백:**

`React.ChangeEvent<HTMLSelectElement>`를 받아 `e.target.value`를 읽은 뒤, `selectionsPath`가 있으면 `setValue(selectionsPath, [e.target.value])`로 데이터 모델에 단일 선택 값을 배열로 감싸서 저장한다. Lit 렌더러의 동작과 일치시키기 위해 배열로 감싼다. 의존성은 `[selectionsPath, setValue]`이다.

**hostStyle 계산:**

`node.weight`가 정의된 경우 `{'--weight': node.weight}` CSS 변수 객체를 생성한다.

**렌더링 구조:**

최상위 `<div className="a2ui-multiplechoice">`에 `hostStyle`을 적용한다. 내부 `<section>`에 `theme.components.MultipleChoice.container` 클래스를 적용한다. `<section>` 안에는 `<label>`(내용: `description`, `htmlFor={id}`, `theme.components.MultipleChoice.label` 클래스)과 `<select>`(`name="data"`, `id`, `theme.components.MultipleChoice.element` 클래스, `stylesToObject(theme.additionalStyles?.MultipleChoice)` 스타일, `onChange={handleChange}`)가 들어간다. `<select>` 안에서 `options` 배열을 map하여 각 항목마다 `<option key={option.value} value={option.value}>{resolveString(option.label)}</option>`을 렌더링한다.

## 동작 흐름

컴포넌트는 내부 선택 상태를 유지하지 않는다. options 목록은 props에서 직접 읽어 렌더링되며, 사용자가 선택을 변경하면 `handleChange`가 즉시 외부 데이터 모델에 배열 형태로 기록한다. 초기 선택값 동기화 로직은 없으므로 브라우저의 기본 선택(첫 번째 option)이 그대로 표시된다.
