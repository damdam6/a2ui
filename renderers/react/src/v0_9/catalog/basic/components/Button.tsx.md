# renderers/react/src/v0_9/catalog/basic/components/Button.tsx

## 개요

`Button` 컴포넌트는 a2ui Basic Catalog의 버튼 UI 요소다. `ButtonApi` 스키마를 기반으로 `createComponentImplementation`을 통해 생성되며, `variant` prop에 따라 CSS 모듈 클래스를 조합하여 스타일 변형을 적용한다. 자식 컴포넌트를 `buildChild`를 통해 렌더링하며, `isValid === false`일 때 버튼을 비활성화한다.

## 의존성

### 외부 패키지
- `@a2ui/web_core/v0_9/basic_catalog` — `ButtonApi` 스키마 가져오기

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation` 유틸리티
- [`../utils`](../utils.ts.md) — `useBasicCatalogStyles` 훅
- `./Button.module.css` — 버튼 CSS 모듈 스타일

## Exports

| 이름 | 종류 |
|------|------|
| `Button` | 상수 (React 컴포넌트, `createComponentImplementation` 반환값) |

## 상세 명세

### `Button`

**시그니처**: `createComponentImplementation(ButtonApi, ({props, buildChild}) => JSX.Element)`

**렌더 함수 동작 로직**:

1. `useBasicCatalogStyles()`를 호출하여 전역 스타일을 주입한다.
2. 클래스 배열 `classes`를 `[styles.button]`으로 초기화한다.
3. `props.variant`를 검사한다:
   - `'primary'`이면 `styles.primary`를 추가한다.
   - `'borderless'`이면 `styles.borderless`를 추가한다.
   - 그 외 값이거나 undefined이면 추가 클래스 없음 (기본 스타일).
4. `<button>` 엘리먼트를 렌더링한다:
   - `className`: `classes.join(' ')` — 조합된 클래스 문자열
   - `onClick`: `props.action` — 클릭 이벤트 핸들러
   - `disabled`: `props.isValid === false` — 엄격한 `=== false` 비교이므로 `undefined`일 때는 비활성화되지 않음
   - 자식: `props.child`가 존재하면 `buildChild(props.child)`를 호출하여 렌더링, 없으면 `null`

**props**:
- `props.variant` (`'primary' | 'borderless' | string | undefined`) — 버튼 스타일 변형
- `props.action` (`() => void | undefined`) — 클릭 핸들러
- `props.isValid` (`boolean | undefined`) — `false`이면 버튼 비활성화
- `props.child` (`ComponentId | undefined`) — 내부에 렌더링할 자식 컴포넌트 ID

## 동작 흐름

props를 받아 variant에 따라 CSS 클래스를 조합하고, 단일 `<button>` 엘리먼트를 반환한다. 내부 상태 없음. `isValid`가 명시적으로 `false`일 때만 `disabled` 속성이 활성화되어 사용자 조작을 차단한다.
