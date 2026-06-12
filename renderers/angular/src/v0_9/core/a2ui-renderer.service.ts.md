# renderers/angular/src/v0_9/core/a2ui-renderer.service.ts

## 개요

Angular 렌더러의 중앙 진입점 서비스다. A2UI 프로토콜 메시지를 처리하고 `SurfaceGroupModel`을 최신 상태로 유지하는 `MessageProcessor`를 관리한다. `RendererConfiguration` 주입 토큰을 통해 카탈로그와 선택적 액션 핸들러를 전달받으며, Angular `OnDestroy` 생명주기에서 모델을 dispose한다.

## 의존성

### 외부 패키지
- `@angular/core`: `Injectable`, `OnDestroy`, `InjectionToken`, `Inject`
- `@a2ui/web_core/v0_9`: `MessageProcessor`, `SurfaceGroupModel`, `ActionListener`(as `ActionHandler`), `A2uiMessage`, `A2uiClientAction`(as `Action`)

### 저장소 내부 모듈
- [`../catalog/types`](../catalog/types.ts.md) — `AngularComponentImplementation`, `AngularCatalog`

## Exports

| 이름 | 종류 |
|------|------|
| `RendererConfiguration` | 인터페이스 |
| `A2UI_RENDERER_CONFIG` | `InjectionToken<RendererConfiguration>` 상수 |
| `A2uiRendererService` | Angular Injectable 클래스 |

## 상세 명세

### `RendererConfiguration`

인터페이스.

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `catalogs` | `AngularCatalog[]` | 필수 | 사용 가능한 컴포넌트와 함수를 담은 카탈로그 배열 |
| `actionHandler` | `(action: Action) => void` | 선택 | 어떤 서피스에서든 컴포넌트가 액션을 발생시킬 때 호출되는 콜백 |

---

### `A2UI_RENDERER_CONFIG`

`new InjectionToken<RendererConfiguration>('A2UI_RENDERER_CONFIG')`로 생성된 Angular 주입 토큰. `A2uiRendererService`를 제공할 때 반드시 함께 제공해야 한다.

---

### `A2uiRendererService`

**데코레이터**: `@Injectable()` (providedIn 미지정 — 명시적 providers 등록 필요)  
**구현 인터페이스**: `OnDestroy`

#### 필드

| 이름 | 접근 | 타입 | 설명 |
|------|------|------|------|
| `_messageProcessor` | private | `MessageProcessor<AngularComponentImplementation>` | 메시지 처리 및 모델 관리 |
| `_catalogs` | private | `AngularCatalog[]` | 주입된 카탈로그 목록 |
| `config` | private (생성자 매개변수) | `RendererConfiguration` | `@Inject(A2UI_RENDERER_CONFIG)`로 주입 |

#### `constructor(@Inject(A2UI_RENDERER_CONFIG) config: RendererConfiguration)`

1. `this._catalogs`에 `config.catalogs`를 할당한다.
2. `new MessageProcessor<AngularComponentImplementation>(this._catalogs, this.config.actionHandler as ActionHandler)`를 생성하여 `this._messageProcessor`에 저장한다.

#### `processMessages(messages: A2uiMessage[]): void`

`this._messageProcessor.processMessages(messages)`를 그대로 위임한다. 에이전트 또는 오케스트레이터로부터 새 메시지가 도착할 때마다 호출해야 한다.

#### `get surfaceGroup(): SurfaceGroupModel<AngularComponentImplementation>`

`this._messageProcessor.model`을 반환한다. 읽기 전용 getter이며 현재 활성 서피스 모델 전체에 접근할 수 있다.

#### `ngOnDestroy(): void`

`this._messageProcessor.model.dispose()`를 호출하여 모든 서피스 모델 리소스를 해제한다.

## 동작 흐름

1. Angular DI가 `A2UI_RENDERER_CONFIG` 토큰을 통해 `RendererConfiguration`을 주입하면 생성자에서 `MessageProcessor`를 즉시 초기화한다.
2. 외부 호출자는 `processMessages()`를 통해 A2UI 메시지를 주입하고, `MessageProcessor`는 이를 파싱하여 `SurfaceGroupModel`을 업데이트한다.
3. 뷰 계층(`ComponentHostComponent` 등)은 `surfaceGroup` getter로 최신 서피스 모델을 읽는다.
4. 컴포넌트가 소멸될 때 `ngOnDestroy`가 모델의 리소스를 정리한다.
