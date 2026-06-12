# renderers/lit/a2ui_explorer/tests/v0_9/35_markdown-text.test.ts

## 개요

`35_markdown-text.json` 예제 파일을 로드하여 Lit 렌더러가 마크다운 텍스트를 올바르게 렌더링하는지 검증하는 브라우저 통합 테스트 파일이다. 렌더링 결과에서 텍스트 콘텐츠와 HTML 태그(특히 `h1`) 존재 여부를 확인한다. `describe` 블록 내에 두 개의 테스트 케이스가 있으며, 각 테스트 전후로 픽스처를 설정·정리한다.

## 의존성

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep` 유틸리티 함수
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 커스텀 엘리먼트 클래스

### 외부 패키지
- 없음 (테스트 프레임워크는 환경에 전역 주입된 `describe`, `it`, `beforeEach`, `afterEach`, `expect` 사용)

## Exports

없음 (테스트 파일이므로 export 없음)

## 동작 흐름

`describe('Example: Markdown Text Support', ...)` 블록:

1. `gallery: LocalGallery`와 `surface: HTMLElement`, `textContent: string` 변수를 블록 스코프에서 선언.
2. **`beforeEach`**: `loadExample('35_markdown-text.json')`으로 예제를 비동기 로드하고, `getSurface(gallery)`로 렌더 표면 엘리먼트를 얻고, `getDeepTextContent(surface)`로 전체 텍스트를 추출해 `textContent`에 저장.
3. **`afterEach`**: `gallery?.remove()`로 DOM에서 갤러리 제거(정리).

### 테스트 케이스

| 테스트 이름 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `'should render text content'` | `textContent`에 문자열 `'Markdown Rendering'`이 포함되는지 확인 (`toContain`) | `beforeEach`에서 로드된 `gallery`, `surface`, `textContent` |
| `'should render markdown HTML tags'` | `querySelectorAllDeep(surface, 'h1')[0]`이 truthy(즉, `h1` 태그가 Shadow DOM 포함 렌더링되어 있음)인지 확인 | `beforeEach`에서 로드된 `gallery`, `surface` |
