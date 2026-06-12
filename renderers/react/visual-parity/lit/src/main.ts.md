# renderers/react/visual-parity/lit/src/main.ts

## 개요

Lit 렌더러 기반 시각적 동등성 테스트 페이지의 진입점이다. URL 쿼리 파라미터(`fixture`, `theme`)를 읽어 해당 픽스처를 A2UI 메시지로 변환한 뒤 Lit 커스텀 엘리먼트(`themed-a2ui-surface`)를 통해 화면에 렌더링한다. `@lit/context`가 `<slot>`을 통해 전파되지 않는 문제를 우회하기 위해, 테마 컨텍스트를 `a2ui-surface`를 직접 자식으로 렌더링하는 래퍼 컴포넌트(`ThemedA2UISurface`)에서 제공한다.

## 의존성

### 외부 패키지

- `@a2ui/lit` — `v0_8` (메시지 프로세서, 타입)
- `@a2ui/lit/ui` — `UI` (컨텍스트 키: `UI.Context.themeContext`, `UI.Context.markdown`)
- `lit` — `LitElement`, `html`
- `lit/decorators.js` — `customElement`, `property`
- `@lit/context` — `provide`
- `@a2ui/markdown-it` — `renderMarkdown`

### 저장소 내부 모듈

- [`../../fixtures`](../../fixtures/index.ts) — `allFixtures`, `FixtureName`, `ComponentFixture` (타입)
- [`../../fixtures/themes`](../../fixtures/themes/index.ts.md) — `getTheme`, `themeNames`, `ThemeName` (타입)

## Exports

이 파일은 ES 모듈 진입점(entry point)이며 외부로 내보내는 심볼이 없다. 브라우저에서 직접 로드되어 `init()` 함수를 즉시 실행한다.

## 상세 명세

### `ThemedA2UISurface` (클래스, `@customElement('themed-a2ui-surface')`)

`LitElement`를 상속한 Lit 커스텀 엘리먼트. `@lit/context`의 `provide` 데코레이터를 사용하여 하위 트리에 테마와 마크다운 렌더러 컨텍스트를 공급한다.

#### 필드

| 필드 | 타입 | 데코레이터 | 기본값 | 설명 |
|---|---|---|---|---|
| `theme` | `v0_8.Types.Theme \| undefined` | `@provide({context: UI.Context.themeContext})`, `@property({attribute: false})` | `undefined` | 하위 트리에 제공할 테마 객체 |
| `markdownRenderer` | `v0_8.Types.MarkdownRenderer` | `@provide({context: UI.Context.markdown})` | `renderMarkdown` | 하위 트리에 제공할 마크다운 렌더러 |
| `surfaceId` | `string` | `@property({attribute: false})` | `''` | `a2ui-surface`에 전달할 서피스 ID |
| `surface` | `any` | `@property({attribute: false})` | `undefined` | 프로세서에서 가져온 서피스 데이터 |
| `processor` | `any` | `@property({attribute: false})` | `undefined` | A2UI 메시지 프로세서 인스턴스 |

#### `render()` 메서드

`html` 태그드 템플릿을 반환하며, `<a2ui-surface>`를 직접 자식으로 렌더링하고 `surfaceId`, `surface`, `processor`를 프로퍼티 바인딩으로 전달한다. 컨텍스트가 `<slot>`을 통해 전파되지 않는 Lit 제약을 우회하는 핵심 패턴이다.

---

### `toValueMap(key: string, value: unknown): v0_8.Types.ValueMap`

픽스처 `data` 객체의 값 하나를 A2UI `ValueMap` 메시지 형식으로 변환하는 헬퍼 함수.

- `boolean` → `{key, valueBoolean: value}`
- `string` → `{key, valueString: value}`
- `number` → `{key, valueNumber: value}`
- 비-null 객체 → `{key, valueMap: Object.entries(value).map(([k, v]) => toValueMap(k, v))}` (재귀)
- 그 외(null 포함) → `{key, valueString: String(value)}`

---

### `dataToMessages(data: Record<string, unknown>, surfaceId: string): v0_8.Types.ServerToClientMessage[]`

픽스처의 `data` 맵 전체를 `dataModelUpdate` 메시지 배열로 변환한다. 동일 부모 경로의 키들을 하나의 메시지로 묶어 덮어쓰기를 방지하는 것이 핵심이다.

동작 단계:
1. `byParentPath: Map<string, ValueMap[]>` 를 생성한다.
2. 각 JSON Pointer 경로(`path`)에 대해 마지막 `/` 위치를 찾아 분리한다.
   - `lastSlash > 0`이면 부모 경로 = `path.substring(0, lastSlash)`, 키 = `path.substring(lastSlash + 1)`
   - `lastSlash === 0`이면 부모 경로 = `'/'`, 키 = `path.substring(1)`
3. `toValueMap(key, value)`로 `ValueMap`을 생성하여 해당 부모 경로의 배열에 추가한다.
4. 각 부모 경로마다 `{dataModelUpdate: {surfaceId, path: parentPath, contents}}` 형태의 메시지를 생성하여 반환한다.

---

### `fixtureToMessages(fixture: ComponentFixture, surfaceId: string): v0_8.Types.ServerToClientMessage[]`

`ComponentFixture` 하나를 완전한 A2UI 서버→클라이언트 메시지 시퀀스로 변환한다.

동작 단계:
1. `fixture.data`가 존재하면 `dataToMessages()`로 초기 데이터 메시지들을 먼저 추가한다.
2. `surfaceUpdate` 메시지를 추가한다: `{surfaceUpdate: {surfaceId, components: [{id, component}, ...]}}`. `fixture.components`를 그대로 매핑한다.
3. `beginRendering` 메시지를 추가한다: `{beginRendering: {root: fixture.root, surfaceId}}`.
4. 위 메시지 배열을 반환한다.

---

### `init()` (함수)

페이지 초기화 진입 함수. 모듈 최하단에서 즉시 호출된다.

동작 단계:
1. `document.getElementById('app')` 로 루트 DOM 요소를 취득한다.
2. `URLSearchParams`로 `fixture`와 `theme` 쿼리 파라미터를 파싱한다.
3. `themeName`이 없으면 `'lit'`을 기본값으로 사용하여 `getTheme()`을 호출한다.
4. `themeName`이 있으면 `document.documentElement`에 `data-theme` 속성을 설정한다(디버깅용).
5. 콘솔에 사용 가능한 테마 목록과 현재 선택된 테마를 로그 출력한다.
6. `fixtureName`이 없거나 `allFixtures`에 존재하지 않으면, 픽스처 목록과 테마 목록을 나열하는 HTML을 `app.innerHTML`에 삽입하고 함수를 종료한다. 각 링크는 `?fixture=<name>&theme=<effectiveTheme>` 형식을 사용한다.
7. 픽스처가 유효하면 `v0_8.Data.A2uiMessageProcessor` 인스턴스를 생성하고 `fixtureToMessages()`로 변환한 메시지들을 `processor.processMessages(messages)`로 처리한다.
8. `processor.getSurfaces().get(surfaceId)`로 서피스 데이터를 가져온다. 실패하면 오류 메시지를 표시하고 종료.
9. `<div class="fixture-container">` 요소를 생성하고 `data-fixture`, `data-theme` 속성을 설정한다.
10. `<themed-a2ui-surface>` 엘리먼트를 생성하고 `theme`, `surfaceId`, `surface`, `processor` 프로퍼티를 설정한다.
11. 컨테이너에 `themedSurface`를 추가하고, `app`에 컨테이너를 추가한다.

## 동작 흐름

```
브라우저 로드
  └→ init() 즉시 호출
       ├→ URL 파라미터 파싱 (fixture, theme)
       ├→ [fixture 미지정] → 목록 HTML 렌더링 후 종료
       └→ [fixture 지정]
            ├→ fixtureToMessages() → 메시지 배열 생성
            │    ├→ dataToMessages() (data 있을 시)
            │    ├→ surfaceUpdate 메시지
            │    └→ beginRendering 메시지
            ├→ A2uiMessageProcessor.processMessages()
            ├→ getSurfaces().get(surfaceId)
            └→ DOM 조립
                 ├→ div.fixture-container 생성
                 └→ themed-a2ui-surface 생성 및 프로퍼티 설정
                      └→ (LitElement render) → <a2ui-surface> 렌더링
```
