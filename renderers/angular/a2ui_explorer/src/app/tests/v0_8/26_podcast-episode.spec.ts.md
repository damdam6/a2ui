# renderers/angular/a2ui_explorer/src/app/tests/v0_8/26_podcast-episode.spec.ts

## 개요

v0.8 카탈로그의 "Podcast Episode (basic)" 예제가 Angular 렌더러에서 올바르게 렌더링되고 재생 버튼이 올바른 액션을 발행하는지 검증하는 Jasmine 테스트 파일이다. 텍스트 렌더링 검증 1개와 버튼 클릭 이벤트 디스패치 검증 1개로 총 2개의 테스트 케이스를 포함한다. `ComponentFixture`를 사용하여 DOM 조작과 이벤트 로그 확인이 가능하다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`, `wait`

## Exports

없음 (테스트 파일이므로 export 없음)

## 테스트 케이스 명세

### `describe`: `'Example: Podcast Episode (basic) (v0.8)'`

#### 픽스처 / 설정
- `textContent: string` — 캔버스 요소의 텍스트 내용
- `fixture: ComponentFixture<DemoComponent>` — Angular 테스트 픽스처
- `beforeEach`: `loadExample('Podcast Episode (basic)', Version.V0_8)`를 `await`하여 `fixture`에 저장하고, `getCanvas().textContent`를 `textContent`에 저장한다.

#### 테스트 케이스: `'should render expected text content'`
- **검증 동작**: 캔버스 텍스트에 다음 항목이 모두 포함되어 있는지 `toContain`으로 단언한다.
  - 버튼 레이블: `'Play Episode'`
  - 팟캐스트 채널명: `'Tech Talk Daily'`
  - 에피소드 제목: `'The Future of AI in Product Design'`
  - 재생 시간: `'45 min'`
  - 날짜: `'Dec 15, 2024'`
  - 설명: `'How AI is transforming the way we design and build products.'`
- **사용 픽스처/모킹**: `textContent`

#### 테스트 케이스: `'should dispatch play action on button click'`
- **검증 동작**: 첫 번째 버튼(`buttons[0]`)을 클릭했을 때 `component.eventsLog[0].action.name`이 `'play'`인지 확인한다.
- **세부 절차**:
  1. `fixture.nativeElement.querySelectorAll('a2ui-button button')`로 버튼 목록 획득
  2. `buttons.length > 0` 및 `btn`이 truthy인지 단언
  3. `btn.click()` 후 `fixture.detectChanges()` 호출
  4. `await wait(10)`으로 비동기 이벤트 처리 대기
  5. `component.eventsLog.length > 0` 단언 후 `loggedAction.name === 'play'` 검증
- **사용 픽스처/모킹**: `fixture`, `DemoComponent.eventsLog`, `wait`

## 동작 흐름

1. `beforeEach`에서 Podcast Episode 예제를 로드하여 `fixture`와 `textContent`를 초기화한다.
2. 첫 번째 테스트는 팟캐스트 에피소드 정보 텍스트 전체를 검증한다.
3. 두 번째 테스트는 "Play Episode" 버튼 클릭 → `detectChanges` → `wait(10)` → 이벤트 로그 확인의 패턴으로 `play` 액션 발행을 검증한다.
