# renderers/angular/src/v0_9/catalog/basic/icon.component.ts

## 개요

A2UI v0.9 Icon 컴포넌트의 Angular 구현체다. Material Icons 폰트 기반의 이름 아이콘과 SVG 경로 기반의 커스텀 아이콘을 모두 지원한다. 이름 문자열은 camelCase로 입력받아 snake_case로 자동 변환하며, `ICON_NAME_OVERRIDES` 테이블에 등록된 특수 이름은 변환 전에 직접 Material Icons 코드명으로 치환된다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog` — `IconApi`

### 저장소 내부 모듈
- [`./basic-catalog-component`](./basic-catalog-component.ts.md) — `BasicCatalogComponent`

## Exports

- `IconComponent` — Angular standalone 컴포넌트 클래스

## 상세 명세

### `ICON_NAME_OVERRIDES` (모듈 스코프 상수)

타입: `Record<string, string>`

camelCase → snake_case 자동 변환으로는 올바른 Material Icons 이름을 얻을 수 없는 예외 케이스를 위한 정적 매핑 테이블이다. 값은 다음과 같다:
- `'play'` → `'play_arrow'`
- `'rewind'` → `'fast_rewind'`
- `'favoriteOff'` → `'favorite_border'`
- `'starOff'` → `'star_border'`

---

### `IconComponent`

`BasicCatalogComponent<typeof IconApi>`를 상속하는 Angular standalone 컴포넌트.

**데코레이터 설정:**
- `selector`: `'a2ui-v09-icon'`
- `standalone: true`
- `imports`: `[]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**스타일 (컴포넌트 캡슐화):**

`.a2ui-icon`은 `display: inline-block`, `width/height/font-size: var(--a2ui-icon-size, 24px)`, `font-family: var(--a2ui-icon-font-family, 'Material Icons')`, `color: var(--a2ui-icon-color, var(--a2ui-text-color-text, var(--a2ui-color-on-background, #333)))`, `font-variation-settings: var(--a2ui-icon-font-variation-settings, 'FILL' 1)`, `line-height: 1`, `vertical-align: middle`. 폰트 렌더링 최적화를 위해 `-webkit-font-smoothing: antialiased`, `text-rendering: optimizeLegibility`, `-moz-osx-font-smoothing: grayscale`, `font-feature-settings: 'liga'`가 설정된다. `.a2ui-icon.svg`에는 `fill: currentColor`가 추가된다.

**computed 시그널:**

- `iconNameRaw` (`readonly`):
  `props()['name']?.value()`. 타입은 문자열, `{ svgPath: string }` 객체, 또는 `undefined`가 될 수 있다.

- `isSvgPath` (`readonly`, 반환 타입 `boolean`):
  `iconNameRaw()`가 `null`이 아닌 객체이고 `'svgPath'` 키를 가지면 `true`, 그렇지 않으면 `false`를 반환한다. `typeof name === 'object' && name !== null && 'svgPath' in name` 조건으로 판별.

- `svgPath` (`readonly`, 반환 타입 `string`):
  `iconNameRaw()`가 `null`이 아닌 객체이고 `'svgPath'` 키를 가지면 `name.svgPath` 문자열을 반환한다. 그렇지 않으면 `''`를 반환한다.

- `iconName` (`readonly`, 반환 타입 `string`):
  이름 기반 아이콘의 최종 Material Icons 코드명을 계산하는 3단계 로직:
  1. `iconNameRaw()`가 `string` 타입이 아니면 `''`를 반환한다.
  2. `ICON_NAME_OVERRIDES[name]`에 값이 있으면 그 오버라이드 값을 반환한다.
  3. 그 외에는 정규식 `name.replace(/[A-Z]/g, letter => \`_${letter.toLowerCase()}\`)`로 camelCase → snake_case 변환 결과를 반환한다.

**템플릿 동작:**

`@if (isSvgPath())`가 `true`이면 `<svg class="a2ui-icon svg" viewBox="0 0 24 24"><path [attr.d]="svgPath()"></path></svg>`를 렌더링한다. `false`이면 `<i class="material-icons a2ui-icon">{{ iconName() }}</i>`를 렌더링한다.

## 동작 흐름

1. `props()['name']`으로부터 raw 값을 `iconNameRaw` computed로 파생한다.
2. `isSvgPath`로 입력이 SVG 경로 객체인지 문자열 이름인지 판별한다.
3. SVG 경로인 경우: `svgPath`에서 `d` 속성 값을 추출해 `<svg>` 엘리먼트로 렌더링한다.
4. 문자열 이름인 경우: `ICON_NAME_OVERRIDES` 테이블 조회 후 매핑이 없으면 camelCase → snake_case 변환을 거쳐 Material Icons 폰트 아이콘 `<i>`로 렌더링한다.
