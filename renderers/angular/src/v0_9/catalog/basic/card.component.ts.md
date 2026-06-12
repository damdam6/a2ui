# renderers/angular/src/v0_9/catalog/basic/card.component.ts

## 개요

A2UI v0.9 Card 컴포넌트의 Angular 구현이다. 그림자와 둥근 모서리를 가진 컨테이너 `<div class="a2ui-card">`를 렌더링하고, `child` 프로퍼티로 지정된 단일 자식 컴포넌트를 `ComponentHostComponent`를 통해 내부에 표시한다. 관련 콘텐츠를 시각적으로 묶는 카드 레이아웃 역할을 한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog` — `CardApi`

### 저장소 내부 모듈
- [`../../core/component-host.component`](../../core/component-host.component.ts.md)
- [`./basic-catalog-component`](./basic-catalog-component.ts.md)

## Exports

- `CardComponent` — Angular 스탠드얼론 컴포넌트 클래스

## 상세 명세

### `CardComponent`

`BasicCatalogComponent<typeof CardApi>`를 상속하는 Angular 컴포넌트.

- 셀렉터: `a2ui-v09-card`
- 변경 감지: `ChangeDetectionStrategy.OnPush`
- 임포트: `ComponentHostComponent`

#### computed 시그널

| 이름 | 반환 타입 | 로직 |
|---|---|---|
| `child` | `Child \| undefined` | `props()['child']?.value()` |

#### 템플릿 로직

최상위 `<div class="a2ui-card">`를 렌더링한다. `@if (child())`가 truthy일 때만 `a2ui-v09-component-host`를 자식으로 렌더링하며, `[componentKey]="child()!"`, `[surfaceId]="surfaceId()"`를 전달한다.

#### CSS 변수 (컴포넌트 내부 스타일)

| 변수 | 기본값 |
|---|---|
| `--a2ui-card-padding` | `var(--a2ui-spacing-m, 16px)` |
| `--a2ui-card-border-radius` | `var(--a2ui-border-radius, 8px)` |
| `--a2ui-card-box-shadow` | `0 2px 4px rgba(0, 0, 0, 0.1)` |
| `--a2ui-card-background` | `var(--a2ui-color-surface, #fff)` |
| `--a2ui-card-border` | `var(--a2ui-border-width, 1px) solid var(--a2ui-color-border, #ccc)` |
| `--a2ui-card-margin` | `var(--a2ui-spacing-m, 16px)` |

## 동작 흐름

컴포넌트 생성 시 부모 클래스에서 basic catalog 스타일이 주입된다. `child` computed 시그널이 평가되어 자식 컴포넌트 키가 결정되고, Angular 변경 감지 시 `@if` 블록이 child 존재 여부에 따라 `ComponentHostComponent`를 렌더링하거나 제거한다.
