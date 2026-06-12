# renderers/angular/src/v0_8/components/image.spec.ts

## 개요

`Image` 컴포넌트에 대한 Angular 단위 테스트 파일이다. URL 존재 여부에 따른 `<img>` 렌더링, alt 텍스트 적용, `usageHint`에 의한 테마 클래스 분기를 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/platform-browser`: `By`

### 저장소 내부 모듈
- [`./image`](./image.ts.md): 테스트 대상 `Image` 컴포넌트
- [`../data/processor`](../data/processor.ts.md): `MessageProcessor` 서비스
- [`../rendering/theming`](../rendering/theming.ts.md): `Theme` 서비스
- [`../rendering/catalog`](../rendering/catalog.ts.md): `Catalog` 서비스

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 / 모킹 설정 (`beforeEach`)

- `mockTheme`: `new Theme()` 후 `components.Image`를 `{all: {'image-all-class': true}, Avatar: {'image-avatar-class': true}}`로 설정.
- `TestBed` 제공자: `MessageProcessor → {resolvePrimitive: (p) => p?.value || p}`, `Theme → mockTheme`, `Catalog → {}`.
- 입력값 설정: `surfaceId: 'surf-1'`, `component: {id: 'img-1', type: 'Image', weight: 1}`, `weight: 1`, `usageHint: null`.

### 테스트 케이스

| 케이스명 | 검증 동작 | 픽스처/모킹 |
|----------|-----------|-------------|
| `should create` | 컴포넌트 인스턴스가 truthy인지 확인 | 기본 설정 |
| `should render <img> if url is provided` | `url: {literalString: 'http://example.com/a.png'}` 설정 시 `<img>` 가 존재하고 `src`가 해당 URL이며 `alt`가 빈 문자열인지, `<section>` className에 `'image-all-class'`가 포함되는지 확인 | `setInput('url', ...)` |
| `should render <img> with altText if provided` | `url`과 `altText: {literalString: 'A beautiful sunset'}` 설정 시 `<img>` alt 속성이 `'A beautiful sunset'`인지 확인 | `setInput('altText', ...)` |
| `should NOT render <img> if url is null` | `usageHint: null`(url 미설정) 상태에서 `<img>` 요소가 없는지 확인 | url 입력 없음 |
| `should apply usageHint class if provided` | `url` 설정 및 `usageHint: 'Avatar'` 설정 시 `<section>` className에 `'image-avatar-class'`가 포함되는지 확인 | `setInput('usageHint', 'Avatar')` |

## 동작 흐름

각 테스트는 `setInput`으로 입력을 구성하고 `detectChanges` 후, `fixture.debugElement.query(By.css(...))`와 `.nativeElement` 속성으로 DOM 상태를 검증한다.
