# renderers/angular/src/v0_8/rendering/theming.ts

## 개요

v0.8 Angular 렌더러에서 테마 상태를 관리하는 `Theme` 서비스를 정의한다. 애플리케이션 루트에 싱글톤으로 제공되며, 컴포넌트들이 테마 속성(컴포넌트별 CSS 클래스, HTML 요소별 설정, 마크다운 스타일링 등)에 접근할 수 있는 공유 저장소 역할을 한다. `DynamicComponent`가 이 서비스를 주입받아 테마 데이터를 참조한다.

## 의존성

### 외부 패키지

- `@angular/core` — `Injectable`

### 저장소 내부 모듈

- [`../types`](../types.ts.md) — `Theme as ThemeType` (타입 전용 임포트)

## Exports

| 이름 | 종류 |
|---|---|
| `Theme` | 클래스 (Angular `@Injectable`) |

## 상세 명세

### `Theme` 클래스

데코레이터: `@Injectable({ providedIn: 'root' })`

애플리케이션 루트 수준에서 자동 제공되는 싱글톤 서비스다.

#### 인스턴스 필드

- `components: ThemeType['components']` — 초기값 `{} as ThemeType['components']`. 컴포넌트 타입 이름을 키로 하는 CSS 클래스 매핑 객체. 예: `{ Text: { all: { 'a2ui-text': true } } }`.
- `elements: ThemeType['elements']` — 초기값 `{} as ThemeType['elements']`. HTML 요소별 테마 설정.
- `markdown: ThemeType['markdown']` — 마크다운 렌더링에 사용되는 CSS 클래스 매핑 객체. 초기값은 모든 키를 빈 배열로 명시:
  - `p: []`, `h1: []`, `h2: []`, `h3: []`, `h4: []`, `h5: []`
  - `ul: []`, `ol: []`, `li: []`
  - `a: []`, `strong: []`, `em: []`
- `additionalStyles?: ThemeType['additionalStyles']` — 선택적 추가 스타일 문자열. 기본값 `undefined`.

#### `update(theme: ThemeType): void`

시그니처: `update(theme: ThemeType): void`

동작 단계:
1. `this.components = theme.components` — 새 컴포넌트 테마로 교체.
2. `this.elements = theme.elements` — 새 요소 테마로 교체.
3. `this.markdown = theme.markdown` — 새 마크다운 스타일로 교체.
4. `this.additionalStyles = theme.additionalStyles` — 추가 스타일로 교체 (`undefined`일 수 있음).

전체 교체 방식이며, 부분 업데이트(merge)는 지원하지 않는다. 서피스에서 서버로부터 테마 데이터를 수신하면 이 메서드를 호출하여 서비스 상태를 갱신한다.

## 동작 흐름

`Theme` 서비스는 싱글톤으로 앱 전체에서 하나의 인스턴스가 유지된다. 렌더러 초기화 시 또는 서버로부터 테마 업데이트 메시지가 도착하면 `update(theme)`를 호출하여 테마를 설정한다. 각 `DynamicComponent` 하위 클래스는 생성자에서 `inject(Theme)`로 이 서비스를 주입받아 `theme.components`, `theme.elements`, `theme.markdown` 등에서 필요한 스타일 정보를 읽는다.
