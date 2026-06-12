# renderers/lit/src/v0_9/a2ui-lit-element.ts

## 개요

v0.9 A2UI 컴포넌트의 기반 추상 클래스 `A2uiLitElement<Api>`를 정의한다. `LitElement`를 상속하며, `context` 프로퍼티가 변경될 때마다 기존 컨트롤러를 정리하고 새 `A2uiController`를 생성하는 생명주기 관리를 담당한다. 자식 컴포넌트 렌더링을 위한 `renderNode` 헬퍼를 제공하며, 구체적인 컨트롤러 생성은 서브클래스에 위임한다.

## 의존성

### 외부 패키지

- `lit` — `LitElement`, `nothing`
- `lit/decorators.js` — `property`
- `@a2ui/web_core/v0_9` — `ComponentContext`, `ComponentApi`, `ComponentId` (타입)
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈

- [`./surface/render-a2ui-node.js`](./surface/render-a2ui-node.ts.md) — `renderA2uiNode`

## Exports

| 이름 | 종류 |
|---|---|
| `A2uiLitElement<Api>` | 추상 클래스 |

## 상세 명세

### 타입: `A2uiChildRef` (파일 내부 비공개)

`type A2uiChildRef = ComponentId | { id: ComponentId; basePath: string }`

자식 컴포넌트 참조를 나타낸다. 단순 ID 문자열이거나, ID와 데이터 컨텍스트 기준 경로 쌍을 담은 객체다.

### 추상 클래스: `A2uiLitElement<Api extends ComponentApi>`

`LitElement`를 상속하는 추상 클래스이다. 서브클래스는 `createController()`와 `render()`를 반드시 구현해야 한다.

#### 필드

| 필드 | 타입 | 접근 | 설명 |
|---|---|---|---|
| `context` | `ComponentContext` | `public`, `@property({type: Object})` | 이 컴포넌트의 A2UI 컴포넌트 컨텍스트. 변경 시 컨트롤러가 교체된다. |
| `controller` | `A2uiController<Api>` | `protected` | 현재 활성 컨트롤러. `context` 변경마다 새 인스턴스로 교체된다. |

#### 추상 메서드: `createController()`

- **시그니처:** `protected abstract createController(): A2uiController<Api>`
- 서브클래스가 반드시 구현해야 하는 팩토리 메서드. 해당 컴포넌트의 API에 바인딩된 새 `A2uiController` 인스턴스를 반환해야 한다. `willUpdate`에서 `context` 변경이 감지될 때마다 호출된다.

#### 메서드: `renderNode(childRef?: A2uiChildRef, customPath?: string)`

- **시그니처:** `protected renderNode(childRef?: A2uiChildRef, customPath?: string): TemplateResult | typeof nothing`
- **동작:**
  1. `childRef`가 없으면 `nothing`을 반환한다.
  2. `this.context.dataContext`에서 `surface`와 `path`(부모 경로 `parentPath`)를 추출한다.
  3. `surface.componentsModel.get(this.context.componentModel.id)`로 서피스에 현재 컴포넌트가 존재하는지 확인한다. 없으면 `nothing`을 반환한다. 이는 예제가 재로드되거나 서피스가 해제된 상태에서 마이크로태스크가 실행될 때 스테일 서피스 접근을 방지한다.
  4. 경로 해석 우선순위: `customPath` > `childRef.basePath` > `parentPath` 순으로 폴백한다.
     - `childRef`가 객체이면 `componentId = childRef.id`, `path = customPath ?? childRef.basePath`로 설정한다.
     - `childRef`가 문자열이면 `componentId = childRef`로 설정한다.
     - 이후에도 `path`가 없으면 `parentPath`를 사용한다.
  5. `renderA2uiNode(new ComponentContext(surface, componentId, path), surface.catalog)`를 호출해 자식 노드를 렌더링하여 반환한다.

#### 메서드: `willUpdate(changedProperties: Map<string, any>)`

- **시그니처:** `willUpdate(changedProperties: Map<string, any>): void`
- Lit 생명주기 콜백 — 업데이트 직전에 호출된다.
- `super.willUpdate(changedProperties)`를 먼저 호출한다.
- `changedProperties`에 `'context'` 키가 있고 `this.context`가 truthy이면:
  1. 기존 `this.controller`가 있으면 `this.removeController(this.controller)`로 Lit 컨트롤러 목록에서 제거하고 `this.controller.dispose()`로 바인더 자원을 해제한다.
  2. `this.createController()`를 호출해 새 컨트롤러를 생성하고 `this.controller`에 할당한다.

## 동작 흐름

서브클래스는 `createController()`와 `render()`만 구현하면 된다. `context` 변경 시 `willUpdate`가 자동으로 컨트롤러를 교체한다. `renderNode`를 통해 자식 컴포넌트를 안전하게 렌더링할 수 있으며, 서피스가 해제된 경우에도 오류 없이 처리된다.
