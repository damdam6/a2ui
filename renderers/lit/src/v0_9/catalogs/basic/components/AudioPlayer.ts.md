# renderers/lit/src/v0_9/catalogs/basic/components/AudioPlayer.ts

## 개요

`a2ui-audioplayer` 커스텀 엘리먼트를 정의하는 파일이다. `BasicCatalogA2uiLitElement`를 상속하며 `AudioPlayerApi` 스키마에 바인딩된다. `url` prop으로 지정된 오디오 소스와 선택적 `description` 텍스트를 렌더링하는 단순한 오디오 플레이어 컴포넌트다. 카탈로그 등록에 사용하는 `A2uiAudioPlayer` 상수도 export한다.

## 의존성

### 외부 패키지

- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`
- `@a2ui/web_core/v0_9/basic_catalog` — `AudioPlayerApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈

- [`../basic-catalog-a2ui-lit-element.js`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`

## Exports

| 이름 | 종류 |
|---|---|
| `A2uiAudioPlayerElement` | 클래스 (커스텀 엘리먼트 `a2ui-audioplayer`) |
| `A2uiAudioPlayer` | 상수 (카탈로그 등록용 객체) |

## 상세 명세

### 클래스: `A2uiAudioPlayerElement`

- **선언:** `@customElement('a2ui-audioplayer') export class A2uiAudioPlayerElement extends BasicCatalogA2uiLitElement<typeof AudioPlayerApi>`

#### 스타일 (`static styles`)

단일 CSS 블록으로 구성된다:
- `:host`: `display: flex`, `flex-direction: column`, `gap: var(--a2ui-spacing-xs, 0.25rem)`
- 배경: `var(--a2ui-audioplayer-background, transparent)` (기본 투명)
- 모서리 반경: `var(--a2ui-audioplayer-border-radius, 0)` (기본 0)
- 패딩: `var(--a2ui-audioplayer-padding, 0)` (기본 0)

#### 메서드: `createController()`

- **시그니처:** `protected createController(): A2uiController<typeof AudioPlayerApi>`
- `new A2uiController(this, AudioPlayerApi)`를 생성하여 반환한다. 부모 클래스의 `willUpdate`에서 `context` 변경 시 호출된다.

#### 메서드: `render()`

- **시그니처:** `render(): TemplateResult | typeof nothing`
- `this.controller.props`가 없으면 `nothing`을 반환한다.
- `props.description`이 truthy이면 `<p>${props.description}</p>`를 렌더링한다. 없으면 `nothing`.
- `<audio src=${props.url} controls></audio>`를 항상 렌더링한다.

### 상수: `A2uiAudioPlayer`

`{ ...AudioPlayerApi, tagName: 'a2ui-audioplayer' }` — `AudioPlayerApi`의 모든 속성에 `tagName: 'a2ui-audioplayer'`를 추가한 객체다. 카탈로그에 이 컴포넌트를 등록할 때 사용한다.

## 동작 흐름

엘리먼트가 DOM에 추가되면 `connectedCallback`(부모 클래스)에서 basic 카탈로그 CSS가 주입된다. `context`가 설정되면 `willUpdate`(부모 클래스)에서 `AudioPlayerApi` 스키마에 바인딩된 `A2uiController`가 생성된다. `render`에서는 description(선택적)과 오디오 요소를 순서대로 출력한다. props 변경 시 컨트롤러가 `requestUpdate()`를 트리거하여 자동으로 재렌더링된다.
