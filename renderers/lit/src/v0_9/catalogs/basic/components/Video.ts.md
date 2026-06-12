# renderers/lit/src/v0_9/catalogs/basic/components/Video.ts

## 개요

`a2ui-video` 커스텀 엘리먼트를 정의하는 파일로, A2UI Basic Catalog의 동영상 컴포넌트를 Lit 웹 컴포넌트로 구현한다. `VideoApi`로부터 `url` 프로퍼티를 받아 HTML5 `<video>` 엘리먼트로 렌더링하며, controls 속성을 기본으로 활성화한다.

## 의존성

### 외부 패키지
- `lit`: `html`, `nothing`, `css`
- `lit/decorators.js`: `customElement`
- `@a2ui/web_core/v0_9/basic_catalog`: `VideoApi`
- `@a2ui/lit/v0_9`: `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.js`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement` 베이스 클래스 제공

## Exports

- `A2uiVideoElement` (클래스): `a2ui-video` 커스텀 엘리먼트 구현체
- `A2uiVideo` (상수 객체): `VideoApi`에 `tagName: 'a2ui-video'`를 병합한 카탈로그 등록용 API 객체

## 상세 명세

### `A2uiVideoElement` 클래스

`BasicCatalogA2uiLitElement<typeof VideoApi>`를 상속하는 Lit 커스텀 엘리먼트. `@customElement('a2ui-video')` 데코레이터로 등록된다.

#### `static styles`
`:host`는 `display: block; width: 100%`로 블록 레벨 컨테이너를 구성한다. 내부 `video` 엘리먼트는 `display: block; width: 100%; height: auto`로 설정되어 가로 방향으로 부모를 채운다. 아래 CSS 커스텀 프로퍼티를 통해 외부에서 스타일 오버라이드가 가능하다:
- `--a2ui-video-border-radius`: 비디오 모서리 반경 (기본값: `0`)

#### `protected createController(): A2uiController`
- 시그니처: `createController(): A2uiController`
- `new A2uiController(this, VideoApi)`를 생성하여 반환한다. 베이스 클래스의 추상 메서드를 구현한다.

#### `render(): TemplateResult | typeof nothing`
- 시그니처: `render(): TemplateResult | typeof nothing`
- `this.controller.props`가 없으면 `nothing`을 반환한다.
- `props.url`을 `src` 속성으로, `controls` 속성을 기본 활성화 상태로, `class="a2ui-video"`를 가진 `<video>` 엘리먼트를 렌더링한다.

### `A2uiVideo` 상수 객체

`VideoApi`의 모든 프로퍼티를 스프레드(`...VideoApi`)하고 `tagName: 'a2ui-video'`를 추가한 평범한 객체 리터럴. 카탈로그에 컴포넌트를 등록할 때 사용된다.

## 동작 흐름

`A2uiController`가 `VideoApi` 스키마에 따라 컴포넌트 모델에서 `props`를 가져온다. `render()`는 `props.url`을 비디오 소스로 사용하여 기본 브라우저 컨트롤이 있는 `<video>` 엘리먼트를 반환한다. 컴포넌트 자체는 재생 상태를 추적하지 않고 단순히 URL을 바인딩하는 역할에 집중한다.
