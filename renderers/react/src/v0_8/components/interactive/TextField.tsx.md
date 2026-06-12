# renderers/react/src/v0_8/components/interactive/TextField.tsx

## 개요

`TextField` 컴포넌트는 텍스트, 숫자, 날짜를 입력받는 필드를 렌더링한다. `textFieldType` 속성에 따라 일반 `<input>` 또는 여러 줄 `<textarea>`로 전환되며, 정규식 기반 유효성 검사와 양방향 데이터 바인딩을 지원한다. Lit 렌더러의 `TextField` 커스텀 엘리먼트와 동일한 DOM 구조를 유지한다.

## 의존성

### 외부 패키지
- `react`: `useState`, `useCallback`, `useEffect`, `useId`, `memo`
- `@a2ui/web_core/types/types`: `Types.TextFieldNode` (타입 전용 import)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md): `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md): `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md): `classMapToString`, `stylesToObject` 유틸

## Exports

- `TextField` — memo로 감싼 named export (React 함수 컴포넌트)
- `default` — `TextField`를 default export로도 제공

## 상세 명세

### `TextFieldType` (타입 별칭, 비공개)

`'shortText' | 'longText' | 'number' | 'date'` 네 가지 문자열 리터럴 유니온 타입. 컴포넌트 파일 내부에서만 사용되며 외부로 export되지 않는다.

### `TextField` (컴포넌트)

**시그니처:** `memo(function TextField({ node, surfaceId }: A2UIComponentProps<Types.TextFieldNode>): JSX.Element)`

**내부 훅 및 변수 초기화:**

1. `useA2UIComponent(node, surfaceId)`로 `theme`, `resolveString`, `setValue`, `getValue`를 가져온다.
2. `resolveString(props.label)`로 레이블 문자열을 결정한다.
3. `props.text?.path`를 `textPath`에 저장한다.
4. `resolveString(props.text) ?? ''`로 초기 값을 결정하고 `useState`로 `value` 상태를 초기화한다 (setter 이름은 `setLocalValue`).
5. `props`를 `Record<string, unknown>`으로 캐스팅한 뒤 `propsRecord.textFieldType || propsRecord.type`을 `TextFieldType`으로 캐스팅해 `fieldType`에 저장한다. 두 속성명을 모두 지원하여 하위 호환성을 유지한다.
6. `props.validationRegexp`를 저장한다.
7. `useState(true)`로 `_isValid` 상태를 초기화한다. 현재는 미래 에러 스타일링을 위해 추적만 하며 렌더링에는 사용하지 않는다 (접두사 `_`로 표시).
8. `useId()`로 고유 `id`를 생성한다.

**useEffect — 외부 데이터 모델 동기화:**

`textPath`와 `getValue`를 의존성으로 갖는다. `textPath`가 존재할 때 `getValue(textPath)`를 호출하여 반환값이 `null`이 아니고 현재 `value`와 다르면 `setLocalValue(String(externalValue))`로 동기화한다.

**`handleChange` 콜백:**

`React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>`를 받아 `e.target.value`를 읽는다. 로컬 상태를 갱신한 뒤, `validationRegexp`가 있으면 `new RegExp(validationRegexp).test(newValue)`로 유효성을 검사해 `setIsValid`를 호출한다. 마지막으로 `textPath`가 있으면 `setValue(textPath, newValue)`로 데이터 모델을 갱신한다. 의존성은 `[validationRegexp, textPath, setValue]`이다.

**`inputType` 결정:**

`fieldType === 'number'`이면 `'number'`, `fieldType === 'date'`이면 `'date'`, 그 외(`'shortText'` 포함 모든 경우)는 `'text'`를 사용한다.

**`isTextArea` 결정:**

`fieldType === 'longText'`이면 `true`, 그 외는 `false`.

**hostStyle 계산:**

`node.weight`가 정의된 경우 `{'--weight': node.weight}` CSS 변수 객체를 생성한다.

**렌더링 구조:**

최상위 `<div className="a2ui-textfield">`에 `hostStyle`을 적용한다. 내부 `<section>`에 `theme.components.TextField.container` 클래스를 적용한다. `label`이 truthy이면 `<label htmlFor={id}>`을 렌더링한다. `isTextArea`가 `true`이면 `<textarea>`를, 아니면 `<input type={inputType}>`을 렌더링한다. 두 요소 모두 `placeholder="Please enter a value"`, `theme.components.TextField.element` 클래스, `stylesToObject(theme.additionalStyles?.TextField)` 스타일을 공유한다.

## 동작 흐름

마운트 시 `fieldType`으로 렌더링할 요소 종류(input/textarea)와 input type을 결정한다. 사용자 입력 시 `handleChange`가 로컬 상태 갱신, 유효성 검사, 외부 데이터 모델 갱신을 순서대로 수행한다. 외부 데이터 모델이 변경되면 path 동기화 effect가 내부 상태를 업데이트한다.
