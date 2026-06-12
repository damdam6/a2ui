# renderers/react/src/v0_8/core/store.ts

## 개요

A2UI 컨텍스트에서 사용하는 TypeScript 인터페이스만을 정의하는 타입 선언 파일이다. 런타임 로직은 없고, `A2UIActions`와 `A2UIContextValue` 두 인터페이스를 export한다. `A2UIProvider.tsx`가 이 파일에 의존하여 컨텍스트 값의 형태를 결정한다.

## 의존성

### 외부 패키지
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 전체 (타입 전용 import)

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `OnActionCallback` 타입

## Exports

| 이름 | 종류 |
|------|------|
| `A2UIActions` | 인터페이스 |
| `A2UIContextValue` | 인터페이스 |

## 상세 명세

### `A2UIActions` 인터페이스

안정적(stable) 액션 객체의 형태를 정의한다. 이 객체는 `useRef`에 저장되어 참조가 변하지 않으며, 리렌더링을 유발하지 않는 `A2UIActionsContext`를 통해 노출된다.

| 메서드/필드 | 시그니처 | 설명 |
|-------------|---------|------|
| `processMessages` | `(messages: Types.ServerToClientMessage[]) => void` | 서버에서 수신한 메시지 배열을 처리 |
| `setData` | `(node: Types.AnyComponentNode \| null, path: string, value: Types.DataValue, surfaceId: string) => void` | 양방향 바인딩을 위해 데이터 모델에 값을 씀 |
| `dispatch` | `(message: Types.A2UIClientEventMessage) => void` | 사용자 액션을 서버로 전달 |
| `clearSurfaces` | `() => void` | 모든 서피스 삭제 |
| `getSurface` | `(surfaceId: string) => Types.Surface \| undefined` | 특정 surfaceId의 서피스 반환 |
| `getSurfaces` | `() => ReadonlyMap<string, Types.Surface>` | 전체 서피스 맵 반환 |
| `getData` | `(node: Types.AnyComponentNode, path: string, surfaceId: string) => Types.DataValue \| null` | 데이터 모델에서 값 읽기 |
| `resolvePath` | `(path: string, dataContextPath?: string) => string` | 상대 경로를 절대 경로로 변환 |

### `A2UIContextValue` 인터페이스

`A2UIActions`를 extends하며, 반응형 상태(reactive state) 필드를 추가한다.

| 필드 | 타입 | 설명 |
|------|------|------|
| `processor` | `Types.MessageProcessor` | `@a2ui/web_core`의 메시지 프로세서 인스턴스 (직접 노출하지 않음) |
| `version` | `number` | React 리렌더링 트리거를 위한 버전 카운터 |
| `onAction` | `OnActionCallback \| null` | 액션 전달 콜백 (실제로는 `dispatch`를 사용) |

## 동작 흐름

이 파일은 타입만 선언하며 실행 코드가 없다. `A2UIProvider.tsx`에서 import해 `actionsRef`와 컨텍스트 값의 타입을 지정하는 데 쓰인다.
