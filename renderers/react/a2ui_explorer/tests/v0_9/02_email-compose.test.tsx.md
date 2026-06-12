# renderers/react/a2ui_explorer/tests/v0_9/02_email-compose.test.tsx

## 개요

`02_email-compose.json` 예제에 대한 브라우저 기반 통합 테스트 파일이다. 이메일 작성 폼 예제가 올바르게 렌더링되고, Send/Discard 버튼 클릭 시 적절한 액션이 디스패치되는지를 검증한다. Jasmine 프레임워크와 `test-utils` 헬퍼를 사용한다.

## 의존성

### 외부 패키지
- `react` — `act`

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.tsx.md) — `loadExample`, `cleanup`, `findButtonByText`, `whenSettled`, `getSurface`

## Exports

없음 (테스트 파일)

## 테스트 케이스

### describe: `'Example: Email Compose'`

**공통 픽스처/훅**:

- `container: HTMLDivElement` — 렌더링된 App의 루트 컨테이너
- `actionSpy: jasmine.Spy` — 액션 디스패치를 가로채는 Jasmine 스파이
- `surface: HTMLElement` — `getSurface(container)`로 가져온 서피스 컨테이너 요소

`beforeEach`: `jasmine.createSpy('onAction')`으로 스파이를 생성하고, `loadExample('02_email-compose.json', actionSpy)`를 `await`하여 컨테이너를 받는다. `getSurface(container)`로 서피스 요소를 추출한다.

`afterEach`: `cleanup()`을 `await`하여 마운트를 해제하고 DOM을 정리한다.

---

#### `'should render text content'`

**검증 동작**: 서피스 내 텍스트 컨텐츠에 이메일 작성 폼의 주요 레이블과 버튼 텍스트가 존재하는지 확인한다.

**검증 항목**:
- `surface.textContent`에 `'FROM'` 포함 여부
- `surface.textContent`에 `'TO'` 포함 여부
- `surface.textContent`에 `'SUBJECT'` 포함 여부
- `surface.textContent`에 `'Send email'` 포함 여부
- `surface.textContent`에 `'Discard'` 포함 여부
- `surface.textContent`에 `'alex@acme.com'` 포함 여부

**모킹**: 없음 (실제 렌더링 사용)

---

#### `'should handle Send button click'`

**검증 동작**: "Send" 텍스트를 가진 버튼을 클릭했을 때 `actionSpy`가 정확히 1회 호출되고, 디스패치된 액션의 `name` 속성이 `'send'`인지 검증한다.

**단계**:
1. `findButtonByText(surface, 'Send')`로 버튼 탐색
2. `act(async () => { sendBtn.click(); })`로 클릭 이벤트 발생
3. `whenSettled()`로 비동기 사이드 이펙트 대기
4. `actionSpy`의 호출 횟수가 `1`인지, 가장 최근 호출의 첫 번째 인수의 `name`이 `'send'`인지 검증

**모킹**: `actionSpy` (Jasmine 스파이, `beforeEach`에서 생성)

---

#### `'should handle Discard button click'`

**검증 동작**: "Discard" 텍스트를 가진 버튼을 클릭했을 때 `actionSpy`가 정확히 1회 호출되고, 디스패치된 액션의 `name` 속성이 `'discard'`인지 검증한다.

**단계**:
1. `findButtonByText(surface, 'Discard')`로 버튼 탐색
2. `act(async () => { discardBtn.click(); })`로 클릭 이벤트 발생
3. `whenSettled()`로 비동기 사이드 이펙트 대기
4. `actionSpy`의 호출 횟수가 `1`인지, 가장 최근 호출의 첫 번째 인수의 `name`이 `'discard'`인지 검증

**모킹**: `actionSpy` (Jasmine 스파이, `beforeEach`에서 생성)
