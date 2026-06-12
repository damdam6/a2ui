# renderers/react/visual-parity/react/src/FixturePage.tsx

## 개요

비주얼 패리티 테스트에서 사용하는 React 컴포넌트 렌더 페이지다. URL 쿼리 파라미터 `?fixture=<name>` 을 읽어 해당 픽스처 데이터를 A2UI 메시지로 변환한 뒤 `processMessages`로 주입하고, `A2UIRenderer`를 통해 화면에 렌더링한다. 픽스처 이름이 없으면 사용 가능한 픽스처 및 테마 목록을 링크 형태로 보여준다.

## 의존성

### 외부 패키지
- `react` — `useEffect`, `useState`, `React`
- `@a2ui/react` — `useA2UI`, `A2UIRenderer`
- `@a2ui/lit/0.8` — 타입만 import (`Types`)

### 저장소 내부 모듈
- [`../../fixtures`](../../fixtures/index.ts.md) — `allFixtures`, `FixtureName`, `ComponentFixture`

## Exports

- `FixturePage` (함수 컴포넌트, named export)

## 상세 명세

### `toValueMap(key: string, value: unknown): Types.ValueMap`

단일 키-값 쌍을 `Types.ValueMap` 객체로 변환하는 순수 함수다.

- `value`가 `boolean`이면 `{key, valueBoolean: value}` 반환
- `value`가 `string`이면 `{key, valueString: value}` 반환
- `value`가 `number`이면 `{key, valueNumber: value}` 반환
- `value`가 `null`이 아닌 객체이면 `Object.entries(value)` 각 항목에 대해 재귀 호출하여 `{key, valueMap: [...]}` 반환
- 나머지 경우에는 `String(value)`로 변환하여 `{key, valueString: ...}` 반환

### `dataToMessages(data: Record<string, unknown>, surfaceId: string): Types.ServerToClientMessage[]`

픽스처의 `data` 맵(경로 → 값)을 `dataModelUpdate` 서버 메시지 배열로 변환하는 함수다.

동작 단계:
1. `byParentPath: Map<string, Types.ValueMap[]>` 를 초기화한다.
2. `data`의 각 `[path, value]` 항목을 순회한다. `path`의 마지막 `/` 위치를 찾아 `parentPath`(마지막 `/` 앞)와 `key`(마지막 `/` 이후)를 분리한다. `/` 가 경로 맨 앞에만 있는 경우(`lastSlash === 0`) `parentPath`는 `'/'`로 설정한다.
3. 같은 `parentPath` 아래의 `ValueMap` 항목들을 배열에 누적한다.
4. `byParentPath` 각 항목마다 `{ dataModelUpdate: { surfaceId, path: parentPath, contents } }` 형태의 메시지를 생성하여 반환 배열에 추가한다.

이 묶음 처리 방식은 같은 부모 경로에 여러 `dataModelUpdate`를 개별 발송할 경우 이전 값이 덮어써지는 문제를 방지한다.

### `fixtureToMessages(fixture: ComponentFixture, surfaceId: string): Types.ServerToClientMessage[]`

`ComponentFixture` 하나를 A2UI 서버 메시지 시퀀스로 변환하는 함수다.

반환 메시지 순서:
1. `fixture.data`가 존재하면 `dataToMessages(fixture.data, surfaceId)` 결과를 앞에 추가한다.
2. `{ surfaceUpdate: { surfaceId, components: [{id, component}, ...] } }` 메시지를 추가한다.
3. `{ beginRendering: { root: fixture.root, surfaceId } }` 메시지를 마지막에 추가한다.

### `FixturePage(): JSX.Element`

URL에서 픽스처를 읽어 렌더링하는 React 함수 컴포넌트다.

내부 상태:
- `fixtureName: FixtureName | null` — URL에서 파싱된 픽스처 이름
- `ready: boolean` — 메시지 처리 완료 여부

`useA2UI()` 훅에서 `processMessages` 함수를 얻는다.

**Effect 1 (마운트 시):** `window.location.search`를 `URLSearchParams`로 파싱하여 `fixture` 파라미터 값을 `FixtureName`으로 캐스팅한 뒤, `allFixtures`에 해당 키가 존재할 때만 `setFixtureName`을 호출한다. 의존성 배열은 `[]`(마운트 1회).

**Effect 2 (`fixtureName` 변경 시):** `fixtureName`이 `null`이면 조기 반환한다. `allFixtures[fixtureName]`으로 픽스처를 가져와 `surfaceId = "fixture-" + fixtureName` 형태로 ID를 만들고, `fixtureToMessages`로 생성한 메시지 배열을 `processMessages`에 전달한 뒤 `setReady(true)`를 호출한다. 의존성 배열은 `[fixtureName, processMessages]`.

**렌더 분기:**
- `fixtureName === null`: 현재 테마(`?theme` 파라미터, 기본값 `'lit'`)를 포함한 목록 뷰를 렌더링한다. `allFixtures`의 키를 나열하는 링크(`?fixture=<name>&theme=<theme>`) 와 `['lit', 'visualParity', 'minimal']` 세 테마 링크를 표시하며 현재 테마에는 `(current)` 표시를 붙인다.
- `!ready`: `<div>Loading...</div>` 를 반환한다.
- 정상 렌더: `<div className="fixture-container" data-fixture={fixtureName}>` 안에 `<A2UIRenderer surfaceId={surfaceId} />`를 렌더링한다.

## 동작 흐름

1. 컴포넌트 마운트 → URL에서 `fixture` 파라미터 파싱 → `fixtureName` 상태 설정
2. `fixtureName` 변경 → 픽스처 데이터를 A2UI 메시지로 변환 → `processMessages` 호출 → `ready = true`
3. `ready === true` → `A2UIRenderer`가 주입된 메시지로 컴포넌트를 화면에 그림
