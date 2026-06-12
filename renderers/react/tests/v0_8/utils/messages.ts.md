# renderers/react/tests/v0_8/utils/messages.ts

## 개요

v0.8 테스트에서 A2UI 서버→클라이언트 메시지 객체를 손쉽게 생성하기 위한 팩토리 함수 모음이다. `Types.ServerToClientMessage` 타입을 기반으로 `surfaceUpdate`, `beginRendering`, `dataModelUpdate`, `deleteSurface` 등의 메시지 구조를 올바른 형태로 조립하여 반환한다. 테스트 코드에서 메시지 구조를 직접 작성하는 반복을 제거한다.

## 의존성

### 외부 패키지
- `@a2ui/web_core/types/types`: `Types` 네임스페이스 (type-only import)

### 저장소 내부 모듈
없음.

## Exports

| 이름 | 종류 |
|---|---|
| `createSurfaceUpdate` | 함수 |
| `createBeginRendering` | 함수 |
| `createSimpleMessages` | 함수 |
| `createDataModelUpdate` | 함수 |
| `createDeleteSurface` | 함수 |
| `createDataModelUpdateSpec` | 함수 |

## 상세 명세

### `createSurfaceUpdate(components, surfaceId?): Types.ServerToClientMessage`

- **매개변수**
  - `components: Array<{id: string; component: Record<string, unknown>}>` — 컴포넌트 목록
  - `surfaceId?: string` — 기본값 `'@default'`
- **반환 타입**: `Types.ServerToClientMessage`
- **동작**: `surfaceUpdate` 키를 가진 메시지 객체를 생성한다. `surfaceId`와 `components` 배열을 담으며, 각 항목은 `{id, component}` 구조다.

### `createBeginRendering(rootId, surfaceId?): Types.ServerToClientMessage`

- **매개변수**
  - `rootId: string` — 렌더링을 시작할 루트 컴포넌트 ID
  - `surfaceId?: string` — 기본값 `'@default'`
- **반환 타입**: `Types.ServerToClientMessage`
- **동작**: `beginRendering` 키를 가진 메시지를 생성한다. `root: rootId`와 `surfaceId`를 담는다.

### `createSimpleMessages(id, componentType, props, surfaceId?): Types.ServerToClientMessage[]`

- **매개변수**
  - `id: string` — 컴포넌트 ID
  - `componentType: string` — 컴포넌트 타입 이름 (예: `'Text'`, `'TextField'`)
  - `props: Record<string, unknown>` — 컴포넌트 props 객체
  - `surfaceId?: string` — 기본값 `'@default'`
- **반환 타입**: `Types.ServerToClientMessage[]` (2개 요소 배열)
- **동작**: 1) `createSurfaceUpdate([{id, component: {[componentType]: props}}], surfaceId)`, 2) `createBeginRendering(id, surfaceId)` 순서로 두 메시지를 배열에 담아 반환한다. 단일 컴포넌트 렌더링에 필요한 최소 메시지 쌍이다.

### `createDataModelUpdate(contents, surfaceId?, path?): Types.ServerToClientMessage`

- **매개변수**
  - `contents: Array<{key: string; valueString?: string; valueNumber?: number; valueBoolean?: boolean; valueMap?: unknown[]}>` — 갱신할 데이터 항목 목록
  - `surfaceId?: string` — 기본값 `'@default'`
  - `path?: string` — 데이터 모델 경로 (선택)
- **반환 타입**: `Types.ServerToClientMessage`
- **동작**: `dataModelUpdate` 키를 가진 메시지 객체를 생성한다. A2UI 스펙에서 애플리케이션 상태를 UI 구조와 독립적으로 갱신할 때 사용한다.

### `createDeleteSurface(surfaceId): Types.ServerToClientMessage`

- **매개변수**
  - `surfaceId: string` — 삭제할 surface ID
- **반환 타입**: `Types.ServerToClientMessage`
- **동작**: `deleteSurface: {surfaceId}` 키를 가진 메시지를 반환한다. A2UI 스펙에서 UI surface와 연관 콘텐츠를 제거할 때 사용한다.

### `createDataModelUpdateSpec(contents, surfaceId?, path?): Types.ServerToClientMessage`

- **매개변수**
  - `contents: Array<{key: string; valueString?: string; valueMap?: unknown[]}>` — 갱신 항목 (valueNumber, valueBoolean 제외)
  - `surfaceId?: string` — 기본값 `'@default'`
  - `path?: string` — 기본값 `'/'`
- **반환 타입**: `Types.ServerToClientMessage`
- **동작**: `createDataModelUpdate`와 동일한 구조이나, 기본 `path`가 `'/'`이고 contents 타입이 더 제한적이다. A2UI 스펙의 JSON 직렬화 `valueString` 패턴을 따른다.

## 동작 흐름

각 함수는 순수 팩토리다. 입력을 받아 `as Types.ServerToClientMessage` 타입 단언으로 캐스팅된 리터럴 객체를 반환한다. 외부 상태나 side-effect가 없다. `createSimpleMessages`만 배열을 반환하며 내부적으로 `createSurfaceUpdate`와 `createBeginRendering`을 합성한다.
