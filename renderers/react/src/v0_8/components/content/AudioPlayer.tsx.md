# renderers/react/src/v0_8/components/content/AudioPlayer.tsx

## 개요

`AudioPlayer` 컴포넌트는 A2UI 노드 트리에서 `AudioPlayerNode`를 받아 HTML `<audio>` 요소와 선택적 설명 텍스트를 렌더링한다. URL이 없으면 렌더링을 생략하며, 테마 클래스와 추가 스타일을 적용하여 Lit 렌더러의 구조를 React에서 동일하게 재현한다. `memo`로 감싸져 불필요한 리렌더링을 방지한다.

## 의존성

### 외부 패키지
- `react` — `memo`
- `@a2ui/web_core/types/types` — `Types.AudioPlayerNode` (타입 전용)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md) — `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md) — `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md) — `classMapToString`, `stylesToObject`

## Exports

- `AudioPlayer` — named export, `React.NamedExoticComponent` (memo로 감싼 함수형 컴포넌트)
- `default` — `AudioPlayer`의 default export

## 상세 명세

### `AudioPlayer`

**시그니처**: `memo(function AudioPlayer({ node, surfaceId }: A2UIComponentProps<Types.AudioPlayerNode>): JSX.Element | null)`

**동작 단계:**
1. `useA2UIComponent(node, surfaceId)`를 호출하여 `theme`과 `resolveString`을 획득한다.
2. `node.properties`를 `props`에 할당한다.
3. `resolveString(props.url)`로 오디오 URL을 문자열로 해석한다. URL이 `null`이면 `null`을 반환하고 렌더링을 중단한다.
4. `resolveString(props.description ?? null)`로 설명 텍스트를 해석한다. `description`이 없으면 `null`을 넘긴다.
5. `node.weight`가 `undefined`가 아니면 `{'--weight': node.weight}`를 `hostStyle`로 설정한다. 없으면 빈 객체 `{}`를 사용한다.
6. 루트 `<div className="a2ui-audio" style={hostStyle}>`를 렌더링한다.
7. 내부에 `<section className={classMapToString(theme.components.AudioPlayer)} style={stylesToObject(theme.additionalStyles?.AudioPlayer)}>`를 렌더링한다.
8. `description`이 truthy이면 `<p>{description}</p>`를 section 안에 삽입한다.
9. `<audio src={url} controls />`를 항상 section 안에 렌더링한다.

**경계 케이스:**
- URL이 null/undefined/빈 문자열이면 컴포넌트 전체가 `null`을 반환한다.
- `description`이 null/undefined/빈 문자열이면 `<p>` 태그를 렌더링하지 않는다.
- `node.weight`가 `undefined`일 때는 `--weight` CSS 변수를 설정하지 않는다.

## 동작 흐름

노드로부터 URL을 해석 → URL 없으면 조기 반환 → 호스트 스타일(CSS 변수) 계산 → 테마 클래스 적용된 section 구조 렌더링 → 선택적 설명 텍스트 + `<audio controls>` 출력.
