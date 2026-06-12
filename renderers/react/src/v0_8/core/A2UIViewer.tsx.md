# renderers/react/src/v0_8/core/A2UIViewer.tsx

## 개요

정적 JSON 컴포넌트 정의와 데이터 모델을 props로 받아 A2UI 컴포넌트 트리를 렌더링하는 고수준 래퍼 컴포넌트 파일이다. 스트리밍 서버 메시지 방식이 아닌 정적 prop 기반 사용 시나리오를 위해 설계되었다. 내부적으로 `A2UIProvider`와 `A2UIRenderer`를 조합하고, JS 객체를 A2UI `ValueMap[]` 형식으로 변환하는 헬퍼 함수를 포함한다. `'use client'` 지시어가 선두에 선언되어 있어 Next.js 등 RSC 환경에서 클라이언트 컴포넌트로 처리된다.

## 의존성

### 외부 패키지
- `react` — `useId`, `useMemo`, `useEffect`, `useRef`
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 전체 (타입 전용)

### 저장소 내부 모듈
- [`./A2UIProvider`](./A2UIProvider.tsx.md) — `A2UIProvider`, `useA2UIActions`
- [`./A2UIRenderer`](./A2UIRenderer.tsx.md) — `A2UIRenderer`
- `../theme/litTheme` — `litTheme` 기본 테마 객체
- [`../types`](../types.ts.md) — `OnActionCallback`

## Exports

| 이름 | 종류 |
|------|------|
| `ComponentInstance` | 인터페이스 |
| `A2UIActionEvent` | 인터페이스 |
| `A2UIViewerProps` | 인터페이스 |
| `A2UIViewer` | React 함수 컴포넌트 |
| `default` | `A2UIViewer`의 default re-export |

## 상세 명세

### `ComponentInstance` 인터페이스

정적 A2UI 컴포넌트 정의의 단일 항목 형태.

| 필드 | 타입 | 설명 |
|------|------|------|
| `id` | `string` | 컴포넌트 고유 ID |
| `component` | `Record<string, unknown>` | 컴포넌트 타입과 속성을 담은 객체 |

### `A2UIActionEvent` 인터페이스

사용자 상호작용 시 `onAction` 콜백에 전달되는 이벤트 형태.

| 필드 | 타입 | 설명 |
|------|------|------|
| `actionName` | `string` | 액션 이름 |
| `sourceComponentId` | `string` | 액션을 발생시킨 컴포넌트 ID |
| `timestamp` | `string` | ISO 8601 타임스탬프 |
| `context` | `Record<string, unknown>` | 액션 컨텍스트 데이터 |

### `A2UIViewerProps` 인터페이스

| 필드 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `root` | `string` | 필수 | 루트 컴포넌트 ID |
| `components` | `ComponentInstance[]` | 필수 | 컴포넌트 정의 배열 |
| `data` | `Record<string, unknown>` | `{}` | 서피스 데이터 모델 |
| `onAction` | `(action: A2UIActionEvent) => void` | 없음 | 액션 발생 시 콜백 |
| `theme` | `Types.Theme` | `litTheme` | 커스텀 테마 |
| `className` | `string` | 없음 | 추가 CSS 클래스 |

### `A2UIViewer` 컴포넌트

**시그니처:** `function A2UIViewer({root, components, data, onAction, theme, className}: A2UIViewerProps): JSX.Element`

**동작 단계:**
1. `useId()`로 `baseId`를 생성한다.
2. `surfaceId`를 `useMemo`로 계산한다. `${root}-${JSON.stringify(components)}`를 문자열로 합쳐 해시를 계산한다(djb2 유사 알고리즘: `hash = 31 * hash + charCodeAt(i)`). 최종 surfaceId는 `surface${baseId.replace(/:/g, '-')}${hash}` 형식이다. 의존성: `[baseId, root, components]`.
3. `handleAction`을 `useMemo`로 생성한다. `onAction`이 없으면 `undefined`를 반환한다. 있으면 `Types.A2UIClientEventMessage`를 받아 `message.userAction`이 존재할 때 `onAction({actionName, sourceComponentId, timestamp, context})`를 호출하는 `OnActionCallback`을 반환한다. 의존성: `[onAction]`.
4. `<A2UIProvider onAction={handleAction} theme={theme}>`로 감싸고 내부에 `<A2UIViewerInner>` 렌더링.

### `A2UIViewerInner` 컴포넌트 (내부, 비공개)

**시그니처:** `function A2UIViewerInner({surfaceId, root, components, data, className}: {...}): JSX.Element`

**동작 단계:**
1. `useA2UIActions()`로 `processMessages`를 얻는다.
2. `lastProcessedRef`를 `useRef<string>('')`로 초기화한다(중복 처리 방지 키 저장).
3. `useEffect`에서: `${surfaceId}-${JSON.stringify(components)}-${JSON.stringify(data)}`를 키로 생성한다. 키가 `lastProcessedRef.current`와 동일하면 즉시 반환(no-op). 다르면 키를 업데이트하고 다음 메시지 배열을 구성해 `processMessages(messages)` 호출:
   - `{beginRendering: {surfaceId, root, styles: {}}}` — 서피스 시작 선언
   - `{surfaceUpdate: {surfaceId, components}}` — 컴포넌트 트리 업데이트
   - `data`가 있고 키가 1개 이상이면: `objectToValueMaps(data)`로 변환 후 `{dataModelUpdate: {surfaceId, path: '/', contents}}` 추가
   - 의존성: `[processMessages, surfaceId, root, components, data]`
4. `className`이 적용된 `div` 안에 `<A2UIRenderer surfaceId={surfaceId} />`를 렌더링한다.

### `objectToValueMaps(obj: Record<string, unknown>): Types.ValueMap[]` (내부 헬퍼)

`Object.entries(obj)`를 순회하며 각 `[key, value]` 쌍에 대해 `valueToValueMap(key, value)`를 호출하고 결과 배열을 반환한다.

### `valueToValueMap(key: string, value: unknown): Types.ValueMap` (내부 헬퍼)

단일 key-value 쌍을 `Types.ValueMap` 형태로 변환한다. 타입별 처리:

| `value` 타입 | 반환 형태 |
|-------------|----------|
| `string` | `{key, valueString: value}` |
| `number` | `{key, valueNumber: value}` |
| `boolean` | `{key, valueBoolean: value}` |
| `null` 또는 `undefined` | `{key}` |
| 배열 | `{key, valueMap: value.map((item, index) => valueToValueMap(String(index), item))}` — 재귀 |
| 객체 | `{key, valueMap: objectToValueMaps(value)}` — 재귀 |
| 그 외 | `{key}` |

## 동작 흐름

```
A2UIViewer props 변경
  → surfaceId 재계산 (root/components 변경 시)
  → handleAction 재생성 (onAction 변경 시)
  → A2UIProvider로 컨텍스트 제공

A2UIViewerInner 마운트/업데이트
  → props 변경 키 비교 (lastProcessedRef)
  → 변경 있을 때만 processMessages() 호출
    → beginRendering + surfaceUpdate [+ dataModelUpdate]
  → A2UIRenderer가 서피스 구독 및 렌더링
```
