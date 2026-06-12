# renderers/angular/src/v0_8/components/icon.ts

## 개요

`Icon`은 Google Material 아이콘(`g-icon`)을 렌더링하는 Angular 컴포넌트다. `name` 입력을 받아 camelCase 또는 TitleCase로 전달된 아이콘 이름을 snake_case로 변환한 뒤 `<span class="g-icon">` 안에 출력한다. `name`이 null이거나 빈 문자열이면 아무것도 렌더링하지 않는다. 접근성을 위해 호스트 요소에 `aria-hidden="true"`, `tabindex="-1"` 속성이 설정된다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `Component`, `computed`, `input`

### 저장소 내부 모듈
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md): `DynamicComponent` 기반 클래스
- [`../types`](../types.ts.md): `IconNode`, `StringValue` 타입

## Exports

| 이름 | 종류 |
|------|------|
| `Icon` | 클래스 (Angular 컴포넌트) |

## 상세 명세

### `Icon` 클래스

**선언:** `export class Icon extends DynamicComponent<IconNode>`

**데코레이터 설정:**
- `selector`: `'a2ui-icon'`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `host`: `{'aria-hidden': 'true', tabindex: '-1'}`

**인라인 스타일:** `:host { display: block; flex: var(--weight); min-height: 0; overflow: auto; }`

**템플릿 동작:**
- `@let resolvedName = this.resolvedName();` 로 값을 로컬 변수에 바인딩
- `@if (resolvedName)` 블록 안에서만 `<section>`과 `<span class="g-icon">` 렌더링
- `<section>`에 `[class]="theme.components.Icon"`, `[style]="theme.additionalStyles?.Icon"` 바인딩

**입력 신호(Inputs):**

| 입력 이름 | 타입 | 기본값 |
|-----------|------|--------|
| `name` | `StringValue \| null` | `null` |

**computed 신호:**

- `resolvedName` — `protected readonly`. `this.resolvePrimitive(this.name())`로 원시 문자열을 추출한 뒤, null/empty인 경우 `''`을 반환하고 값이 있으면 `this.toSnakeCase(rawName)`를 통해 snake_case로 변환해 반환한다.

**메서드:**

#### `private toSnakeCase(str: string): string`

두 단계 변환:
1. 문자열 첫 글자가 대문자이면 소문자로 변환 (TitleCase 처리): `.replace(/^[A-Z]/, letter => letter.toLowerCase())`
2. 이후 대문자 앞에 `_`를 삽입하고 소문자로 변환 (camelCase 처리): `.replace(/[A-Z]/g, letter => \`_${letter.toLowerCase()}\`)`

예시: `'arrowForward'` → `'arrow_forward'`, `'ArrowForward'` → `'arrow_forward'`

## 동작 흐름

1. `name` 입력 신호가 갱신되면 `resolvedName` computed 신호가 재계산된다.
2. `resolvePrimitive`로 `StringValue`를 문자열로 풀고, 빈 값이면 아무것도 렌더링하지 않는다.
3. 값이 있으면 `toSnakeCase`로 변환 후 `<span class="g-icon">` 내부에 텍스트 노드로 출력한다.
