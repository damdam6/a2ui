# renderers/react/src/v0_8/types.ts

## 개요

A2UI React 렌더러 v0_8의 공유 타입 정의 파일이다. 외부 패키지(`@a2ui/web_core`)의 타입들을 한 곳에서 재내보내어 소비자가 단일 진입점을 통해 접근할 수 있게 한다. 또한 React 렌더러 고유의 타입(`A2UIComponentProps`, `ComponentLoader`, `ComponentRegistration`, `OnActionCallback`, `A2UIProviderConfig`)을 정의한다.

## 의존성

### 외부 패키지

- `react` (타입 전용) — `ComponentType`
- `@a2ui/web_core/types/types` (타입 전용) — `Types` 네임스페이스 전체
- `@a2ui/web_core/types/primitives` (타입 전용) — `Primitives` 네임스페이스 전체

### 저장소 내부 모듈

없음.

## Exports

### 재내보내기 (Re-exports)

| 이름 | 원본 | 설명 |
|------|------|------|
| `Types` | `@a2ui/web_core/types/types` | 타입 네임스페이스 전체 |
| `Primitives` | `@a2ui/web_core/types/primitives` | 프리미티브 타입 네임스페이스 전체 |
| `AnyComponentNode` | `Types.AnyComponentNode` | 모든 컴포넌트 노드의 유니온 타입 |
| `Surface` | `Types.Surface` | 서피스 데이터 타입 |
| `SurfaceID` | `Types.SurfaceID` | 서피스 식별자 타입 |
| `Theme` | `Types.Theme` | 테마 구성 타입 |
| `ServerToClientMessage` | `Types.ServerToClientMessage` | 서버→클라이언트 메시지 타입 |
| `A2UIClientEventMessage` | `Types.A2UIClientEventMessage` | 클라이언트 이벤트 메시지 타입 |
| `Action` | `Types.Action` | 액션 타입 |
| `DataValue` | `Types.DataValue` | 데이터 값 타입 |
| `MessageProcessor` | `Types.MessageProcessor` | 메시지 프로세서 타입 |
| `StringValue` | `Primitives.StringValue` | 문자열 값 프리미티브 타입 |
| `NumberValue` | `Primitives.NumberValue` | 숫자 값 프리미티브 타입 |
| `BooleanValue` | `Primitives.BooleanValue` | 불리언 값 프리미티브 타입 |

### 자체 정의 타입

| 이름 | 종류 | 설명 |
|------|------|------|
| `A2UIComponentProps` | 제네릭 인터페이스 | A2UI React 컴포넌트가 공통으로 받는 props |
| `ComponentLoader` | 제네릭 타입 별칭 | 컴포넌트를 비동기로 로드하는 함수 타입 |
| `ComponentRegistration` | 제네릭 인터페이스 | 레지스트리에 등록되는 컴포넌트 항목 |
| `OnActionCallback` | 타입 별칭 | 사용자 액션 발생 시 호출되는 콜백 타입 |
| `A2UIProviderConfig` | 인터페이스 | A2UI Provider 설정 옵션 |

## 상세 명세

### 인터페이스 `A2UIComponentProps<T extends Types.AnyComponentNode = Types.AnyComponentNode>`

| 필드 | 타입 | 설명 |
|------|------|------|
| `node` | `T` | A2UI 메시지 프로세서가 해석한 컴포넌트 노드 |
| `surfaceId` | `string` | 이 컴포넌트가 속한 서피스의 식별자 |

제네릭 `T`는 `Types.AnyComponentNode`를 기본값으로 하며, 특정 컴포넌트 노드 타입으로 좁힐 수 있다.

### 타입 별칭 `ComponentLoader<T extends Types.AnyComponentNode = Types.AnyComponentNode>`

`() => Promise<{ default: ComponentType<A2UIComponentProps<T>> }>` 형태의 함수 타입. React의 동적 import 패턴(`() => import('./Foo')`)과 일치하여 lazy loading을 지원한다.

### 인터페이스 `ComponentRegistration<T extends Types.AnyComponentNode = Types.AnyComponentNode>`

| 필드 | 타입 | 필수 여부 | 설명 |
|------|------|-----------|------|
| `component` | `ComponentType<A2UIComponentProps<T>> \| ComponentLoader<T>` | 필수 | 직접 로드된 컴포넌트 또는 lazy loader 함수 |
| `lazy` | `boolean` | 선택 | `true`이면 `component`를 lazy loader로 처리 |

### 타입 별칭 `OnActionCallback`

`(message: Types.A2UIClientEventMessage) => void | Promise<void>` — 버튼 클릭 등 사용자 액션 발생 시 호출되는 콜백. 동기 및 비동기 처리 모두 허용한다.

### 인터페이스 `A2UIProviderConfig`

| 필드 | 타입 | 필수 여부 | 설명 |
|------|------|-----------|------|
| `onAction` | `OnActionCallback` | 선택 | 사용자 액션 디스패치 시 호출되는 콜백 |
| `theme` | `Types.Theme` | 선택 | 초기 테마 설정 |

## 동작 흐름

이 파일은 런타임 코드를 포함하지 않는 순수 타입 모듈이다. 모든 선언은 타입 레벨에서만 존재하며, 컴파일 이후 JavaScript 출력에 영향을 주지 않는다. 렌더러 내 다른 모듈들이 이 파일을 통해 공통 타입을 일관되게 참조하게 한다.
