# renderers/angular/src/v0_9/catalog/basic/image.component.ts

## 개요

A2UI v0.9 Image 컴포넌트의 Angular 구현체다. 이미지 URL, 대체 텍스트(alt), object-fit 방식, 형태 변형(variant)을 프롭으로 받아 단일 `<img>` 엘리먼트를 렌더링한다. variant 값은 CSS 클래스명으로 사용되며, 각 클래스가 미리 정의된 크기와 모양을 제어한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog` — `ImageApi`

### 저장소 내부 모듈
- [`./basic-catalog-component`](./basic-catalog-component.ts.md) — `BasicCatalogComponent`

## Exports

- `ImageComponent` — Angular standalone 컴포넌트 클래스

## 상세 명세

### `ImageComponent`

`BasicCatalogComponent<typeof ImageApi>`를 상속하는 Angular standalone 컴포넌트.

**데코레이터 설정:**
- `selector`: `'a2ui-v09-image'`
- `standalone: true`
- `imports`: `[]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**스타일 (컴포넌트 캡슐화):**

`.a2ui-image` 기본 스타일: `display: block`, `max-width: 100%`, `height: auto`, `border-radius: var(--a2ui-image-border-radius, var(--a2ui-border-radius, 8px))`.

variant별 추가 스타일:
- `.icon`: `width: var(--a2ui-image-icon-size, 24px)`, `height: var(--a2ui-image-icon-size, 24px)`
- `.avatar`: `width: var(--a2ui-image-avatar-size, 40px)`, `height: var(--a2ui-image-avatar-size, 40px)`, `border-radius: 50%`
- `.smallFeature`: `max-width: var(--a2ui-image-small-feature-size, 100px)`
- `.largeFeature`: `max-height: var(--a2ui-image-large-feature-size, 400px)`
- `.header`: `height: var(--a2ui-image-header-size, 200px)`
- `mediumFeature`: 별도 CSS 클래스 스타일 없음(기본 `.a2ui-image` 스타일 적용)

**computed 시그널:**

- `url` (`readonly`): `props()['url']?.value()`. URL 문자열 또는 `undefined`. `<img src>`에 직접 바인딩된다.
- `description` (`readonly`): `props()['description']?.value() || ''`. alt 텍스트로 사용되며 프롭이 없거나 falsy이면 빈 문자열.
- `fit` (`readonly`): `props()['fit']?.value() || 'cover'`. `object-fit` CSS 속성값. 기본값 `'cover'`.
- `variant` (`readonly`): `props()['variant']?.value() || 'default'`. CSS 클래스명으로 사용. 기본값 `'default'`(별도 스타일 없음).

**템플릿 동작:**

`<img [src]="url()" [alt]="description()" [style.object-fit]="fit()" [class]="'a2ui-image ' + variant()">` 단일 엘리먼트를 렌더링한다. `class` 속성은 항상 `'a2ui-image '` 문자열 뒤에 variant 값을 이어붙인 형태다.

## 동작 흐름

1. `props()`에서 `url`, `description`, `fit`, `variant`를 computed 시그널로 파생한다.
2. `<img>` 엘리먼트의 `src`, `alt`, `object-fit` 인라인 스타일, CSS 클래스를 각 시그널에 바인딩한다.
3. variant 클래스가 변경되면 OnPush 변경 감지에 의해 DOM이 업데이트되어 해당 variant의 크기/모양 스타일이 적용된다.
