# renderers/react/src/v0_9/catalog/basic/components/ChoicePicker.tsx

## 개요

`ChoicePicker` 컴포넌트는 a2ui Basic Catalog의 다중/단일 선택 UI 요소다. `ChoicePickerApi` 스키마를 기반으로 생성되며, `variant`(`mutuallyExclusive` vs 다중 선택), `displayStyle`(`chips` vs 라디오/체크박스), `filterable` 옵션을 조합하여 세 가지 이상의 렌더링 형태를 지원한다. 내부에 필터 문자열 상태를 보유한다.

## 의존성

### 외부 패키지
- `react` — `useState` 훅 가져오기
- `@a2ui/web_core/v0_9/basic_catalog` — `ChoicePickerApi` 스키마 가져오기

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation` 유틸리티
- [`../utils`](../utils.ts.md) — `useBasicCatalogStyles` 훅
- `./ChoicePicker.module.css` — CSS 모듈 스타일

## Exports

| 이름 | 종류 |
|------|------|
| `ChoicePicker` | 상수 (React 컴포넌트, `createComponentImplementation` 반환값) |

## 상세 명세

### 타입 별칭 (비공개)

#### `_Option`
`type _Option = any` — 옵션 항목 타입. `ChoicePickerApi` 스키마의 깊이 중첩된 옵션 타입을 `z.infer`가 올바르게 추론하지 못하는 문제로 인해 임시로 `any`를 사용한다고 주석에 명시되어 있다.

### `ChoicePicker`

**시그니처**: `createComponentImplementation(ChoicePickerApi, ({props, context}) => JSX.Element)`

**내부 상태**:
- `filter: string` — 현재 필터 입력 문자열. 초기값 `''`. `setFilter`로 갱신.

**렌더 함수 동작 로직**:

1. `useBasicCatalogStyles()` 호출.
2. `values`: `props.value`가 배열이면 그대로, 아니면 `[]`.
3. `isMutuallyExclusive`: `props.variant === 'mutuallyExclusive'` 여부.

#### `onToggle(val: string): void`
선택 토글 핸들러:
- `isMutuallyExclusive`이면 `props.setValue([val])` — 단일 선택으로 대체.
- 아니면 `values`에 `val`이 이미 포함되어 있으면 제거(`filter`), 없으면 추가(`...values, val`)한 새 배열을 `props.setValue`에 전달.

#### 옵션 필터링 (`options`)
`props.options || []`를 순회하여, `props.filterable`이 `false`이거나 `filter`가 `''`이거나 `opt.label`을 lowercase로 변환했을 때 `filter.toLowerCase()`를 포함하는 항목만 유지한다.

#### `listClasses: string`
`styles.options`에 `props.displayStyle === 'chips'`이면 `styles.chips`를 추가한 클래스 문자열.

**렌더 트리**:
- 최외곽 `<div className={styles.host}>`
  - `props.label`이 있으면 `<strong className={styles.label}>{props.label}</strong>`
  - `props.filterable`이 있으면 `<input type="text" placeholder="Filter options..." value={filter} onChange={e => setFilter(e.target.value)} className={styles.filterInput} />`
  - `<div className={listClasses}>`: 필터링된 `options` 배열을 `map` 순회
    - `displayStyle === 'chips'`이면 각 항목을 `<button>` 으로 렌더링:
      - `onClick`: `() => onToggle(opt.value)`
      - `className`: `styles.chip` + (선택됨이면 `styles.selected`)
      - `aria-pressed`: `isSelected`
      - 내용: `opt.label`
    - 그 외이면 `<label className={styles.optionLabel}>` 로 렌더링:
      - `<input type={isMutuallyExclusive ? 'radio' : 'checkbox'} checked={isSelected} onChange={() => onToggle(opt.value)} name={isMutuallyExclusive ? 'choice-${context.componentModel.id}' : undefined} />`
      - `<span className={styles.optionText}>{opt.label}</span>`

**props**:
- `props.value` (`string[] | undefined`) — 현재 선택된 값 배열
- `props.setValue` (`(vals: string[]) => void`) — 외부 상태 업데이트 콜백
- `props.variant` (`'mutuallyExclusive' | string | undefined`) — 단일/다중 선택 모드
- `props.displayStyle` (`'chips' | string | undefined`) — 렌더링 형태
- `props.options` (`Array<{label: string, value: string}> | undefined`) — 선택 가능한 항목 목록
- `props.filterable` (`boolean | undefined`) — 필터 입력 표시 여부
- `props.label` (`string | undefined`) — 그룹 레이블

**context**:
- `context.componentModel.id` — radio 버튼 그룹의 `name` 속성에 사용

## 동작 흐름

컴포넌트 마운트 시 `filter` 상태가 `''`으로 초기화된다. 렌더마다 `options`를 필터 상태에 따라 동적으로 계산하고, `displayStyle`과 `isMutuallyExclusive`에 따라 버튼 또는 라디오/체크박스 목록을 렌더링한다. 사용자의 선택 변경은 `onToggle` → `props.setValue` 경로로 상위에 전파되고, 필터 입력은 로컬 `filter` 상태만 변경한다.
