# renderers/react/visual-parity/fixtures/components/video.ts

## 개요

Video 컴포넌트의 비주얼 패리티 테스트용 픽스처 모음이다. URL을 `literalString`으로 직접 지정하는 기본 방식과 데이터 모델 path 바인딩으로 URL을 주입하는 방식 두 가지를 각각 별도 픽스처로 제공한다.

## 의존성

### 저장소 내부 모듈

- [`../types`](../types.ts.md) — `ComponentFixture` 타입을 type-only import

### 외부 패키지

없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `videoBasic` | 상수 (`ComponentFixture`) | literalString URL을 사용하는 기본 Video 픽스처 |
| `videoWithPathBinding` | 상수 (`ComponentFixture`) | 데이터 모델 path 바인딩을 사용하는 Video 픽스처 |
| `videoFixtures` | 상수 (객체) | 위 2개 픽스처를 묶은 그룹 객체 |

## 상세 명세

### `videoBasic: ComponentFixture`

- `root`: `'video-1'`
- 단일 컴포넌트: `id: 'video-1'`
- `Video.url`: `{literalString: 'https://www.w3schools.com/html/mov_bbb.mp4'}`
- `data` 필드 없음 — 정적 URL

### `videoWithPathBinding: ComponentFixture`

- `root`: `'video-2'`
- `data`: `{'/media/videoUrl': 'https://www.w3schools.com/html/mov_bbb.mp4'}` — 데이터 모델 초기값 주입
- 단일 컴포넌트: `id: 'video-2'`
- `Video.url`: `{path: '/media/videoUrl'}` — 데이터 모델의 `/media/videoUrl` 경로에 바인딩
- 두 픽스처 모두 동일한 MP4 파일(`mov_bbb.mp4`, Big Buck Bunny 클립)을 사용하므로 렌더 결과가 시각적으로 동일해야 한다

### `videoFixtures: object`

`videoBasic`과 `videoWithPathBinding`을 키-값으로 묶은 집계 객체.

## 동작 흐름

`videoBasic`은 URL을 컴포넌트 정의 내에 직접 포함하고, `videoWithPathBinding`은 `data` 필드를 통해 렌더 전 데이터 모델에 URL을 주입한 뒤 path 바인딩으로 참조한다. 두 방식의 시각적 동등성을 검증하는 것이 이 두 픽스처의 핵심 목적이다.
