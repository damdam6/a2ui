# renderers/react/src/v0_8/components/layout/index.ts

## 개요

레이아웃 컴포넌트 모듈의 진입점(barrel) 파일이다. `Row`, `Column`, `List`, `Card`, `Tabs`, `Modal` 여섯 가지 레이아웃 컴포넌트를 각각의 개별 모듈에서 re-export한다. 외부에서 레이아웃 컴포넌트를 임포트할 때 이 파일 하나만 참조하면 된다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- `./Row` — `Row` 컴포넌트 정의
- `./Column` — `Column` 컴포넌트 정의
- `./List` — `List` 컴포넌트 정의
- `./Card` — `Card` 컴포넌트 정의
- `./Tabs` — `Tabs` 컴포넌트 정의
- `./Modal` — `Modal` 컴포넌트 정의

## Exports

| 이름 | 종류 | 출처 |
|------|------|------|
| `Row` | React 컴포넌트 | `./Row` |
| `Column` | React 컴포넌트 | `./Column` |
| `List` | React 컴포넌트 | `./List` |
| `Card` | React 컴포넌트 | `./Card` |
| `Tabs` | React 컴포넌트 | `./Tabs` |
| `Modal` | React 컴포넌트 | `./Modal` |

## 상세 명세

이 파일은 로직 없이 named re-export만 수행한다. 각 export 문은 해당 동일 이름의 모듈에서 named export를 그대로 전달한다.

## 동작 흐름

파일이 임포트되는 시점에 각 하위 모듈 파일이 로드되며, 각 컴포넌트가 모듈 스코프에 바인딩된다. 이후 소비자가 `import { Row } from '.../layout'` 형태로 사용할 수 있다.
