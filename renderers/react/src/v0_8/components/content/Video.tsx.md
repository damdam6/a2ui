# renderers/react/src/v0_8/components/content/Video.tsx

## 개요

`Video` 컴포넌트는 일반 동영상 URL과 YouTube URL을 모두 지원하는 영상 재생 컴포넌트다. URL이 YouTube 형식이면 `<iframe>`으로 임베드하고, 그 외에는 HTML `<video>` 요소를 사용한다. URL이 없으면 렌더링을 생략한다.

## 의존성

### 외부 패키지
- `react` — `memo`
- `@a2ui/web_core/types/types` — `Types.VideoNode` (타입 전용)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md) — `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md) — `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md) — `classMapToString`, `stylesToObject`

## Exports

- `Video` — named export, memo로 감싼 함수형 컴포넌트
- `default` — `Video`의 default export

## 상세 명세

### `getYouTubeVideoId(url: string): string | null`

비공개 헬퍼 함수. URL에서 YouTube 동영상 ID를 추출한다.

**매개변수:** `url` (string) — 검사할 URL 문자열.

**반환 타입:** YouTube 동영상 ID 문자열 또는 `null`.

**동작:** 정규식 패턴 배열 `[/(?:youtube\.com\/watch\?v=|youtu\.be\/|youtube\.com\/embed\/)([^&\s?]+)/]`을 순서대로 적용한다. URL이 일치하고 `match.length > 1`이면 `match[1]`(비-null 단언 `!` 사용)을 반환한다. 일치하지 않으면 `null`을 반환한다.

지원하는 YouTube URL 형식:
- `youtube.com/watch?v=VIDEO_ID`
- `youtu.be/VIDEO_ID`
- `youtube.com/embed/VIDEO_ID`

### `Video`

**시그니처**: `memo(function Video({ node, surfaceId }: A2UIComponentProps<Types.VideoNode>): JSX.Element | null)`

**동작 단계:**
1. `useA2UIComponent(node, surfaceId)`로 `theme`과 `resolveString`을 획득한다.
2. `resolveString(props.url)`로 URL을 해석한다.
3. URL이 null이면 `null`을 반환하고 중단한다.
4. `getYouTubeVideoId(url)`을 호출하여 `youtubeId`를 획득한다.
5. `node.weight`가 있으면 `{'--weight': node.weight}`를 `hostStyle`로 설정하고, 없으면 `{}`를 사용한다.
6. 루트 `<div className="a2ui-video" style={hostStyle}>`를 렌더링한다.
7. 내부에 `<section className={classMapToString(theme.components.Video)} style={stylesToObject(theme.additionalStyles?.Video)}>`를 렌더링한다.
8. `youtubeId`가 truthy이면 `<iframe>`을 렌더링한다:
   - `src`: `"https://www.youtube.com/embed/${youtubeId}"`
   - `title`: `"YouTube video player"`
   - `allow`: `"accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"`
   - `allowFullScreen`: 설정
   - 인라인 스타일: `{border: 'none', width: '100%', aspectRatio: '16/9'}`
9. `youtubeId`가 null이면 `<video src={url} controls />`를 렌더링한다.

**경계 케이스:**
- URL이 null이면 컴포넌트 전체가 `null`을 반환한다.
- YouTube URL이 아닌 모든 URL은 네이티브 `<video>` 태그로 처리된다.

## 동작 흐름

URL 해석 → null 확인 → YouTube ID 추출 시도 → 호스트 스타일 계산 → YouTube면 iframe 임베드, 아니면 video 태그 렌더링.
