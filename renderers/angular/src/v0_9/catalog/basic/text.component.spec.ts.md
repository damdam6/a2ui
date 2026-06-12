# renderers/angular/src/v0_9/catalog/basic/text.component.spec.ts

## 개요

`TextComponent`의 Angular 단위 테스트 파일이다. Markdown 렌더링, 각 non-markdown variant(`h1`~`h5`, `caption`) 렌더링, 그리고 `text` 프로퍼티 누락 처리를 검증한다. `MarkdownRenderer`를 jasmine spy로 교체하여 비동기 렌더링을 제어 가능한 형태로 테스트한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/platform-browser`: `By`

### 저장소 내부 모듈
- [`./text.component`](./text.component.ts.md): 테스트 대상
- [`../../core/markdown`](../../core/markdown.ts.md): `MarkdownRenderer` (spy로 교체)
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md): `A2uiRendererService`, `A2UI_RENDERER_CONFIG` (실제 서비스, `catalogs: []` 설정)
- [`../../core/test-utils`](../../core/test-utils.ts.md): `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## 픽스처 및 모킹

### `mockMarkdownRenderer`
`jasmine.createSpyObj('MarkdownRenderer', ['render'])`. `render.and.callFake`로 `(text: string) => Promise.resolve('<p>${text}</p>')` 형태의 비동기 함수를 설정한다.

### `defaultProps`
- `text`: `createBoundProperty('Hello World')`

`surfaceId`는 `'surf1'`로 설정된다.

## 테스트 케이스

모든 Markdown 관련 테스트는 `await fixture.whenStable()` 후 `detectChanges()`를 다시 호출하여 비동기 Promise 완료를 기다린다.

### `should create`
`detectChanges()` 후 `component`가 truthy임을 확인한다.

### `should render the markdown text`
`detectChanges()` 및 `await fixture.whenStable()` 후, `.a2ui-text` 엘리먼트가 존재하고 내부 `<p>` 태그가 존재하며 텍스트가 `'Hello World'`임을 검증한다. `mockMarkdownRenderer.render`가 `'Hello World'`와 함께 호출되었음도 확인한다.

### `should handle variant h1`
`variant`를 `'h1'`으로 설정 후 `whenStable()`. `mockMarkdownRenderer.render`가 호출되지 않았음을 검증(non-markdown 경로). `.a2ui-text.h1` span 내부에 `<h1>` 엘리먼트가 있고 텍스트가 `'Heading'`임을 확인한다.

### `should handle variant caption`
`variant`를 `'caption'`으로 설정. non-markdown임을 확인하고 `.a2ui-text.caption` span 내부에 `<em>` 엘리먼트가 있으며 텍스트가 `'Caption'`임을 검증한다.

### `should handle variant h2`
`variant`를 `'h2'`로 설정. `.a2ui-text.h2` 내부에 `<h2>` 엘리먼트, 텍스트 `'Heading'`을 검증한다.

### `should handle variant h3`
`variant`를 `'h3'`로 설정. `.a2ui-text.h3` 내부에 `<h3>` 엘리먼트, 텍스트 `'Heading'`을 검증한다.

### `should handle variant h4`
`variant`를 `'h4'`로 설정. `.a2ui-text.h4` 내부에 `<h4>` 엘리먼트, 텍스트 `'Heading'`을 검증한다.

### `should handle variant h5`
`variant`를 `'h5'`로 설정. `.a2ui-text.h5` 내부에 `<h5>` 엘리먼트, 텍스트 `'Heading'`을 검증한다.

### `should handle missing text property`
`setComponentProps(fixture, {} as ComponentToProps<TextComponent>)`로 props를 비운 뒤 `whenStable()`. `mockMarkdownRenderer.render`가 빈 문자열 `''`과 함께 호출되었음을 검증한다. 이는 `text` computed의 `|| ''` 기본값 처리를 확인한다.
