# renderers/react/src/v0_9/catalog/basic/components/CheckBox.tsx

## 개요

`CheckBox` 컴포넌트는 a2ui Basic Catalog의 체크박스 입력 UI 요소다. `CheckBoxApi` 스키마를 기반으로 생성되며, 선택 상태 변경 시 `props.setValue`를 통해 외부에 값을 전달하는 제어 컴포넌트(controlled component) 방식으로 동작한다. 유효성 검사 오류가 존재할 때 에러 스타일과 첫 번째 오류 메시지를 표시하는 기능을 포함한다.

## 의존성

### 외부 패키지
- `react` — `React`, `React.CSSProperties`, `React.ChangeEvent`, `React.useId` 사용
- `@a2ui/web_core/v0_9/basic_catalog` — `CheckBoxApi` 스키마 가져오기

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation` 유틸리티
- [`../utils`](../utils.ts.md) — `useBasicCatalogStyles` 훅

## Exports

| 이름 | 종류 |
|------|------|
| `CheckBox` | 상수 (React 컴포넌트, `createComponentImplementation` 반환값) |

## 상세 명세

### `CheckBox`

**시그니처**: `createComponentImplementation(CheckBoxApi, ({props}) => JSX.Element)`

**렌더 함수 내 지역 함수 및 상태**:

#### `onChange(e: React.ChangeEvent<HTMLInputElement>): void`
체크박스 변경 이벤트 핸들러. `e.target.checked` 불리언 값을 `props.setValue`에 전달한다.

#### `uniqueId: string`
`React.useId()`로 생성된 고유 ID. `<input>`의 `id`와 `<label>`의 `htmlFor`에 연결하여 접근성(label-input 연결)을 확보한다.

#### `hasError: boolean`
`props.validationErrors`가 존재하고 길이가 0보다 큰 경우 `true`. 에러 스타일 분기 및 오류 메시지 표시에 사용된다.

**스타일 객체 목록** (모두 `React.CSSProperties`):

| 변수명 | 용도 | 주요 CSS 변수 |
|--------|------|--------------|
| `containerStyle` | 최외곽 flex 컬럼 컨테이너 | `--a2ui-checkbox-margin`, `--a2ui-spacing-m` |
| `rowStyle` | 체크박스+라벨 수평 배치 | `--a2ui-checkbox-gap`, `--a2ui-spacing-s` |
| `inputBaseStyle` | 체크박스 기본 스타일 | `--a2ui-checkbox-size (1rem)`, `--a2ui-checkbox-border`, `--a2ui-checkbox-border-radius (4px)` |
| `inputErrorStyle` | 에러 시 체크박스 추가 스타일 | `--a2ui-checkbox-color-error (red)` 의 1px solid outline |
| `labelBaseStyle` | 라벨 기본 스타일 | `--a2ui-checkbox-label-font-size`, `--a2ui-checkbox-label-font-weight (bold)` |
| `labelErrorStyle` | 에러 시 라벨 색상 | `--a2ui-checkbox-color-error (red)` |
| `errorStyle` | 오류 메시지 텍스트 스타일 | `--a2ui-font-size-xs (0.75rem)`, `--a2ui-checkbox-color-error (red)`, `marginTop: '4px'` |

**렌더 트리 구조**:
- 최외곽 `<div style={containerStyle}>`
  - `<div style={rowStyle}>` — 가로 배치
    - `<input type="checkbox" id={uniqueId} checked={!!props.value} onChange={onChange} style={inputBaseStyle + (hasError ? inputErrorStyle : {})} />`
    - `props.label`이 존재하면: `<label htmlFor={uniqueId} style={labelBaseStyle + (hasError ? labelErrorStyle : {})}>{props.label}</label>`
  - `hasError`이면: `<span style={errorStyle}>{props.validationErrors?.[0]}</span>` — 첫 번째 오류만 표시

**props**:
- `props.value` (`boolean | undefined`) — 체크 상태 (`!!` 로 강제 변환)
- `props.setValue` (`(val: boolean) => void`) — 외부 상태 업데이트 콜백
- `props.label` (`string | undefined`) — 체크박스 레이블 텍스트
- `props.validationErrors` (`string[] | undefined`) — 유효성 검사 오류 메시지 배열

## 동작 흐름

렌더 시 에러 여부를 계산하고 인라인 스타일을 구성한다. 사용자가 체크박스를 토글하면 `onChange`가 호출되어 `props.setValue`로 새 상태를 전파한다. 에러가 있을 때 입력란과 레이블에 에러 스타일을 적용하고, 오류 메시지 중 첫 번째만 하단에 표시한다.
