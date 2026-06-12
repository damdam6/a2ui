# renderers/react/src/v0_8/hooks/useA2UIComponent.ts

## 개요

개별 A2UI 컴포넌트 구현을 위한 베이스 훅 파일이다. 테마 접근, 데이터 바인딩(읽기/쓰기), `StringValue`/`NumberValue`/`BooleanValue` 해석, 액션 디스패치, 접근성을 위한 고유 ID 생성 기능을 하나의 훅 `useA2UIComponent`로 묶어 제공한다. 내부적으로 `useA2UIState()`를 구독하여 데이터 모델 변경 시 컴포넌트가 리렌더링되도록 보장한다.

## 의존성

### 외부 패키지
- `react` — `useCallback`, `useId`, `useMemo`
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 전체 (타입 전용)
- `@a2ui/web_core/types/primitives` — `Primitives` 네임스페이스 전체 (타입 전용)

### 저장소 내부 모듈
- [`../core/A2UIProvider`](../core/A2UIProvider.tsx.md) — `useA2UIActions`, `useA2UIState`
- `../theme/ThemeContext` — `useTheme`

## Exports

| 이름 | 종류 |
|------|------|
| `UseA2UIComponentResult` | 인터페이스 |
| `useA2UIComponent` | 훅 함수 |

## 상세 명세

### `UseA2UIComponentResult` 인터페이스

| 필드 | 타입 | 설명 |
|------|------|------|
| `theme` | `Types.Theme` | 현재 테마 |
| `resolveString` | `(value: Primitives.StringValue \| null \| undefined) => string \| null` | StringValue 해석 |
| `resolveNumber` | `(value: Primitives.NumberValue \| null \| undefined) => number \| null` | NumberValue 해석 |
| `resolveBoolean` | `(value: Primitives.BooleanValue \| null \| undefined) => boolean \| null` | BooleanValue 해석 |
| `setValue` | `(path: string, value: Types.DataValue) => void` | 데이터 모델에 값 쓰기 |
| `getValue` | `(path: string) => Types.DataValue \| null` | 데이터 모델에서 값 읽기 |
| `sendAction` | `(action: Types.Action) => void` | 사용자 액션 디스패치 |
| `getUniqueId` | `(prefix: string) => string` | 접근성용 고유 ID 생성 |

### `useA2UIComponent<T extends Types.AnyComponentNode>(node: T, surfaceId: string): UseA2UIComponentResult`

**제네릭:** `T extends Types.AnyComponentNode` — 구체적인 노드 타입을 유지한다.

**동작 단계:**
1. `useA2UIActions()` — 안정 액션 객체를 얻는다(리렌더링 비유발).
2. `useTheme()` — 현재 테마를 읽는다.
3. `useId()` — `baseId`를 생성한다(SSR 및 Concurrent Mode 호환).
4. `useA2UIState()` — 반환값을 사용하지 않고 호출만 한다. 데이터 모델이 `setData`로 변경될 때 이 컴포넌트가 리렌더링되도록 상태를 구독하기 위함이다. `memo()`는 컨텍스트 트리거 리렌더링을 차단하지 않는다.

**내부 콜백 (모두 `useCallback`):**

#### `resolveString(value: Primitives.StringValue | null | undefined): string | null`
의존성: `[actions, node, surfaceId]`.
1. `value`가 falsy이면 `null` 반환.
2. `typeof value !== 'object'`이면 `null` 반환.
3. `value.literalString !== undefined`이면 `value.literalString` 반환.
4. `value.literal !== undefined`이면 `String(value.literal)` 반환.
5. `value.path`가 있으면 `actions.getData(node, value.path, surfaceId)` 호출; 결과가 `null`이 아니면 `String(data)`, 아니면 `null` 반환.
6. 그 외 `null` 반환.

#### `resolveNumber(value: Primitives.NumberValue | null | undefined): number | null`
의존성: `[actions, node, surfaceId]`.
1. `value`가 falsy이면 `null`.
2. `typeof value !== 'object'`이면 `null`.
3. `value.literalNumber !== undefined`이면 `value.literalNumber`.
4. `value.literal !== undefined`이면 `Number(value.literal)`.
5. `value.path`가 있으면 `actions.getData(...)` 호출; 결과를 `Number(data)` 반환.
6. 그 외 `null`.

#### `resolveBoolean(value: Primitives.BooleanValue | null | undefined): boolean | null`
의존성: `[actions, node, surfaceId]`.
1. `value`가 falsy이면 `null`.
2. `typeof value !== 'object'`이면 `null`.
3. `value.literalBoolean !== undefined`이면 `value.literalBoolean`.
4. `value.literal !== undefined`이면 `Boolean(value.literal)`.
5. `value.path`가 있으면 `actions.getData(...)` 호출; 결과를 `Boolean(data)` 반환.
6. 그 외 `null`.

#### `setValue(path: string, value: Types.DataValue): void`
의존성: `[actions, node, surfaceId]`. `actions.setData(node, path, value, surfaceId)` 직접 위임.

#### `getValue(path: string): Types.DataValue | null`
의존성: `[actions, node, surfaceId]`. `actions.getData(node, path, surfaceId)` 직접 위임.

#### `sendAction(action: Types.Action): void`
의존성: `[actions, node, surfaceId]`.
1. 빈 `actionContext` 객체를 생성한다.
2. `action.context`가 있으면 각 `item`에 대해 우선순위 순으로 값을 해석한다:
   - `item.value.literalString !== undefined` → `actionContext[item.key] = item.value.literalString`
   - `item.value.literalNumber !== undefined` → `actionContext[item.key] = item.value.literalNumber`
   - `item.value.literalBoolean !== undefined` → `actionContext[item.key] = item.value.literalBoolean`
   - `item.value.path` → `actions.resolvePath(item.value.path, node.dataContextPath)`로 절대 경로 변환 후 `actions.getData(node, resolvedPath, surfaceId)` 값 할당
3. `actions.dispatch({userAction: {name: action.name, sourceComponentId: node.id, surfaceId, timestamp: new Date().toISOString(), context: actionContext}})` 호출.

#### `getUniqueId(prefix: string): string`
의존성: `[baseId]`. `${prefix}${baseId}`를 반환한다.

**최종 반환:** 위 8개 콜백과 `theme`를 `useMemo`로 묶어 반환한다. 의존성: `[theme, resolveString, resolveNumber, resolveBoolean, setValue, getValue, sendAction, getUniqueId]`.

## 동작 흐름

```
컴포넌트 렌더
  → useA2UIActions() — 안정 액션 (리렌더 없음)
  → useTheme() — 테마 읽기
  → useId() — 고유 ID 기반
  → useA2UIState() — 데이터 모델 변경 구독

데이터 변경 (actions.setData 호출)
  → version++ → useA2UIState() 구독자 리렌더링
  → resolveString/Number/Boolean이 최신 데이터 반환

사용자 인터랙션
  → sendAction(action)
    → context 항목 해석
    → actions.dispatch({userAction: {...}})
      → onActionRef.current(message) 호출
```
