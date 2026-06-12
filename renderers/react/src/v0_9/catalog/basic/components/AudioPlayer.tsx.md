# renderers/react/src/v0_9/catalog/basic/components/AudioPlayer.tsx

## 개요

`AudioPlayer` 컴포넌트는 a2ui Basic Catalog의 오디오 재생 UI 요소다. `AudioPlayerApi` 스키마를 기반으로 `createComponentImplementation`을 통해 생성되며, 선택적 설명 텍스트와 브라우저 기본 `<audio controls>` 엘리먼트를 수직 방향으로 배치한다. 모든 시각적 속성은 CSS 변수(custom properties)를 통해 테마 재정의가 가능하도록 설계되어 있다.

## 의존성

### 외부 패키지
- `react` — `React.CSSProperties` 타입 사용
- `@a2ui/web_core/v0_9/basic_catalog` — `AudioPlayerApi` 스키마 가져오기

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation` 유틸리티
- [`../utils`](../utils.ts.md) — `useBasicCatalogStyles` 훅

## Exports

| 이름 | 종류 |
|------|------|
| `AudioPlayer` | 상수 (React 컴포넌트, `createComponentImplementation` 반환값) |

## 상세 명세

### `AudioPlayer`

**시그니처**: `createComponentImplementation(AudioPlayerApi, ({props}) => JSX.Element)`

`createComponentImplementation`에 `AudioPlayerApi` 스키마와 렌더 함수를 전달하여 생성된 컴포넌트다.

**렌더 함수 동작 로직**:

1. `useBasicCatalogStyles()`를 호출하여 Basic Catalog의 전역 CSS를 주입한다.
2. `containerStyle`을 구성한다:
   - `display: 'flex'`, `flexDirection: 'column'` — 자식 요소를 수직 배치
   - `gap: 'var(--a2ui-spacing-xs, 0.25rem)'` — 설명 텍스트와 오디오 엘리먼트 사이 간격
   - `background: 'var(--a2ui-audioplayer-background, transparent)'` — 기본값 투명
   - `borderRadius: 'var(--a2ui-audioplayer-border-radius, 0)'` — 기본값 0
   - `padding: 'var(--a2ui-audioplayer-padding, 0)'` — 기본값 0
3. `props.description`이 존재하면 `<span>` 엘리먼트로 설명 텍스트를 렌더링한다:
   - `fontSize: 'var(--a2ui-font-size-xs, 0.75rem)'`
   - `color: 'var(--a2ui-text-caption-color, light-dark(#666, #aaa))'`
4. `<audio src={props.url} controls />` 엘리먼트를 렌더링하여 브라우저 기본 재생 컨트롤을 제공한다.

**props**:
- `props.url` (`string`) — 오디오 파일 URL (`<audio>` 의 `src` 속성으로 전달)
- `props.description` (`string | undefined`) — 선택적 설명 텍스트; 존재할 때만 렌더링

## 동작 흐름

컴포넌트가 마운트되면 `useBasicCatalogStyles`를 통해 전역 스타일이 주입된다. 이후 컨테이너 `<div>` 안에 `props.description` 유무에 따라 선택적으로 캡션 `<span>`을 먼저 렌더링하고, 그 아래에 브라우저 네이티브 `<audio controls>` 엘리먼트를 배치한다. 데이터 흐름은 단방향이며 내부 상태를 보유하지 않는다.
