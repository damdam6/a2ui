# renderers/angular/src/v0_8/components/divider.ts

## 개요

`Divider`는 수평 구분선(`<hr>`)을 렌더링하는 단순 Angular 컴포넌트다. 추가 입력이나 로직 없이 `DynamicComponent`를 상속하고 테마의 `Divider` 클래스와 추가 스타일을 적용한다. 레이아웃 내 시각적 구분 용도로 사용된다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `Component`

### 저장소 내부 모듈
- [`../types`](../types.ts.md): `DividerNode` 타입
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md): `DynamicComponent` 기반 클래스

## Exports

| 이름 | 종류 |
|------|------|
| `Divider` | 클래스 (Angular 컴포넌트) |

## 상세 명세

### `Divider` 클래스

**선언:** `export class Divider extends DynamicComponent<DividerNode>`

**데코레이터 설정:**
- `selector`: `'a2ui-divider'`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `template`: `'<hr [class]="theme.components.Divider" [style]="theme.additionalStyles?.Divider"/>'`

**인라인 스타일:**
- `:host`: `display: block; min-height: 0; overflow: auto;`
- `hr`: `height: 1px; background: #ccc; border: none;`

클래스 본문에 추가 필드, 메서드, 입력 신호는 없다. 모든 동작은 `DynamicComponent` 상속을 통해 제공된다.

## 동작 흐름

컴포넌트가 DOM에 마운트되면 단일 `<hr>` 요소를 렌더링하며, `theme.components.Divider`에 설정된 클래스와 `theme.additionalStyles?.Divider`의 인라인 스타일을 적용한다. 자체 입력 처리나 이벤트 처리는 없다.
