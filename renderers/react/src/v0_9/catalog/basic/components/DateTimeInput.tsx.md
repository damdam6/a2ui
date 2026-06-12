# renderers/react/src/v0_9/catalog/basic/components/DateTimeInput.tsx

## 개요

`DateTimeInput` 컴포넌트는 a2ui Basic Catalog의 날짜/시간 입력 UI 요소다. `DateTimeInputApi` 스키마를 기반으로 생성되며, `enableDate`와 `enableTime` prop 조합에 따라 HTML `<input>` 엘리먼트의 `type`을 `datetime-local`, `date`, `time` 중 하나로 동적으로 결정한다. 선택적 레이블과 함께 제어 컴포넌트(controlled) 방식으로 동작한다.

## 의존성

### 외부 패키지
- `react` — `React`, `React.CSSProperties`, `React.ChangeEvent`, `React.useId` 사용
- `@a2ui/web_core/v0_9/basic_catalog` — `DateTimeInputApi` 스키마 가져오기

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation` 유틸리티
- [`../utils`](../utils.ts.md) — `useBasicCatalogStyles` 훅

## Exports

| 이름 | 종류 |
|------|------|
| `DateTimeInput` | 상수 (React 컴포넌트, `createComponentImplementation` 반환값) |

## 상세 명세

### `DateTimeInput`

**시그니처**: `createComponentImplementation(DateTimeInputApi, ({props}) => JSX.Element)`

**렌더 함수 동작 로직**:

1. `useBasicCatalogStyles()` 호출.

#### `onChange(e: React.ChangeEvent<HTMLInputElement>): void`
입력 변경 핸들러. `e.target.value` 문자열을 `props.setValue`에 전달한다.

#### `uniqueId: string`
`React.useId()`로 생성된 고유 ID. `<input>` 의 `id`와 `<label>`의 `htmlFor`를 연결한다.

#### `type` 결정 로직
- 초기값: `'datetime-local'`
- `props.enableDate && !props.enableTime` → `type = 'date'`
- `!props.enableDate && props.enableTime` → `type = 'time'`
- 두 조건 모두 해당 안 되면 `'datetime-local'` 유지 (둘 다 true이거나 둘 다 false인 경우 포함)

#### `style` 객체 (`React.CSSProperties`)
- `backgroundColor: 'var(--a2ui-datetimeinput-background, var(--a2ui-color-input, #fff))'`
- `color: 'var(--a2ui-datetimeinput-color, var(--a2ui-color-on-input, #333))'`
- `border: 'var(--a2ui-datetimeinput-border, var(--a2ui-border))'`
- `borderRadius: 'var(--a2ui-datetimeinput-border-radius, var(--a2ui-border-radius))'`
- `padding: 'var(--a2ui-datetimeinput-padding, var(--a2ui-spacing-s))'`
- `boxSizing: 'border-box'`

**렌더 트리**:
- 최외곽 `<div>` — `display: 'flex'`, `flexDirection: 'column'`, `gap: 'var(--a2ui-spacing-xs, 0.25rem)'`
  - `props.label`이 있으면 `<label htmlFor={uniqueId}>` — `fontSize`, `fontWeight`에 CSS 변수 적용
  - `<input id={uniqueId} type={type} style={style} value={props.value || ''} onChange={onChange} min={typeof props.min === 'string' ? props.min : undefined} max={typeof props.max === 'string' ? props.max : undefined} />`

**props**:
- `props.value` (`string | undefined`) — 현재 입력값 (빈 문자열로 fallback)
- `props.setValue` (`(val: string) => void`) — 외부 상태 업데이트 콜백
- `props.label` (`string | undefined`) — 입력란 레이블
- `props.enableDate` (`boolean | undefined`) — 날짜 입력 활성화 여부
- `props.enableTime` (`boolean | undefined`) — 시간 입력 활성화 여부
- `props.min` (`string | undefined`) — 최솟값 (문자열인 경우에만 `min` 속성으로 전달)
- `props.max` (`string | undefined`) — 최댓값 (문자열인 경우에만 `max` 속성으로 전달)

## 동작 흐름

렌더 시 `enableDate`/`enableTime` 조합으로 입력 타입을 결정하고, `React.useId()`로 레이블-입력 연결을 확보한다. 사용자가 값을 변경하면 `onChange` → `props.setValue` 경로로 상위에 전파된다. `min`/`max`는 타입이 문자열인 경우에만 HTML 속성으로 전달하여 타입 안전성을 유지한다.
