# renderers/react/src/v0_8/hooks/useA2UI.ts

## 개요

A2UI의 주 퍼블릭 API 훅 파일이다. `useA2UI` 훅 하나와 그 반환 타입 `UseA2UIResult`를 export한다. 서버 메시지 처리 및 서피스 접근에 필요한 메서드와 현재 버전 번호를 묶어 제공하며, 상태 변경 시 리렌더링이 발생하도록 `useA2UIState`를 구독한다.

## 의존성

### 외부 패키지
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 전체 (타입 전용)

### 저장소 내부 모듈
- [`../core/A2UIProvider`](../core/A2UIProvider.tsx.md) — `useA2UIActions`, `useA2UIState`

## Exports

| 이름 | 종류 |
|------|------|
| `UseA2UIResult` | 인터페이스 |
| `useA2UI` | 훅 함수 |

## 상세 명세

### `UseA2UIResult` 인터페이스

| 필드 | 타입 | 설명 |
|------|------|------|
| `processMessages` | `(messages: Types.ServerToClientMessage[]) => void` | 서버 메시지 처리 |
| `getSurface` | `(surfaceId: string) => Types.Surface \| undefined` | 특정 서피스 조회 |
| `getSurfaces` | `() => ReadonlyMap<string, Types.Surface>` | 전체 서피스 맵 조회 |
| `clearSurfaces` | `() => void` | 전체 서피스 삭제 |
| `version` | `number` | 상태 변경 카운터 |

### `useA2UI(): UseA2UIResult`

**동작 단계:**
1. `useA2UIActions()`를 호출하여 안정적 액션 객체(`actions`)를 얻는다.
2. `useA2UIState()`를 호출하여 반응형 상태(`state`)를 구독한다. 이로 인해 이 훅을 사용하는 컴포넌트는 `version`이 변경될 때마다 리렌더링된다.
3. 두 값을 조합하여 `UseA2UIResult` 형태의 객체를 반환한다:
   - `processMessages`: `actions.processMessages`
   - `getSurface`: `actions.getSurface`
   - `getSurfaces`: `actions.getSurfaces`
   - `clearSurfaces`: `actions.clearSurfaces`
   - `version`: `state.version`

**경계 케이스:** `A2UIProvider` 외부에서 호출 시 `useA2UIActions()` 또는 `useA2UIState()`가 에러를 throw한다.

**주의:** 액션만 필요하고 리렌더링을 원하지 않는 경우에는 `useA2UIActions()`를 직접 사용해야 한다.

## 동작 흐름

```
useA2UI() 호출
  → useA2UIActions() → 안정 액션 객체 (참조 불변)
  → useA2UIState() → {version} 구독 (변경 시 리렌더링 유발)
  → {processMessages, getSurface, getSurfaces, clearSurfaces, version} 반환
```
