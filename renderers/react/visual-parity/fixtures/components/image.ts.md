# renderers/react/visual-parity/fixtures/components/image.ts

## 개요

Image 컴포넌트의 비주얼 패리티 테스트용 픽스처 모음이다. picsum.photos 서비스의 seed 파라미터를 사용해 테스트 실행 간에 일관된 플레이스홀더 이미지 URL을 제공한다. `usageHint` 값의 모든 주요 변형(기본, avatar, header, icon, largeFeature, mediumFeature, smallFeature)을 각각 하나의 픽스처로 커버한다.

## 의존성

### 저장소 내부 모듈

- [`../types`](../types.ts.md) — `ComponentFixture` 타입을 type-only import

### 외부 패키지

없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `imageBasic` | 상수 (`ComponentFixture`) | 기본 Image 픽스처 |
| `imageAvatar` | 상수 (`ComponentFixture`) | `usageHint: 'avatar'` Image 픽스처 |
| `imageHeader` | 상수 (`ComponentFixture`) | `usageHint: 'header'` Image 픽스처 |
| `imageIcon` | 상수 (`ComponentFixture`) | `usageHint: 'icon'` Image 픽스처 |
| `imageLargeFeature` | 상수 (`ComponentFixture`) | `usageHint: 'largeFeature'` Image 픽스처 |
| `imageMediumFeature` | 상수 (`ComponentFixture`) | `usageHint: 'mediumFeature'` Image 픽스처 |
| `imageSmallFeature` | 상수 (`ComponentFixture`) | `usageHint: 'smallFeature'` Image 픽스처 |
| `imageFixtures` | 상수 (객체) | 위 7개 픽스처를 하나의 객체로 묶은 그룹 |

## 상세 명세

### 파일-수준 상수 (내부 헬퍼)

세 개의 picsum.photos URL 상수가 파일 상단에 정의되어 있으며, 모든 픽스처에서 재사용된다.

- `placeholderUrl`: `'https://picsum.photos/seed/a2ui/150/150'` — seed `a2ui`, 150×150 크기의 정사각형 이미지
- `avatarUrl`: `'https://picsum.photos/seed/avatar/64/64'` — seed `avatar`, 64×64 크기의 소형 이미지
- `largeUrl`: `'https://picsum.photos/seed/large/400/200'` — seed `large`, 400×200 크기의 가로형 이미지

### `imageBasic: ComponentFixture`

- `root`: `'img-basic'`
- `components` 배열에 단일 컴포넌트 엔트리 포함
  - `id`: `'img-basic'`, `component.Image.url.literalString`: `placeholderUrl` (150×150)
  - `usageHint` 없음: 렌더러의 기본 이미지 스타일 적용

### `imageAvatar: ComponentFixture`

- `root`: `'img-avatar'`
- 단일 컴포넌트: `id: 'img-avatar'`, `url.literalString`: `avatarUrl` (64×64), `usageHint: 'avatar'`
- avatar 힌트에 따라 렌더러가 원형 또는 소형 표시 방식을 적용하는지 검증하는 용도

### `imageHeader: ComponentFixture`

- `root`: `'img-header'`
- 단일 컴포넌트: `id: 'img-header'`, `url.literalString`: `largeUrl` (400×200), `usageHint: 'header'`
- 헤더용 이미지 렌더링 방식 검증

### `imageIcon: ComponentFixture`

- `root`: `'img-icon'`
- 단일 컴포넌트: `id: 'img-icon'`, `url.literalString`: `avatarUrl` (64×64), `usageHint: 'icon'`
- icon 힌트에 따른 소형 인라인 이미지 렌더링 방식 검증

### `imageLargeFeature: ComponentFixture`

- `root`: `'img-large'`
- 단일 컴포넌트: `id: 'img-large'`, `url.literalString`: `largeUrl` (400×200), `usageHint: 'largeFeature'`

### `imageMediumFeature: ComponentFixture`

- `root`: `'img-medium'`
- 단일 컴포넌트: `id: 'img-medium'`, `url.literalString`: `placeholderUrl` (150×150), `usageHint: 'mediumFeature'`

### `imageSmallFeature: ComponentFixture`

- `root`: `'img-small'`
- 단일 컴포넌트: `id: 'img-small'`, `url.literalString`: `avatarUrl` (64×64), `usageHint: 'smallFeature'`

### `imageFixtures: object`

7개의 개별 픽스처 상수(`imageBasic`, `imageAvatar`, `imageHeader`, `imageIcon`, `imageLargeFeature`, `imageMediumFeature`, `imageSmallFeature`)를 키-값 쌍으로 묶은 단순 객체. 상위 집계 모듈에서 스프레드로 병합하거나 반복 처리할 때 사용된다.

## 동작 흐름

파일은 세 URL 상수를 먼저 선언한 뒤, 각 픽스처를 `ComponentFixture` 형태의 독립 상수로 정의한다. 각 픽스처는 `root` ID와 해당 ID를 가진 단일 컴포넌트 엔트리로 구성된다. 마지막에 `imageFixtures` 객체로 전체 픽스처를 집계한다. `data` 필드는 어느 픽스처에도 사용되지 않으며 모든 URL은 `literalString`으로 하드코딩되어 런타임 데이터 모델 의존 없이 안정적으로 렌더링된다.
