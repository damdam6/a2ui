# renderers/angular/a2ui_explorer/src/app/kitchen-sink-surface.ts

## 개요

v0.9 프로토콜 메시지 시퀀스로 구성된 "Kitchen Sink" 데모 Surface 정의 상수를 내보내는 데이터 파일이다. `createSurface`, `updateDataModel`, `updateComponents` 세 단계 메시지를 배열로 담고 있으며, 다양한 컴포넌트 타입과 데이터 바인딩 패턴을 한 Surface에 집약해 렌더러의 핵심 기능을 일괄 확인하는 데 사용된다. 로직 없이 순수 데이터 상수만 포함한다.

## 의존성

없음 (외부 패키지 및 내부 모듈 import 없음).

## Exports

- `KITCHEN_SINK_SURFACE` (상수, 객체 배열)

## 상세 명세

### `KITCHEN_SINK_SURFACE` 상수

**타입**: 타입 어노테이션 없는 리터럴 배열 (TypeScript가 구조를 추론)

세 개의 v0.9 프로토콜 메시지 객체로 구성된다.

**메시지 1 — `createSurface`**
- `version`: `'v0.9'`
- `createSurface.surfaceId`: `'demo-surface'`
- `createSurface.catalogId`: `'demo'`

**메시지 2 — `updateDataModel`**
- `version`: `'v0.9'`
- `updateDataModel.surfaceId`: `'demo-surface'`
- `updateDataModel.path`: `'/'`
- `updateDataModel.value`: 초기 데이터 모델 객체
  - `user.name`: `'Guest'`
  - `user.email`: `''`
  - `form.submitted`: `false`
  - `form.responseMessage`: `''`
  - `settings.theme`: `'light'`

**메시지 3 — `updateComponents`**
- `version`: `'v0.9'`
- `updateComponents.surfaceId`: `'demo-surface'`
- `updateComponents.components`: 14개 컴포넌트 정의 배열 (선언 순서대로)

| id | component | 주요 속성 |
|---|---|---|
| `root` | `Column` | `align: 'start'`, `justify: 'start'`, `children: ['header', 'form-section', 'footer']` |
| `header` | `Row` | `children: ['logo', 'welcome-text']`, `align: 'center'` |
| `logo` | `Text` | `text: 'A2UI v0.9'`, `weight: 700` |
| `welcome-text` | `Text` | `text`: `formatString` 호출 표현식 — `value: 'Welcome, {{/user/name}}!'` |
| `form-section` | `Card` | `child: 'form-column'` |
| `form-column` | `Column` | `children: ['name-field', 'email-field', 'submit-btn', 'result-msg']` |
| `name-field` | `TextField` | `label: 'Your Name'`, `value: { path: '/user/name' }` |
| `satisfaction-slider` | `CustomSlider` | `label: 'Satisfaction Level'`, `value: { path: '/user/satisfaction' }`, `min: 0`, `max: 10` (form-column children에 미포함 — 미연결 상태) |
| `email-field` | `TextField` | `label: 'Email Address'`, `value: { path: '/user/email' }`, `variant: 'shortText'` |
| `submit-btn` | `Button` | `child: 'submit-text'`, `variant: 'primary'`, `action.event.name: 'submit_form'`, `action.event.context`: `{ name: { path: '/user/name' }, email: { path: '/user/email' } }` |
| `submit-text` | `Text` | `text: 'Submit'` |
| `result-msg` | `Text` | `text: { path: '/form/responseMessage' }` |
| `footer` | `Row` | `children: ['copy-text']` |
| `copy-text` | `Text` | `text: 'Powered by web_core v0.9'` |

## 동작 흐름

이 상수는 독립적으로 실행 로직을 갖지 않는다. 소비 측(demo-catalog 등)에서 `KITCHEN_SINK_SURFACE` 배열을 AgentStub의 `initializeDemo` 메서드에 넘기면, 배열 원소가 순서대로 처리되어 `'demo-surface'` Surface가 생성되고 초기 데이터 모델이 설정된 뒤 컴포넌트 트리가 구성된다.
