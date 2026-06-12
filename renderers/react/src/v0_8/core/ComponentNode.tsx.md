# renderers/react/src/v0_8/core/ComponentNode.tsx

## 개요

A2UI 컴포넌트 노드를 타입에 따라 동적으로 렌더링하는 단일 책임 컴포넌트 파일이다. 레지스트리에서 `node.type`에 해당하는 React 컴포넌트를 조회하여 렌더링하며, 지연 로딩(lazy load) 컴포넌트를 `React.Suspense`로 감싸 처리한다. `React.memo`로 메모이제이션되어 있으며, 래퍼 `div`를 추가하지 않아 Lit 렌더러의 `:host` 요소 구조와 DOM 계층이 동일하게 유지된다.

## 의존성

### 외부 패키지
- `react` — `Suspense`, `useMemo`, `memo`
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 전체 (타입 전용)

### 저장소 내부 모듈
- [`../registry/ComponentRegistry`](../registry/ComponentRegistry.ts.md) — `ComponentRegistry` 클래스

## Exports

| 이름 | 종류 |
|------|------|
| `ComponentNode` | React 컴포넌트 (memo, named export) |
| `default` | `ComponentNode`의 default re-export |

## 상세 명세

### `LoadingFallback` (내부 상수)

`memo`로 감싼 React 함수 컴포넌트. `className="a2ui-loading"`, `padding: '8px'`, `opacity: 0.5` 스타일의 `div` 안에 `"Loading..."` 텍스트를 렌더링한다. `Suspense`의 폴백으로 사용된다.

### `ComponentNodeProps` 인터페이스 (내부)

| 필드 | 타입 | 설명 |
|------|------|------|
| `node` | `Types.AnyComponentNode \| null \| undefined` | 렌더링할 컴포넌트 노드 (안전을 위해 null/undefined 허용) |
| `surfaceId` | `string` | 이 컴포넌트가 속한 서피스 ID |
| `registry` | `ComponentRegistry` (선택) | 커스텀 레지스트리; 미제공 시 싱글톤 사용 |

### `ComponentNode` 컴포넌트

**시그니처:** `memo(function ComponentNode({node, surfaceId, registry}: ComponentNodeProps): JSX.Element | null)`

**동작 단계:**
1. `registry`가 제공되면 그것을, 아니면 `ComponentRegistry.getInstance()`(싱글톤)를 `actualRegistry`로 설정한다.
2. `nodeType`을 계산한다. `node`가 존재하고, `typeof node === 'object'`이며, `'type' in node`가 참이면 `node.type`을 사용한다. 그렇지 않으면 `null`.
3. `useMemo`로 `Component`를 조회한다: `nodeType`이 있으면 `actualRegistry.get(nodeType)`, 없으면 `null`. 의존성: `[actualRegistry, nodeType]`. (Hooks 규칙 준수를 위해 조건에 무관하게 항상 호출한다.)
4. `nodeType`이 `null`인 경우:
   - `node`가 존재하면 `console.warn('[A2UI] Invalid component node (not resolved?):', node)` 경고 출력.
   - `null`을 반환한다.
5. `Component`가 `null`인 경우: `console.warn('[A2UI] Unknown component type: ${nodeType}')` 경고 출력 후 `null`을 반환한다.
6. 정상 경우: `<Suspense fallback={<LoadingFallback />}>` 안에 `<Component node={node as Types.AnyComponentNode} surfaceId={surfaceId} />`를 렌더링한다. `node`는 step 4에서 유효성이 검증되었으므로 타입 단언이 안전하다.

## 동작 흐름

```
node 수신
  → nodeType 추출 (null이면 경고 후 null 반환)
  → 레지스트리에서 Component 조회 (없으면 경고 후 null 반환)
  → Suspense로 감싸 Component 렌더링
    → lazy 컴포넌트이면 로딩 중 LoadingFallback 표시
    → 로드 완료 후 실제 컴포넌트 렌더링
```
