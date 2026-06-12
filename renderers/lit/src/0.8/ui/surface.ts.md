# renderers/lit/src/0.8/ui/surface.ts

## 개요

`Surface`는 A2UI의 최상위 렌더링 진입점 커스텀 엘리먼트(`a2ui-surface`)다. `Surface` 데이터 객체를 받아 로고, 색상 팔레트, 폰트 등 서피스 스타일을 CSS 커스텀 프로퍼티로 변환하고, 내부에 `<a2ui-root>`를 생성해 컴포넌트 트리 전체를 렌더링한다. `primaryColor` 하나로부터 `color-mix()`를 이용해 `--p-0`부터 `--p-100`까지 17단계 색상 팔레트를 프로그래밍 방식으로 생성하는 로직을 포함한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/style-map.js` — `styleMap`
- `@a2ui/web_core/types/types` — `Types.SurfaceID`, `Types.Surface`
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor`

### 저장소 내부 모듈
- [`./root.js`](./root.ts.md) — `Root` (기반 클래스)

## Exports

| 이름 | 종류 |
|---|---|
| `Surface` | 클래스 (`@customElement('a2ui-surface')`, `Root` 상속) |

## 상세 명세

### 클래스: `Surface`

`Root`를 상속하며 `@customElement('a2ui-surface')`로 등록된다.

#### 필드 및 프로퍼티

| 이름 | 타입 | 기본값 | 데코레이터 | 설명 |
|---|---|---|---|---|
| `surfaceId` | `Types.SurfaceID \| null` | `null` | 서피스 식별자. `Root`의 동명 프로퍼티를 재선언. `@property()` |
| `surface` | `Types.Surface \| null` | `null` | 렌더링할 서피스 데이터(스타일, 컴포넌트 트리 포함). `@property()` |
| `processor` | `A2uiMessageProcessor \| null` | `null` | 데이터 바인딩 처리기. `Root`의 동명 프로퍼티를 재선언. `@property()` |
| `enableCustomElements` | `boolean` | `false` | 커스텀 엘리먼트 오버라이드 허용 여부. `@property()` |

#### `static styles`

`structuralStyles`를 포함하지 않는 인라인 전용 CSS:
- `:host { display: flex; min-height: 0; max-height: 100%; flex-direction: column; gap: 16px; }`
- `#surface-logo { display: flex; justify-content: center; }`
- `#surface-logo img { width: 50%; max-width: 220px; }`
- `a2ui-root { flex: 1; }`

---

#### `private #renderLogo(): TemplateResult | typeof nothing`

`this.surface?.styles.logoUrl`이 없으면(falsy) `nothing`을 즉시 반환한다. 존재하면 `<div id="surface-logo"><img src=${logoUrl} /></div>` 구조를 반환한다.

---

#### `private #renderSurface(): TemplateResult`

`this.surface?.styles`의 키-값 쌍을 순회하며 `styles: Record<string, string>` 맵을 구성한 뒤 `<a2ui-root>`를 렌더링한다.

스타일 키 처리 (`switch` 분기):

**`'primaryColor'` 케이스**: 하나의 색 값(`value`)으로 17개 CSS 변수를 생성한다.
- `--p-100` = `'#ffffff'`
- `--p-99` = `color-mix(in srgb, ${value} 2%, white 98%)`
- `--p-98` = `color-mix(in srgb, ${value} 4%, white 96%)`
- `--p-95` = `color-mix(in srgb, ${value} 10%, white 90%)`
- `--p-90` = `color-mix(in srgb, ${value} 20%, white 80%)`
- `--p-80` = `color-mix(in srgb, ${value} 40%, white 60%)`
- `--p-70` = `color-mix(in srgb, ${value} 60%, white 40%)`
- `--p-60` = `color-mix(in srgb, ${value} 80%, white 20%)`
- `--p-50` = `value` (원색 그대로)
- `--p-40` = `color-mix(in srgb, ${value} 80%, black 20%)`
- `--p-35` = `color-mix(in srgb, ${value} 70%, black 30%)`
- `--p-30` = `color-mix(in srgb, ${value} 60%, black 40%)`
- `--p-25` = `color-mix(in srgb, ${value} 50%, black 50%)`
- `--p-20` = `color-mix(in srgb, ${value} 40%, black 60%)`
- `--p-15` = `color-mix(in srgb, ${value} 30%, black 70%)`
- `--p-10` = `color-mix(in srgb, ${value} 20%, black 80%)`
- `--p-5` = `color-mix(in srgb, ${value} 10%, black 90%)`
- `--0` = `'#00000'` (주의: 원본 코드에 5자리 hex 값으로 기록됨)

색상 범위 설계: 0(검정) ~ 50(원색) ~ 100(흰색)이지만, 0→50 구간과 50→100 구간에 각각 절반의 범위를 배분하므로 혼합 비율이 두 배씩 증가한다.

**`'font'` 케이스**: `--font-family`와 `--font-family-flex`를 동일 `value`로 설정한다.

**기타 키**: 현재 처리 로직 없음 (switch 내 매칭 case 없음).

완성된 `styles` 맵을 `styleMap(styles)`로 `<a2ui-root>`의 `style` 속성에 적용한다. `.surfaceId`, `.processor`, `.childComponents`(= `surface.componentTree`가 있으면 `[surface.componentTree]`, 없으면 `null`), `.enableCustomElements`를 `<a2ui-root>`에 전달한다.

---

#### `render(): TemplateResult | typeof nothing`

`this.surface`가 `null`이면 `nothing`을 반환한다. 있으면 `html\`${[this.#renderLogo(), this.#renderSurface()]}\`` 패턴으로 로고와 서피스를 순서대로 렌더링한다.

## 동작 흐름

외부에서 `surface` 프로퍼티가 설정되면 `render()`가 실행된다. `#renderLogo()`가 로고 URL이 있을 때 상단 이미지를 표시하고, `#renderSurface()`가 서피스 스타일을 CSS 커스텀 프로퍼티로 변환하여 `<a2ui-root>`의 인라인 스타일로 주입한다. `<a2ui-root>`는 `surface.componentTree`를 단일 루트 노드로 받아 전체 하위 컴포넌트 트리를 렌더링한다. `--p-*` 변수는 `<a2ui-root>` 스코프 내 모든 하위 컴포넌트에서 참조 가능하다.
