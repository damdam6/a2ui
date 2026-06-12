# renderers/angular/src/v0_8/components/surface.ts

## 개요

`Surface` 컴포넌트는 a2ui 렌더링 트리의 최상위 진입점 역할을 하는 Angular 독립형 컴포넌트다. `surfaceId`로 식별되는 서피스 데이터를 외부 입력(`surface` 인풋) 또는 `MessageProcessor`의 내부 Map에서 가져와 `componentTree`의 루트 컴포넌트를 동적으로 렌더링한다. `processor.version()` 시그널을 추적하여 인플레이스 변경(in-place mutation)에도 반응적으로 업데이트한다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `Component`, `computed`, `inject`, `input`

### 저장소 내부 모듈
- [`../data`](../data/index.ts.md) — `MessageProcessor` (주입 대상)
- [`../rendering/renderer`](../rendering/renderer.ts.md) — `Renderer` 디렉티브
- [`../types`](../types.ts.md) — `Surface as SurfaceType`, `SurfaceID` 타입

## Exports

| 이름 | 종류 |
|------|------|
| `Surface` | 클래스 (Angular Component) |

## 상세 명세

### `Surface` 클래스

`@Component` 데코레이터 설정:
- `selector`: `'a2ui-surface'`
- `imports`: `[Renderer]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `styles`: `:host { display: block; width: 100%; height: 100%; }`

#### 필드

| 이름 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `processor` | `MessageProcessor` (private) | `inject(MessageProcessor)` | 서피스 데이터 조회에 사용 |
| `surfaceId` | `InputSignal<SurfaceID>` | required | 렌더링할 서피스의 식별자 |
| `surfaceInput` | `InputSignal<SurfaceType \| null>` | `null` (alias: `'surface'`) | 외부에서 직접 전달되는 서피스 데이터. 템플릿 바인딩 시 `[surface]` 입력명 사용 |

#### `surface` computed (protected)
1. `this.processor.version()`을 호출하여 인플레이스 변경 추적을 위한 의존성을 등록한다.
2. `this.surfaceInput()`이 null이 아니면 반환한다.
3. null이면 `this.processor.getSurfaces().get(this.surfaceId())`를 조회하고, 없으면 `null`을 반환한다.

#### `rootComponent` computed (protected)
1. `this.processor.version()`을 다시 호출하여 동일한 인플레이스 변경 추적을 적용한다.
2. `this.surface()?.componentTree ?? null`을 반환한다.

#### 템플릿 구조
`@if (rootComponent())` 블록: `rootComponent`가 non-null일 때만 `<ng-container a2ui-renderer [surfaceId]="surfaceId()" [component]="rootComponent()!" />`를 렌더링한다.

## 동작 흐름

1. `surfaceId`와 선택적 `surface` 입력을 받아 서피스 데이터를 결정한다.
2. `surfaceInput`이 있으면 해당 데이터를 우선 사용하고, 없으면 프로세서의 서피스 Map을 조회한다.
3. `processor.version()` 시그널로 외부에서 Map이 직접 변경되어도 Angular의 OnPush 변경 감지가 올바르게 트리거된다.
4. `rootComponent`가 결정되면 `Renderer`를 통해 전체 컴포넌트 트리를 재귀적으로 렌더링한다.
5. 서피스 데이터가 없으면 아무것도 렌더링하지 않는다.
