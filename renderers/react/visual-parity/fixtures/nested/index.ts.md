# renderers/react/visual-parity/fixtures/nested/index.ts

## 개요

`nested` 픽스처 서브디렉토리의 공개 진입점 역할을 하는 배럴(barrel) 모듈이다. `layouts.ts`에서 정의된 모든 심볼을 와일드카드로 재내보내는 동시에, `nestedFixtures` 객체를 명시적으로도 재내보낸다. 이 파일 자체에는 독립적인 로직이 없으며, 소비자가 상위 경로(`../nested`)만으로 nested 레이아웃 픽스처 전체에 접근할 수 있도록 한다.

## 의존성

### 저장소 내부 모듈

- [`./layouts`](./layouts.ts.md) — 모든 nested 레이아웃 픽스처 상수 및 `nestedFixtures` 집계 객체를 정의

### 외부 패키지

없음.

## Exports

| 이름 | 종류 | 비고 |
|---|---|---|
| `* from './layouts'` | 와일드카드 재내보내기 | `layouts.ts`의 모든 공개 심볼 |
| `nestedFixtures` | 상수(객체) | 명시적 재내보내기 (와일드카드와 중복이지만 의도적으로 강조) |

## 상세 명세

이 파일에는 독자적으로 선언된 심볼이 없다. 두 줄의 `export` 문만 존재한다.

- `export * from './layouts'` — `layouts.ts`에서 내보내는 모든 이름(`nestedCardInList`, `nestedForm`, `nestedRowInColumn`, `nestedColumnInRow`, `nestedDashboard`, `nestedProfile`, `nestedSettings`, `nestedFixtures`)을 그대로 이 모듈의 외부 인터페이스로 노출한다.
- `export {nestedFixtures} from './layouts'` — 동일 모듈에서 `nestedFixtures`를 다시 한 번 명시적으로 재내보낸다. 이는 배럴 사용 시 핵심 집계 심볼임을 강조하기 위한 의도적 중복이다.

## 동작 흐름

파일 실행 시 실제 코드는 수행되지 않는다. TypeScript 모듈 시스템이 `layouts.ts`를 로드한 뒤 해당 심볼들을 이 모듈의 공개 API로 게시한다.
