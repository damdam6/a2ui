# renderers/react/src/v0_8/components/content/Text.tsx

## 개요

`Text` 컴포넌트는 마크다운 텍스트를 파싱하여 HTML로 렌더링하는 콘텐츠 컴포넌트다. `usageHint`에 따라 텍스트에 마크다운 헤딩 접두사나 이탤릭 서식을 자동으로 추가하고, 테마의 마크다운 설정을 반영하여 렌더링된 HTML 요소에 CSS 클래스를 적용한다. Lit 렌더러의 마크다운 디렉티브 동작을 정확히 재현한다.

## 의존성

### 외부 패키지
- `react` — `useMemo`, `memo`
- `@a2ui/web_core/types/types` — `Types.TextNode`, `Types.Theme`, `Types.StringValue` (타입 전용)
- `markdown-it` — 마크다운 파싱 및 HTML 렌더링

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md) — `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md) — `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md) — `classMapToString`, `stylesToObject`, `mergeClassMaps`

## Exports

- `Text` — named export, memo로 감싼 함수형 컴포넌트
- `default` — `Text`의 default export

## 상세 명세

### 타입 별칭 및 인터페이스

- `UsageHint`: `'h1' | 'h2' | 'h3' | 'h4' | 'h5' | 'caption' | 'body'` — 텍스트 사용 목적 리터럴 유니온 타입. 파일 내부에서만 사용된다.
- `HintedStyles`: 인터페이스. 각 UsageHint 키(`h1`, `h2`, `h3`, `h4`, `h5`, `body`, `caption`)를 선택적 필드로 가지며, 각 값은 `Record<string, string>`이다. 테마의 추가 스타일이 사용 목적별로 분리되어 있는 경우를 나타낸다.

### `isHintedStyles(styles: unknown): styles is HintedStyles`

비공개 타입 가드 함수.

**동작:** `styles`가 객체이고 `null`이 아니며 배열이 아닐 때, `['h1', 'h2', 'h3', 'h4', 'h5', 'caption', 'body']` 중 하나 이상의 키를 가지면 `true`를 반환한다. null과 배열은 `false`를 반환한다.

### `markdownRenderer: MarkdownIt` (모듈 수준 상수)

`new MarkdownIt()`으로 생성된 모듈 수준 singleton 인스턴스. 기본 설정을 사용한다:
- `html: false` (기본값) — 보안상 원시 HTML 비활성화
- `linkify: false` (기본값) — URL 자동 링크화 비활성화
- `breaks: false` (기본값) — 줄바꿈 `\n`을 `<br>`로 변환하지 않음
- `typographer: false` (기본값) — 스마트 따옴표/대시 변환 비활성화

렌더링이 동기적이므로 인스턴스의 renderer rules를 일시적으로 수정 후 복원하는 패턴이 안전하다.

### `TAG_TO_TOKEN: Record<string, string>` (모듈 수준 상수)

HTML 태그 이름을 markdown-it 토큰 이름으로 매핑하는 조회 테이블. 값:
- `'p'` → `'paragraph'`
- `'h1'` → `'heading'`
- `'h2'` → `'heading'`
- `'h3'` → `'heading'`
- `'h4'` → `'heading'`
- `'h5'` → `'heading'`
- `'h6'` → `'heading'`
- `'ul'` → `'bullet_list'`
- `'ol'` → `'ordered_list'`
- `'li'` → `'list_item'`
- `'a'` → `'link'`
- `'strong'` → `'strong'`
- `'em'` → `'em'`

### `toClassArray(classes: string[] | Record<string, boolean>): string[]`

비공개 헬퍼 함수. 테마의 클래스 맵을 문자열 배열로 정규화한다.

**동작:** 입력이 배열이면 그대로 반환한다. 객체이면 `Object.entries`로 순회하여 값이 `true`인 키만 골라 배열로 반환한다.

### `renderWithTheme(text: string, markdownTheme: Types.Theme['markdown']): string`

비공개 함수. 테마 클래스를 markdown-it renderer rules에 임시 등록하여 마크다운을 테마가 적용된 HTML로 렌더링한다.

**매개변수:**
- `text`: 렌더링할 마크다운 문자열.
- `markdownTheme`: `Types.Theme['markdown']` 타입의 테마 설정 객체.

**반환 타입:** 렌더링된 HTML 문자열.

**동작 단계:**
1. `appliedKeys: string[]` 배열을 초기화하여 이 호출에서 등록한 renderer rule 키를 추적한다.
2. `markdownTheme`을 `Record<string, string[] | Record<string, boolean>> | undefined`로 캐스팅하여 `themeMap`에 저장한다.
3. `themeMap`이 존재하면 각 `[tag, classes]` 쌍을 순회한다:
   - `classes`가 falsy이면 건너뛴다.
   - `TAG_TO_TOKEN[tag]`가 없으면 건너뛴다.
   - `key = "${tokenName}_open"` 형식으로 renderer rule 키를 구성한다. (예: `'heading_open'`)
   - `appliedKeys`에 해당 key가 아직 없으면 추가한다.
   - `markdownRenderer.renderer.rules[key]`에 클로저 함수를 등록한다. 이 함수는 렌더링 시 토큰의 `token.tag`로 테마 맵을 재조회하여 각 클래스를 `token.attrJoin('class', cls)`로 누적하고, `self.renderToken()`의 결과를 반환한다.
4. `markdownRenderer.render(text)`로 동기적으로 HTML을 생성한다.
5. `appliedKeys`에 있는 모든 renderer rule 키를 `delete`하여 모듈 수준 렌더러를 원상태로 복원한다.
6. 생성된 HTML 문자열을 반환한다.

**특이사항:** `h1`~`h6`는 모두 `TAG_TO_TOKEN`에서 `'heading'`으로 매핑되므로, 같은 `heading_open` rule이 등록된다. 하지만 클로저 내부에서 `token.tag`로 재조회하므로 헤딩 수준별로 서로 다른 클래스를 적용할 수 있다.

### `Text`

**시그니처**: `memo(function Text({ node, surfaceId }: A2UIComponentProps<Types.TextNode>): JSX.Element | null)`

**동작 단계:**
1. `useA2UIComponent(node, surfaceId)`로 `theme`과 `resolveString`을 획득한다.
2. `resolveString(props.text)`로 텍스트 값을 해석한다.
3. `props.usageHint as UsageHint | undefined`로 사용 목적 힌트를 추출한다.
4. `mergeClassMaps(theme.components.Text.all, usageHint ? theme.components.Text[usageHint] : {})`로 기본 클래스와 힌트별 클래스를 병합한다.
5. `useMemo`로 `additionalStyles`를 계산한다. 의존성: `[theme.additionalStyles?.Text, usageHint]`.
   - `theme.additionalStyles?.Text`가 없으면 `undefined`.
   - `isHintedStyles(textStyles)`가 true이면: `(usageHint ?? 'body')`에 해당하는 힌트별 스타일을 `stylesToObject`로 변환한다.
   - 그렇지 않으면: 전체 스타일을 `stylesToObject`로 변환한다.
6. `useMemo`로 `renderedContent`를 계산한다. 의존성: `[textValue, theme.markdown, usageHint]`.
   - `textValue`가 null 또는 undefined이면 `null`을 반환한다.
   - `usageHint`에 따라 `markdownText`에 접두사를 추가한다:
     - `'h1'` → `"# "` 접두사 추가
     - `'h2'` → `"## "` 접두사 추가
     - `'h3'` → `"### "` 접두사 추가
     - `'h4'` → `"#### "` 접두사 추가
     - `'h5'` → `"##### "` 접두사 추가
     - `'caption'` → `*${markdownText}*` 형태로 감싸기 (이탤릭)
     - 그 외(body 및 undefined) → 변환 없음
   - `{__html: renderWithTheme(markdownText, theme.markdown)}`를 반환한다.
7. `renderedContent`가 null이면 `null`을 반환하고 중단한다.
8. `node.weight`가 있으면 `{'--weight': node.weight}`를 `hostStyle`로 설정하고, 없으면 `{}`를 사용한다.
9. 루트 `<div className="a2ui-text" style={hostStyle}>`를 렌더링한다.
10. 내부에 `<section className={classMapToString(classes)} style={additionalStyles} dangerouslySetInnerHTML={renderedContent} />`를 렌더링한다.

**경계 케이스:**
- `textValue`가 null/undefined이면 `null` 반환.
- `usageHint`가 `undefined`이면 body 처리(접두사 없음).
- 마크다운 내 원시 HTML은 `html: false` 기본 설정으로 비활성화되어 XSS를 방지한다.

## 동작 흐름

텍스트 해석 → 힌트별 클래스 병합 → 힌트별 스타일 계산(useMemo) → 힌트에 따른 마크다운 접두사 추가 → 테마 renderer rules 임시 적용 후 HTML 생성 → renderer rules 제거(복원) → `dangerouslySetInnerHTML`로 section에 삽입.
