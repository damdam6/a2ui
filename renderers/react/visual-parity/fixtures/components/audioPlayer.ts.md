# renderers/react/visual-parity/fixtures/components/audioPlayer.ts

## 개요

`AudioPlayer` 컴포넌트의 시각적 동등성(visual parity) 테스트를 위한 픽스처 정의 파일이다. 두 가지 케이스를 제공한다: 리터럴 URL을 사용하는 기본 케이스와 데이터 모델 경로 바인딩을 사용하는 케이스. 파일 주석에 따르면 `description` prop은 A2UI spec에 정의되어 있지만 Lit 렌더러에서 미구현 상태이므로, React/Lit 간 parity 테스트에서는 `url`만 검증 대상으로 삼는다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `ComponentFixture` (타입 import)

## Exports

- `audioPlayerBasic`: `ComponentFixture` — 리터럴 URL로 AudioPlayer를 렌더링하는 픽스처
- `audioPlayerWithPathBinding`: `ComponentFixture` — 데이터 모델 경로에서 URL을 읽는 픽스처
- `audioPlayerFixtures`: `{ audioPlayerBasic, audioPlayerWithPathBinding }` — 네임드 컬렉션 객체

## 상세 명세

### `audioPlayerBasic: ComponentFixture`

- `root`: `'audio-1'`
- `components`: 단일 컴포넌트 항목
  - `id`: `'audio-1'`
  - `component.AudioPlayer.url`: `{literalString: 'https://www.w3schools.com/html/horse.mp3'}`
- `data`: 없음 (데이터 모델 초기값 불필요)

### `audioPlayerWithPathBinding: ComponentFixture`

- `root`: `'audio-2'`
- `data`: `{'/media/audioUrl': 'https://www.w3schools.com/html/horse.mp3'}` — 데이터 모델 초기값으로 해당 경로에 URL을 설정한다.
- `components`: 단일 컴포넌트 항목
  - `id`: `'audio-2'`
  - `component.AudioPlayer.url`: `{path: '/media/audioUrl'}` — 데이터 모델 경로 바인딩

### `audioPlayerFixtures`

`audioPlayerBasic`과 `audioPlayerWithPathBinding`을 하나의 객체로 묶어 내보내는 네임스페이스 역할의 상수.

## 동작 흐름

픽스처는 시각적 parity 테스트 러너가 소비한다. 테스트 러너는 `root` 컴포넌트 ID를 사용하여 렌더링 진입점을 결정하고, `data`가 있으면 데이터 모델에 초기값을 주입한 후 `components` 배열을 컴포넌트 레지스트리에 등록하여 React와 Lit 양쪽으로 렌더링한 결과를 비교한다.
