# renderers/angular/a2ui_explorer/src/app/tests/v0_9/18_track-list.spec.ts

## 개요

`Track List` 예제 컴포넌트의 렌더링을 검증하는 Angular 통합 테스트 파일이다. 플레이리스트 이름과 세 트랙 각각의 곡명·아티스트·재생 시간 텍스트가 올바르게 표시되는지, 그리고 세 개의 앨범 아트 이미지 URL이 올바른지 확인한다. 인터랙션 테스트는 포함하지 않는다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component.ts`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에서 실행하며 별도로 export하는 항목은 없다.

## 테스트 케이스

### `describe('Example: Track List')`

픽스처 변수: `fixture: ComponentFixture<DemoComponent>`, `textContent: string`

**`beforeEach`**
- `loadExample('Track List')`를 비동기로 호출하여 `fixture`를 얻는다.
- `getCanvas().textContent`를 `textContent`에 저장한다.

---

**`should render playlist name`**
- 검증 동작: 텍스트에 `'Focus Flow'` 플레이리스트 이름이 포함되어 있는지 확인한다.
- 픽스처/모킹: `textContent` 사용.

---

**`should render Weightless track details`**
- 검증 동작: 텍스트에 `'Weightless'`, `'Marconi Union'`, `'8:09'` 가 포함되어 있는지 확인한다.
- 픽스처/모킹: `textContent` 사용.

---

**`should render Clair de Lune track details`**
- 검증 동작: 텍스트에 `'Clair de Lune'`, `'Debussy'`, `'5:12'` 가 포함되어 있는지 확인한다.
- 픽스처/모킹: `textContent` 사용.

---

**`should render Ambient Light track details`**
- 검증 동작: 텍스트에 `'Ambient Light'`, `'Brian Eno'`, `'6:45'` 가 포함되어 있는지 확인한다.
- 픽스처/모킹: `textContent` 사용.

---

**`should render images`**
- 검증 동작: `querySelectorAll('img')`로 이미지 요소 배열을 수집하고 길이가 3 이상인지 확인한다. 각 `img` 요소의 `src` 배열에 다음 세 URL이 모두 포함되어 있는지 검증한다:
  - `'https://images.unsplash.com/photo-1470225620780-dba8ba36b745?w=50&h=50&fit=crop'` (Weightless)
  - `'https://images.unsplash.com/photo-1511379938547-c1f69419868d?w=50&h=50&fit=crop'` (Clair de Lune)
  - `'https://images.unsplash.com/photo-1507838153414-b4b713384a76?w=50&h=50&fit=crop'` (Ambient Light)
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리.

## 동작 흐름

`beforeEach`에서 `'Track List'` 예제를 로드하고 텍스트를 추출한다. 플레이리스트 이름 검증 1개, 트랙별 세부 정보 검증 3개, 이미지 URL 검증 1개로 구성된 총 5개의 `it` 블록이 순서 독립적으로 실행된다.
