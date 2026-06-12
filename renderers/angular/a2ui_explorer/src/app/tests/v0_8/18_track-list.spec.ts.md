# renderers/angular/a2ui_explorer/src/app/tests/v0_8/18_track-list.spec.ts

## 개요

v0.8 카탈로그의 "Track List (basic)" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 파일이다. 음악 트랙 목록(트랙 번호, 제목, 아티스트, 재생 시간)이 캔버스 DOM에 모두 포함되어 있는지 텍스트 기반으로 확인한다. 인터랙션 테스트 없이 텍스트 렌더링 검증만 수행하는 단순한 구조다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample` 임포트

## Exports

없음 (테스트 파일이므로 export 없음)

## 테스트 케이스 명세

### `describe`: `'Example: Track List (basic) (v0.8)'`

#### 픽스처 / 설정
- `textContent: string` — 각 테스트 실행 전 캔버스 요소의 텍스트 내용을 담는 변수
- `beforeEach`: `loadExample('Track List (basic)', Version.V0_8)`를 `await`하여 예제를 로드한 후, `getCanvas().textContent`를 `textContent`에 저장한다.

#### 테스트 케이스: `'should render expected text content'`
- **검증 동작**: 캔버스 텍스트에 다음 항목이 모두 포함되어 있는지 `toContain`으로 단언한다.
  - 트랙 번호: `'1'`, `'2'`, `'3'`
  - 트랙 1: 제목 `'Focus Flow'`, 아티스트 `'Weightless'` / `'Marconi Union'`, 재생 시간 `'8:09'`
  - 트랙 2: 제목 `'Clair de Lune'`, 아티스트 `'Debussy'`, 재생 시간 `'5:12'`
  - 트랙 3: 제목 `'Ambient Light'`, 아티스트 `'Brian Eno'`, 재생 시간 `'6:45'`
- **사용 픽스처/모킹**: `beforeEach`에서 설정한 `textContent`, `loadExample`, `getCanvas`

## 동작 흐름

1. `beforeEach`에서 `loadExample`을 호출해 v0.8 카탈로그의 Track List 예제를 로드한다.
2. `getCanvas().textContent`로 렌더링된 캔버스 DOM 전체 텍스트를 추출한다.
3. 단일 `it` 블록에서 3개 트랙의 번호·제목·아티스트·재생 시간이 모두 텍스트에 포함되는지 검증한다.
