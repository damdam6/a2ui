# renderers/react/tests/v0_9/integration-scenarios.test.tsx

## 개요

실제 JSON 예시 파일(specification 디렉토리)을 `MessageProcessor`로 파싱하여 `A2uiSurface` React 컴포넌트로 종단 간(end-to-end) 렌더링하는 통합 테스트 파일이다. 단위 테스트용 헬퍼 없이 실제 `@a2ui/react/v0_9` 공개 API를 사용하며, 브라우저 DOM 환경에서 사용자 인터랙션까지 포함한 전체 렌더링 파이프라인을 검증한다. 세 가지 실제 예시(Markdown Text, Task Card, Login Form)를 대상으로 한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`, `screen`, `act`, `fireEvent`
- `react` — `React`(JSX, `React.StrictMode`)
- `@a2ui/web_core/v0_9` — `MessageProcessor`
- `@a2ui/react/v0_9` — `A2uiSurface`, `basicCatalog`

### 저장소 내부 모듈 (JSON 픽스처)
- `../../../../specification/v0_9/catalogs/basic/examples/35_markdown-text.json`
- `../../../../specification/v0_9/catalogs/basic/examples/07_task-card.json`
- `../../../../specification/v0_9/catalogs/basic/examples/09_login-form.json`

## Exports

없음 (테스트 전용 파일).

## 상세 명세 — 테스트 케이스

### `describe('Gallery Integration Tests')`

각 테스트는 동일한 패턴을 따른다:
1. `new MessageProcessor([basicCatalog as any], async () => {})` 로 프로세서를 생성한다. 두 번째 인자로 빈 비동기 함수를 액션 핸들러로 전달한다.
2. `processor.processMessages(exXxx.messages as any[])` 로 JSON 파일의 `messages` 배열을 처리한다.
3. `processor.model.getSurface(surfaceId)` 로 surface 모델을 가져오고 `expect(surface).toBeDefined()`로 유효성을 확인한다.
4. `render(<React.StrictMode><A2uiSurface surface={surface as any} /></React.StrictMode>)` 로 렌더링한다.
5. 특정 텍스트나 인터랙션을 검증한다.

---

- **`renders Markdown Text -> "Markdown Rendering"`**:
  - 픽스처: `35_markdown-text.json`, surface ID: `'gallery-markdown-text'`
  - 검증: `screen.getByText('Markdown Rendering').toBeInTheDocument()`

- **`renders Task Card -> content visibility`**:
  - 픽스처: `07_task-card.json`, surface ID: `'gallery-task-card'`
  - 검증: `'Review pull request'`와 `'Backend'` 텍스트가 DOM에 존재함

- **`handles Login form -> input updates data model`**:
  - 픽스처: `09_login-form.json`, surface ID: `'gallery-login-form'`
  - `screen.getByLabelText('Email')`로 email 입력 필드를 찾는다.
  - `act` 블록 내에서 `fireEvent.change(emailInput, {target: {value: 'alice@example.com'}})` 실행
  - `surface!.dataModel.get('/email')`이 `'alice@example.com'`임을 확인하여 데이터 모델과 UI가 양방향으로 동기화됨을 검증한다.

## 동작 흐름

JSON 메시지 배열 → `MessageProcessor.processMessages` → surface 모델 구성 → `A2uiSurface` 렌더링 → DOM 어설션 순서로 흐른다. Login Form 테스트에서는 추가로 `fireEvent`를 통한 인터랙션과 `surface.dataModel` 상태 검증이 이어진다. `React.StrictMode`로 래핑하여 이중 렌더링 이슈를 실제 환경과 유사하게 재현한다.
