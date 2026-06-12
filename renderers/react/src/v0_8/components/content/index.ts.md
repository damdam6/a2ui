# renderers/react/src/v0_8/components/content/index.ts

## 개요

콘텐츠 컴포넌트 디렉터리의 배럴(barrel) 파일로, 같은 디렉터리 내 개별 컴포넌트 파일들에서 named export를 모아 재수출한다. 상위 모듈이 각 파일을 직접 참조하지 않고 이 파일 하나를 통해 모든 콘텐츠 컴포넌트에 접근할 수 있도록 한다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`./Text`](./Text.tsx.md) — `Text` 컴포넌트
- [`./Image`](./Image.tsx.md) — `Image` 컴포넌트
- [`./Icon`](./Icon.tsx.md) — `Icon` 컴포넌트
- [`./Divider`](./Divider.tsx.md) — `Divider` 컴포넌트
- [`./Video`](./Video.tsx.md) — `Video` 컴포넌트
- [`./AudioPlayer`](./AudioPlayer.tsx.md) — `AudioPlayer` 컴포넌트

## Exports

- `Text` — 마크다운 텍스트 렌더링 컴포넌트 (`./Text`에서)
- `Image` — 이미지 렌더링 컴포넌트 (`./Image`에서)
- `Icon` — Material Symbols 아이콘 컴포넌트 (`./Icon`에서)
- `Divider` — 구분선 컴포넌트 (`./Divider`에서)
- `Video` — 동영상(일반/YouTube) 렌더링 컴포넌트 (`./Video`에서)
- `AudioPlayer` — 오디오 플레이어 컴포넌트 (`./AudioPlayer`에서)

## 동작 흐름

파일은 여섯 개의 `export { ... } from '...'` 구문만 포함한다. 런타임 로직 없음.
