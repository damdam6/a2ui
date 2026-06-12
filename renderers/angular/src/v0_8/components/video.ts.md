# renderers/angular/src/v0_8/components/video.ts

## 개요

비디오 미디어를 렌더링하는 Angular standalone 컴포넌트다. `url` 입력이 유효한 경우에만 `<section>` 컨테이너와 `controls` 속성이 있는 `<video>` 요소를 렌더링하며, URL이 `null`이면 DOM을 완전히 비운다. 테마 클래스와 추가 스타일을 컨테이너에 바인딩한다.

## 의존성

### 외부 패키지
- `@angular/core` — `ChangeDetectionStrategy`, `Component`, `computed`, `input`

### 저장소 내부 모듈
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md) — 기반 클래스 `DynamicComponent`
- [`../types`](../types.ts.md) — 타입 `StringValue`, `VideoNode`

## Exports

| 이름 | 종류 |
|---|---|
| `Video` | Angular 컴포넌트 클래스 |

## 상세 명세

### `Video` 클래스

`DynamicComponent<VideoNode>`를 상속하는 Angular 컴포넌트.

**데코레이터 메타데이터**
- `selector`: `'a2ui-video'`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `template`:
  - `@let resolvedUrl = this.resolvedUrl()`으로 계산 신호 값을 로컬 변수에 바인딩.
  - `@if (resolvedUrl)` 블록 안에 `<section [class]="theme.components.Video" [style]="theme.additionalStyles?.Video">`와 그 안에 `<video controls [src]="resolvedUrl">`를 배치.
- `styles`:
  - `:host`: `display: block; flex: var(--weight); min-height: 0; overflow: auto`
  - `video`: `display: block; width: 100%; box-sizing: border-box`

---

#### 입력 신호(Inputs)

| 이름 | 타입 | 기본값 | 필수 |
|---|---|---|---|
| `url` | `StringValue \| null` | — | 필수(`input.required`) |

---

#### `resolvedUrl` (protected readonly computed)

`computed(() => this.resolvePrimitive(this.url()))`. `url` 신호 값을 `resolvePrimitive`에 전달하여 실제 문자열로 변환한다. 결과가 falsy이면 `@if` 블록이 렌더링을 건너뛴다.

## 동작 흐름

컴포넌트가 초기화되면 `url` 입력 신호를 감시하고, `resolvedUrl` 계산 신호를 통해 실제 URL 문자열로 변환한다. URL이 유효하면 테마 스타일이 적용된 `<section>` 안에 `<video>` 요소를 렌더링하고, URL이 `null`이거나 빈 값이면 `@if` 조건이 false가 되어 DOM에서 완전히 제거된다.
