# renderers/angular/src/v0_8/rendering/dynamic-component.ts

## 개요

v0.8 렌더러에서 모든 동적 컴포넌트의 기반이 되는 추상 Angular 지시자(`DynamicComponent`)를 정의한다. `Renderer`가 동적으로 생성하는 컴포넌트들은 이 클래스를 상속받아야 하며, 서버 액션 디스패치와 데이터 경로 해석, 고유 ID 생성 등 공통 기능을 제공한다. 모듈 수준의 `idCounter` 변수를 통해 인스턴스별 고유 ID를 발급한다.

## 의존성

### 외부 패키지

- `@angular/core` — `Directive`, `inject`, `input`

### 저장소 내부 모듈

- [`../data`](../data/index.ts.md) — `MessageProcessor`
- [`./theming`](theming.ts.md) — `Theme`
- [`../types`](../types.ts.md) — `A2UIClientEventMessage`, `Action`, `AnyComponentNode`, `BooleanValue`, `NumberValue`, `ServerToClientMessage`, `StringValue`, `SurfaceID` (타입 전용 임포트)

## Exports

| 이름 | 종류 |
|---|---|
| `DynamicComponent<T>` | 추상 클래스 (Angular `@Directive`) |

## 상세 명세

### 모듈 레벨 변수

- `idCounter: number` — 초깃값 `0`. `getUniqueId`가 호출될 때마다 1씩 증가하는 전역 카운터. 모듈 로드 후 재설정되지 않으므로 컴포넌트 인스턴스 수명 전반에 걸쳐 단조 증가한다.

### `DynamicComponent<T extends AnyComponentNode = AnyComponentNode>` (추상 클래스)

데코레이터: `@Directive({ host: { '[style.--weight]': 'weight()' } })`

호스트 바인딩: `--weight` CSS 커스텀 속성에 `weight()` signal 값을 바인딩하여, 레이아웃 엔진이 flex weight를 CSS로 제어할 수 있게 한다.

#### 주입된 의존성

- `processor: MessageProcessor` — `inject(MessageProcessor)`로 주입, `protected readonly`. 데이터 경로 해석 및 서버 메시지 디스패치에 사용.
- `theme: Theme` — `inject(Theme)`로 주입, `protected readonly`. 테마 데이터 접근에 사용.

#### 입력 신호 (signal inputs)

- `surfaceId = input.required<SurfaceID | null>()` — 현재 컴포넌트가 속한 서피스 ID. `null`일 수 있다.
- `component = input.required<T>()` — 렌더링할 컴포넌트 노드 데이터.
- `weight = input.required<string | number>()` — 레이아웃 가중치. 호스트의 `--weight` CSS 변수에 반영된다.

#### `sendAction(action: Action): Promise<ServerToClientMessage[]>`

시그니처: `protected sendAction(action: Action): Promise<ServerToClientMessage[]>`

동작 단계:
1. `this.component()`로 현재 컴포넌트 노드를, `this.surfaceId() ?? undefined`로 서피스 ID를 얻는다.
2. 빈 `context` 객체(`Record<string, unknown>`)를 초기화한다.
3. `action.context`가 존재하면 각 항목을 순회한다:
   - `item.value.literalBoolean !== undefined`이면 `context[item.key] = item.value.literalBoolean`
   - `item.value.literalNumber !== undefined`이면 `context[item.key] = item.value.literalNumber`
   - `item.value.literalString !== undefined`이면 `context[item.key] = item.value.literalString`
   - `item.value.path`가 존재하면 `processor.resolvePath`로 경로를 해석한 뒤 `processor.getData`로 값을 조회하여 `context[item.key]`에 저장
4. `A2UIClientEventMessage` 객체를 구성한다: `userAction.name`, `userAction.sourceComponentId`(= `component.id`), `userAction.surfaceId`(= `surfaceId!`), `userAction.timestamp`(= `new Date().toISOString()`), `userAction.context`.
5. `processor.dispatch(message)`를 호출하고 그 Promise를 반환한다.

#### `resolvePrimitive` (오버로드 3개)

시그니처 (오버로드):
- `protected resolvePrimitive(value: StringValue | null): string | null`
- `protected resolvePrimitive(value: BooleanValue | null): boolean | null`
- `protected resolvePrimitive(value: NumberValue | null): number | null`

구현 시그니처: `protected resolvePrimitive(value: StringValue | BooleanValue | NumberValue | null)`

동작 단계:
1. `this.component()`, `this.surfaceId()`를 얻는다.
2. `value`가 falsy이거나 `typeof value !== 'object'`이면 `null` 반환.
3. `'literal' in value`이고 `value.literal`이 `null`/`undefined`가 아니면 `value.literal` 반환.
4. `value.path`가 존재하면 `processor.getData(component, value.path, surfaceId ?? undefined)`의 결과를 `any`로 캐스팅하여 반환.
5. `'literalString' in value`이면 `value.literalString` 반환.
6. `'literalNumber' in value`이면 `value.literalNumber` 반환.
7. `'literalBoolean' in value`이면 `value.literalBoolean` 반환.
8. 어느 분기도 해당하지 않으면 `null` 반환.

경계 케이스: 값이 `path` 방식으로 참조될 때 데이터 모델에서 런타임에 조회된다. 타입 안전성은 오버로드 선언으로 제공되지만 구현 내부에서는 `any` 캐스팅이 사용된다.

#### `getUniqueId(prefix: string): string`

시그니처: `protected getUniqueId(prefix: string): string`

`idCounter`를 후위 증가(`idCounter++`)하면서 `` `${prefix}-${idCounter++}` `` 형식의 문자열을 반환한다. 예: `prefix = 'slider'`이고 `idCounter`가 현재 `5`이면 `'slider-5'`를 반환하고 `idCounter`는 `6`이 된다.

## 동작 흐름

`DynamicComponent`는 직접 인스턴스화되지 않고, 구체 컴포넌트 클래스들이 상속하여 사용한다. `Renderer`가 동적으로 컴포넌트를 생성할 때 `setInput('surfaceId', ...)`, `setInput('component', ...)`, `setInput('weight', ...)`로 입력을 주입한다. 사용자 인터랙션이 발생하면 하위 클래스에서 `sendAction`을 호출하고, 데이터 바인딩이 필요한 속성은 `resolvePrimitive`를 통해 리터럴 또는 데이터 경로에서 값을 해석한다.
