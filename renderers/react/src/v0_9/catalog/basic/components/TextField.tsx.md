# renderers/react/src/v0_9/catalog/basic/components/TextField.tsx

## 개요

`TextField` 컴포넌트는 a2ui basic catalog의 `TextFieldApi` 스키마를 따르는 React 구현체다. 단일행 `<input>`, 여러 줄 `<textarea>`, 숫자 입력, 비밀번호 입력을 `variant` prop 하나로 통합 지원하며, 유효성 오류 메시지와 레이블도 렌더링한다. CSS Module(`TextField.module.css`)로 스타일을 적용한다.

## 의존성

### 외부 패키지
- `react` — `React`, `React.useId`, `React.ChangeEvent`
- `@a2ui/web_core/v0_9/basic_catalog` — `TextFieldApi`
- CSS Module: `./TextField.module.css` — `styles.host`, `styles.label`, `styles.input`, `styles.invalid`, `styles.error`

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation`
- [`../utils`](../utils.ts.md) — `useBasicCatalogStyles`

## Exports

| 이름 | 종류 |
|------|------|
| `TextField` | 상수 (ReactComponentImplementation) |

## 상세 명세

### `TextField`

`createComponentImplementation(TextFieldApi, renderFn)` 호출로 생성된 컴포넌트 구현체다.

**렌더 함수 시그니처:** `({props}) => JSX.Element`

**매개변수 (props 필드):**
- `props.label` — 입력 필드 위에 표시할 레이블 문자열 (옵셔널)
- `props.value` — 현재 입력값 (문자열, 옵셔널). 없으면 `''` 사용
- `props.variant` — 입력 유형 결정자: `'longText'` → textarea, `'number'` → number input, `'obscured'` → password input, 나머지(기본) → text input
- `props.setValue` — 입력값 변경 시 호출되는 콜백 `(value: string) => void`
- `props.validationErrors` — 유효성 오류 문자열 배열 (옵셔널)

**내부 로직:**

1. `useBasicCatalogStyles()`로 global 스타일 주입.
2. `onChange` 핸들러: `React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>` 이벤트를 받아 `e.target.value`를 `props.setValue`에 전달.
3. `isLong = props.variant === 'longText'` — textarea 여부 결정.
4. `type` 결정 로직:
   - `props.variant === 'number'` → `'number'`
   - `props.variant === 'obscured'` → `'password'`
   - 그 외 → `'text'`
5. `React.useId()`로 `uniqueId` 생성 → `<label>`과 `<input>/<textarea>` 연결.
6. `hasError = props.validationErrors && props.validationErrors.length > 0` — 오류 존재 여부.
7. `inputClasses`: `` `${styles.input} ${hasError ? styles.invalid : ''}` ``

**렌더 구조:**
- 최상위 `<div className={styles.host}>`
- `props.label`이 있으면 `<label htmlFor={uniqueId} className={styles.label}>{props.label}</label>`
- `isLong`이면 `<textarea id={uniqueId} className={inputClasses} value={props.value || ''} onChange={onChange} />`
- 그 외면 `<input id={uniqueId} type={type} className={inputClasses} value={props.value || ''} onChange={onChange} />`
- `hasError`이면 `<span className={styles.error}>{props.validationErrors![0]}</span>` (첫 번째 오류만 표시)

## 동작 흐름

사용자 입력 → `onChange` → `props.setValue(e.target.value)` 호출 → 상위에서 `props.value` 업데이트 → 재렌더링. 유효성 오류가 있으면 입력 요소에 `styles.invalid` 클래스가 추가되고 첫 번째 오류 메시지가 아래에 표시된다. `props.value`가 falsy이면 `''`로 대체하여 uncontrolled 경고를 방지한다.
