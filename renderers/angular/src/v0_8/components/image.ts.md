# renderers/angular/src/v0_8/components/image.ts

## 개요

`Image`는 URL을 받아 `<img>` 요소를 렌더링하는 Angular 컴포넌트다. URL이 null이면 아무것도 렌더링하지 않는다. `usageHint`(예: `'Avatar'`)에 따라 테마에서 hint별 클래스를 추가로 적용하며, `@a2ui/web_core/styles`의 `Styles.merge` 유틸로 기본 클래스와 hint 클래스를 합산한다. `altText` 입력을 통해 접근성 대체 텍스트를 지정할 수 있다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `Component`, `computed`, `input`
- `@a2ui/web_core/types/primitives`: `Primitives.StringValue`
- `@a2ui/web_core/styles/index`: `Styles.merge`

### 저장소 내부 모듈
- [`../types`](../types.ts.md): `ImageNode`, `ResolvedImage` 타입
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md): `DynamicComponent` 기반 클래스

## Exports

| 이름 | 종류 |
|------|------|
| `Image` | 클래스 (Angular 컴포넌트) |

## 상세 명세

### `Image` 클래스

**선언:** `export class Image extends DynamicComponent<ImageNode>`

**데코레이터 설정:**
- `selector`: `'a2ui-image'`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**인라인 스타일:**
- `:host`: `display: block; flex: var(--weight); min-height: 0; overflow: auto;`
- `img`: `display: block; width: 100%; height: 100%; box-sizing: border-box;`

**템플릿 동작:**
- `@let resolvedUrl = this.resolvedUrl();`, `@let resolvedAltText = this.resolvedAltText();` 로 로컬 변수에 바인딩
- `@if (resolvedUrl)` 조건부로 `<section [class]="classes()" [style]="theme.additionalStyles?.Image">` 와 `<img [src]="resolvedUrl" [alt]="resolvedAltText" />` 렌더링

**입력 신호(Inputs):**

| 입력 이름 | 타입 | 기본값 |
|-----------|------|--------|
| `url` | `Primitives.StringValue \| null` | `null` |
| `usageHint` | `ResolvedImage['usageHint'] \| null` | `null` |
| `fit` | `ResolvedImage['fit'] \| null` | `null` |
| `altText` | `Primitives.StringValue \| null` | `null` |

**computed 신호:**

- `resolvedUrl` — `protected readonly`. `this.resolvePrimitive(this.url())`로 URL 문자열 추출. null이면 템플릿 `@if` 가 전체 요소를 숨긴다.
- `resolvedAltText` — `protected readonly`. `this.resolvePrimitive(this.altText()) || ''`로 alt 텍스트 추출. 값이 없으면 빈 문자열.
- `classes` — `protected`. `usageHint()` 값을 확인해 `Styles.merge(this.theme.components.Image.all, usageHint ? this.theme.components.Image[usageHint] : {})` 로 클래스 맵을 병합한다. `usageHint`가 없으면 기본 `all` 클래스만 적용된다.

## 동작 흐름

1. `url` 입력 신호가 설정되면 `resolvedUrl` computed가 문자열로 변환한다.
2. URL이 null이면 아무것도 렌더링되지 않는다.
3. URL이 있으면 `<section>`에 병합된 클래스를 적용하고, `<img>`에 src와 alt를 바인딩한다.
4. `usageHint`가 있으면 해당 hint 이름 키로 테마의 추가 클래스를 병합한다 (예: `'Avatar'` hint → `theme.components.Image.Avatar` 클래스).
