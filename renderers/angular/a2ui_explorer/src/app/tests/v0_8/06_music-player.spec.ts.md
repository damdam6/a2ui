# renderers/angular/a2ui_explorer/src/app/tests/v0_8/06_music-player.spec.ts

## 개요

v0.8 버전의 "Music Player (basic)" 예제 컴포넌트가 Angular 렌더러에서 올바르게 동작하는지 검증하는 Jasmine 테스트 스위트다. 텍스트 콘텐츠 렌더링과 세 개의 제어 버튼(이전 곡, 재생/일시정지, 다음 곡)에 대한 액션 디스패치를 각각 독립적인 테스트 케이스로 검증한다. `DemoComponent`의 `eventsLog`를 통해 버튼 클릭 시 올바른 액션 이름이 기록되는지 확인한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent` 클래스
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`, `wait` 유틸리티

## Exports

이 파일은 `describe` 블록으로 테스트를 등록할 뿐 명시적으로 export하는 항목이 없다.

## 테스트 케이스 명세

### 스위트: `Example: Music Player (basic) (v0.8)`

**픽스처 및 공유 변수**
- `textContent: string` — 캔버스 요소의 텍스트 내용
- `fixture: ComponentFixture<DemoComponent>` — Angular 컴포넌트 픽스처

**`beforeEach`**
`loadExample('Music Player (basic)', Version.V0_8)`를 비동기로 호출하여 픽스처를 초기화하고, `getCanvas().textContent`로 렌더링된 텍스트를 추출해 `textContent`에 저장한다.

---

#### 테스트 1: `should render expected text content`
- **검증 동작**: 캔버스에 렌더링된 텍스트가 음악 플레이어에 필요한 모든 정보를 포함하는지 확인한다.
- **검증 항목**: `'Blinding Lights'`(곡 제목), `'The Weeknd'`(아티스트명), `'1:48'`(현재 재생 시간), `'4:22'`(전체 길이), `'pause'`(재생 상태 아이콘/텍스트) 포함 여부를 `toContain`으로 각각 단언한다.
- **픽스처/모킹**: `loadExample` 호출 결과 반환된 `textContent` 사용.

---

#### 테스트 2: `should dispatch previous action on button click`
- **검증 동작**: `a2ui-button button` 셀렉터로 조회한 버튼 목록의 첫 번째 버튼(인덱스 0)을 클릭했을 때 `previous` 액션이 디스패치되는지 검증한다.
- **검증 항목**: 버튼 목록 길이가 0보다 크고, 첫 번째 버튼이 존재하며, 클릭 후 10ms 대기 뒤 `component.eventsLog[0].action.name`이 `'previous'`인지 단언한다.
- **픽스처/모킹**: `fixture.componentInstance`로 컴포넌트 인스턴스 접근, `wait(10)` 비동기 대기.

---

#### 테스트 3: `should dispatch playPause action on button click`
- **검증 동작**: 버튼 목록의 두 번째 버튼(인덱스 1)을 클릭했을 때 `playPause` 액션이 디스패치되는지 검증한다.
- **검증 항목**: 버튼 목록 길이가 1보다 크고, 두 번째 버튼이 존재하며, 클릭 후 10ms 대기 뒤 `component.eventsLog[0].action.name`이 `'playPause'`인지 단언한다.
- **픽스처/모킹**: `fixture.componentInstance`, `wait(10)`.

---

#### 테스트 4: `should dispatch next action on button click`
- **검증 동작**: 버튼 목록의 세 번째 버튼(인덱스 2)을 클릭했을 때 `next` 액션이 디스패치되는지 검증한다.
- **검증 항목**: 버튼 목록 길이가 2보다 크고, 세 번째 버튼이 존재하며, 클릭 후 10ms 대기 뒤 `component.eventsLog[0].action.name`이 `'next'`인지 단언한다.
- **픽스처/모킹**: `fixture.componentInstance`, `wait(10)`.

## 동작 흐름

각 테스트는 `beforeEach`에서 v0.8 Music Player 예제를 로드하고 캔버스 텍스트를 가져오는 것으로 시작한다. 텍스트 렌더링 테스트는 `textContent`를 직접 검사한다. 버튼 액션 테스트는 `fixture.nativeElement`에서 `a2ui-button button` 셀렉터로 버튼을 조회하고, 해당 인덱스의 버튼을 클릭한 후 `fixture.detectChanges()`와 `wait(10)`으로 변경 사항을 반영한 다음, `eventsLog`의 첫 번째 항목에서 액션 이름을 검증한다.
