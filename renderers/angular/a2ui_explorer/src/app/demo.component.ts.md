# renderers/angular/a2ui_explorer/src/app/demo.component.ts

## 개요

A2UI Angular 렌더러의 메인 대시보드 컴포넌트다. 왼쪽 사이드바에 예제 목록을 나열하고, 중앙 캔버스에 선택한 예제를 실제 Surface 컴포넌트로 렌더링하며, 오른쪽 인스펙터 패널에서 `createSurface` 메시지·DataModel·이벤트 로그를 실시간으로 열람하고 편집할 수 있다. v0.8과 v0.9 두 프로토콜 버전을 런타임에 전환할 수 있으며, standalone 컴포넌트로서 자체 `providers` 배열 안에서 서비스를 모두 등록한다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectorRef`, `Component`, `OnInit`, `inject`, `OnDestroy`, `effect`, `signal`
- `@angular/common`: `CommonModule`
- `@a2ui/angular/v0_9`: `A2uiRendererService`, `A2UI_RENDERER_CONFIG`, `SurfaceComponent` (as `SurfaceComponentV09`), `AngularCatalog`
- `@a2ui/angular/v0_8`: `provideMarkdownRenderer`, `Surface` (as `SurfaceV08`), `Catalog` (as `CatalogV08`), `DEFAULT_CATALOG` (as `DEFAULT_CATALOG_V08`)
- `@a2ui/web_core/v0_9`: `A2uiClientAction`

### 저장소 내부 모듈
- [`./agent-stub.service`](./agent-stub.service.ts.md) — `AgentStubService`
- [`./agent-stub-v08.service`](./agent-stub-v08.service.ts.md) — `AgentStubV08Service`
- [`./agent-stub-v09.service`](./agent-stub-v09.service.ts.md) — `AgentStubV09Service`
- [`./demo-catalog`](./demo-catalog.ts.md) — `DemoCatalog`
- [`./action-dispatcher.service`](./action-dispatcher.service.ts.md) — `ActionDispatcher`
- [`./types`](./types.ts.md) — `A2uiExample`, `A2UI_VERSION`, `A2UI_EXAMPLES`, `Version`

## Exports

- `DemoComponent` (클래스, Angular standalone 컴포넌트)

## 상세 명세

### `DemoComponent` 클래스

**데코레이터**: `@Component`
- `selector`: `'a2ui-v0-9-demo'`
- `standalone: true`
- `imports`: `[CommonModule, SurfaceComponentV09, SurfaceV08]`
- 인라인 `template` 및 `styles` 포함
- **providers 배열** (선언 순서대로):
  1. `A2uiRendererService` — 직접 클래스 제공
  2. `{ provide: AngularCatalog, useClass: DemoCatalog }` — `DemoCatalog`로 대체
  3. `{ provide: CatalogV08, useValue: DEFAULT_CATALOG_V08 }` — v0.8 기본 카탈로그 값 제공
  4. `provideMarkdownRenderer()` — 마크다운 렌더러 등록
  5. `ActionDispatcher` — 직접 클래스 제공
  6. `AgentStubService` 팩토리 — `A2UI_VERSION`이 `Version.V0_8`이면 `AgentStubV08Service`를, 아니면 `AgentStubV09Service`를 반환; `deps: [AgentStubV09Service, AgentStubV08Service, A2UI_VERSION]`
  7. `AgentStubV08Service` — 직접 클래스 제공
  8. `AgentStubV09Service` — 직접 클래스 제공
  9. `A2UI_RENDERER_CONFIG` 팩토리 — `AngularCatalog`와 `ActionDispatcher`를 받아 `{ catalogs: [catalog], actionHandler: (action) => dispatcher.dispatch(action) }` 객체를 반환; `deps: [AngularCatalog, ActionDispatcher]`

**implements**: `OnInit`, `OnDestroy`

#### 필드 / 상태

| 필드 | 타입 | 초기값 | 설명 |
|---|---|---|---|
| `Version` | `typeof Version` | `Version` enum | 템플릿에서 열거형을 직접 참조하기 위해 클래스에 노출 |
| `rendererService` (private) | `A2uiRendererService` | `inject(...)` | 데이터 모델 직접 접근용 |
| `agentStub` (private) | `AgentStubService` | `inject(...)` | 예제 초기화·이벤트 로그·신호 소스 |
| `cdr` (private) | `ChangeDetectorRef` | `inject(...)` | 수동 변경 감지 트리거 |
| `version` (readonly) | `Version` | `inject(A2UI_VERSION)` | 현재 활성 프로토콜 버전 |
| `examples` (readonly) | `Array<A2uiExample>` | `inject(A2UI_EXAMPLES)` | 사이드바에 표시할 예제 목록 |
| `selectedExample` | `A2uiExample \| undefined` | `undefined` | 현재 선택된 예제 |
| `surfaceId` | signal(string) | `agentStub.surfaceId` | 렌더링할 surface의 ID |
| `inspectTab` | `'data' \| 'events'` | `'data'` | 인스펙터 탭 상태 (현재 템플릿에서 미사용) |
| `currentCreateSurfaceMessageJson` | `string` | `''` | `createSurface` 메시지 텍스트에리어에 표시할 JSON 문자열 |
| `messageError` | `string \| null` | `null` | `createSurface` JSON 파싱 오류 메시지; 유효하면 `null` |
| `currentDataModelJson` | `string` | `''` | DataModel 텍스트에리어에 표시할 JSON 문자열 |
| `dataModelError` | `string \| null` | `null` | DataModel JSON 파싱 오류 메시지; 유효하면 `null` |
| `jsonInputFocused` | `WritableSignal<boolean>` | `signal(false)` | 텍스트에리어 포커스 여부; `true`이면 자동 갱신 effect가 중단됨 |
| `isDataModelFolded` | `boolean` | `false` | DataModel 패널 접힘 여부 |
| `isSurfaceMessageFolded` | `boolean` | `false` | Surface Message 패널 접힘 여부 |
| `isEventsLogFolded` | `boolean` | `false` | Events Log 패널 접힘 여부 |

#### `constructor()`

두 개의 `effect`를 등록한다.

**DataModel 동기화 effect**: `jsonInputFocused()`가 `true`이면 즉시 반환해 자동 갱신을 억제한다. 그렇지 않으면 `agentStub.dataModel()`을 구독해 `JSON.stringify(data, null, 2)`로 포맷된 값을 `currentDataModelJson`에 반영하고 `cdr.detectChanges()`를 호출한다.

**createSurface 메시지 동기화 effect**: 동일한 포커스 가드 후, `agentStub.currentCreateSurfaceMessage()`를 구독해 메시지가 있으면 `JSON.stringify(msg, null, 2)`, 없으면 빈 문자열을 `currentCreateSurfaceMessageJson`에 저장하고 `cdr.detectChanges()`를 호출한다.

#### `ngOnInit(): void`

`window`가 존재하면 `localStorage`에서 `'isDataModelFolded'`, `'isSurfaceMessageFolded'`, `'isEventsLogFolded'` 세 키를 읽어 문자열 `'true'` 비교로 각 접힘 상태를 복원한다. 이후 `selectExampleFromUrl()`을 호출한다.

#### `ngOnDestroy(): void`

빈 구현. `OnDestroy` 인터페이스 준수 목적.

#### `get eventsLog()`

`agentStub.eventsLog()` 신호의 현재 값을 반환한다.

#### `clearEventsLog(): void`

`agentStub.eventsLog.set([])`로 이벤트 로그 배열을 초기화한다.

#### `onVersionChange(event: Event): void`

`event.target`을 `HTMLSelectElement`로 캐스팅해 `select.value`를 가져온다. `window`가 존재하면 현재 URL의 `version` 쿼리 파라미터를 새 값으로 교체하고 `window.location.href`를 갱신해 페이지 전체를 재로드한다. 버전 전환 시 Angular DI 트리를 완전히 재초기화하기 위한 의도적 전체 재로드다.

#### `selectExample(example: A2uiExample): void`

1. `selectedExample`를 전달된 예제로 설정한다.
2. `window.location.hash`를 `slugify(example.name)` 결과로 갱신한다.
3. `agentStub.initializeDemo(example.messages)`를 호출해 렌더러에 메시지 시퀀스를 전달한다.
4. `cdr.detectChanges()`로 뷰를 즉시 업데이트한다.

#### `getActionType(action: A2uiClientAction): string`

`action.name`이 truthy이면 그 값을 반환하고, falsy이면 `'Action'` 문자열을 반환한다.

#### `onSurfaceMessageChange(event: Event): void`

`event.target`을 `HTMLTextAreaElement`로 캐스팅해 현재 값을 `currentCreateSurfaceMessageJson`에 저장한다. `JSON.parse`를 시도해:
- **성공**: `messageError = null`. 파싱 결과 객체에 `createSurface` 키가 없거나 `selectedExample`가 `undefined`이면 조기 반환한다. 두 조건 모두 충족되면 `selectedExample.messages`를 순회해 `'createSurface' in m`인 메시지를 파싱 결과로 교체한 `updatedMessages`를 구성하고 `agentStub.initializeDemo(updatedMessages)`를 호출한 뒤 `cdr.detectChanges()`를 실행한다.
- **실패**: `messageError`에 `e instanceof Error ? e.message : 'Invalid JSON'`을 저장하고 `console.error(e)`를 호출한다.

#### `onSurfaceMessageFocus(): void`

`jsonInputFocused.set(true)`를 호출해 자동 갱신 effect를 멈춘다.

#### `onSurfaceMessageBlur(): void`

`jsonInputFocused.set(false)`로 포커스 가드를 해제한다. 이후 `currentCreateSurfaceMessageJson`를 파싱해 성공하면 `JSON.stringify(parsed, null, 2)`로 재포맷해 저장한다. 파싱 실패 시 무시한다.

#### `onDataModelChange(event: Event): void`

`event.target`을 `HTMLTextAreaElement`로 캐스팅해 `currentDataModelJson`을 업데이트한다. `JSON.parse` 시도 후:
- **성공**: `dataModelError = null`. `rendererService.surfaceGroup?.getSurface(surfaceId())`로 해당 Surface를 찾아 `surface?.dataModel.set('/', parsed)`로 루트 데이터 모델을 교체한다.
- **실패**: `dataModelError`에 오류 메시지를 저장하고 `console.error(e)`를 호출한다.

#### `onDataModelFocus(): void`

`jsonInputFocused.set(true)`.

#### `onDataModelBlur(): void`

`jsonInputFocused.set(false)`. `currentDataModelJson`을 파싱해 성공하면 2칸 들여쓰기로 재포맷, 실패 시 무시한다.

#### `toggleDataModel(): void`

`isDataModelFolded`를 토글하고 `localStorage.setItem('isDataModelFolded', String(this.isDataModelFolded))`에 저장한다.

#### `toggleSurfaceMessage(): void`

`isSurfaceMessageFolded`를 토글하고 `localStorage.setItem('isSurfaceMessageFolded', ...)`에 저장한다.

#### `toggleEventsLog(): void`

`isEventsLogFolded`를 토글하고 `localStorage.setItem('isEventsLogFolded', ...)`에 저장한다.

#### `private slugify(text: string): string`

문자열을 소문자로 변환하고 (`toLowerCase()`), `[^a-z0-9]+` 패턴의 연속 비영숫자 구간을 `'-'`로 대체하며, 선두와 말미의 `'-'`를 제거해 URL-safe slug를 반환한다.

#### `private selectExampleFromUrl(): void`

`window.location.hash.substring(1)`로 현재 해시를 읽는다(`||` 연산자로 빈 문자열 기본값 처리). `examples`에서 `slugify(ex.name) === hash`인 첫 번째 항목을 찾고, 없으면 `examples[0]`을 기본값으로 사용한다. 예제를 찾지 못하면 즉시 반환하고, 찾으면 `selectExample(example)`을 호출한다.

## 동작 흐름

1. `ngOnInit` 시점에 `localStorage`로부터 패널 접힘 상태를 복원하고, `selectExampleFromUrl()`로 URL 해시 또는 첫 번째 예제를 자동 선택한다.
2. 예제 선택 시 `agentStub.initializeDemo()`가 메시지 시퀀스를 처리해 `surfaceId` signal을 설정하고, v0.9 또는 v0.8 Surface 컴포넌트가 캔버스에 렌더링된다.
3. 두 `effect`가 `agentStub`의 signal 변화를 감지해 인스펙터 패널의 textarea를 자동으로 갱신한다. `jsonInputFocused`가 `true`이면 갱신을 건너뛰어 사용자 편집 내용을 보호한다.
4. 사용자가 textarea를 직접 편집하면 실시간으로 렌더러 상태(Surface 메시지 또는 데이터 모델)가 업데이트된다.
5. 버전 변경은 URL 쿼리 파라미터를 교체하고 전체 페이지를 재로드해 Angular DI 트리를 완전히 재구성한다.
