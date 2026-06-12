# renderers/lit/src/0.8/ui/component-registry.ts

## 개요

`ComponentRegistry` 클래스는 A2UI의 커스텀 컴포넌트 등록소 역할을 한다. 타입 이름(`typeName`)과 커스텀 엘리먼트 생성자를 매핑해 저장하고, 필요 시 브라우저의 `customElements` 레지스트리에 태그명을 등록한다. 선택적으로 JSON 스키마를 함께 등록해 인라인 카탈로그 데이터를 제공할 수 있다. 모듈 수준 싱글턴 인스턴스 `componentRegistry`가 함께 내보내진다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`./ui.js`](./ui.ts.md) — `CustomElementConstructorOf` 타입

## Exports

- `ComponentRegistry` (클래스)
- `componentRegistry` (상수, `ComponentRegistry` 인스턴스)

## 상세 명세

### 클래스 `ComponentRegistry`

#### 필드

| 필드 | 타입 | 초기값 | 가시성 | 설명 |
|------|------|--------|--------|------|
| `schemas` | `Map<string, unknown>` | `new Map()` | `private` | 타입 이름 → JSON 스키마 매핑 |
| `registry` | `Map<string, CustomElementConstructorOf<HTMLElement>>` | `new Map()` | `private` | 타입 이름 → 생성자 함수 매핑 |

#### 메서드 `register(typeName, constructor, tagName?, schema?): void`

커스텀 컴포넌트를 등록한다.

매개변수:
- `typeName: string` — 영숫자만 허용되는 타입 식별자.
- `constructor: CustomElementConstructorOf<HTMLElement>` — 커스텀 엘리먼트 생성자.
- `tagName?: string` — 등록할 HTML 태그명. 생략 시 `a2ui-custom-${typeName.toLowerCase()}`로 자동 생성된다.
- `schema?: unknown` — 컴포넌트의 JSON 스키마. 제공 시 `schemas` 맵에 저장된다.

동작 로직:
1. `typeName`이 `/^[a-zA-Z0-9]+$/` 정규식에 매치하지 않으면 `Error('[Registry] Invalid typeName ...')`를 던진다.
2. `this.registry.set(typeName, constructor)`로 생성자를 저장한다.
3. `schema`가 있으면 `this.schemas.set(typeName, schema)`로 스키마를 저장한다.
4. `actualTagName`을 결정한다: `tagName`이 있으면 그대로 사용, 없으면 `a2ui-custom-${typeName.toLowerCase()}`.
5. `customElements.getName(constructor)`로 해당 생성자가 이미 등록됐는지 확인한다:
   - 이미 등록됐고 `existingName !== actualTagName`이면 충돌 에러를 던진다.
   - 이미 등록됐고 이름이 동일하면 조기 반환한다.
6. `customElements.get(actualTagName)`이 undefined인 경우에만 `customElements.define(actualTagName, constructor)`를 호출한다(중복 정의 방지).

#### 메서드 `get(typeName: string): CustomElementConstructorOf<HTMLElement> | undefined`

`typeName`에 해당하는 생성자를 `registry` 맵에서 조회해 반환한다. 없으면 `undefined`를 반환한다.

#### 메서드 `getInlineCatalog(): {components: {[key: string]: unknown}}`

`schemas` 맵의 모든 항목을 일반 객체 `{components: {...}}`로 변환해 반환한다. 타입명 키와 스키마 값 쌍으로 구성된다. 빈 스키마 맵이면 `{components: {}}`를 반환한다.

### 상수 `componentRegistry`

`new ComponentRegistry()`로 생성된 모듈 수준 싱글턴. 애플리케이션 전체에서 단일 레지스트리로 공유된다.

## 동작 흐름

애플리케이션 초기화 시 커스텀 컴포넌트를 `componentRegistry.register(...)`로 등록한다. `Root.renderComponentTree` 및 `Root.renderCustomComponent`는 `componentRegistry.get(component.type)`을 호출해 등록된 생성자를 조회하고, 있으면 해당 생성자로 인스턴스를 생성해 렌더링한다. `getInlineCatalog()`는 등록된 모든 컴포넌트의 스키마를 카탈로그 형태로 집계하는 데 사용된다.
