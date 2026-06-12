# renderers/angular/a2ui_explorer/src/app/custom-slider.component.ts

## 개요

`CustomSliderComponent`는 기본 카탈로그에 포함되지 않는 커스텀 컴포넌트로, A2UI 렌더러가 외부에서 정의된 컴포넌트 타입을 처리할 수 있는지 검증하는 용도로 사용된다. `CatalogComponent` 기반 클래스를 상속하여 `label`, `value`, `min`, `max` 네 가지 props를 통해 HTML `<input type="range">`를 렌더링하며, 값 변경 시 `onUpdate`를 통해 데이터 모델에 반영한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`, `ChangeDetectionStrategy`
- `@angular/common` — `CommonModule`
- `zod` — `z`
- `@a2ui/web_core/v0_9` — `ComponentApi`

### 저장소 내부 모듈
- [`src/v0_9/core/catalog_component`](../../../../../src/v0_9/core/catalog_component.ts) — `CatalogComponent`

## Exports

| 이름 | 종류 |
|---|---|
| `CustomSliderComponent` | 클래스 (Angular 컴포넌트) |
| `customSliderComponentDeclaration` | 상수 (객체) |

## 상세 명세

### 모듈 레벨 상수

#### `customSliderApi`
타입: `ComponentApi` (satisfies 키워드로 검사됨)

- `name: 'CustomSlider'` — A2UI 카탈로그에서 이 컴포넌트를 식별하는 이름
- `schema`: `zod` 스키마로 정의된 props 명세. 네 가지 선택적 필드를 갖는다:
  - `label: z.string().optional()`
  - `value: z.number().optional()`
  - `min: z.number().optional()`
  - `max: z.number().optional()`
  - 전체 스키마는 `as any`로 캐스팅되어 내부 타입 불일치를 우회한다.

### 클래스 `CustomSliderComponent extends CatalogComponent<typeof customSliderApi>`

**데코레이터**: `@Component`
- `selector`: `'a2ui-custom-slider'`
- `standalone: true`
- `imports`: `[CommonModule]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**템플릿 구조:**
- 최상위 `<div class="custom-slider-container">`
- `<label>` 요소: `props()['label']?.value() || 'Value'` 텍스트와 현재 `props()['value']?.value()` 값을 표시
- `<input type="range">`: `min`, `max`, `value` 속성을 각 prop의 `.value()` 또는 기본값(0 또는 100)으로 바인딩. `(input)` 이벤트로 `handleInput`을 호출

**스타일:**
- `.custom-slider-container`: `padding: 10px`, `border: 1px dashed blue`, `border-radius: 4px`
- `input`: `width: 100%`

#### 메서드

##### `handleInput(event: Event): void`
- 매개변수: `event: Event` — `<input type="range">`의 input 이벤트
- 반환 타입: `void`
- 동작: `event.target`을 `HTMLInputElement`로 캐스팅하여 `value`를 `Number()`로 변환한다. `this.props()['value']?.onUpdate(val)`을 호출해 A2UI 데이터 모델에 변경 사항을 반영한다. `onUpdate`가 존재하지 않으면 옵셔널 체이닝으로 안전하게 무시된다.

### 상수 `customSliderComponentDeclaration`

```
{
  ...customSliderApi,           // name, schema
  component: CustomSliderComponent,
}
```

스프레드 연산자로 `customSliderApi`의 모든 필드를 복사하고 `component` 필드를 추가한 객체다. `DemoCatalog`의 `extraComponents` 배열에 이 객체를 전달하여 렌더러에 컴포넌트를 등록한다.

## 동작 흐름

`DemoCatalog` 생성 시 `extraComponents: [customSliderComponentDeclaration]`을 통해 이 컴포넌트가 카탈로그에 등록된다. A2UI 메시지에서 `type: 'CustomSlider'`인 컴포넌트 노드를 만나면 렌더러가 이 컴포넌트를 인스턴스화하고 `props` Signal을 통해 속성을 바인딩한다. 사용자가 슬라이더를 조작하면 `handleInput`이 `onUpdate`를 통해 데이터 모델 변경을 발행한다.
